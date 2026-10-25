# 🚀 multicore-wakeup (AF-Kernel Lab)

A beginner-friendly, minimal x86_64 bare-metal UEFI learning lab written in pure Rust and Assembly. 

This lab demonstrates how to write a raw UEFI boot application in Rust, set up a custom GDT, render text directly onto the physical framebuffer using the Graphics Output Protocol (GOP), and wake up a secondary CPU core using Local APIC Inter-Processor Interrupts (IPIs) with a low-level assembly trampoline.

---

## 🛠️ What This Code Does

- **UEFI Boot Entry:** Initializes via UEFI boot services and exits boot services to take control of raw hardware.
- **Framebuffer Graphics (GOP):** Locates the UEFI Graphics Output Protocol framebuffer and renders text strings using an 8x8 bitmap font.
- **Secondary Core Activation:** Queries processor info using UEFI MP Services, copies trampoline binary code to memory (`0x8000`), and uses Local APIC registers to wake up Core 1.
- **Custom GDT Setup:** Manually configures a Global Descriptor Table and reloads segment registers using `retfq`.
- **In-Memory RAM Disk:** Allocates a static buffer acting as a RAM disk and copies a mock file header (`Author.txt`) into it.
- **Panic Handler:** Triggers the system PC speaker via PIT channel 2 if a panic occurs.

---

## ⚙️ Requirements

- Nightly Rust toolchain with the UEFI target installed:
    rustup target add x86_64-unknown-uefi

---

## 🔨 Build Instructions

Format your code and compile the optimized release binary for UEFI:
    cargo fmt && cargo build --target x86_64-unknown-uefi --release

---

## 💿 Creating a Bootable ISO (Commands)

To package your compiled EFI binary into a bootable UEFI ISO image (`AetherOS.iso`) for testing in QEMU or a VM, run these commands directly in your terminal:

    # 1. Create a 50MB FAT32 ESP image
    dd if=/dev/zero of=esp.img bs=1M count=50
    mkfs.fat -F 32 -n "AETHEROS" esp.img

    # 2. Mount and copy the compiled kernel binary to EFI boot path
    mkdir -p mnt
    sudo mount esp.img mnt
    sudo mkdir -p mnt/EFI/BOOT
    sudo cp target/x86_64-unknown-uefi/release/multiprocessor_wakeup.efi mnt/EFI/BOOT/BOOTX64.EFI
    sync
    sudo umount mnt
    rmdir mnt

    # 3. Generate the hybrid bootable ISO using xorriso
    xorriso -as mkisofs \
      -R -J \
      -V "AETHEROS" \
      -o AetherOS.iso \
      -partition_offset 16 \
      -append_partition 2 0xef esp.img \
      -e --interval:appended_partition_2:all:: \
      -no-emul-boot \
      -isohybrid-gpt-basdat \
      esp.img

---

## ⚡ Flashing Directly to a USB Drive (Commands)

> ⚠️ **Warning:** Double-check your USB drive partition path (e.g., `/dev/sda1` or `/dev/sdb1`) before running these commands to prevent accidental data loss on your system drives!

To flash the kernel directly onto a FAT32-formatted USB partition:

    # 1. Format partition to FAT32
    sudo mkfs.fat -F 32 /dev/sda1

    # 2. Mount USB partition
    sudo mount /dev/sda1 /mnt

    # 3. Create UEFI boot directory and copy kernel binary
    sudo mkdir -p /mnt/EFI/BOOT
    sudo cp target/x86_64-unknown-uefi/release/multiprocessor_wakeup.efi /mnt/EFI/BOOT/BOOTX64.EFI

    # 4. Sync and unmount
    sync
    sudo umount /mnt

    echo "[+] USB is ready to boot!"

---

## 👨‍💻 Author

- **Arshman Farhan** (AF-Kernel Lab)
