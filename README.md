# GhostBar

An ultra-light gaming overlay and instant-replay clipper for Windows 11. It works like Xbox Game Bar, uses a small
fraction of the RAM and CPU, and never knocks you out of a borderless fullscreen game.

* **About 7 MB to download** (exe + DLLs), **~10 MB and 0 % CPU when you aren't gaming**, and **~10-16 MB in Task
  Manager while it records a game**.
* Instant replay, screenshots, per-app audio, Discord-sized exports, Spotify controls, web tabs, any app inside the overlay.
* No hooks, no injection, no game memory access: safe with Vanguard, EAC and BattlEye.

## Quick start

1. Download `GhostBar-win64.zip` from the **Releases** page (or the latest **Actions** build artifact).
2. Unzip anywhere and run `GhostBar.exe`. A ghost icon appears in the tray.
3. Press **Win+J** in a game (Borderless / Windowed Fullscreen) to open the overlay, **Alt+F10** to save the last
   30 seconds, and **Alt+F11** for a screenshot.

The replay buffer starts by itself when a game starts and stops when it closes (see "Lightweight by design").

Requirements: Windows 11 (or Windows 10 22H2), an NVIDIA, AMD or Intel GPU with a hardware video encoder, the
Microsoft Visual C++ Redistributable (already installed on virtually every gaming PC) and the WebView2 Runtime
(preinstalled on Windows 11) for web tabs.

## Lightweight by design

| What | How |
|---|---|
| **Nothing runs when you aren't gaming** | The replay buffer is *auto-start with games* by default. It starts when a game from your list (25 popular titles) runs, or when any borderless/fullscreen non-browser app comes to the front (auto-detect), and stops when that process exits. Otherwise GhostBar is a ~10 MB tray icon with no timers, no polling and 0 % CPU. |
| **Small number in Task Manager** | The working set is trimmed every few seconds while recording and right after the overlay closes. Task Manager's *Memory* column stays around **7-16 MB** while recording (the encoded buffer itself is "standby" memory that Windows reclaims on demand). |
| **One dropdown for the cost** | *Profile*: **Lightest** (720p30, ~10 MB buffer), **Light** (1080p60 10 Mbps, ~40 MB, default), **Balanced**, **Quality**. The estimate is shown next to the setting. |
| **Overlay costs nothing when closed** | GPU device, swap chain, fonts, bitmaps, thumbnails, PDH counters and the mic meter thread are all released on close; the first frame is drawn before the window appears. |
| **Slim FFmpeg** | An overlay vcpkg port builds FFmpeg with only the codecs GhostBar uses (hardware H.264/HEVC/AV1 encoders, AAC, MP4/MKV, D3D11 decode): `avcodec` 13.3 → 3.5 MB, `avformat` 2.1 → 0.8 MB. |
| **Efficient threads** | The audio threads wake 25x/s instead of 100x/s; capture/encode threads opt out of EcoQoS, everything else opts in while hidden. |
| **Web tabs on a diet** | Hidden tabs are suspended and set to low memory; *Free web memory* can drop them immediately (they reload from the saved URL and scroll position), after 2/10/30 min, or never. |

## Status

| | Status |
|---|---|
| a. Tray app, settings persistence, Win+J hook, overlay with correct focus hand-back | **done, built, run** |
| b. Audio widget (devices, master/mic, per-app volume, mic meter, push-to-mute) | **done, built** |
| c. Replay buffer, save-clip hotkey, toast, manual recording, per-game auto-start | **done, built, clip verified** |
| d. Full clip settings UI + Settings window, clip editor (trim), "Optimize for Discord/…" exports | **done, built, exports verified** |
| e. Web tab widget (WebView2, suspend/restore, discard) | **done, built, WebView2 pipeline self-tested** |
| f. App window widget (DWM thumbnail, bring-to-overlay, experimental embed) | **done, built** |
| g. Layouts/profiles, per-game switching, pinned HUD, System / Notes / Diagnostics widgets | **done, built** |
| h. Much lighter (auto-start with games, profiles, memory trim, slim FFmpeg, async overlay open) | **done, measured** |
| i. New look (5 themes, floating bar, gradients, soft shadows, animations, scroll bars, UI scale, app icon) | **done, rendered and checked** |
| j. Screenshots, per-app audio, Now playing / Timer / Quick launch widgets, after-save actions | **done, tested** |

