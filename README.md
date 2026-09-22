# EcuFlash Splitter

This is a byte-preserving utility for breaking a large ECU binary into sequential raw `.bin` parts. It defaults to **2048 KiB per part = 2,097,152 bytes**.

It creates a JSON manifest containing:

- the original file size and SHA-256;
- each part filename;
- each part's original start offset and end offset;
- original and stored lengths;
- a SHA-256 for every part.

The manifest can be used to verify the parts and rejoin them exactly. The tool does not recalculate ECU checksums, decrypt, compress, or otherwise modify calibration data.

## Windows GUI

1. Install Python 3 if it is not already installed.
2. Double-click `EcuFlash_Splitter_GUI.bat`.
3. Select the ECU binary and an output folder.
4. Leave the chunk size at `2048` unless your EcuFlash workflow requires a different limit.
5. Click **Split binary**.

The last part is normally shorter than 2048 KiB. Enable **Pad final part to full chunk** only when a downstream workflow specifically requires fixed-size files. The manifest records the padding and the Join operation strips it back out.

After editing one or more parts in EcuFlash, select the manifest and output filename, enable **Allow edited parts when joining**, and click **Join parts**. The tool still requires every part to have the exact recorded file size and remain in the original order. It will report the new SHA-256; a changed hash is expected after calibration edits.

## Command line

From a Command Prompt in this folder:

```text
py -3 ecuflash_splitter.py split "C:\path\to\ecu.bin" -o "C:\path\to\ecu_parts"
py -3 ecuflash_splitter.py verify "C:\path\to\ecu_parts\ecu.ecuflash-manifest.json"
py -3 ecuflash_splitter.py join "C:\path\to\ecu_parts\ecu.ecuflash-manifest.json" -o "C:\path\to\rejoined_ecu.bin"
py -3 ecuflash_splitter.py join "C:\path\to\ecu_parts\ecu.ecuflash-manifest.json" -o "C:\path\to\edited_rejoined_ecu.bin" --allow-modified
```

Use `--overwrite` only when replacing files intentionally. The normal `verify` and `join` commands require the original hashes. Use `--allow-modified` only when you intentionally edited the parts and want to rebuild the combined binary. Use `--pad-last FF` for a fixed-size final part, or another byte value if the receiving workflow specifies it.

## Addressing note

Each output part is a raw binary whose first byte is displayed at offset zero by a normal binary editor. The manifest's `start_offset_hex` is the original ROM offset for that part. If you create EcuFlash definitions for individual parts, use those offsets as the base address rather than treating every part as the beginning of the original ROM.

Splitting a file does not by itself make an ECU image flashable. Checksum correction, segment headers, encryption, bootloader rules, and write order remain specific to the ECU and EcuFlash definition being used.
