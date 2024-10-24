# interfacing with device drivers

## Technical requirements

- A linux based host system
- A micro SD
- Raspberry Pi 3
- A 5V 1A DC power supply
- AN Ethernet cable and port for network connectivity

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
