# Cryptween Binaries

Binary distribution repository for Cryptween projects native modules.

## Structure

```
releases/
├── {project-name}-v{version}/
│   ├── windows-x64-{module}.node
│   ├── linux-x64-{module}.node
│   └── macos-x64-{module}.node
```

## Projects

- **cryptween-daemon**: Cross-exchange trading daemon native modules

## Usage

Binaries are automatically built and published via GitHub Actions on version tags.

Download format:
```
https://github.com/cryptween/cryptween-binaries/releases/download/{project-name}-v{version}/{platform}-{arch}-{module}.node
```