![All widgets, rendered offscreen by `GhostBar.exe --snapshot`](docs-overlay.png)

### Measured on the dev machine (Windows 11, Ryzen + RX 9070 XT, 4K monitor)

| Scenario | Result |
|---|---|
| Idle (no game), overlay hidden | **~10 MB private, 7 threads, 0.000 CPU-s over 10 s** |
| **Recording a running game** (Valorant, "Light" profile), Task Manager *Memory* column | **7-16 MB** (sampled every 5 s; before: ~114 MB) |
| Same, CPU | ~10 % of one core = **0.3 % of this 32-thread PC** |
| Benchmark `--bench-replay`, working set before → after trim | Lightest 75 → 10 MB, Light 93 → 10 MB, Balanced 106 → 10 MB, Quality 141 → 12 MB |
| Where the ~143 MB *commit* goes (Light) | 4 MB baseline, +31 MB GPU driver's D3D11 device, +36 MB AMD encoder runtime, +3 MB audio, + the encoded buffer (~38 MB at 30 s). The driver parts are mostly cold after a trim. |
| Encoded ring buffer, 19 s | Lightest 9 MB, Light 24 MB, Balanced 37 MB, Quality 72 MB |
| Install size (exe + DLLs) | **7.0 MB** (was 17.4 MB with the full FFmpeg) |
| Overlay open | GPU device 61 ms; widget setup is now asynchronous (PDH counters, mic meter, clip list) |
| Per-app audio (powershell.exe playing a 0.04-amplitude tone) | captured peak **0.040**, timeline 8.39 s of audio over 8.50 s |
| Screenshot (WGC one-shot) | ~100-300 ms, saved as PNG |
| Save a 15.5 s 4K60 clip (312 MB) from the ring buffer | **296 ms**, no re-encode, valid MP4 (H.264 + AAC 48 kHz stereo) |
| Save a 20.9 s 1080p60 clip (70 MB) | **58 ms** |
| Lossless trim 10 s out of a 21 s clip | **78 ms** |
| Discord 10 MB export of a 21 s 1080p60 clip | **9.5 MB**, 1080p30, one pass, 3.4 s (GPU decode + AMF encode) |
| WhatsApp 16 MB / Custom 5 MB / Discord 50 MB | 15.5 MB / 4.93 MB / 48.8 MB, all under budget |
| Shorts/TikTok 9:16 | 606×1080 crop of a 1080p source (never upscaled), 3.5 s |
| Timeline thumbnails (10 frames) | ~100 ms |
| WebView2 tab: create → navigate → scroll capture/restore → Low memory → TrySuspend → Resume | all OK, page loaded in 0.7 s |
| All exports and the lossless trim re-verified on the slim FFmpeg | Discord 8.1 MB, WhatsApp 13.2 MB, custom 3.5 MB (4 MB target), Shorts 606x1080, 10/10 thumbnails |
| Compiler warnings (MSVC /W3) | 0 |

"Commit" is what the process has reserved; the number in Task Manager is the *active private working set*, which the
trim keeps small. The buffer holds encoded packets, so its size is set by bitrate × length; the Clipping widget shows
the estimate next to the profile.

---

## Why C++20 + Win32 (not Rust)

Every hard part of this app is a Microsoft C/C++ API: Win32 windows and hooks, DirectComposition, Direct2D,
D3D11 video processing, Windows.Graphics.Capture through C++/WinRT, Core Audio COM, WebView2 and libavcodec (C).
In C++ each of these is a direct header include, with no binding layer to keep in sync or fight. The same applies to
the undocumented-but-stable `IPolicyConfig` and to FFmpeg's `AVD3D11VADeviceContext`, which wraps our own D3D11 device so
frames never leave the GPU. Rust with `windows-rs` would work, but FFmpeg's hardware-context structs and the WebView2
COM surface are much rougher there. The result is a single ~860 KB exe plus the FFmpeg DLLs.

