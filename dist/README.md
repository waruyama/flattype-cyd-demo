# dist/

Build output for the Flattype CYD demo.

- `firmware.bin` — merged ESP32 image (bootloader + partition table + app, 4 MB) for the Cheap Yellow Display (ESP32-2432S028R). Flashes at offset `0x0`. Free to use and redistribute — see [`LICENSE.txt`](LICENSE.txt).
- `index.html`, `manifest.json` — local copy of the [esp-web-tools](https://esphome.github.io/esp-web-tools/) installer page. The hosted version at <https://waruyama.github.io/flattype-cyd-demo-install/> is the one users should normally use; this copy is for offline / local serving.

## Flash manually

```bash
esptool.py --chip esp32 --port /dev/ttyUSB0 write_flash 0x0 firmware.bin
# or
espflash write-bin 0x0 firmware.bin
```
