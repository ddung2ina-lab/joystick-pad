# Joystick Gamepad firmware

`JoystickPad.uf2` is the working W25Q32 boot-code version used on the RP2040 Pro Micro board in this project.

- USB HID gamepad plus a serial interface; product name `Joystick Gamepad`.
- VID `239A`, PID `CAFE`.
- Six front keys on GPIO 2–7; joystick click on GPIO 8; analog X/Y on GPIO 26/27.
- Joystick supply: **3V3**; all grounds shared.
- W25Q32 boot code, input logic, and HID mapping retained from the working binary. The public binary changes the USB product label to `Joystick Gamepad` and replaces absolute diagnostic build paths with relative filenames. Binary comparisons verify that only these text strings changed. No separate hardware reflash test is claimed for these text-only changes.

SHA-256 of the distributed UF2:

```text
230913ba278be849b420fdbeb9ccf1f927f1677f98f05bc3cc8f6692183a7b80
```

## Flashing

Hold BOOT while plugging the board into a PC with a USB data cable. Release BOOT when `RPI-RP2` appears, then copy `JoystickPad.uf2` to that drive. The board restarts automatically. Reconnect normally without BOOT, with the stick centred and untouched during start-up.

## Rebuilding the sketch

The Arduino `.ino` source is editable. Button mappings are the `{GPIO, HID button number}` entries in the `buttons[]` array. The default stick-click entry is `{8, 14}` (L3); for a digital L2 button mapping, change that entry to `{8, 9}` and rebuild. This changes a button assignment; it does not add an analog trigger axis. Axis swapping, inversion, and filtering constants are also in the sketch. Copy the newly built UF2 to the board after editing; changing the source file alone does not change already flashed firmware.

The local toolchain used Arduino-Pico **6.3.2** from [Earle Philhower's Arduino-Pico project](https://github.com/earlephilhower/arduino-pico), with these settings:

- Board: Generic RP2040
- Flash allocation: 2 MB, no filesystem
- USB stack: Adafruit TinyUSB
- Required boot2 property for this board's flash: `boot2_W25Q32JVxQ_4_padded_checksum`

After installing that board package, run from the repository root:

```sh
arduino-cli compile \
  --fqbn rp2040:rp2040:generic:flash=2097152_0,usbstack=tinyusb \
  --build-property build.boot2=boot2_W25Q32JVxQ_4_padded_checksum \
  --output-dir build \
  firmware/JoystickPad
```

The line continuation above uses a POSIX shell; on Windows PowerShell use one line or PowerShell's backtick continuation. Do not omit the boot2 property when rebuilding for the tested Winbond W25Q32 board. Generic SPI boot code did not enumerate over normal USB on that board.

A new build can have a different checksum from the distributed UF2. The checksum above identifies the actual repaired binary, not every rebuild of the source. Boards with different flash chips may require a different boot2 selection.


## Prepare a UF2 for publication

The compiled UF2 is distributed through Releases, not tracked in the source repository. After building, remove absolute build-user paths from diagnostic strings with:

```sh
python firmware/make_public_uf2.py build/JoystickPad.ino.uf2 JoystickPad.uf2
```

This keeps flash addresses, boot code, and program instructions unchanged. The script only replaces NUL-terminated absolute user-directory paths in diagnostic strings. The published file's checksum above identifies the actual release asset.
