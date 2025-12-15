# Publishing Guide for Yisangsay

This guide covers how to convert your GIF to ASCII frames and publish the application to various package managers and platforms.

## 1. Converting GIF to ASCII Art Frames

To replace the current ASCII frames with frames extracted from `yi-sang-limbus-company.gif`, you'll need to:

### Step 1: Extract GIF Frames

Use ImageMagick to extract individual frames from the GIF:

```bash
# Install ImageMagick (if not already installed)
# Windows (with chocolatey): choco install imagemagick
# Linux: sudo apt install imagemagick
# macOS: brew install imagemagick

# Extract frames
magick yi-sang-limbus-company.gif -coalesce frame_%03d.png
```

This will create numbered PNG files (frame_000.png, frame_001.png, etc.)

### Step 2: Convert Frames to ASCII

Use one of these tools to convert each frame to ASCII art:

#### Option A: jp2a (Linux/macOS)
```bash
# Install jp2a
# Linux: sudo apt install jp2a
# macOS: brew install jp2a

# Convert each frame (adjust width to 64 characters)
for file in frame_*.png; do
    jp2a --width=64 --output="${file%.png}.txt" "$file"
done
```

#### Option B: ascii-image-converter (Cross-platform)
```bash
# Install (Go required)
go install github.com/TheZoraiz/ascii-image-converter@latest

# Convert each frame
for file in frame_*.png; do
    ascii-image-converter "$file" -W 64 -C --save-txt "${file%.png}.txt"
done
```

#### Option C: img2txt from libcaca (Cross-platform)
```bash
# Install libcaca
# Windows: Available via MSYS2
# Linux: sudo apt install caca-utils
# macOS: brew install libcaca

# Convert frames
for file in frame_*.png; do
    img2txt -W 64 "$file" > "${file%.png}.txt"
done
```

### Step 3: Update the Project

1. Replace the existing frame files in the `frames/` directory with your new ASCII art frames
2. Update `src/frames.rs` to match the number of frames you have
3. Adjust animation intervals in the `ANIMATE1_FRAMES` and `ANIMATE2_FRAMES` arrays
4. Test the animation: `cargo run -- animate -v 1`

**Tips for better ASCII art:**
- Experiment with different widths (32, 48, 64 characters)
- Try different ASCII converters - each produces different results
- Consider using colored ASCII if your GIF has distinct colors (requires enabling color support)
- Manually edit frames for better quality if needed

---

## 2. Publishing to Cargo (crates.io)

Cargo is the official Rust package registry. Publishing here makes installation easy with `cargo install yisangsay`.

### Prerequisites

