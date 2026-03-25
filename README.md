# FalQon-OS
A PQC privacy oriented OS for Developers

# CPU architecture
Instruction set (x86-64, ARM64, RISC-V?)
Your boot assembly, calling convention, and register names depend entirely on this.
uname -m

# CPU
CPU vendor & microarchitecture
Intel vs AMD changes MSR addresses, MTRR behaviour, and spectre mitigations needed.
cat /proc/cpuinfo | grep -m1 "model name"

# CPU
Core count & hyperthreading
Needed to set up the APIC table, SMP bootstrap, and scheduler affinity masks.
lscpu | grep -E "^CPU\(s\)|Thread"

# CPU
CPU feature flags (SSE, AVX, RDRAND…)
RDRAND is required for hardware entropy; AVX-512 unlocks faster PQ crypto routines.
grep flags /proc/cpuinfo | head -1

# CPU
Memory
Total RAM & layout (NUMA nodes?)
Your physical memory allocator bitmap must match exactly what the bootloader reports.
dmidecode -t memory | grep -E "Size|Locator"

# Memory
Page size & paging levels (4-level vs 5-level)
5-level paging (LA57) changes your virtual address space from 256 TiB to 128 PiB.
grep -m1 la57 /proc/cpuinfo

# Memory
Firmware & boot
BIOS vs UEFI
UEFI needs a GPT disk + EFI partition; legacy BIOS uses MBR + INT 0x10. Completely different boot paths.
[ -d /sys/firmware/efi ] && echo UEFI || echo BIOS

# Firmware
Secure Boot state
If enabled, your kernel image must be signed with a key enrolled in the firmware DB.
mokutil --sb-state

# Firmware
ACPI tables presence
ACPI provides the memory map, CPU topology, interrupt routing — your kernel parses these at boot.
ls /sys/firmware/acpi/tables/

# Firmware
Devices & I/O
PCI device list (NIC, storage, GPU…)
Each device needs a driver. Knowing the vendor:device IDs lets you look up the datasheet.
lspci -nn

# Devices
Storage controller type (AHCI, NVMe, virtio?)
Completely different register interfaces. virtio-blk is the easiest target for QEMU development.
lspci | grep -i "storage\|SATA\|NVMe"
Devices
Security features
TPM presence & version
TPM 2.0 enables measured boot and remote attestation — key for a quantum-resistant trust chain.
cat /sys/class/tpm/tpm0/tpm_version_major 2>/dev/null || echo none 
