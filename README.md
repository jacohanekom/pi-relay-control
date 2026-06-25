# pi-relay-control

A GPIO relay control daemon for Raspberry Pi 5. Exposes a simple TCP socket interface to turn a relay on/off and query its state, with persistent state across restarts.

## Requirements

- Raspberry Pi 5 (ARM64)
- `liblgpio-dev` (build) / `liblgpio` (runtime)
- Relay connected to BCM GPIO pin 5 (configurable)

## Build

```bash
make
```

## Install

```bash
# Install locally
sudo make install

# Or build and install a Debian package
dpkg-buildpackage -us -uc
sudo apt install ../pi-relay-control_*.deb
```

## Configuration

Edit `/etc/pi-relay-control.conf`:

```
gpio_pin 5    # BCM GPIO pin number
port 7778     # TCP socket port
```

## Usage

Control the relay with any TCP client, e.g. `nc`:

```bash
echo "on"     | nc localhost 7778    # Turn relay ON  → OK RELAY=ON
echo "off"    | nc localhost 7778    # Turn relay OFF → OK RELAY=OFF
echo "status" | nc localhost 7778    # Query state    → RELAY=ON
```

### Commands

| Command  | Response                                    |
|----------|---------------------------------------------|
| `on`     | `OK RELAY=ON`                               |
| `off`    | `OK RELAY=OFF`                              |
| `status` | `RELAY=ON` or `RELAY=OFF`                   |
| other    | `ERR unknown command. Use: on | off | status` |

## Service management

```bash
sudo systemctl start   pi-relay-control
sudo systemctl stop    pi-relay-control
sudo systemctl enable  pi-relay-control   # start on boot
sudo systemctl status  pi-relay-control
```

The service runs as root, restarts automatically on failure (5 s delay), and persists relay state to `/var/lib/relay_control/state` so the relay returns to its last position after a reboot.
