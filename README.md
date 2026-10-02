# PrBoom+ 2.666 — Portable Linux AppImage

![Linux](https://img.shields.io/badge/Platform-Linux%20x86__64-blue?logo=linux&logoColor=white)
![AppImage](https://img.shields.io/badge/Format-AppImage-orange?logo=appimage&logoColor=white)
![License](https://img.shields.io/badge/License-GPL%20v2-green)
![Release](https://img.shields.io/github/v/release/augvalle/prboom-plus?label=Latest%20Release)

A portable, zero-installation **AppImage** build of the classic **PrBoom+ 2.666** Doom source port for Linux. 

Designed to run seamlessly across all modern Linux distributions—including immutable desktop systems like **Fedora Silverblue / Kinoite**, SteamOS, Ubuntu, Debian, Arch, and openSUSE—without needing `root` access or system dependencies.

---

## ⚡ Quick Start

### 1. Download
Grab the latest release from the [**Releases Page**](https://github.com/augvalle/prboom-plus/releases/latest).

### 2. Make it Executable
Open your terminal and run:

```bash
chmod +x PrBoom_Plus-x86_64.AppImage
```

### 3. Launch & Play
Make sure you have a valid Doom IWAD (e.g., `DOOM2.WAD`, `DOOM.WAD`, `TNT.WAD`, `PLUTONIA.WAD`, or `freedoom2.wad`). 

#### Option A (Recommended): Auto-Detection
Place your `.WAD` files in the standard Linux Doom directory:
```bash
mkdir -p ~/.local/share/games/doom
cp /path/to/DOOM2.WAD ~/.local/share/games/doom/
```
Then launch the AppImage directly:
```bash
./PrBoom_Plus-x86_64.AppImage
```

#### Option B: Specify IWAD Path
```bash
./PrBoom_Plus-x86_64.AppImage -iwad /path/to/DOOM2.WAD
```

---

## ✨ Features of this Build

* **Maximum System Compatibility:** Compiled on Ubuntu 20.04 LTS (`glibc` forward-compatibility) to ensure it runs out-of-the-box on older and modern distros alike.
* **Self-Contained:** Bundles all necessary dynamic dependencies (`SDL2`, `SDL2_mixer`, `libfluidsynth`, `libmad`, `libvorbis`, etc.) along with the required `prboom-plus.wad` engine data.
* **Immutable OS Friendly:** Works out of the box on Fedora Silverblue/Kinoite, Steam Deck (Desktop Mode), and openSUSE MicroOS without layering RPMs.

---

## 🛠️ Building from Source

If you want to build this AppImage yourself inside a containerized environment (e.g., Distrobox/Podman):

```bash
# 1. Spin up an Ubuntu 20.04 container
distrobox create -n appimage-builder -i ubuntu:20.04
distrobox enter appimage-builder

# 2. Install build essentials
sudo apt update && sudo apt install -y \
    build-essential cmake git libsdl2-dev libsdl2-image-dev \
    libsdl2-mixer-dev libsdl2-net-dev libpcre2-dev libmad0-dev \
    libfluidsynth-dev libdumb1-dev libvorbis-dev libflac-dev wget file

# 3. Clone and compile
git clone https://github.com/coelckers/prboom-plus.git
cd prboom-plus && mkdir build && cd build
cmake -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_BUILD_TYPE=Release ..
make -j$(nproc)
make install DESTDIR=AppDir

# 4. Copy required WAD data & setup AppRun
find . -iname "prboom-plus.wad" -exec cp {} AppDir/usr/bin/ \;

cat << 'EOF' > AppDir/AppRun
#!/bin/sh
SELF="$(readlink -f "$0")"
HERE="$(dirname "$SELF")"
case "$HERE" in
  */usr/bin) APPDIR="$(dirname "$(dirname "$HERE")")" ;;
  *)         APPDIR="$HERE" ;;
esac
export DOOMWADPATH="$APPDIR/usr/bin:$APPDIR/usr/share/games/doom:$APPDIR/usr/share/prboom-plus:$DOOMWADPATH"
exec "$APPDIR/usr/bin/prboom-plus" "$@"
EOF
chmod +x AppDir/AppRun

# 5. Download linuxdeploy and package
wget [https://github.com/linuxdeploy/linuxdeploy/releases/download/continuous/linuxdeploy-x86_64.AppImage](https://github.com/linuxdeploy/linuxdeploy/releases/download/continuous/linuxdeploy-x86_64.AppImage)
chmod +x linuxdeploy-x86_64.AppImage
./linuxdeploy-x86_64.AppImage --appimage-extract

# Create minimal desktop entries
cat <<EOF> prboom-plus.desktop
[Desktop Entry]
Version=1.0
Type=Application
Name=PrBoom+
Exec=prboom-plus %F
Icon=prboom-plus
Categories=Game;
EOF

echo "iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mP8z8BQDwAEhQGAhKmMIQAAAABJRU5ErkJggg==" | base64 -d > prboom-plus.png

OUTPUT=PrBoom_Plus-x86_64.AppImage ./squashfs-root/AppRun --appdir AppDir --desktop-file prboom-plus.desktop --icon-file prboom-plus.png --output appimage
```

---

## 📜 Credits & Acknowledgments

* **PrBoom+ Development Team & Upstream:** Originally created by Colin Phipps, Florian Schulze, Andrey Budko (entryway), Christoph Oelckers, and contributors.
* **Upstream Repository:** [coelckers/prboom-plus](https://github.com/coelckers/prboom-plus)
* **AppImage Tooling:** [linuxdeploy](https://github.com/linuxdeploy/linuxdeploy) & [AppImageKit](https://github.com/AppImage/AppImageKit)
* **License:** Distributed under the terms of the [GNU General Public License v2](LICENSE).
