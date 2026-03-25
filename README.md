# FalQon-OS
A PQC privacy oriented OS for Developers

## CPU architecture
Instruction set (x86-64, ARM64, RISC-V?)
Your boot assembly, calling convention, and register names depend entirely on this.
uname -m

###### x86_64

#### CPU
CPU vendor & microarchitecture
Intel vs AMD changes MSR addresses, MTRR behaviour, and spectre mitigations needed.
cat /proc/cpuinfo | grep -m1 "model name"

###### 13th Gen Intel(R) Core(TM) i7-13700H
#### CPU
Core count & hyperthreading
Needed to set up the APIC table, SMP bootstrap, and scheduler affinity masks.
lscpu | grep -E "^CPU\(s\)|Thread"
       
     CPU(s):                                  20
     Thread(s) per core:                      2
     CPU(s) scaling MHz:                      17%

#### CPU
CPU feature flags (SSE, AVX, RDRAND…)
RDRAND is required for hardware entropy; AVX-512 unlocks faster PQ crypto routines.
grep flags /proc/cpuinfo | head -1
    
      fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc art arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf tsc_known_freq pni pclmulqdq dtes64 monitor ds_cpl vmx smx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch cpuid_fault epb ssbd ibrs ibpb stibp ibrs_enhanced tpr_shadow flexpriority ept vpid ept_ad fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid rdseed adx smap clflushopt clwb intel_pt sha_ni xsaveopt xsavec xgetbv1 xsaves split_lock_detect user_shstk avx_vnni dtherm ida arat pln pts hwp hwp_notify hwp_act_window hwp_epp hwp_pkg_req hfi vnmi umip pku ospke waitpkg gfni vaes vpclmulqdq rdpid movdiri movdir64b fsrm md_clear serialize arch_lbr ibt flush_l1d arch_capabilities

#### CPU
Memory
Total RAM & layout (NUMA nodes?)
Your physical memory allocator bitmap must match exactly what the bootloader reports.
dmidecode -t memory | grep -E "Size|Locator"
        
      Size: 16 GB
      Locator: DIMM A
      Bank Locator: BANK 0
      Non-Volatile Size: None
      Volatile Size: 16 GB
      Cache Size: None
      Logical Size: None
      Size: 16 GB
      Locator: DIMM B
      Bank Locator: BANK 0
      Non-Volatile Size: None
      Volatile Size: 16 GB
      Cache Size: None
      Logical Size: None

## Memory
Page size & paging levels (4-level vs 5-level)
5-level paging (LA57) changes your virtual address space from 256 TiB to 128 PiB.
grep -m1 la57 /proc/cpuinfo
 ###### None 
#### Memory
Firmware & boot
BIOS vs UEFI
UEFI needs a GPT disk + EFI partition; legacy BIOS uses MBR + INT 0x10. Completely different boot paths.
[ -d /sys/firmware/efi ] && echo UEFI || echo BIOS

 ###### UEFI

## Firmware
Secure Boot state
If enabled, your kernel image must be signed with a key enrolled in the firmware DB.
mokutil --sb-state
###### SecureBoot disabled
#### Firmware
ACPI tables presence
ACPI provides the memory map, CPU topology, interrupt routing — your kernel parses these at boot.
ls /sys/firmware/acpi/tables/
        
        APIC  BOOT  DBG2  DMAR  dynamic  FACS  HPET  MCFG  NHLT  SSDT1   SSDT11  SSDT13  SSDT15  SSDT2  SSDT4  SSDT6  SSDT8  TPM2   UEFI2
        BGRT  data  DBGP  DSDT  FACP     FPDT  LPIT  MSDM  PHAT  SSDT10  SSDT12  SSDT14  SSDT16  SSDT3  SSDT5  SSDT7  SSDT9  UEFI1  WSMT

