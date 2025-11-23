# Build ArchBANG ISO

Build the ArchBANG ISO image using the project's build script.

## Usage

```
/build-iso [options]
```

## What This Does

1. Runs the `./build` script to create the ISO
2. Validates the build configuration
3. Reports build status and output

## Prerequisites

- All build dependencies installed (mkarchiso, etc.)
- Sufficient disk space (typically 2-5GB for build artifacts)
- sudo access (build script requires elevated privileges)

## Build Script

The `/home/mrgreen/archbang/build` script is the main build executable.

## Options

You can pass additional options to the build command:
- `-c` : Clean previous builds before starting
- `-m` : Minimal build (skip some optional steps)

## Example

```
/build-iso -c
```

This rebuilds the ISO from scratch, cleaning previous artifacts.

## Troubleshooting

- Check disk space if build fails
- Ensure pacman.conf is correct
- Verify packages.x86_64 has valid package names
- Review the build script for detailed error messages
