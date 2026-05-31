# Taskbar Music Lounge

Native-style Windhawk tool mod that adds a compact music panel to the Windows 11 taskbar area. It shows current media metadata, album art, playback controls, and per-app volume control.

## Features

- Event-driven media updates through Windows GSMTC events.
- Current track title, artist, and rounded album art.
- Playback controls: previous, play/pause, next.
- Disabled controls are detected from GSMTC capabilities and ignored safely.
- Mouse wheel changes the volume of the active media app through Core Audio sessions.
- Smart visibility behavior for fullscreen apps, idle/paused playback, and taskbar auto-hide.
- Acrylic/blur background with rounded Windows 11 styling.
- DPI-aware positioning for scaled displays.
- Horizontal ticker animation for long track names.
- Light/dark text color support with optional manual color.

## Requirements

- Windows 11.
- Windhawk with tool mod support.
- A media app that exposes metadata through Global System Media Transport Controls.
- For best layout results, disable the default Widgets button if it overlaps the panel.

## Installation

1. Open Windhawk.
2. Create or edit a local mod.
3. Paste the contents of `mod.cpp`.
4. Compile and enable the mod.
5. Adjust the mod settings in Windhawk.

The mod is configured for `explorer.exe` and runs as a Windhawk tool mod.

## Controls

- Left click previous icon: previous track, if supported by the current media session.
- Left click play/pause icon: toggle playback, if supported.
- Left click next icon: next track, if supported.
- Mouse wheel over the panel: change the active media app volume.

Volume scrolling changes the Windows audio session volume for the app that owns the current GSMTC session. For browser playback, this usually means the browser process such as `chrome.exe`, `msedge.exe`, or `firefox.exe`, not a single YouTube tab. If the matching audio session cannot be found, the system master volume is left unchanged.

## Settings

| Setting | Default | Description |
| --- | ---: | --- |
| `PanelWidth` | `300` | Panel width in logical pixels. |
| `PanelHeight` | `48` | Panel height in logical pixels. |
| `FontSize` | `11` | Track text font size. |
| `ButtonScale` | `1.0` | Scales playback icons and hit targets. |
| `HideFullscreen` | `false` | Hide the panel while fullscreen or presentation mode is active. |
| `IdleTimeout` | `0` | Hide after this many paused seconds. `0` disables idle hiding. |
| `OffsetX` | `2` | Horizontal offset from the taskbar left edge. |
| `OffsetY` | `2` | Vertical offset from the taskbar center. |
| `AutoTheme` | `true` | Use system light/dark mode for text color. |
| `TextColor` | `0xFFFFFF` | Manual text color when `AutoTheme` is disabled. |
| `BgOpacity` | `0` | Acrylic tint opacity from `0` to `255`. |

## Architecture

The mod uses two worker threads:

- UI thread: owns the hidden/topmost media window, painting, hit testing, taskbar positioning, visibility timers, and mouse input.
- Media events thread: owns the GSMTC manager, subscribes to media events, and dispatches media commands.

Media state is stored behind a mutex and copied into the paint path. Album art uses `shared_ptr<Bitmap>` so the paint thread can safely draw while the media thread updates metadata.

GSMTC events used:

- `CurrentSessionChanged`
- `SessionsChanged`
- `PlaybackInfoChanged`
- `MediaPropertiesChanged`

The code avoids constant media polling. UI refresh messages are coalesced so repeated media events do not flood the window message queue.

## Build Notes

The mod uses these libraries:

```text
-lole32 -luuid -ldwmapi -lgdi32 -luser32 -lwindowsapp -lshcore -lgdiplus -lshell32
```

Main Windows APIs used:

- Windows Media Control / GSMTC for media metadata and playback commands.
- Core Audio (`IAudioSessionManager2`, `ISimpleAudioVolume`) for per-app volume.
- GDI+ for drawing text, icons, and album art.
- DWM and undocumented acrylic composition attribute for the panel background.
- WinEvent hooks to track taskbar movement and auto-hide state.

## Known Limitations

- Browser volume is controlled per browser process, not per tab.
- Some apps expose metadata but do not support next/previous controls; those buttons are disabled.
- Some media players may not expose album art through GSMTC.
- Acrylic behavior can vary across Windows builds and themes.
- The settings section in `mod.cpp` is the source of truth for Windhawk settings.

## File Layout

- `mod.cpp`: complete Windhawk mod implementation.
- `README.md`: project documentation.
