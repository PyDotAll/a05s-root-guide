# Tool verification notes

Use trusted sources for firmware and flashing tools. Verify release checksums or signatures
where publishers provide them, and do not store authentication tokens or device identifiers in
project files, terminal transcripts, or screenshots.

## LZ4 content-size check

When repacking an image for Odin, include the LZ4 content-size field:

```bash
lz4 -B6 --content-size init_boot.img init_boot.img.lz4
xxd -l 8 -p init_boot.img.lz4
```

The frame header should begin with `04224d186c`. A header beginning with `04224d1864` lacks the
content-size field and may be rejected by Odin.

Check installed tool versions with each tool's version option; do not assume another device's
toolchain or firmware matches yours.
