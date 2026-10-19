# capture_portal

Bash script for lazy linux user

## Features

- **Start portal** --- open a live capture preview using `ffplay`.
- **Capture-card detection** --- scans `/dev/video*` devices and, when
  available, uses `v4l2-ctl` to show device information and supported
  formats/resolutions.
- **Device selection** --- choose a video device such as
  `/dev/video0`.
- **Resolution selection** --- choose a preset or enter a custom
  resolution.
- **Frame-rate selection** --- choose a preset FPS or enter a custom
  value.
- **Input-format selection** --- MJPEG, YUYV422, or let FFmpeg choose.
- **Playback synchronization** --- choose video, audio, or external
  clock synchronization and configure frame dropping.
- **Persistent configuration** --- settings are loaded at startup and
  saved when you change the device, resolution, FPS, format, or
  sync/frame-drop mode.

## Requirements

- Bash
- FFmpeg with `ffplay` enabled
- A Linux V4L2 video device / capture card

Recommended for device detection:

- `v4l-utils` (provides `v4l2-ctl`)

On Debian, Ubuntu, and Raspberry Pi OS, install the dependencies with:

```bash
sudo apt update
sudo apt install ffmpeg v4l-utils
```

Package names may differ on other Linux distributions.

## Installation and usage

Put `capture_portal.sh` in a directory of your choice, then run:

```bash
chmod +x capture_portal.sh
./capture_portal.sh
```

If you prefer, run it with Bash directly:

```bash
bash capture_portal.sh
```

The script checks for `ffplay` at startup. Connect the capture card
before launching the portal for the best chance of detecting it.

## Menu

  --------------------------------------------------------------------------------------

  Option                                             Action

  -------------------------------------------------- -----------------------------------

  `1) Start portal`                                  Start the live preview

  `2) Detect capture cards / list supported modes`   Scan video devices and display
                                                     supported modes where `v4l2-ctl` is
                                                     available

  `3) Select video device`                           Choose a device path

  `4) Select resolution`                             Pick a preset, enter a custom
                                                     resolution, or inspect supported
                                                     modes

  `5) Select frame rate`                             Choose a preset FPS or enter a
                                                     custom value

  `6) Select input format`                           Choose MJPEG, YUYV422, or automatic
                                                     format selection

  `7) Select sync / frame-drop mode`                 Choose playback clock and
                                                     frame-drop behavior

`8) Exit`                                          Close the script
--------------------------------------------------------------------------------------

### Default settings

- Device: `/dev/video0`
- Resolution: `1920x1080`
- Frame rate: `60 FPS`
- Input format: `mjpeg`
- Sync: `video`
- Frame dropping: enabled

The capture device must support the selected resolution, frame rate, and
input format together. If the preview fails to start, use the detector
to inspect the device's supported modes and choose a compatible
combination.

## Automatic configuration saving

The script stores settings in:

```text
~/.config/capture_portal/config
```

If `XDG_CONFIG_HOME` is set, it uses:

```text
$XDG_CONFIG_HOME/capture_portal/config
```

Changes to the device, resolution, FPS, input format, and
sync/frame-drop mode are saved automatically. The next launch loads the
saved settings.

To reset to defaults, exit the script and remove the configuration file:

```bash
rm ~/.config/capture_portal/config
```

If you use a custom `XDG_CONFIG_HOME`, remove the
`capture_portal/config` file under that directory instead. A new
configuration with default values will be created the next time the
script runs.

## Sync and VSync note

The sync menu configures `ffplay` playback synchronization (`-sync`) and
frame-dropping behavior (`-framedrop` / `-noframedrop`). **This is not a
guaranteed hardware/display VSync toggle.** Actual display
synchronization depends on FFmpeg, the graphics/display driver, and the
display pipeline.

For low-latency preview, the default video clock with frame dropping is
a reasonable starting point. If frames appear choppy or the device
cannot sustain the selected mode, try a lower resolution or FPS.

## Troubleshooting

### No capture device appears

1. Check that the capture card is connected.

2. List video nodes:

   ```bash
   ls -l /dev/video*
   ```

3. If installed, list V4L2 devices:

   ```bash
   v4l2-ctl --list-devices
   ```

4. Check kernel messages if the device is not detected:

   ```bash
   dmesg | tail -80
   ```

### Permission denied

Check that your user has permission to access the device. On some
distributions, adding your user to the `video` group may be appropriate;
log out and back in for group changes to take effect. Follow your
distribution's device-permission policy.

### `ffplay` fails to open the stream

The chosen device may not support the selected format, resolution, and
FPS combination. Use the detector's supported-mode listing and try a
compatible setting. Some capture cards expose multiple `/dev/video*`
nodes, and not every node necessarily provides the expected video
stream.

### `v4l2-ctl` is missing

The preview can still use `/dev/video*` devices, but detailed device and
supported-mode information requires `v4l-utils`.
