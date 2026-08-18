<div align="center">
  <img src="docs/images/logo.png" alt="MediaPeek logo" width="96">
  <h1>MediaPeek</h1>
  <p>A now-playing media card that hides in the top edge of your Windows screen.<br>
  Zero screen space, zero clicks wasted — it isn't there until you want it.</p>
  <p><b>English</b> | <a href="README.zh-CN.md">中文</a></p>
  <img src="docs/images/edge-reveal.gif" alt="Rest the cursor on the top screen edge and the card slides out" width="680">
</div>

## Why

Switching windows just to see what's playing breaks flow. MediaPeek puts the current
track — cover, title, artist — one cursor-flick away, on top of everything including
fullscreen apps, while taking **no screen space at all** when idle: the hidden card's
pixels are literally click-through.

It talks to your player through Windows SMTC, so NetEase Cloud Music, QQ Music,
Spotify, browsers and most other players work with zero setup.

## Features

### Reveal on demand

Rest the cursor on the top screen edge for a beat (~150 ms, so fly-bys to a maximized
window's close button never trigger it) and the card slides out. It stays while hovered
and tucks itself away 600 ms after you leave.

![Edge-dwell reveal](docs/images/edge-reveal.gif)

### Now-playing peeks

The card also surfaces on its own: once when the track changes (NetEase Cloud Music),
then a 3.5-second peek every minute while music is playing. Automatic displays are
two-stage — the title glides in first, then the card stretches to make room for the
cover and artist. Move the cursor onto a peek to take it over.

![Two-stage now-playing peek](docs/images/auto-peek.gif)

The card's left side wears an ambient glow sampled from the album art, and while music
is playing a thin beam of light sweeps its hairline border — both kept strictly inside
the capsule silhouette, so the click-through promise holds.

### Gesture controls

No buttons by default: tap the card to play / pause, flick left or right to switch
tracks (with an instant sideways nudge as feedback), and the card glides to the new
title's width on its own.

![Card gestures](docs/images/gestures.gif)

### Button controls

Prefer explicit transport buttons? Enable them from the tray menu and a
previous / play-pause / next cluster appears on the card's right side.

![Button controls](docs/images/buttons.gif)

### Lives in the tray

Left-click the tray icon to peek the card, right-click for the menu: launch at
startup, control mode, interface language, store updates (packaged builds only),
restart and exit.

### Startup greetings

Every launch plays a short greeting that grows out of the screen edge: a welcome card
with usage hints on first run, a compact hello capsule afterwards.

![First-run welcome card](docs/images/welcome.gif)
![Hello capsule](docs/images/hello.gif)

### And the quiet details

- Always on top without stealing focus; no taskbar button, no Alt+Tab entry — and it
  keeps working over fullscreen video and games.
- Content-sized card: the capsule always wraps the full title and artist, no marquee,
  no truncation, and animates between widths on track changes.
- Multi-language UI (中文 / English), following the system language by default,
  switchable anytime from the tray menu.
- Single instance, crash-safe logging, settings persisted to
  `%AppData%\ImmersiveBar\settings.json`.

## Install

- **Microsoft Store (MSIX):** the listing is being prepared; packaged builds get
  automatic updates and a manifest-declared startup task. See
  [Packaging](#packaging-for-the-microsoft-store) to build the package yourself.
- **Portable exe:** a self-contained single-file build runs from anywhere — see
  [Build from source](#build-from-source).

Requires Windows 10 19041+ (developed on Windows 11).

## Build from source

Prerequisites: .NET 8 SDK with the Windows desktop workload.

```powershell
dotnet build ImmersiveBar.sln
dotnet test tests/ImmersiveBar.Tests
dotnet run --project src/ImmersiveBar
```

Produce the portable single-file exe:

```powershell
dotnet publish src/ImmersiveBar -c Release -r win-x64 --self-contained true `
  /p:PublishSingleFile=true /p:IncludeNativeLibrariesForSelfExtract=true
# Output: src/ImmersiveBar/bin/Release/net8.0-windows10.0.19041.0/win-x64/publish/ImmersiveBar.exe
```

## Packaging for the Microsoft Store

```powershell
powershell -ExecutionPolicy Bypass -File packaging\Pack-Msix.ps1          # unsigned, for store submission
powershell -ExecutionPolicy Bypass -File packaging\Pack-Msix.ps1 -Sign    # dev-signed, for local sideloading
```

The script publishes, assembles and validates `artifacts\msix\ImmersiveBar.msix`.
Before submitting, set the `Identity` values in `packaging\Package.appxmanifest` from
Partner Center and bump `Identity@Version`. Packaged builds self-update through the
Store and auto-start via the manifest `StartupTask` instead of the registry Run key.

## How it works

- C# / WPF on .NET 8 (`net8.0-windows10.0.19041.0`), **zero third-party
  dependencies** — WinRT SMTC and the tray icon are driven through raw interop.
- `ImmersiveBar.Core` is the pure logic layer (view model + SMTC abstraction, no WPF
  references, UI dispatch injected) and is covered by xUnit tests; `ImmersiveBar` is
  the WPF shell (layered click-through window, AppBar docking, animation choreography,
  tray, settings, localization).
- The window spans the monitor's top edge but reserves nothing (zero-height AppBar);
  all reveals are render-thread `DoubleAnimation`s, so motion stays smooth on
  high-refresh displays.
