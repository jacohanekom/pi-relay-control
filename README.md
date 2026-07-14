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

### From the APT repository

CI publishes to a signed APT repository (shared with other aipicam Raspberry Pi packages) hosted on Cloudflare R2, with two channels:

- **`main`** — pushing a `v*` tag publishes the clean release version here.
- **`nightly`** — every push (to any branch, and PRs) publishes a dev build here, versioned with a `+<UTC timestamp>` suffix.

```bash
curl -fsSL https://repo.aipicam.com/pubkey.asc | sudo gpg --dearmor -o /usr/share/keyrings/aipicam.gpg

# stable releases
echo "deb [signed-by=/usr/share/keyrings/aipicam.gpg] https://repo.aipicam.com main main" | sudo tee /etc/apt/sources.list.d/aipicam.list

# or nightly builds instead
echo "deb [signed-by=/usr/share/keyrings/aipicam.gpg] https://repo.aipicam.com nightly main" | sudo tee /etc/apt/sources.list.d/aipicam.list

sudo apt-get update
sudo apt-get install pi-relay-control
```

Builds run on GitHub's native `ubuntu-24.04-arm` hosted runner (no QEMU). Uses the same `R2_ACCOUNT_ID`, `R2_ACCESS_KEY_ID`, `R2_SECRET_ACCESS_KEY`, `GPG_PRIVATE_KEY`, and `GPG_KEY_ID` repo secrets described in [pi-block-cpu-cores](../pi-block-cpu-cores)'s README, since it publishes into the same shared repo.

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
