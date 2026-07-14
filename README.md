# File Packer & Unpacker — Java

A command-line tool that packs multiple files from a directory into a single
archive and restores them back to their original state. Built in Java using
core file I/O streams, with a custom binary header format for storing per-file
metadata inside the archive.

---

## What It Does

**Packing** traverses a source directory, collects all `.txt` files, and writes
them sequentially into a single packed archive. Before each file's data, a
fixed-length 100-byte header is written containing the filename and file size.
After packing, the tool prints a report showing files scanned and files packed.

**Unpacking** reads the packed archive, parses each 100-byte header to extract
the filename and file size, creates the output file, reads exactly that many bytes
from the archive, and writes them to the new file. The loop continues until the
archive is fully consumed. A completion report shows the total files restored.

```
-----------------------------------------------------
------- Marvellous Packer Unpacker CUI Module -------
-----------------------------------------------------
----------------- Packing Activity ------------------

Enter the name of Directory that you want to pack :
> reports

Enter the name of packed file to create :
> archive.pak

File packed with name : report_q1.txt
File packed with name : report_q2.txt
File packed with name : summary.txt

-----------------------------------------------------
Packing activity completed..
Number of files scanned : 5
Number of files packed  : 3
-----------------------------------------------------
```

```
-----------------------------------------------------
------- Marvellous Packer Unpacker CUI Module -------
-----------------------------------------------------
---------------- Unpacking Activity -----------------

Enter the name of packed file to open :
> archive.pak

File dropped with name : report_q1.txt
File dropped with name : report_q2.txt
File dropped with name : summary.txt

-----------------------------------------------------
Unpacking activity completed..
Number of files unpacked : 3
-----------------------------------------------------
```

---

## Archive Format

The archive uses a simple sequential binary format. Each packed file is stored
as a fixed-size header followed immediately by the file's raw bytes:

```
┌─────────────────────────────────────────────────────┐
│  HEADER  (100 bytes, space-padded)                  │
│  Format: "<filename> <filesize>                   " │
│  Example: "report_q1.txt 2048                     " │
├─────────────────────────────────────────────────────┤
│  FILE DATA  (filesize bytes)                        │
│  Raw bytes of the original file                     │
├─────────────────────────────────────────────────────┤
│  HEADER  (100 bytes)  — next file                   │
├─────────────────────────────────────────────────────┤
│  FILE DATA  — next file                             │
└─────────────────────────────────────────────────────┘
```

The fixed 100-byte header length allows the unpacker to read headers with a
single `fiobj.read(Header, 0, 100)` call, then parse the filename and size
with `String.trim()` and `split(" ")` before reading exactly `filesize` bytes
of data.

---

## Key Implementation Details

**Packer (`program542.java`)**

- Opens a `FileOutputStream` on the destination packed file
- Uses `File.listFiles()` to traverse the source directory
- Filters for `.txt` files using `String.endsWith(".txt")`
- Constructs each header as `"<name> <size>"` padded with spaces to exactly 100 bytes
- Writes the header with `foobj.write(Header.getBytes(), 0, 100)`
- Streams file bytes with a 1024-byte buffer loop:
  `while ((iRet = fiobj.read(Buffer)) != -1) foobj.write(Buffer, 0, iRet)`
- Tracks and reports files scanned vs. files packed

**Unpacker (`program541.java`)**

- Opens a `FileInputStream` on the packed archive
- Loops with `fiobj.read(Header, 0, 100)` — reads each 100-byte header
- Trims whitespace and splits on space to extract filename (`Tokens[0]`) and size (`Tokens[1]`)
- Creates the output file with `File.createNewFile()`
- Reads exactly `FileSize` bytes: `fiobj.read(Buffer, 0, FileSize)`
- Writes to the output file and closes the stream
- Reports total files unpacked

---

## How the Project Was Built

The implementation was developed incrementally, with each version adding one
capability on top of the previous one:

| Stage | Capability added |
|---|---|
| File creation | `File.createNewFile()`, `FileOutputStream` basics |
| File reading | `FileInputStream`, buffered reads |
| File copying | Source-to-destination streaming |
| Directory traversal | `File.listFiles()`, filename and size extraction |
| Header construction | Fixed 100-byte padding for metadata |
| Packing | Writing headers + file data sequentially |
| Unpacking | Header parsing, file reconstruction, loop until EOF |

---

## Project Structure

```
file-packer-unpacker-java/
│
├── PackFront.java/            # Packer implementation (incremental versions)
│   ├── program501.java        # Initial setup
│   ├── program502.java – ...  # Incremental steps
│   └── program542.java        # Final complete packer
│
├── UnpackFront.java/          # Unpacker implementation (incremental versions)
│   ├── program533.java        # String parsing fundamentals
│   ├── program534.java – ...  # Incremental steps
│   └── program541.java        # Final complete unpacker
│
└── README.md
```

---

## Build and Run

**Requirements:** JDK 8 or later

**Compile and run the packer:**
```bash
cd "PackFront.java"
javac program542.java
java program542
```

**Compile and run the unpacker:**
```bash
cd "UnpackFront.java"
javac program541.java
java program541
```

**Quick test:**
```bash
# 1. Create a test directory with some text files
mkdir test_dir
echo "Hello from file one" > test_dir/file1.txt
echo "Hello from file two" > test_dir/file2.txt

# 2. Run the packer — enter "test_dir" and "archive.pak" when prompted
java program542

# 3. Run the unpacker in a different directory — enter "archive.pak" when prompted
mkdir output_dir && cd output_dir
java -cp .. program541
# file1.txt and file2.txt are recreated here
```

---

## Design Decisions

**Why a fixed 100-byte header?**
A fixed-length header makes the unpacking loop simple and reliable — read exactly
100 bytes, parse, read exactly `filesize` bytes, repeat. Variable-length headers
would require a delimiter or a length prefix, adding complexity. 100 bytes is
enough to hold any reasonable filename plus size without truncation.

**Why filter for `.txt` files only?**
The current implementation targets text files to keep the scope focused on
demonstrating the core packing mechanism. The filter is a single `endsWith()`
check and can be removed or generalized to support any file type.

**Why separate files for each development step?**
The incremental file structure preserves the learning process — each version
adds exactly one capability. This makes it easy to trace how each piece of the
final implementation was developed and tested.

---

## Future Improvements

- Support all file types (remove `.txt` filter, handle binary files)
- Add magic number header at archive start for format validation
- Add per-file checksum (MD5) and write a log file after packing
- Add encryption/decryption on the byte stream during pack/unpack
- Replace filename+size header with a structured binary format using `DataOutputStream`
- Add AWT GUI for directory selection and progress display
- Support nested directory packing

---

## License

Niraj Vijaysinh Nale
