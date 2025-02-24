## How to add VEYE mipi cameras drivers on OrangePi

### 1. Setting Up the Orange Pi Linux SDK (orangepi-build)

After setting up the Orange Pi Linux SDK, perform an initial build to compile the image for flashing. Use the following command to initiate the build process:

```bash
sudo ./build.sh
```

### 2. Modifying the Configuration File

Navigate to the `orangepi-build/userpatches/config-default.conf` file and update the `IGNORE_UPDATES` variable. Set it to `"yes"` as shown below:

```bash
IGNORE_UPDATES="yes"
```

This step ensures that updates will be ignored during the build process.

### 3. Copying the Camera Driver Source Code

Next, copy all the source files, including the `Kconfig` and `Makefile`, from the `cam_drv_src` directory. These files should be placed in the following directory:

```bash
orangepi-build/kernel/orange-pi-5.10-rk35xx/drivers/media/i2c
```

### 4. Copying Configuration Files

Copy all the files from the `config` directory to the following path in the build directory:

```bash
orangepi-build/external/config/kernel
```

This step ensures that the necessary kernel configurations are in place for the build process.

### 5. Copying Device Tree Files

Copy the files from the `dts` directory into the following path:

```bash
orangepi-build/kernel/orange-pi-5.10-rk35xx/arch/arm64/boot/dts/rockchip
```

**Important Note**: You can only choose one of the device tree files: either `mvcam` or `cam2m`, not both. After copying, touch the files in the target directory to update their timestamps and ensure compilation.

### 6. Building the Final Image

Finally, navigate to the build directory and execute the build command to compile the final image:

```bash
sudo ./build.sh
```

This command will build the system image based on the updated configuration and source files. Once the build is complete, you will have a system image ready for flashing onto your device.

