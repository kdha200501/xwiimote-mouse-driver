# Setting up the WiiMote as a pointing device

This guide walks through getting the WiiMote working as a mouse with
`xwiimote-mouse-driver`.

## Dependencies

Before building, the [libxwiimote](https://github.com/xwiimote/xwiimote)
library (and its build toolchain) is required. On a Fedora-family system the
following three packages provide it:

```
sudo dnf install libtool ncurses-devel systemd-devel
```

- `libtool` - build tooling needed to build libxwiimote
- `ncurses-devel` - terminal UI used by the xwiimote test utilities
- `systemd-devel` - provides the `libudev` headers/pc file that libxwiimote
  links against

## Pair the WiiMote

##### Start scanning

```shell
$ bluetoothctl scan on
```

Take the battery cover off and press the red `SYNC` button - the LEDs
blink to signify pairing. The WiiMote will appear in the scan output as
`Nintendo RVL-CNT-01` - take note of its **MAC address** from there.

> [!NOTE]
>
> Pairing with the `1`+`2` buttons is temporary - the WiiMote does
> not keep the pairing and cannot reconnect later (`bluetoothd` logs
> `Host is down`). Pair with the red `SYNC` button instead, which stores
> the pairing permanently.



Optional: if a previous setup is not working, remove the old pairing
(which may have been created with the temporary `1`+`2` method) before
pairing again:

```shell
$ bluetoothctl remove AA:BB:CC:DD:EE:FF
```



##### Pair, trust and connect

```shell
$ bluetoothctl pair AA:BB:CC:DD:EE:FF
$ bluetoothctl trust AA:BB:CC:DD:EE:FF
$ bluetoothctl connect AA:BB:CC:DD:EE:FF
```

> [!TIP]
>
> The trust command removes the need to enter PIN



Verify:

```shell
$ bluetoothctl info AA:BB:CC:DD:EE:FF | grep -E "Paired|Trusted|Connected"
```

> [!NOTE]
>
> To reconnect later, press any single button such as A. The WiiMote connects
> to the computer by itself - do not use `1`+`2` or `SYNC` for re-connection.



If it fails, press a button while watching `journalctl -u bluetooth -f`
and try fresh batteries.

## Build and run the driver

##### Build

Configure and build:

```shell
$ git checkout customize/main
$ cmake -B build -S .
$ cmake --build build
```

The driver binary is `build/xwiimote-mouse-driver`.

##### Run

The driver creates a virtual mouse from the wiimote. It requires root
access:

```shell
$ sudo ./build/xwiimote-mouse-driver
```

Options:

| Option | Description |
| ------ | ----------- |
| `--socket-path=<path>` | Path to the control socket (default `./wiimote-mouse.sock`) |
| `--config-file=<path>` | Path to the config file (default `./wiimote-mouse.conf`) |
| `--help` | Print a help message |
| `--version` | Print the version number |

On first run, the driver writes a default `wiimote-mouse.conf` next to the
socket file. If no wiimote is connected, the driver waits and auto-reconnects
when one appears.

> [!NOTE]
>
> The driver grabs the **first** available wiimote. With multiple wiimotes
> connected, only one is used and the rest are ignored until the active one
> disconnects.

##### Calibrate and configure

With the driver running, start the configuration utility (needs
`tkinter` and `numpy` available to python3):

```shell
$ python3 ./src/py/configurator.py --socket-path ./wiimote-mouse.sock
```

It is used to calibrate the IR sensor to your screen and to map wiimote
buttons to mouse/keyboard buttons - see the
[project documentation](https://mrapplejuice.github.io/xwiimote-mouse-driver/#)
for details.
