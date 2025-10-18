# fxa500-espat-driver


## Нотую процес збирання прошивки esp-at.

Прошиква тут:
https://github.com/espressif/esp-at

Шо нам треба поправити:

1. Конфігурацію виводів:

Что нам відомо:
- UART0 is used to download firmware and log output
- UART1 is used to send AT commands and receive AT responses
- Both UART0 and UART1 use 115200 baud rate for communication by default.

Порт UART для AT-команд.

| ESP32-C6-mini    | MCU             |
| ---------------- | --------------- |
| UART1_RXD, GPIO6 | UART9_TX, PD15  |
| UART1_TXD, GPIO7 | UART9_RX, PD14  |
| UART1_CTS, GPIO5 | UART9_RTS, PD13 |
| UART1_RTS, GPIO4 | UART9_CTS, PD0  |

Порт UART для Логів та прошивки.

| ESP32-C6-mini     | MCU             |
| ----------------- | --------------- |
| UART0_RXD, GPIO17 | UART1_TX, PA9   |
| UART0_TXD, GPIO16 | UART1_RX, PA10  |

Подивимось шо в [офіційній прошивці](https://docs.espressif.com/projects/esp-at/en/latest/esp32c6/AT_Binary_Lists/esp_at_binaries.html)

Начебто все співпадає с налаштуваннями офіційної прошивки.

Так шо спочатку пробуємо прошити як є.
Слідуй [інструкції](https://docs.espressif.com/projects/esp-at/en/latest/esp32c6/Get_Started/Downloading_guide.html).

Я нотую шо робив на MacOS

```
mkdir dist
cd dist
wget https://dl.espressif.com/esp-at/firmwares/esp32c6/ESP32-C6-4MB-AT-V4.1.1.0.zip
unzip ./ESP32-C6-4MB-AT-V4.1.1.0.zip
python -m venv .venv
. ./.venv/bin/activate
./.venv/bin/pip install esptool
```

Підключаю через USB. Дивимось чи бачить комп модуль:
```
ls /dev/cu*
```

В моєму випадку це: `/dev/cu.usbmodem22301`

```
cd ./ESP32-C6-4MB-AT-V4.1.1.0/ESP32-C6-4MB-AT-V4.1.1.0

esptool.py --chip auto --port /dev/cu.usbmodem22301 --baud 115200 --before default_reset --after hard_reset write_flash -z --flash_mode dio --flash_freq 80m --flash_size 4MB 0x0 bootloader/bootloader.bin 0x60000 esp-at.bin 0x8000 partition_table/partition-table.bin 0xd000 ota_data_initial.bin 0x1e000 at_customize.bin 0x1f000 customized_partitions/mfg_nvs.bin
```


```

--flash_mode dio --flash_freq 80m --flash_size 4MB
0x0 bootloader/bootloader.bin
0x8000 partition_table/partition-table.bin
0xd000 ota_data_initial.bin
0x1e000 at_customize.bin
0x1f000 customized_partitions/mfg_nvs.bin
0x60000 esp-at.bin
``


Слідуємо [інструкції](https://docs.espressif.com/projects/esp-at/en/latest/esp32/Compile_and_Develop/How_to_set_AT_port_pin.html)

esp-at присутній як субмодуль, то треба клонувати цей репозиторій з ключем `git clone --recurse-submodules` або виконати:


```
git submodule update --init --recursive
```

