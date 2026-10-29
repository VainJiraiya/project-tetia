# 🚀 KDE Plasma 6 + Kirigami + PySide6: The Starting Carepackage

> **Your one-stop reference** for building native KDE Plasma 6 applications with Python, PySide6, and Kirigami—on NixOS.

---

## 📋 Table of Contents

1. [What You Have](#1-what-you-have)
2. [The Flake](#2-the-flake)
3. [VS Code on NixOS](#3-vs-code-on-nixos)
4. [Your First Project](#4-your-first-project)
5. [Python ↔ QML Bridge](#5-python--qml-bridge)
6. [Kirigami Essentials](#6-kirigami-essentials)
7. [Roadmap: Media Library App](#7-roadmap-media-library-app)
8. [NixOS Troubleshooting](#8-nixos-troubleshooting)
9. [Resources](#9-resources)

---

## 1. What You Have

This carepackage contains:

- **`flake.nix`** — A reproducible Nix development shell with everything you need:
  - Python 3 + PySide6 (official Qt6 Python bindings)
  - KDE Frameworks 6: Kirigami, KConfig, KIO, Baloo, KFileMetaData
  - Qt6: Base, Declarative (QML), Multimedia, SVG
  - Build tools: CMake, Ninja, pkg-config
  - Distribution tools: flatpak-builder, appstream

- **This guide** — How to use it, plus a project template and roadmap.

**You do NOT need:**
- `venv` / `virtualenv` / `pipx`
- System-wide Python packages
- Manual Qt/KDE library installation

The flake *is* your environment.

---

## 2. The Flake

### 2.1 Enter the Shell

```bash
cd your-project/
git init          # Flakes only see git-tracked files
git add flake.nix
nix develop       # Enter the development shell
```

You should see the welcome banner and PySide6 version.

### 2.2 What Each Package Does

| Package | Purpose |
|---------|---------|
| `python3.withPackages (ps: [ pyside6 ])` | Python + official Qt6 bindings. Includes `shiboken6` automatically. |
| `kdePackages.kirigami` | KDE's adaptive UI framework. Provides `org.kde.kirigami` QML imports. |
| `kdePackages.kirigami-addons` | Extra components: `AboutPage`, `SettingsPage`, `FormLayout`, etc. |
| `kdePackages.qqc2-desktop-style` | **Critical.** The Plasma 6 visual style. Without this, your app looks generic. |
| `kdePackages.kconfig` | Persistent settings (window size, last folder, preferences). |
| `kdePackages.kio` | Native file dialogs, network shares, trash, recent files. |
| `kdePackages.baloo` | File indexer. Search "all photos from 2023" instantly. |
| `kdePackages.kfilemetadata` | Extract EXIF, ID3, video codecs, comic metadata. |
| `kdePackages.knewstuff` | Let users download themes/scrapers from within your app. |
| `kdePackages.qtwayland` | Wayland support. Required if you run a Wayland session (default on NixOS KDE). |
| `qt6.qtbase` | Core Qt: `QtCore`, `QtGui`, `QtNetwork`. |
| `qt6.qtdeclarative` | QML runtime, `QtQuick`, `QtQml`. |
| `qt6.qtmultimedia` | Video/audio playback for your media app. |
| `qt6.qtsvg` | SVG icons and scalable graphics. |
| `qt6.qtimageformats` | WebP, TIFF, and other image formats. |
| `qt6.qt5compat` | Compatibility layer for transitioning Qt5→Qt6 code. |
| `flatpak-builder` | Build Flatpaks for distribution on Flathub. |
| `appstream` | Generate AppStream metadata (required by Flatpak/app stores). |

### 2.3 Environment Variables

The `shellHook` sets these automatically:

- **`QT_PLUGIN_PATH`** — Where Qt finds platform/style plugins
- **`QML2_IMPORT_PATH`** — Where QML finds `org.kde.kirigami` and friends
- **`QT_QUICK_CONTROLS_STYLE=org.kde.desktop`** — Forces the Plasma 6 native look
- **`XDG_DATA_DIRS`** — KDE data directories for icons, themes, etc.

**Without these, your app will either crash on launch or look like a generic Qt app.**

---

## 3. VS Code on NixOS

### 3.1 Should You Add VS Code to Your User Packages?

**No.** Do not add `pkgs.vscode` to `home.packages` or `environment.systemPackages`.

Instead, use **Home Manager's dedicated VS Code module**:

```nix
# ~/.config/home-manager/home.nix  (or your user packages.nix)
programs.vscode = {
  enable = true;

  # Optional: pre-install extensions declaratively
  extensions = with pkgs.vscode-extensions; [
    # mkhl.direnv          # Auto-load Nix env in VS Code
    # ms-python.python     # Python language support
    # ms-python.vscode-pylance
  ];

  # Optional: declarative settings.json
  userSettings = {
    "python.defaultInterpreterPath" = "\${env:PYTHONPATH}";
  };
};
```

**Why `programs.vscode.enable` instead of `home.packages`?**

- Creates proper `.desktop` entries and MIME associations
- Can manage extensions and settings declaratively
- Handles the NixOS wrapper correctly (avoids `Cannot open shared object` errors)
- Integrates with `xdg-desktop-portal` for native file dialogs

### 3.2 The Direnv Workflow (Recommended)

Install `nix-direnv` via Home Manager:

```nix
# home.nix
programs.direnv = {
  enable = true;
  nix-direnv.enable = true;
};
```

Then in your project root:

```bash
echo "use flake" > .envrc
direnv allow
```

Now whenever you `cd` into your project, the flake loads automatically. VS Code (with the **direnv** extension) will pick up the environment without launching from terminal.

### 3.3 VS Code Extensions for This Stack

| Extension | Why |
|-----------|-----|
| **mkhl.direnv** | Auto-loads your Nix flake when opening the folder |
| **ms-python.python** | Python language server, debugging |
| **ms-python.vscode-pylance** | Better type checking and IntelliSense |
| **bbenno.qt-qml** | QML syntax highlighting and basic completion |
| **jnoortheen.nix-ide** | Nix language support |

**Launch workflow:**
```bash
cd your-project/
nix develop          # Enter shell
code .               # Launch VS Code with env vars set
```

Or with direnv: just `code .` from anywhere after `direnv allow`.

### 3.4 Qt Creator (Optional)

For heavy QML UI work, Qt Creator is still king:

```bash
# Add pkgs.qtcreator to the flake packages, then:
nix develop
qtcreator &
```

Use Qt Creator for:
- QML visual designer
- QML profiler
- Qt-specific debugging

Use VS Code for:
- Python logic
- Git integration
- General project navigation

---

## 4. Your First Project

### 4.1 Directory Structure

```
my-kde-app/
├── flake.nix              # The dev shell
├── .envrc                 # direnv: "use flake"
├── pyproject.toml         # Python packaging metadata
├── org.kde.myapp.desktop  # Desktop entry
├── org.kde.myapp.svg      # App icon
├── README.md
└── src/
    ├── __init__.py
    ├── __main__.py        # Entry point: python -m src
    ├── app.py             # Main Python application logic
    └── qml/
        └── Main.qml       # Kirigami QML UI
```

### 4.2 `pyproject.toml`

```toml
[project]
name = "my-kde-app"
version = "0.1.0"
description = "A native KDE Plasma 6 application"
requires-python = ">=3.11"
dependencies = [
    # PySide6 comes from the Nix flake, not pip
]

[project.scripts]
myapp = "src.__main__:main"

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

### 4.3 `src/__main__.py`

```python
#!/usr/bin/env python3
import sys
import signal
from pathlib import Path

from PySide6.QtGui import QGuiApplication
from PySide6.QtCore import QUrl
from PySide6.QtQml import QQmlApplicationEngine


def main():
    app = QGuiApplication(sys.argv)
    app.setApplicationName("my-kde-app")
    app.setOrganizationDomain("kde.org")

    # Allow Ctrl+C to quit
    signal.signal(signal.SIGINT, signal.SIG_DFL)

    engine = QQmlApplicationEngine()

    # Load Main.qml from the qml/ directory
    qml_path = Path(__file__).parent / "qml" / "Main.qml"
    engine.load(QUrl.fromLocalFile(str(qml_path.resolve())))

    if not engine.rootObjects():
        sys.exit(-1)

    sys.exit(app.exec())


if __name__ == "__main__":
    main()
```

### 4.4 `src/qml/Main.qml`

```qml
import QtQuick
import QtQuick.Controls as Controls
import org.kde.kirigami as Kirigami

Kirigami.ApplicationWindow {
    id: root
    title: "Hello Kirigami"
    width: 600
    height: 400

    // This is the native Plasma 6 page stack
    pageStack.initialPage: Kirigami.Page {
        title: "Welcome"

        Controls.Label {
            anchors.centerIn: parent
            text: "Hello from Kirigami on NixOS!"
            font.pointSize: 16
        }
    }
}
```

### 4.5 Running It

```bash
nix develop          # Enter the flake shell
python -m src        # Run the app
```

You should see a native-looking KDE window with "Welcome" in the header.

---

## 5. Python ↔ QML Bridge

This is the core pattern for your media app. You write logic in Python and expose it to QML.

### 5.1 Exposing a Python Class to QML

```python
# src/backend.py
from PySide6.QtCore import QObject, Signal, Slot, Property
from PySide6.QtQml import QmlElement

# This declares the QML import "org.kde.myapp 1.0"
QML_IMPORT_NAME = "org.kde.myapp"
QML_IMPORT_MAJOR_VERSION = 1


@QmlElement
class MediaBackend(QObject):
    countChanged = Signal()

    def __init__(self, parent=None):
        super().__init__(parent)
        self._count = 0

    @Property(int, notify=countChanged)
    def count(self):
        return self._count

    @count.setter
    def count(self, value):
        if self._count != value:
            self._count = value
            self.countChanged.emit()

    @Slot(result=str)
    def scanFolder(self, path: str) -> str:
        """Scan a folder and return a summary."""
        from pathlib import Path
        files = list(Path(path).glob("*"))
        self.count = len(files)
        return f"Found {len(files)} items"
```

### 5.2 Using It in QML

```qml
import QtQuick
import QtQuick.Controls as Controls
import org.kde.kirigami as Kirigami
import org.kde.myapp 1.0

Kirigami.ApplicationWindow {
    id: root

    MediaBackend {
        id: backend
    }

    pageStack.initialPage: Kirigami.Page {
        title: "Media Scanner"

        Column {
            anchors.centerIn: parent
            spacing: Kirigami.Units.largeSpacing

            Controls.Label {
                text: "Files found: " + backend.count
                font.pointSize: 20
            }

            Controls.Button {
                text: "Scan ~/Pictures"
                onClicked: {
                    let result = backend.scanFolder("/home/YOURNAME/Pictures");
                    console.log(result);
                }
            }
        }
    }
}
```

### 5.3 Key Rules

- Inherit from `QObject`
- Use `@QmlElement` to auto-register the type
- Properties need a `Signal` for change notification
- Use `@Slot` for methods callable from QML
- Use `@Property(type, notify=signal)` for bindable data

---

## 6. Kirigami Essentials

Kirigami is KDE's convergent UI framework. It adapts between desktop and mobile.

### 6.1 Core Components

| Component | Use For |
|-----------|---------|
| `Kirigami.ApplicationWindow` | Root window. Has a built-in page stack. |
| `Kirigami.Page` | A screen/page in your app. |
| `Kirigami.PageStack` | Navigation stack. Push/pop pages. |
| `Kirigami.Action` | Toolbar and menu actions. |
| `Kirigami.Card` | Content cards (your media grid items). |
| `Kirigami.CardsGridView` | A grid of cards. Perfect for image thumbnails. |
| `Kirigami.CardsListView` | A list of cards. Good for file lists. |
| `Kirigami.OverlaySheet` | Bottom sheet / modal panel. |
| `Kirigami.GlobalDrawer` | Hamburger sidebar menu. |

### 6.2 Your Media Grid (The Core UI)

```qml
import QtQuick
import org.kde.kirigami as Kirigami

Kirigami.CardsGridView {
    id: grid
    model: mediaModel   // Your Python QAbstractListModel

    delegate: Kirigami.Card {
        banner {
            source: model.thumbnailUrl
            title: model.title
        }
        contentItem: Controls.Label {
            text: model.tags
            elide: Text.ElideRight
        }

        // Click to open
        MouseArea {
            anchors.fill: parent
            onClicked: pageStack.push("qrc:/qml/DetailPage.qml", { itemId: model.id })
        }
    }
}
```

### 6.3 Kirigami Units

Always use Kirigami's spacing units instead of hardcoded pixels:

```qml
spacing: Kirigami.Units.largeSpacing      # 18px equivalent
padding: Kirigami.Units.gridUnit          # Base grid unit (≈18px on desktop)
font.pointSize: Kirigami.Theme.defaultFont.pointSize
```

This ensures your app scales correctly on different DPIs and form factors.

---

## 7. Roadmap: Media Library App

Your dream app: a unified library for photos, videos, comics, and manga.

### Stage 1: The Shell (Week 1)
**Goal:** A working Kirigami app with a sidebar and empty pages.

- Set up the flake and project structure
- Create `ApplicationWindow` with `GlobalDrawer` (sidebar)
- Add placeholder pages: "Photos", "Videos", "Comics"
- Learn `pageStack.push()` navigation

### Stage 2: The Image Grid (Week 2)
**Goal:** Browse a folder of images.

- Build a `QAbstractListModel` in Python
- Populate it from `pathlib.Path(folder).glob("*.jpg")`
- Display in `Kirigami.CardsGridView`
- Generate thumbnails using `QImageReader` + `QPixmapCache`
- Click to open fullscreen image viewer

### Stage 3: The Comic Reader (Weeks 3-4)
**Goal:** Read CBZ/CBR archives.

- Add `zipfile` / `rarfile` support
- Extract images to a temporary cache
- Build a "reader" page with pagination (arrow keys, scroll)
- Remember last-read page with `KConfig`
- Add reading direction toggle (L→R, R→L, vertical)

### Stage 4: Video Player (Week 5)
**Goal:** Basic video browsing.

- Use `QtMultimedia.MediaPlayer` in QML
- Add a video grid with duration thumbnails
- Integrate with Plasma's MPRIS2 (media controls widget)

### Stage 5: The Unified Library (Weeks 6-8)
**Goal:** One database, multiple providers.

- SQLite database for metadata
- Abstract `MediaProvider` base class:
  - `FolderProvider` (photos/videos)
  - `ComicProvider` (CBZ/CBR)
  - `VideoProvider` (thin wrapper)
- Global search across all providers
- Tagging system

### Stage 6: KDE Integration (Ongoing)
**Goal:** Feel truly native.

- `KFileMetaData` for auto-extraction
- `Baloo` integration for fast search
- `KIO` for network shares (SMB, SFTP, Google Drive)
- `KNewStuff` for downloadable scrapers
- Flatpak packaging for Flathub

---

## 8. NixOS Troubleshooting

### "Failed to load QML module org.kde.kirigami"

**Cause:** `QML2_IMPORT_PATH` is not set or wrong.

**Fix:** Ensure you're inside `nix develop`. The `shellHook` sets this. If running outside, set it manually:
```bash
export QML2_IMPORT_PATH="/run/current-system/sw/lib/qt-6/qml"
```

### "QQmlApplicationEngine failed to load component"

**Cause:** Qt can't find the KDE style plugin.

**Fix:** Check `QT_PLUGIN_PATH` includes `qqc2-desktop-style`.

### App looks like Windows 95 / generic Qt

**Cause:** `QT_QUICK_CONTROLS_STYLE` is not set to `org.kde.desktop`.

**Fix:** The flake sets this. If running from a launcher, ensure the env var is inherited.

### PySide6 ImportError on NixOS

**Cause:** Running Python outside the flake shell.

**Fix:** Always run from `nix develop` or use direnv.

### Wayland black screen / crash

**Cause:** Missing Wayland platform plugin.

**Fix:** The flake includes `qtwayland`. If issues persist, force X11:
```bash
QT_QPA_PLATFORM=xcb python -m src
```

### "Cannot mix incompatible Qt library"

**Cause:** PySide6 from pip mixed with Qt from Nix.

**Fix:** Never `pip install pyside6`. Use only the Nix-provided PySide6.

---

## 9. Resources

### Official KDE Documentation
- [KDE Develop: Setup with Python](https://develop.kde.org/docs/getting-started/kirigami/setup-python/)
- [KDE Develop: Python App Tutorial](https://develop.kde.org/docs/getting-started/python/python-app/)
- [KDE Develop: Kirigami Components](https://develop.kde.org/docs/getting-started/kirigami/)
- [KDE Invent (source code)](https://invent.kde.org/)

### Qt / PySide6
- [PySide6 Official Docs](https://doc.qt.io/qtforpython-6/)
- [Qt6 QML Reference](https://doc.qt.io/qt-6/qmlreference.html)

### NixOS
- [NixOS Python Wiki](https://wiki.nixos.org/wiki/Python)
- [Nixpkgs Python Manual](https://ryantm.github.io/nixpkgs/languages-frameworks/python/)
- [NixOS Search](https://search.nixos.org/)

### Community
- **KDE Matrix:** [#kde-devel:kde.org](https://matrix.to/#/#kde-devel:kde.org)
- **NixOS Matrix:** [#nixos:kde.org](https://matrix.to/#/#nixos:kde.org)
- **r/kde** and **r/NixOS** for quick questions

---

## Quick Command Reference

```bash
# Enter dev shell
nix develop

# Format the flake
nix fmt

# Check flake outputs
nix flake show

# Run your app
python -m src

# Launch Qt Creator (if enabled in flake)
qtcreator &

# Launch VS Code with env
nix develop --command code .

# Or with direnv: just `code .` after `direnv allow`
```

---

**Happy hacking.** You're building the media app KDE needs.
