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