#### Firmware
Devices & I/O
PCI device list (NIC, storage, GPU…)
Each device needs a driver. Knowing the vendor:device IDs lets you look up the datasheet.
lspci -nn

              0000:00:00.0 Host bridge [0600]: Intel Corporation Raptor Lake-P 6p+8e cores Host Bridge/DRAM Controller [8086:a706]
              0000:00:01.0 PCI bridge [0604]: Intel Corporation Device [8086:a70d]
              0000:00:02.0 VGA compatible controller [0300]: Intel Corporation Raptor Lake-P [Iris Xe Graphics] [8086:a7a0] (rev 04)
              0000:00:04.0 Signal processing controller [1180]: Intel Corporation Raptor Lake Dynamic Platform and Thermal Framework Processor Participant [8086:a71d]
              0000:00:06.0 System peripheral [0880]: Intel Corporation RST VMD Managed Controller [8086:09ab]
              0000:00:07.0 PCI bridge [0604]: Intel Corporation Raptor Lake-P Thunderbolt 4 PCI Express Root Port #0 [8086:a76e]
              0000:00:07.1 PCI bridge [0604]: Intel Corporation Device [8086:a73f]
              0000:00:08.0 System peripheral [0880]: Intel Corporation GNA Scoring Accelerator module [8086:a74f]
              0000:00:0a.0 Signal processing controller [1180]: Intel Corporation Raptor Lake Crashlog and Telemetry [8086:a77d] (rev 01)
              0000:00:0d.0 USB controller [0c03]: Intel Corporation Raptor Lake-P Thunderbolt 4 USB Controller [8086:a71e]
              0000:00:0d.2 USB controller [0c03]: Intel Corporation Raptor Lake-P Thunderbolt 4 NHI #0 [8086:a73e]
              0000:00:0e.0 RAID bus controller [0104]: Intel Corporation Volume Management Device NVMe RAID Controller Intel Corporation [8086:a77f]
              0000:00:12.0 Serial controller [0700]: Intel Corporation Alder Lake-P Integrated Sensor Hub [8086:51fc] (rev 01)
              0000:00:12.6 Serial bus controller [0c80]: Intel Corporation Device [8086:51fb] (rev 01)
              0000:00:14.0 USB controller [0c03]: Intel Corporation Alder Lake PCH USB 3.2 xHCI Host Controller [8086:51ed] (rev 01)
              0000:00:14.2 RAM memory [0500]: Intel Corporation Alder Lake PCH Shared SRAM [8086:51ef] (rev 01)
              0000:00:14.3 Network controller [0280]: Intel Corporation Raptor Lake PCH CNVi WiFi [8086:51f1] (rev 01)
              0000:00:15.0 Serial bus controller [0c80]: Intel Corporation Alder Lake PCH Serial IO I2C Controller #0 [8086:51e8] (rev 01)
              0000:00:15.1 Serial bus controller [0c80]: Intel Corporation Alder Lake PCH Serial IO I2C Controller #1 [8086:51e9] (rev 01)
              0000:00:16.0 Communication controller [0780]: Intel Corporation Alder Lake PCH HECI Controller [8086:51e0] (rev 01)
              0000:00:1c.0 PCI bridge [0604]: Intel Corporation Alder Lake-P PCH PCIe Root Port #4 [8086:51bb] (rev 01)
              0000:00:1f.0 ISA bridge [0601]: Intel Corporation Raptor Lake LPC/eSPI Controller [8086:519d] (rev 01)
              0000:00:1f.3 Multimedia audio controller [0401]: Intel Corporation Raptor Lake-P/U/H cAVS [8086:51ca] (rev 01)
              0000:00:1f.4 SMBus [0c05]: Intel Corporation Alder Lake PCH-P SMBus Host Controller [8086:51a3] (rev 01)
              0000:00:1f.5 Serial bus controller [0c80]: Intel Corporation Alder Lake-P PCH SPI Controller [8086:51a4] (rev 01)
              0000:01:00.0 PCI bridge [0604]: Intel Corporation Device [8086:4fa1] (rev 01)
              0000:02:01.0 PCI bridge [0604]: Intel Corporation Device [8086:4fa4]
              0000:03:00.0 Display controller [0380]: Intel Corporation DG2 [Arc A370M] [8086:5693] (rev 05)
              0000:a6:00.0 Unassigned class [ff00]: Realtek Semiconductor Co., Ltd. RTS5260 PCI Express Card Reader [10ec:5260] (rev 01)
              10000:e0:06.0 PCI bridge [0604]: Intel Corporation Raptor Lake PCIe 4.0 Graphics Port [8086:a74d]
              10000:e1:00.0 Non-Volatile memory controller [0108]: SK hynix Platinum P41/PC801 NVMe Solid State Drive [1c5c:1959]

## Devices
Storage controller type (AHCI, NVMe, virtio?)
Completely different register interfaces. virtio-blk is the easiest target for QEMU development.
lspci | grep -i "storage\|SATA\|NVMe"

              0000:00:0e.0 RAID bus controller: Intel Corporation Volume Management Device NVMe RAID Controller Intel Corporation
              10000:e1:00.0 Non-Volatile memory controller: SK hynix Platinum P41/PC801 NVMe Solid State Drive
#### Devices
Security features
TPM presence & version
TPM 2.0 enables measured boot and remote attestation — key for a quantum-resistant trust chain.
cat /sys/class/tpm/tpm0/tpm_version_major 2>/dev/null || echo none 
##### 2
