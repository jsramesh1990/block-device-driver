
#  Yocto Block Device Driver Project

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