1. Create an account on [crates.io](https://crates.io/)
2. Get your API token: Go to Account Settings → API Tokens → New Token
3. Save the token locally:
   ```bash
   cargo login <your-api-token>
   ```

### Pre-publication Checklist

Ensure your `Cargo.toml` has these required fields:

```toml
[package]
name = "yisangsay"
version = "0.1.0"
edition = "2024"
description = "Yisangsay is a CLI program like cowsay, but instead of a talking cow, it's Yi Sang from Limbus Company!"
license = "GPL-3.0"
repository = "https://github.com/VectorSophie/yisangsay-rs"
authors = ["VectorSophie <jay7math@gmail.com>"]
keywords = ["cli", "ascii-art", "cowsay", "limbus-company"]
categories = ["command-line-utilities"]
readme = "README.md"

# Exclude unnecessary files from the package
exclude = [
    ".github/",
    "*.gif",
    "*.png",
    ".gitignore"
]
```

### Publishing Steps

1. **Test the package build**
   ```bash
   cargo build --release
   cargo test
   ```

2. **Dry run the publish**
   ```bash
   cargo publish --dry-run
   ```
   This verifies everything without actually publishing.

3. **Publish to crates.io**
   ```bash
   cargo publish
   ```

4. **Verify the publication**
   - Visit https://crates.io/crates/yisangsay
   - Test installation: `cargo install yisangsay`

### Updating Versions

For subsequent releases:

1. Update the version in `Cargo.toml` (follow [Semantic Versioning](https://semver.org/))
2. Create a git tag:
   ```bash
   git tag v0.2.0
   git push origin v0.2.0
   ```
3. Publish: `cargo publish`

---

## 3. Publishing to APT (Debian/Ubuntu Packages)

APT packages allow Debian and Ubuntu users to install via `apt install yisangsay`.

### Option A: Using cargo-deb (Recommended)

`cargo-deb` automatically creates `.deb` packages from your Cargo project.

#### Installation
```bash
cargo install cargo-deb
```

#### Configuration

Add to your `Cargo.toml`:

```toml
[package.metadata.deb]
maintainer = "VectorSophie <jay7math@gmail.com>"
copyright = "2024, VectorSophie <jay7math@gmail.com>"
extended-description = """\
Yisangsay is a fun CLI tool inspired by cowsay. \
Instead of a cow, Yi Sang from Limbus Company delivers your messages \
in ASCII art form with animations."""
depends = "$auto"
section = "utils"
priority = "optional"
assets = [
    ["target/release/yisangsay", "usr/bin/", "755"],
    ["README.md", "usr/share/doc/yisangsay/", "644"],
]
```

#### Build the .deb package
```bash
cargo deb
```

The package will be created in `target/debian/yisangsay_0.1.0_amd64.deb`

#### Test the package
```bash
sudo dpkg -i target/debian/yisangsay_0.1.0_amd64.deb
yisangsay say "Testing the package!"
```

### Option B: Personal Package Archive (PPA) for Ubuntu

To distribute via APT repositories:

1. **Create an account on Launchpad**
   - Visit [launchpad.net](https://launchpad.net)
   - Set up GPG keys for signing packages

2. **Create a PPA**
   ```bash
   # On Launchpad, create a new PPA (e.g., ppa:vectorsophie/yisangsay)
   ```

3. **Build source package**
   ```bash
   # Install packaging tools
   sudo apt install devscripts debhelper dh-cargo

   # Create Debian packaging files
   mkdir debian
   cd debian
   ```

   Create these files in the `debian/` directory:

   **debian/control:**
   ```
   Source: yisangsay
   Section: utils
   Priority: optional
   Maintainer: VectorSophie <jay7math@gmail.com>
   Build-Depends: debhelper (>= 11), dh-cargo, cargo, rustc
   Standards-Version: 4.5.0

   Package: yisangsay
   Architecture: any
   Depends: ${shlibs:Depends}, ${misc:Depends}
   Description: CLI program featuring Yi Sang from Limbus Company
    Yisangsay is like cowsay, but with Yi Sang from Limbus Company
    delivering your messages in ASCII art with animations.
   ```

   **debian/rules:**
   ```makefile
   #!/usr/bin/make -f
   %:
   	dh $@ --buildsystem=cargo
   ```

   **debian/changelog:**
   ```
   yisangsay (0.1.0-1) focal; urgency=medium

     * Initial release

    -- VectorSophie <jay7math@gmail.com>  Mon, 16 Dec 2024 00:00:00 +0000
   ```

4. **Build and upload**
   ```bash
   debuild -S -sa
   dput ppa:vectorsophie/yisangsay ../yisangsay_0.1.0-1_source.changes
   ```

5. **Users can then install**
   ```bash
   sudo add-apt-repository ppa:vectorsophie/yisangsay
   sudo apt update
   sudo apt install yisangsay
   ```

### Option C: Host .deb files via GitHub Releases

For simpler distribution without maintaining a PPA:

1. Build the .deb package using cargo-deb
2. Create a GitHub release
3. Upload the .deb file as a release asset
4. Users can download and install:
   ```bash
   wget https://github.com/VectorSophie/yisangsay-rs/releases/download/v0.1.0/yisangsay_0.1.0_amd64.deb
   sudo dpkg -i yisangsay_0.1.0_amd64.deb
   ```

---

## 4. Windows Publishing Strategy

Windows users have several installation options:

### Option A: GitHub Releases with Binary

**Most straightforward for users without Rust installed:**

1. **Build Windows binary**
   ```bash
   cargo build --release --target x86_64-pc-windows-msvc
   ```

2. **Create release archive**
   ```bash
   # PowerShell
   Compress-Archive -Path target/release/yisangsay.exe,README.md -DestinationPath yisangsay-windows-x64.zip
   ```

3. **Upload to GitHub Releases**
   - Create a release on GitHub
   - Upload the `.zip` file
   - Users download, extract, and add to PATH

**Usage instructions for users:**
```powershell
# Download and extract yisangsay-windows-x64.zip
# Add to PATH (optional)
$env:Path += ";C:\path\to\yisangsay"

# Or use directly
.\yisangsay.exe say "Hello from Yi Sang"
```

### Option B: Chocolatey Package

Chocolatey is a popular Windows package manager.

1. **Create Chocolatey package**

   Create a `yisangsay.nuspec` file:
   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <package xmlns="http://schemas.microsoft.com/packaging/2015/06/nuspec.xsd">
     <metadata>
       <id>yisangsay</id>
       <version>0.1.0</version>
       <title>Yisangsay</title>
       <authors>VectorSophie</authors>
       <projectUrl>https://github.com/VectorSophie/yisangsay-rs</projectUrl>
       <licenseUrl>https://github.com/VectorSophie/yisangsay-rs/blob/main/LICENSE</licenseUrl>
       <requireLicenseAcceptance>false</requireLicenseAcceptance>
       <description>Yisangsay is a CLI program like cowsay, but featuring Yi Sang from Limbus Company!</description>
       <summary>Yi Sang ASCII art in your terminal</summary>
       <tags>cli ascii-art cowsay limbus-company</tags>
     </metadata>
     <files>
       <file src="tools\**" target="tools" />
     </files>
   </package>
   ```

   Create `tools/chocolateyinstall.ps1`:
   ```powershell
   $ErrorActionPreference = 'Stop'
   $packageName = 'yisangsay'
   $toolsDir = "$(Split-Path -parent $MyInvocation.MyCommand.Definition)"
   $url64 = 'https://github.com/VectorSophie/yisangsay-rs/releases/download/v0.1.0/yisangsay-windows-x64.zip'

   $packageArgs = @{
     packageName   = $packageName
     unzipLocation = $toolsDir
     url64bit      = $url64
     checksum64    = 'INSERT_SHA256_CHECKSUM'
     checksumType64= 'sha256'
   }

   Install-ChocolateyZipPackage @packageArgs
   ```

2. **Test locally**
   ```powershell
   choco pack
   choco install yisangsay -s . -y
   ```

3. **Publish to Chocolatey Community Repository**
   ```powershell
   choco apikey --key YOUR-API-KEY --source https://push.chocolatey.org/
   choco push yisangsay.0.1.0.nupkg --source https://push.chocolatey.org/
   ```

4. **Users install with**
   ```powershell
   choco install yisangsay
   ```

### Option C: winget (Windows Package Manager)

winget is Microsoft's official package manager for Windows 10/11.

1. **Fork the winget-pkgs repository**
   ```bash
   git clone https://github.com/microsoft/winget-pkgs
   cd winget-pkgs
   ```

2. **Create package manifest**

   Create `manifests/v/VectorSophie/Yisangsay/0.1.0/` directory with these files:

   **VectorSophie.Yisangsay.yaml:**
   ```yaml
   PackageIdentifier: VectorSophie.Yisangsay
   PackageVersion: 0.1.0
   DefaultLocale: en-US
   ManifestType: version
   ManifestVersion: 1.5.0
   ```

   **VectorSophie.Yisangsay.locale.en-US.yaml:**
   ```yaml
   PackageIdentifier: VectorSophie.Yisangsay
   PackageVersion: 0.1.0
   PackageLocale: en-US
   Publisher: VectorSophie
   PublisherUrl: https://github.com/VectorSophie
   PackageName: Yisangsay
   PackageUrl: https://github.com/VectorSophie/yisangsay-rs
   License: GPL-3.0
   LicenseUrl: https://github.com/VectorSophie/yisangsay-rs/blob/main/LICENSE
   ShortDescription: CLI program featuring Yi Sang from Limbus Company
   Description: Yisangsay is like cowsay, but with Yi Sang from Limbus Company delivering your messages in ASCII art with animations.
   Tags:
   - cli
   - ascii-art
   - cowsay
   ManifestType: defaultLocale
   ManifestVersion: 1.5.0
   ```

   **VectorSophie.Yisangsay.installer.yaml:**
   ```yaml
   PackageIdentifier: VectorSophie.Yisangsay
   PackageVersion: 0.1.0
   Installers:
   - Architecture: x64
     InstallerType: zip
     InstallerUrl: https://github.com/VectorSophie/yisangsay-rs/releases/download/v0.1.0/yisangsay-windows-x64.zip
     InstallerSha256: INSERT_SHA256_HERE
   ManifestType: installer
   ManifestVersion: 1.5.0
   ```

3. **Submit pull request**
   - Test with `winget validate --manifest manifests/v/VectorSophie/Yisangsay/0.1.0`
   - Create PR to microsoft/winget-pkgs

4. **Users install with**
   ```powershell
   winget install VectorSophie.Yisangsay
   ```

### Option D: Scoop

Scoop is another popular Windows package manager, focused on command-line tools.

1. **Create a bucket (repository)**
   ```bash
   git init scoop-yisangsay
   cd scoop-yisangsay
   ```

2. **Create manifest** (`yisangsay.json`):
   ```json
   {
     "version": "0.1.0",
     "description": "CLI program featuring Yi Sang from Limbus Company",
     "homepage": "https://github.com/VectorSophie/yisangsay-rs",
     "license": "GPL-3.0",
     "architecture": {
       "64bit": {
         "url": "https://github.com/VectorSophie/yisangsay-rs/releases/download/v0.1.0/yisangsay-windows-x64.zip",
         "hash": "sha256:INSERT_HASH",
         "extract_dir": ""
       }
     },
     "bin": "yisangsay.exe"
   }
   ```

3. **Publish the bucket**
   ```bash
   git add yisangsay.json
   git commit -m "Add yisangsay package"
   git push origin main
   ```

4. **Users install with**
   ```powershell
   scoop bucket add yisangsay https://github.com/VectorSophie/scoop-yisangsay
   scoop install yisangsay
   ```

---

## Summary of Publishing Options

| Platform | Difficulty | Reach | Auto-updates | Recommendation |
|----------|-----------|-------|--------------|----------------|
| **Cargo** | Easy | Rust users | ✓ | Essential |
| **GitHub Releases** | Easy | All | ✗ | Essential |
| **cargo-deb** | Easy | Debian/Ubuntu | ✗ | Recommended |
| **PPA** | Hard | Ubuntu | ✓ | Optional |
| **Chocolatey** | Medium | Windows | ✓ | Recommended |
| **winget** | Medium | Windows 10/11 | ✓ | Recommended |
| **Scoop** | Easy | Windows | ✓ | Optional |

### Recommended Priority

1. **Cargo (crates.io)** - Essential for Rust users
2. **GitHub Releases** - Binary downloads for all platforms
3. **cargo-deb** - Easy Debian/Ubuntu support
4. **winget or Chocolatey** - Choose one for Windows users

---

## Automation with GitHub Actions

Consider setting up CI/CD to automate releases:

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]

    steps:
      - uses: actions/checkout@v3

      - name: Install Rust
        uses: actions-rs/toolchain@v1
        with:
          toolchain: stable

      - name: Build
        run: cargo build --release

      - name: Create Release
        uses: softprops/action-gh-release@v1
        with:
          files: |
            target/release/yisangsay
            target/release/yisangsay.exe
```

This automatically builds and publishes binaries when you create a git tag.
