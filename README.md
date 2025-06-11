# OpenBMC Hello World Recipe Deployment Guide

> **This is a Hello World recipe being deployed on a prebuilt Romulus image. Let me know if you errors and importantly a better way to do this** 

> ### **CAUTION**: THIS MAYBE NOT THE RIGHT WAY TO DO IT, BUT THIS WORKS AND THE ONLY WAY I COULD MAKE IT WORK.

## Prerequisites

Before starting, ensure you have:
- QEMU installed
- OpenBMC prerequisites installed
- Romulus prebuilt image (search online for download instructions)

## Steps to Replicate

### Build Recipe

1. **Create workspace directory**
   ```bash
   mkdir helloworld
   cd helloworld
   ```

2. **Create source file**
   ```bash
   # Create helloworld.c file in the helloworld directory
   touch helloworld.c
   ```

3. **Setup OpenBMC environment**
   ```bash
   # Clone the OpenBMC repository
   git clone https://github.com/openbmc/openbmc.git
   cd openbmc
   
   # Set up environment for Romulus machine
   source setup romulus
   ```

4. **Initialize recipe workspace**
   ```bash
   # Set up workspace for editing recipes and source files
   devtool add helloworld path/to/helloworld
   ```

5. **Configure recipe files**
   - Edit `helloworld.bb` with the content provided in the Materials section below
   - Create `files` directory at the same level as `helloworld.bb`
   - Add `helloworld.c` source file inside the `files` directory

6. **Build the recipe**
   ```bash
   devtool build helloworld
   ```

### Deploy Binary to Running Machine

1. **Start Romulus machine**
   - Run the prebuilt Romulus image on QEMU ARM
   - Ensure SSH ports are open and port 443 is forwarded

2. **Login to Romulus**
   - Username: `root`
   - Password: `0penBmc`

3. **Get machine IP address**
   ```bash
   ifconfig
   # Note the IP address
   ```

4. **Deploy to target machine**
   ```bash
   # From another terminal on host machine
   devtool deploy-target helloworld root@<romulus_ip>
   ```

5. **Test the deployment**
   ```bash
   # On the Romulus machine
   helloworld
   ```

## Materials

### helloworld.bb Recipe File

```bitbake
SUMMARY = "Simple helloworld application"
SECTION = "examples"
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://${COMMON_LICENSE_DIR}/MIT;md5=0835ade698e0bcf8506ecda2f7b4f302"

SRC_URI = "file://helloworld.c"

S = "${WORKDIR}/sources"
UNPACKDIR = "${S}"

do_compile() {
    ${CC} ${LDFLAGS} helloworld.c -o helloworld
}

do_install() {
    install -d ${D}${bindir}
    install -m 0755 helloworld ${D}${bindir}
}
```

> **Note**: The checksum is of the MIT license file found in the OpenBMC directory (/home/uddalak/trybmc/openbmc/meta/files/common-licenses/MIT)

## Workspace File Structure

### Inside Romulus Workspace
```
workspace/
├── appends/
│   └── helloworld.bbappend
├── conf/
│   └── layer.conf
├── README
└── recipes/
    └── helloworld/
        ├── files/
        │   └── helloworld.c
        └── helloworld.bb
```

### Inside build/romulus Directory
```
build/romulus/
├── bitbake-cookerdaemon.log
├── cache/
├── conf/
├── downloads/
├── sstate-cache/
├── tmp/
└── workspace/
```

## Troubleshooting

- Ensure all prerequisites are properly installed
- Verify the Romulus image is running and accessible via SSH
- Check that the IP address is correctly noted and used in the deploy command
- Confirm the recipe builds successfully before attempting deployment
