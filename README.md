#  Yocto Block Device Driver Project

[![Yocto Project](https://img.shields.io/badge/Yocto-Project-blue.svg)](https://www.yoctoproject.org/)
[![Linux Kernel](https://img.shields.io/badge/Linux-Kernel%20Driver-orange.svg)](https://www.kernel.org/)
[![Block Device](https://img.shields.io/badge/Block%20Device-Driver-green.svg)](https://en.wikipedia.org/wiki/Device_driver)
[![Embedded Linux](https://img.shields.io/badge/Embedded-Linux-red.svg)](https://en.wikipedia.org/wiki/Embedded_Linux)
[![BitBake](https://img.shields.io/badge/Build-BitBake-yellow.svg)](https://github.com/openembedded/bitbake)
[![RAM Disk](https://img.shields.io/badge/RAM-Disk%20Driver-purple.svg)](https://en.wikipedia.org/wiki/RAM_drive)
[![Driver Development](https://img.shields.io/badge/Driver-Development-8b0000.svg)](https://en.wikipedia.org/wiki/Device_driver_development)
[![Kernel Module](https://img.shields.io/badge/Kernel-Module-0066cc.svg)](https://en.wikipedia.org/wiki/Loadable_kernel_module)
[![Makefile](https://img.shields.io/badge/Build-Makefile-lightgrey.svg)](https://en.wikipedia.org/wiki/Makefile)
[![Educational](https://img.shields.io/badge/Educational-Portfolio%20Project-ff69b4.svg)](https://en.wikipedia.org/wiki/Educational_software)
[![C Programming](https://img.shields.io/badge/C-Embedded%20Programming-009688.svg)](https://en.wikipedia.org/wiki/C_(programming_language))
[![IOCTL](https://img.shields.io/badge/IOCTL-Interface-blueviolet.svg)](https://en.wikipedia.org/wiki/Ioctl)
[![Character Device](https://img.shields.io/badge/Character%20Device-Driver-32CD32.svg)](https://en.wikipedia.org/wiki/Device_file)
[![Prototyping](https://img.shields.io/badge/Prototyping-Environment-FF8C00.svg)](https://en.wikipedia.org/wiki/Software_prototyping)
[![GitHub Portfolio](https://img.shields.io/badge/GitHub-Portfolio%20Ready-181717.svg)](https://github.com)
[![Cross-Platform](https://img.shields.io/badge/Cross--Platform-Linux%20Kernel-333333.svg)](https://en.wikipedia.org/wiki/Cross-platform)
[![Open Source](https://img.shields.io/badge/Open%20Source-Hardware/Software-brightgreen.svg)](https://opensource.org/)
[![System Programming](https://img.shields.io/badge/System-Programming-4B0082.svg)](https://en.wikipedia.org/wiki/System_programming)
[![Low-Level](https://img.shields.io/badge/Low--Level-Programming-800000.svg)](https://en.wikipedia.org/wiki/Low-level_programming_language)
[![Virtual Device](https://img.shields.io/badge/Virtual-Device%20Driver-2E8B57.svg)](https://en.wikipedia.org/wiki/Virtual_device)
[![Build Automation](https://img.shields.io/badge/Build-Automation-483D8B.svg)](https://en.wikipedia.org/wiki/Build_automation)

This project provides a **Yocto-like environment** for learning and demonstrating how a **block device driver** works, without requiring a full Yocto build.

It includes:

* A **RAM-backed block device driver** (`myblock.c`)
* A **user-space test application** (`user_app.c`)
* A **Makefile** to build both driver and app
* A **BitBake-style recipe** (`.bb`)
* A **dummy BitBake script** to simulate Yocto builds

This project is ideal for **prototyping, and GitHub portfolio demonstrations**.

---

#  Project Structure

```
yocto-block/
│── README.md
│── Makefile
│── scripts/
│   └── bitbake.sh        # Yocto build script
│
├── meta-myblock/
│   ├── conf/
│   │   └── layer.conf
│   └── recipes-myblock/
│       └── block-sample/
│           ├── block-sample_1.0.bb
│           ├── Makefile
│           ├── myblock.c
│           └── user_app.c
│
└── build/                # Auto-generated build artifacts
```

---

#  How the yocto Build Works

1. Run the BitBake script:

```bash
sh scripts/bitbake.sh block-sample
```

2. The script:

* Copies sources into a  `build/` folder
* Executes the **Makefile**
* Builds the kernel module (`myblock.ko`)
* Builds the user-space app (`user_app`)

3. Artifacts are generated in:

```
build/block-sample/
```

---

#  Block Device Driver — Registration & Probe

### 1️ Registration in Linux

The driver is registered in `myblock_init()`:

```c
dev.gd = alloc_disk(1);  // Allocate gendisk
dev.gd->major = register_blkdev(0,"myblock");  // Register major dynamically
dev.gd->queue = dev.queue;               // Assign request queue
strcpy(dev.gd->disk_name, "myblock");   // Set device name
set_capacity(dev.gd, NUM_SECTORS);      // Set device size
add_disk(dev.gd);                        // Add disk → /dev/myblock appears
```

* `register_blkdev()` → allocates major number
* `alloc_disk()` → allocates device structure
* `add_disk()` → makes the device accessible in `/dev`

### 2️ Probe Function Concept

* Typical platform drivers use a **probe function** called by the kernel when a matching device is found.
* In this RAM-backed driver, there is **no physical hardware**, so `module_init(myblock_init)` acts as the **probe**.
* During probe/init, the driver allocates memory, sets up queues, and registers the device.

---

#  Block Device Working Flow

The following diagram shows how the block device driver interacts with the system:

![Block Device Driver Flow](https://media.geeksforgeeks.org/wp-content/uploads/20200603084935/driver-21.png)


#        Working Flow (Block Diagram )

![driver overview](https://github.com/user-attachments/assets/3100bdaa-b897-4f09-ad18-7bc8bce9650a)





### 1️ Initialization

```
module_init(myblock_init)
        │
        ├── alloc_disk()
        ├── register_blkdev()
        ├── blk_init_queue()
        └── add_disk() → /dev/myblock appears
```

### 2️ User-Space Interaction

**Write Flow:**

```
User App write()
      │
      ▼
Block Layer → Request Queue → myblock_request()
      │
      ▼
Data stored in internal RAM buffer
```

**Read Flow:**

```
User App read()
      │
      ▼
Block Layer → Request Queue → myblock_request()
      │
      ▼
Data copied from RAM buffer → User App
```

### 3️ Cleanup

```
module_exit(myblock_exit)
        │
        ├── del_gendisk()
        ├── unregister_blkdev()
        └── blk_cleanup_queue()
```

---

#  Example Usage

### 1. Build Driver and App

```bash
make
```

### 2. Load Driver

```bash
sudo insmod myblock.ko
ls /dev/myblock   # Check device exists
```

### 3. Write to Device

```bash
echo "Hello Block Device" > /dev/myblock
```

### 4. Read from Device

```bash
cat /dev/myblock
# Output: Hello Block Device
```

### 5. Use User-Space App

```bash
./user_app
# Example Output:
# User App: Writing "Hello"
# User App: Read back "Hello"
```

### 6. Unload Driver

```bash
sudo rmmod myblock
```

---




