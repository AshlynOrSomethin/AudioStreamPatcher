# AudioStream Patcher

A downstream patch and automated build system for Firefox-based browsers that changes how audio volume is handled on Linux.

## What does this do?

Firefox-based browsers normally forward volume changes from individual tabs to the system audio backend through `cubeb`. On Linux systems using PipeWire or PulseAudio, this can cause per-tab volume sliders to affect the browser stream's system mixer volume.

This project patches Firefox's `AudioStream` pipeline so volume adjustments are applied in software before audio is sent to the system.

The result:

- Per-tab volume controls still work.
- Browser audio is adjusted internally.
- The system mixer volume for the browser stream is no longer modified.
- Other applications are unaffected.

The patch applies at the browser audio pipeline level, so it works with regular web audio, HTML5 video, and playback paths that browser extensions cannot reliably intercept at the final audio output stage.

## Supported browsers

This patching approach is intended for Firefox-based browsers, including:

- Firefox
- Firefox ESR
- Zen Browser
- Floorp

This repository's current automated workflow is focused on Zen Browser.

## How it works

The patch replaces backend volume control:

```text
Browser volume change
	|
	v
cubeb_stream_set_volume()
	|
	v
PipeWire / PulseAudio mixer
```

with software-side processing:

```text
Browser volume change
	|
	v
AudioStream stores volume level
	|
	v
PCM samples are scaled in the browser
	|
	v
PipeWire / PulseAudio receives unchanged stream volume
```

This keeps volume handling inside the browser instead of exposing it to the system audio mixer.

## Automated builds

This repository maintains automated downstream builds using GitHub Actions.

Builds are generated from upstream browser source, the patch is applied during compilation, and release artifacts are published from successful runs.

Generated packages currently include:

- Portable tar archives
- Arch Linux packages
- Debian packages
- RPM packages
- Flatpak bundles
- AppImages

## Installation

Download the package format appropriate for your distribution from the Releases page.

Available formats:

### Debian / Ubuntu

Install the `.deb` package:

```bash
sudo apt install ./zen-patched_VERSION_amd64.deb
```

### Arch Linux

Install the `.pkg.tar.zst` package:

```bash
sudo pacman -U zen-patched-linux-x86_64.pkg.tar.zst
```

### RPM-based distributions

Install the `.rpm` package:

```bash
sudo rpm -i zen-patched-VERSION.x86_64.rpm
```

### AppImage

Make it executable and run:

```bash
chmod +x zen-patched-x86_64.AppImage
./zen-patched-x86_64.AppImage
```

### Flatpak

Install the generated Flatpak bundle:

```bash
flatpak install ./zen-patched.flatpak
```

## Building locally

This project is designed around automated builds, but the general process is:

1. Clone this repository.
2. Clone the upstream browser source.
3. Apply the patch.
4. Update the locale pack data if required by the upstream build.
5. Build the browser using the upstream build system.
6. Package the resulting build artifacts.

The GitHub Actions workflow in this repository is the reference build environment.

## License

This repository contains downstream patches and build tooling.

The patches are provided under the Mozilla Public License 2.0 (MPL-2.0).

Upstream browser source code remains subject to its original licensing terms.

## Why this exists

Linux audio stacks provide powerful stream-level controls, but browser tab volume controls interacting with the system mixer can be undesirable for some workflows.

This project keeps browser volume behavior isolated to the browser itself while preserving normal system audio management.