# Circuitpython port for ESP32-S2

ESP32-S2 is very interesting for its native USB support.
Currently S2 has circuitpyhton and no micropython.

Pinout:

    GND  GND
    3V3  3.3V
    EN   10k 3.3V
    IO0  BTN GND
    IO18 10k 3.3V (pullup for floating RXD)
    IO19 D-
    IO20 D+
    # JTAG pinout 1
    IO11 TDI
    IO12 TCK
    IO13 TDO
    IO14 TMS - BLUE LED - 549ohm - 3.3V
    # JTAG pinout 2
    IO35 TDI
    IO36 TCK
    IO37 TDO
    IO38 TMS - BLUE LED - 549ohm - 3.3V

See also module internal schematics in
[ESP32-S2 WROVER datasheet](https://www.espressif.com/sites/default/files/documentation/esp32-s2-wrover_esp32-s2-wrover-i_datasheet_en.pdf)

To upload circuitpython hold BTN and plug in the board to PC USB.
Serial port "/dev/ttyACM0" should appear on linux, other OS should
get similar serial device with different name.
Using the latest esptool.py v3.0-dev upload the circuitpython for
[Adafruit CircuitPython Saola-1 WROVER board](https://adafruit-circuit-python.s3.amazonaws.com/index.html?prefix=bin/espressif_saola_1_wrover/en_US/).
The latest may not be the one that works best, so here's last known good:

    wget -c https://adafruit-circuit-python.s3.amazonaws.com/bin/espressif_saola_1_wrover/en_US/adafruit-circuitpython-espressif_saola_1_wrover-en_US-20200827-fe73cfb.bin
    python3 esptool.py --chip esp32s2 -p /dev/ttyACM0 --no-stub -b 460800 --before=default_reset --after=hard_reset write_flash --flash_mode qio --flash_freq 40m --flash_size 16MB 0x0000 $1

Power OFF/ON the board without pressing BTN.
S2 should now enumerate as USB-serial "/dev/ttyACM0" and 
USB-storage device "Espressi Saola 1 w/WROVER 1.0" what makes
it very practical, just copy "ecp5.py" and "blink.bit" to the
root of its filesystem and connect to its serial port:

    screen /dev/ttyACM0
    >>> from ecp5 import idcode,prog,flash,flash_read
    IDCODE: 0x21111043
    >>> prog("blink.bit")
    102400 bytes uploaded in 46 ms (2226 kB/s)
    True
    >>> flash("blink.bit")
    102400 bytes uploaded in 16320 ms (6 kB/s)
    4K blocks: 25 total, 25 erased, 25 written.
    True
    >>>

Circuitpython bugs: for the first time, prog("blink.bit")
may be syntax error, for the second time it will succeed:

    >>> from ecp5 import prog,flash
    IDCODE: 0x21111043
    >>> prog("blink.bit")
    Traceback (most recent call last):
    File "<stdin>", line 1
    SyntaxError: invalid syntax
    >>> prog("blink.bit")
    102400 bytes uploaded in 45 ms (2275 kB/s)
    True

To load "autostart.bit" bitstream at power ON, make "main.py":

    import ecp5
    ecp5.prog("autostart.bit")
    ecp5.idcode()

Last "ecp5.idcode()" is to release JTAG pins to high-Z mode
and allow other devices sharing JTAG bus to program ECP5 when
ESP32-S2 is inactive.

Uploading a new bitstream is easy - just overwrite new "autostart.bit" to
USB flash disk and power OFF/ON the board or restart circuitpython
by pressing Ctrl-D at empty prompt:

    >>> Ctrl-D

Warning: there is high chance of linux to freeze during restarting
circuitpython because linux has still some critical bugs at removing and
inserting USB devices.