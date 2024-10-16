- Package to build an image

```bash
sudo apt install gawk wget git diffstat unzip texinfo gcc build-essential chrpath socat cpio python3 python3-pip python3-pexpect xz-utils debianutils iputils-ping python3-git python3-jinja2 python3-subunit zstd liblz4-tool file locales libacl1
sudo locale-gen en_US.UTF-8
```

- Package to build Project documentation

```bash
sudo apt install git make inkscape texlive-latex-extra
sudo apt install sphinx python3-saneyaml python3-sphinx-rtd-theme
```

- Yocto Project Development Tasks Manual.

  - Git 1.8.3.1 or greater
  - tar 1.28 or greater
  - Python 3.8.0 or greater.
  - gcc 8.0 or greater.
  - GNU make 4.0 or greater

  * Link documentation: https://docs.yoctoproject.org/ref-manual/system-requirements.html#supported-linux-distributions

- clone repo poky with branch dunfell

- start build

```bash
cd poky
source oe-init-build-env
bitbake core-image-minimal
```

- simulate your image using QEMU

```bash
runqemu qemux86-64
# link using qemu: https://docs.yoctoproject.org/dev-manual/qemu.html#using-the-quick-emulator-qemu

```

- Link doc: https://docs.yoctoproject.org/brief-yoctoprojectqs/index.html