## Anti-cheat safety (Vanguard, EAC, BattlEye)

GhostBar never touches a game process:

* **No DLL injection and no Present/Vulkan hooks.** The overlay is our own top-level window, composed by DWM.
* **No game memory reads or writes, no kernel driver.**
* **Capture uses only official OS APIs**: Windows.Graphics.Capture, with DXGI Desktop Duplication as a fallback.
* **Input**: a `WH_KEYBOARD_LL` hook. It is an OS callback that runs in *our* process; nothing is loaded into the game.
  Two things are injected through `SendInput`, both from our process and neither aimed at the game:
  * a no-op key (VK 0xFF) only when a **Win+** combo fires, so releasing Win doesn't open Start. PowerToys uses the same
    technique.
  * a zero-movement mouse event, only if Windows refuses to give the overlay focus (foreground lock).
* The foreground watcher is an **out-of-context** WinEvent hook (`WINEVENT_OUTOFCONTEXT`), so no DLL is mapped into
  other processes.

## Build

Prerequisites:
* Visual Studio 2022 Build Tools: "Desktop development with C++" with the Windows 11 SDK (10.0.26100) and "C++ CMake tools".
* Git and vcpkg (`git clone https://github.com/microsoft/vcpkg` then `bootstrap-vcpkg.bat`), with `VCPKG_ROOT` set.

From an **x64 Native Tools** prompt:

```bat
cmake --preset release
cmake --build --preset release
```

The output is `build\release\GhostBar.exe`; the FFmpeg DLLs are copied next to it automatically.

The first configure builds FFmpeg from source (about 6 minutes; later builds use the binary cache). `vcpkg-configuration.json`
points vcpkg at `vcpkg-ports/ffmpeg`, a copy of the stock port with extra `configure` flags that leave out every codec
GhostBar doesn't use.

> **Long paths:** vcpkg's FFmpeg build fails if the vcpkg install directory path is very long (>260 chars, e.g. deep
> inside `AppData`). In that case configure with a short install root and build dir:
> `cmake -S . -B C:\gbbuild -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_TOOLCHAIN_FILE=%VCPKG_ROOT%\scripts\buildsystems\vcpkg.cmake -DVCPKG_INSTALLED_DIR=C:\gbdeps`

### Dependencies (vcpkg manifest: `vcpkg.json`)

| Dependency | Why |
|---|---|
| `ffmpeg[avcodec,avformat,swresample,nvcodec,amf,qsv]` | `h264/hevc/av1_nvenc`, `_amf`, `_qsv` hardware encoders; AAC; MP4/MKV muxing; audio resampling. `nvcodec` and `amf` are header-only shims that load the GPU driver's runtime at run time, so one exe works on every vendor. `qsv` pulls in libvpl. FFmpeg is built LGPL (no `gpl` feature). |
| `nlohmann-json` | settings.json / state.json |
| `webview2` | WebView2 SDK headers + `WebView2Loader.dll` (copied next to the exe). The Evergreen WebView2 Runtime itself ships with Windows 11. |
| Windows SDK | D3D11, DXGI, D2D, DirectWrite, DirectComposition, WIC, Core Audio, C++/WinRT, PDH |

## Using it

