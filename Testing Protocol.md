# interfacing with device drivers

## Technical requirements

- A linux based host system
- A micro SD
- Raspberry Pi 3
- A 5V 1A DC power supply
- AN Ethernet cable and port for network connectivity

## Testing

- SPI testing
- I2c testing
- UART testing
- PWM testing
- USB testing
- Ethernet testing

## Componets of the system

- In linux, there are threee main types of device driver:

  - Character
  - Block
  - Network

- Example: Can access the GPIO driver through a group of files in `/sys/class/gpio`

  Link doc: https://lancesimms.com/RaspberryPi/HackingRaspberryPi4WithYocto_Introduction.html
  Link doc: https://www.ics.com/blog/control-raspberry-pi-gpio-pins-python
  Link doc: https://www.raspberrypi.com/documentation/computers/os.html#use-python-on-a-raspberry-pi
  Link doc: https://realpython.com/python-raspberry-pi/#interacting-with-physical-components
  Link doc: https://www.kdnuggets.com/build-a-command-line-app-with-python-in-7-easy-steps
  Link doc: https://episyche.com/blog/how-to-build-a-cli-tool-using-python

# Structure file in layer for testing protocol

```
meta-protocol
└─recipes-protocol
    └─test-protocol
        ├─ test-protocol.bb
        └─ test-protocol
            ├─ test-protocol.service
            ├─ python-test-protocol
            └─ test-protocol.sh
```

# Managing Services Files In Linux

- System services are programs that run in the background, often started during system boot and managed by the system’s init system.
- One effective way to manage services is through the use of `systemd service files`.
- Linux services manage application processes, ensuring they start automatically and remain running, even after system reboots or crashes.

## .services file?

### Creating a systemd service file

- A systemd service file is a `configuration file` that provides information to the init system about how to manage a specific service. These files are typically stored in the `/etc/systemd/system` dicrectory and have a `.service` extension.
- Exmaple for a hypothetical application called `myapp` that runs as a service:

```myapp.service
[Unit]
Description=UAT Python App
After=network.target

[Service]
User=admin
WorkingDirectory=/home/admin/workDir/
ExecStart=/home/admin/workDir/venv/bin/python app.py
Restart=always

[Install]
WantedBy=multi-user.target
```

- Explain the contents of the file:

  - [Unit]: Contains metadata and dependencies for the service.
    - Description: A human-readable description of the service.
    - After: Specifies that the service should start after the network.target.
  - [Service]: Defines how the service should be managed.
    - User: Specifies the system user name.
    - WorkingDirectory: The path of the application which needs to be hosted.
    - ExecStart: The command to start the service.
    - Restart: Specifies the restart behavior (in this case, always restart).
  - [Install]: Defines when and how the service should be started during boot.
    - WantedBy: Specifies the target that this service should be started with (multi-user.target in this case).

### Managing Services with systemd Commands

- Viewing Services Logs:

```bash
journalctl -ef -u myapp.service
# displays the logs for the specified service in real-time
```

### Reloading Services configuration

```bash
sudo systemctl daemon-reload
```

### Starting, Stopping, Enabling, Restart and Disabling Services

- Starting a service:

```bash
sudo systemctl start myapp.service
```

- Stopping a service:

```bash
sudo systemctl stop myapp.service
```

- Enabling a service:

```bash
sudo systemctl enable myapp.service
```

- Restarting a service:

```bash
sudo systemctl restart myapp.service
```

- Disabling a service:

```bash
sudo systemctl disable myapp.service
```

## systemctl

- `systemctl` is a command-line utility that provides a convenient way to manage system services.
- Note:
  - /etc/systemd/system/ – For custom or overridden service files.
  - /lib/systemd/system/ – For default service files provided by installed packages.

# Python interface with protocol (Raspberry Pi)

## GPIO

```python
import RPi.GPIO as GPIO
import time

GPIOO.setmode(GPIO.BCM)
# code here
```

## SPI

## I2C

## UART

## PWM

## USB

## Ethernet

# Command Line Interface (CLI) for packages using Python

```testing-protocol.sh
#!/bin/bash
python3 test-protocol.py
# Run the script
./testing-protocol.sh
```

# Create a .service file

- This is a service file for the test-protocol package.

```testing-protocol.service
[Unit]
Description=Testing Protocol
After=network.target

[Service]
User=admin
WorkingDirectory=/home/admin/workDir/
ExecStart=/home/admin/workDir/venv/bin/python test-protocol.py
Restart=always

[Install]
WantedBy=multi-user.target

```

# Customizing the file .bb to build with yocto

- This is a recipe file for the test-protocol package.

```testing-protocol.bb
SUMMARY = "Testing Protocol"
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://${COMMON_LICENSE_DIR}/MIT;md5=0835ade698e0bcf8506ecda2f7b4f302"

SRC_URI = "file://test-protocol.py"

S = "${WORKDIR}"

inherit setuptools3

RDEPENDS_${PN} += "python3-rpi.gpio"

do_install() {
    install -d ${D}${bindir}
    install -m 0755 ${S}/test-protocol.py ${D}${bindir}
}

FILES_${PN} += "${bindir}/test-protocol.py"
```
