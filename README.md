# motionphoto-cli

motionphoto-cli combines a still image and its companion video into one Google/Samsung Motion Photo. It can process an individual image/video pair or scan a directory, using EXIF metadata to match files when filenames are unreliable.

This repository is a command-line derivative of the original [MotionPhoto2 project](https://github.com/PetrVys/MotionPhoto2). The original graphical workflow did not fit the headless homeserver automation where this tool is used, so this version removes the Gooey interface and its dependencies. It is intended to run from a terminal, shell script, cron job, container, or server without a desktop environment.

## Why this version exists

Immich stores the still image and companion video as separate files. In the homeserver backup workflow, those files are merged into Motion Photos before the results are synchronized to a Google Pixel and uploaded to Google Photos as an additional backup copy.

The converter itself is independent of Immich, Syncthing, and Google Photos. It can process any compatible image/video directory. The homeserver workflow is documented separately in the [TS VM guide](<HOMELAB_REPOSITORY_URL>/ts_vm/README.md).

Compared with the original GUI-oriented workflow, this version provides:

- Command-line usage without Gooey.
- Fewer Python dependencies.
- A smaller Linux release binary.
- Directory processing suitable for scheduled jobs.
- EXIF-based pairing when filenames do not match.
- Separate output directories so the source library remains unchanged.
- Copying of files that are not part of a motion-photo pair.

## Installation

### Release binary

The release binary is the simplest option for a supported platform.

1. Download the release archive for your platform.
2. Extract it.
3. On Unix-like systems, make the executable runnable:

   ```sh
   chmod +x motionphoto2
   ```

4. Confirm that the command-line interface starts:

   ```sh
   ./motionphoto2 --help
   ```

The Linux binary is built against an earlier glibc version so it can run on older systems such as Debian 12.

ExifTool is required by motionphoto-cli and must be installed separately and available on `PATH`. See the [ExifTool installation instructions](https://exiftool.org/install.html), or follow the [homelab ExifTool setup](<HOMELAB_REPOSITORY_URL>/ts_vm/README.md#541-install-the-pipeline-dependencies).

### Python environment

The source workflow is documented and tested with Python 3.11.

```sh
python3.11 -m venv .venv
.venv/bin/python -m pip install --upgrade pip
.venv/bin/pip install -r requirements.txt
.venv/bin/python motionphoto2.py --help
```

The command-line source version does not require Gooey. Keep ExifTool installed separately and available on `PATH`.

## Usage

### Process a directory

This example scans a directory, pairs files using EXIF metadata, writes Motion Photos to a separate directory, and copies files that were not muxed:

```sh
./motionphoto2 \
  --input-directory /srv/photos/input \
  --output-directory /srv/photos/motionphoto-staging \
  --exif-match \
  --copy-unmuxed
```

With the Python source checkout, replace `./motionphoto2` with:

```sh
.venv/bin/python motionphoto2.py \
  --input-directory /srv/photos/input \
  --output-directory /srv/photos/motionphoto-staging \
  --exif-match \
  --copy-unmuxed
```

Use `--recursive` to process subdirectories. With `--exif-match`, image/video pairing uses Live Photo metadata instead of relying only on filenames. Use `--incremental-mode` when repeating scans and you want to skip photos already present in the output directory.

### Process an individual pair

```sh
./motionphoto2 \
  --input-image ImageFile.HEIC \
  --input-video VideoFile.MP4 \
  --output-file MotionPhoto.HEIC
```

### Use it from a scheduled script

```sh
#!/bin/sh
set -eu

cd /opt/motionphoto-cli
./motionphoto2 \
  --input-directory /srv/photos/input \
  --output-directory /srv/photos/motionphoto-staging \
  --exif-match \
  --copy-unmuxed \
  --incremental-mode
```

The script can then be called by cron or another timer service. Keep synchronization and backup actions as separate steps so that conversion and transfer can be checked independently.

### Options

- `--input-directory DIR` — process photos and videos in a directory.
- `--output-directory DIR` — save results to a separate directory.
- `--recursive` — process subdirectories recursively.
- `--exif-match` — match image/video pairs using Live Photo metadata.
- `--copy-unmuxed` — copy files that are not part of a muxed pair.
- `--incremental-mode` — skip photos already muxed in the output.
- `--no-xmp` — skip XMP processing and use the supported non-XMP motion-photo path.
- `--overwrite` — replace the original image; use with care.
- `--delete-video` — delete the source video after muxing; use with care.
- `--keep-temp` — keep temporary muxing files.

Run `./motionphoto2 --help` for the complete option list.

## Limitations

This version does not provide the original graphical interface. It is intended for command-line and automated workflows.

Motion Photo compatibility depends on the input image, companion video, metadata, and the receiving application. Test representative files before processing a large library. Google Photos applies its own compatibility and HDR processing rules.

## Credits and license

This project is derived from [Petr Vyskocil's MotionPhoto2](https://github.com/PetrVys/MotionPhoto2). The upstream project and its README contain the complete contributor credits and project history.

The original MIT license and copyright notice are retained in [LICENSE](LICENSE):

```text
Copyright (c) 2024 Petr Vyskocil
```

This derivative preserves the upstream license and identifies its command-line and automation-focused changes.