| Action | Default |
|---|---|
| Toggle overlay | **Win+J** (low-level hook; works for reserved Win-combos too, e.g. Win+K, since the hook swallows them) |
| Fallback toggle | **Alt+Z** (`RegisterHotKey`; works even if the hook is blocked) |
| Close overlay | **Esc** or the toggle again |
| Save last N seconds | **Alt+F10** (works without opening the overlay) |
| Start/stop recording | **Alt+F9** |
| Screenshot (PNG of the game's monitor) | **Alt+F11** |
| Push-to-mute | not bound (hold-to-mute; set in Settings) |

All hotkeys can be rebound in **Settings**: the gear icon in the overlay's top bar, or **Settings…** in the tray menu,
which opens the same panel as a normal window. Click a binding, then press the new combo. Settings also has every
clip option: capture source (game's monitor, a specific monitor, or the game window), resolution including custom
W×H with letterbox/crop, FPS, codec, encoder, CQP quality slider with presets or VBR/CBR bitrate, encoder preset
P1–P7, keyframe interval, audio tracks, mic device, AAC bitrate, game/mic volume, noise gate threshold, container,
clips folder, per-game subfolders, file-name template and a max folder size. Pipeline changes restart the buffer
when you release the mouse; output options apply instantly.

The tray icon (a ghost in your accent colour) has Open, Replay on/off, Save clip, Record, Settings, folders, Start
with Windows and Exit.

### Screenshots, after-save actions, per-app audio

* **Screenshot** (Alt+F11, tray menu or `--screenshot`): a one-shot capture through the same official Windows capture API
  as the replay buffer. It works even when the buffer is off. Files go to `<clips folder>\Screenshots\<Game>\`.
* **After saving a clip** (Settings → *After saving a clip*): copy the file to the clipboard (paste it straight into
  Discord), play a confirmation sound (off by default: it would be recorded into the next clip's game audio), and
  **also make an optimized copy** (Discord 10 MB, Discord 50 MB, WhatsApp, Shorts) in the background on the GPU, at
  low priority. The shareable copy is what ends up on the clipboard.
* **Only selected apps on the game-audio track** (Settings → *Clip audio* → *Track 1: only selected apps*): tick
  e.g. Valorant and Discord and nothing else (music, notifications, browser) lands in your clips. It uses Windows'
  official process-loopback API (Windows 11 / 10 build 20348+), captures each app *and its child processes*, picks up
  apps that start later, and falls back to the whole desktop mix if the OS doesn't support it.

### Now playing, Timer, Quick launch

* **Now playing**: title, artist, cover art, progress and previous / play-pause / next for whatever is playing (Spotify,
  YouTube in a browser, VLC…). It uses the official Windows media-session API and polls once a second only while the
  widget is shown or pinned.
* **Timer**: stopwatch with laps, or countdown with 30 s / 1 / 2 / 5 / 10 min presets and a custom `mm:ss` box. It keeps
  running while the overlay is closed (wall-clock based, it even survives a restart) and beeps + toasts at zero. Pin it
  to see it on the HUD.
* **Quick launch**: tiles for the apps (real exe icons) and links you reach for mid-game. **+ App…** opens a file picker,
  **+ Link** takes a name and URL, the pencil toggles delete badges.

### Themes and interface size

Settings → *Appearance*: **Midnight** (default), **Graphite**, **Light**, **Aurora**, **Ember**, accent colour (swatches or
hue), panel opacity, corner radius, background dim, and **Interface size** (80-150 %, for 4K screens). The overlay uses a
floating pill top bar, gradient panels, layered soft shadows, animated hover/toggle/slider states and thin scroll bars;
the font is *Segoe UI Variable* on Windows 11.

### Clip editor and "Optimize for…"

Every clip in the Clipping widget has ✂ (trim/export) and ⇪ (compress for Discord) buttons. The editor shows a
preview, a thumbnail timeline with keyframe ticks and draggable In/Out handles, and:

* **Save trimmed copy**: lossless stream copy. It is instant and starts on the keyframe at or before In, at most 1 s
  early with the default 1 s keyframes.
* **Optimize for**: Discord (10 MB), Nitro Basic (50 MB), Nitro (500 MB), WhatsApp (16 MB / 720p), X/Twitter,
  YouTube Shorts/TikTok (9:16 with a draggable crop box on the preview), or a custom size in MB. GhostBar computes the
  bitrate from the clip length and size budget. If that is too low for clean video, it steps down resolution, then
  fps. The editor shows the planned output (e.g. `1920x1080 · 30 fps · H.264 3.5 Mbps · AAC 96 kbps · ≈9.0 MB`)
  before you export. Exports use GPU decode, the GPU scaler and the hardware encoder on a Below Normal thread. If a
  result comes out over budget, it is re-encoded with a corrected bitrate (a true second pass). Audio tracks are mixed
  into one, because Discord and phones only play the first track.

### Web tabs (Steam chat, Discord web, YouTube, wikis…)

"+" → **Web tabs** adds the widget. It has tabs, back/forward/reload, an address bar (plain words search with your
search engine), favourites (☆ toggles, the list button opens one) and a per-tab pin ("keep running while hidden").
All tabs share one WebView2 profile in `%AppData%\GhostBar\WebView2`, so logins survive restarts. Each tab's URL,
title and scroll position are saved. Pop-up windows open as new tabs.

Memory: WebView2 isn't started until a web widget is first shown. When the overlay closes, every tab that isn't
pinned or playing audio is hidden and `TrySuspend()`ed, and all tabs drop to `MemoryUsageTargetLevel = Low`. Music
keeps playing if "Keep audio tabs running" is on. After *N* minutes hidden (default 30, set in Settings → Web tabs),
a single one-shot timer fully discards idle tabs. They reload lazily from the saved URL and scroll position the next
time you look at them.

### App windows (Steam, Discord, Spotify, any app)

"+" → **App window**, then **+** in the widget lists running windows; pick one and its exe is remembered (if it isn't
running later, the widget offers **Launch**).
* **Live preview**: `DwmRegisterThumbnail`. DWM draws a live copy inside the widget, and GhostBar copies no pixels.
* **Click the preview** to bring the real window to the widget's rectangle, topmost above the overlay, where you can
  use it normally. Clicking the overlay keeps it on top. When the overlay closes, the window's original placement,
  z-order and topmost state are restored, or it is minimized if you enable "Minimize on close". Then focus goes back
  to the game.
* **Embed (experimental)** uses `SetParent`. It is off by default because it breaks many Chromium/Electron apps.

Web pages and app previews are native surfaces, so they are temporarily hidden while another panel or a menu overlaps
them, or while you drag the widget.

### Layouts, per-game profiles and the pinned HUD

* The top bar's layout picker switches layouts or makes a new one (a copy of the current one). Settings → Layouts
  renames or deletes the current layout and has **"Use automatically for <game>.exe"**. The overlay then opens with
  that layout whenever that game is in front, and returns to your general layout for other apps.
* **Pin** (header pin icon on Clipping, Audio, System, Notes, Diagnostics): the widget stays on screen as a
  **click-through HUD** while the overlay is closed. It never takes focus and never appears in clips or streams.
  **Scroll the mouse wheel over the pin** to set the HUD opacity (20–100 %). The HUD only redraws if something is
  pinned: 1 Hz, or 10 Hz for the mic meter.

### More widgets

* **System**: CPU/GPU with 60 s graphs, RAM, VRAM, temperature when the firmware exposes ACPI thermal zones (many
  desktops don't; GPU/CPU sensors need vendor drivers), and ping to a host you choose (ICMP, every 2 s, only while
  visible).
* **Notes**: a word-wrapped text pad, autosaved 500 ms after you stop typing.
* **Diagnostics**: GhostBar's own private memory, working set, CPU, GPU-3D and GPU-encoder use, threads/handles and
  open web views, plus the replay buffer's encoder, capture API, output size, buffer RAM and seconds, audio tracks,
  and encoded/dropped frames (green / yellow / red).

### Command line (Stream Deck, AutoHotkey, scripts)

```text
GhostBar.exe --save-clip | --toggle-overlay | --toggle-recording | --screenshot | --settings | --exit
GhostBar.exe --export <file> <discord10|discord50|discord500|whatsapp|twitter|shorts|custom|lossless>
             [--start s] [--end s] [--mb N] [--crop 0..1]
GhostBar.exe --thumbs <file>          (diagnostic: decode the editor's timeline thumbnails)
GhostBar.exe --selftest-web           (diagnostic: WebView2 create/navigate/scroll/suspend/resume, hidden window)
GhostBar.exe --snapshot <out.png> [midnight|graphite|light|aurora|ember]
                                      (diagnostic: render every widget offscreen to a PNG; nothing is shown or saved)
GhostBar.exe --bench-replay <lightest|light|balanced|quality> [seconds] [noaudio] [folder [apps exe1 exe2 ...]]
                                      (diagnostic: run the replay buffer headlessly and report RAM / CPU / buffer size;
                                       with a folder it also saves a clip; "apps" tests per-app audio)
```

The first group controls the running instance. `--export` runs headlessly and prints the plan and result to the
console.

Data lives in `%AppData%\GhostBar\`:
* `settings.json`: hotkeys, theme, clip settings
* `state.json`: layouts, widget positions/sizes, z-order, minimized state, scroll offsets, widget data
* `ghostbar.log`: diagnostics (encoder choice, clip save times, focus hand-back results)

Both JSON files are saved 500 ms after the last change and on exit. Each save goes to a temp file first and is then
swapped in with `ReplaceFile`, which keeps a `.bak`. A crash mid-write never corrupts them; a damaged file falls back to
the `.bak`.

Clips go to `%USERPROFILE%\Videos\GhostBar\<Game>\<Game>_<date>_<time>.mp4` (template and folder are configurable).

## How the fullscreen behaviour works

* **Window:** `WS_POPUP` with `WS_EX_TOPMOST | WS_EX_TOOLWINDOW | WS_EX_NOREDIRECTIONBITMAP`, so there is no taskbar or
  Alt+Tab entry. Content is a Direct2D-drawn flip-model swap chain (premultiplied alpha) attached through
  DirectComposition, the modern form of a per-pixel-alpha layered window. Borderless and "fullscreen optimized" games keep
  running under it; DWM composes it without a display mode change.
* **Open:** remember the foreground HWND (the game) and cover that game's monitor. Release `ClipCursor`. Take focus,
  and if the foreground lock refuses, send one empty mouse input from our process and retry. Fade in with a DComp
  opacity animation that DWM runs, at 90 ms by default. Optional background dim of 0–80 %.
* **Close:** `SetForegroundWindow(game)` *while we are still the foreground process* (always allowed), hide, and restore
  the game's cursor clip. The game gets mouse and keyboard back with zero clicks. Alt+Tab or clicking another app also
  closes the overlay.
* **Exclusive fullscreen:** detected with `SHQueryUserNotificationState == QUNS_RUNNING_D3D_FULL_SCREEN` plus a check
  that the window covers its monitor. The first time, a toast suggests Borderless/Windowed Fullscreen. The overlay then
  opens **view-only** (`WS_EX_NOACTIVATE | WS_EX_TRANSPARENT`), so the game keeps focus and isn't minimized.
* **Never in clips/streams:** `SetWindowDisplayAffinity(WDA_EXCLUDEFROMCAPTURE)` on the overlay and on the toast.
* **Hidden:** no rendering and no timers. The GPU device and swap chain are released, so they are recreated on open.
  PDH stats close and the UI thread switches to EcoQoS (`SetThreadInformation(ThreadPowerThrottling)`). Capture and
  encode threads explicitly opt out of throttling.

### Advanced: uiAccess build (overlay above other topmost windows)

Windows only honours `uiAccess="true"` for a **signed** exe started from a **secure folder** (`C:\Program Files\…`).
uiAccess windows sit in a higher z-band than normal topmost windows. The keyboard hook also keeps working when an
elevated (admin) game has focus.

1. Build: `cmake --preset release-uiaccess && cmake --build --preset release-uiaccess`
2. Create a personal code-signing certificate (elevated PowerShell):
   ```powershell
   $cert = New-SelfSignedCertificate -Type CodeSigningCert -Subject "CN=GhostBar Local" -CertStoreLocation Cert:\CurrentUser\My
   Export-Certificate -Cert $cert -FilePath $env:TEMP\ghostbar.cer
   Import-Certificate -FilePath $env:TEMP\ghostbar.cer -CertStoreLocation Cert:\LocalMachine\Root
   Import-Certificate -FilePath $env:TEMP\ghostbar.cer -CertStoreLocation Cert:\LocalMachine\TrustedPublisher
   ```
3. Sign: `signtool sign /fd SHA256 /n "GhostBar Local" build\release-uiaccess\GhostBar.exe`
4. Copy the exe and the FFmpeg DLLs to `C:\Program Files\GhostBar\` and run it from there.

The certificate is trusted only on your own PC. Delete it from both stores when you no longer need it.

## Architecture

```
src/
  main.cpp                 single instance, COM, App::Run
  app/app.*                tray, message window, hotkey routing, replay auto-start logic
  core/                    settings.h / state.h (JSON schemas), store.* (debounced atomic save), util, log
  input/hotkeys.*          WH_KEYBOARD_LL hook thread + RegisterHotKey fallback, combo parser, rebind capture
  overlay/gfx.*            D3D11 + D2D + DirectComposition swap chain, DWM-side fade
  overlay/ui.*             immediate-mode toolkit: buttons, sliders, toggles, dropdowns, layers, pop-ups
  overlay/overlay.*        the overlay window: open/close/focus, top bar, widget frames, drag/resize/snap
  overlay/toast.*          no-activate, capture-excluded notification
  overlay/widgets/         clip, audio, settings, editor, web, app, media (SMTC), timer + launcher, system/notes/diagnostics, factory
  capture/screenshot.*     one-shot WGC screenshot -> PNG
  encode/app_audio.*       per-app audio: process-loopback clients mixed on an absolute wall-clock timeline
  web/web_env.*            shared lazily-created WebView2 environment (persistent profile)
  overlay/panel_window.*   standalone window hosting a widget (the Settings window)
  media/media.*            probe, thumbnails, export presets/planning, lossless trim, GPU re-encode
  audio/audio_controller.* Core Audio: endpoints, sessions, notifications, IPolicyConfig, mic meter
  capture/                 CaptureDevice, WGC source, DXGI duplication source, D3D11 VideoProcessor (BGRA -> NV12)
  encode/                  video_encoder (FFmpeg NVENC/AMF/QSV on D3D11 frames), audio (WASAPI + AAC + mixer)
  replay/                  packet_ring (encoded ring buffer), muxer (stream-copy MP4/MKV, live MKV), replay_service
  system/system.*          fullscreen detection, EcoQoS, autostart, PDH stats, game/foreground watcher
```

**Replay data path:** WGC frame → one GPU copy → D3D11 VideoProcessor scales and converts to NV12, writing *directly
into an FFmpeg encoder surface* on the same device → hardware encoder → `AVPacket`s in a RAM ring. The keyframe
interval is 1 s and there are no B-frames, so every cut starts cleanly on a keyframe and dts == pts. A save takes
ref-counted packet references under a short lock, then a worker thread stream-copies them to MP4 (`+faststart`).
Audio: WASAPI loopback (track 1, "Game") and microphone (track 2, "Mic"), plus an optional mixed track 3. All are
resampled to 48 kHz, aligned to the same QPC session clock (gaps are filled with silence, so A/V never drifts) and
encoded to AAC. The video thread runs at Above Normal priority, paced by a high-resolution waitable timer at constant
frame rate.

## Known limitations (current build)

* The clip editor previews frames at the In/Out points; it has no video playback (use ▶ to open the clip in your
  player).
* Exports are H.264 only, for compatibility: every chat app and browser plays it inline.
* Per-app audio needs Windows 11 / 10 build 20348+; apps that render audio through a different process than the one
  you picked (some launchers) need that process ticked instead.
* HDR desktops are captured as SDR (8-bit BGRA).
* If a game runs **as administrator**, a non-elevated GhostBar's low-level hook can't see keys while that game has focus.
  The `RegisterHotKey` fallback (Alt+Z) and the save-clip combo then only work if you run GhostBar elevated or use the
  uiAccess build.
* Opening the overlay recreates its GPU device (that is how idle RAM stays near 10 MB). The open time is written to
  `ghostbar.log` ("overlay opened in … ms").

* Web pages and app previews are hidden (not clipped) while another panel overlaps them: DirectComposition content
  can't be drawn on top of native child windows or DWM thumbnails.
* Temperatures come from ACPI thermal zones only (no kernel sensor driver, by design). On many desktops that reports
  nothing.
* Not yet verified on screen in a real game: opening the overlay, focus hand-back, the HUD, and bring-to-overlay.
  Everything else was tested headlessly on the dev PC. TESTING.md is the checklist.

## Possible next steps

* An HDR capture path (FP16 frame pool → P010 → HEVC/AV1 Main10).
* A proper installer (MSIX or Inno Setup) that also performs the uiAccess install to Program Files.
