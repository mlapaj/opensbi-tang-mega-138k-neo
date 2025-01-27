# Intro
This is port of OpenSBI. Original readme can be found: [here](README_orig.md)

I have added support for Andes A25 core. I was basing on atomic examples from Gowin and U-Boot sources for other Andes CPUS.

**WARNING: I'm not taking any responsibility if this code is working not correctly. Use at your own risk.**

Following problems identified:
 - A25 core has no double trap feature (or at least i do not know how to enable it) - i had to disable CLEAR_MDT macro.
 - A25 core has problems with atomic operations. There is a workaround in gcc: `-mno-b19758`, however, this did not worked correctly,
   I have found that reading memory before doing amoswp is fixing issue. I'm not sure if this fix is proper. I have no time to deeply analyze it.
 - I had to adopt device tree. I have used modified DT from U-Boot

# How to use

You need to get my U-Boot repo and generate device tree for Aneng RISC Core
In my case I had to disable semihosting feature - my JLink debugger is not supporting semihosting

Since I`m not using normal boot procedure and this project is right now in heavy development to run OpenSBI you need to use following script 
executed in  mega138k/project2_ae350_risc_gpio/sw_atomic_raw of [vhdl_playground repo](https://github.com/mlapaj/vhdl_playground/tree/main/mega138k/project2_ae350_risc_gpio/sw_atomic_ddr) :

```
set logging file mylog.txt
set logging on
set pagination off
set disassemble-next-line on
target extended-remote :2331

file hello_world
monitor reset
restore hello_world
set $pc = _start
b hello_world.c:59
c
 x /1xb 0x40000
 set $i = 10
while $i <= 17
    eval "set $x%d = 0", $i
    set $i = $i + 1
end
set $pc=0

file fw_jump.elf
restore fw_jump.elf
restore u-boot.dtb binary 0x200000
#hartId is in a0
#dtb addres in a1
set $a1=0x200000
b generic_early_init
stepi
```


If everything is configured correctly, you should get following message on your UART:
```
OpenSBI v1.6
   ____                    _____ ____ _____
  / __ \                  / ____|  _ \_   _|
 | |  | |_ __   ___ _ __ | (___ | |_) || |
 | |  | | '_ \ / _ \ '_ \ \___ \|  _ < | |
 | |__| | |_) |  __/ | | |____) | |_) || |_
  \____/| .__/ \___|_| |_|_____/|____/_____|
        | |
        |_|

Platform Name               : andestech,a25
Platform Features           : medeleg
Platform HART Count         : 1
Platform IPI Device         : andes_plicsw
Platform Timer Device       : andes_plmt @ 800000000Hz
Platform Console Device     : uart8250
Platform HSM Device         : ---
Platform PMU Device         : andes_pmu
Platform Reboot Device      : ---
Platform Shutdown Device    : ---
Platform Suspend Device     : ---
Platform CPPC Device        : ---
Firmware Base               : 0x0
Firmware Size               : 321 KB
Firmware RW Offset          : 0x40000
Firmware RW Size            : 65 KB
Firmware Heap Offset        : 0x47000
Firmware Heap Size          : 37 KB (total), 2 KB (reserved), 10 KB (used), 24 KB (free)
Firmware Scratch Size       : 4096 B (total), 252 B (used), 3844 B (free)
Runtime SBI Version         : 2.0
Standard SBI Extensions     : time,rfnc,ipi,base,hsm,pmu,dbcn,legacy,vendor
Experimental SBI Extensions : fwft,dbtr,sse

Domain0 Name                : root
Domain0 Boot HART           : 0
Domain0 HARTs               : 0*
Domain0 Region00            : 0xf0300000-0xf0300fff M: (I,R,W) S/U: (R,W)
Domain0 Region01            : 0x00040000-0x0005ffff M: (R,W) S/U: ()
Domain0 Region02            : 0x00000000-0x0003ffff M: (R,X) S/U: ()
Domain0 Region03            : 0xe6000000-0xe60fffff M: (I,R,W) S/U: ()
Domain0 Region04            : 0xe6400000-0xe67fffff M: (I,R,W) S/U: ()
Domain0 Region05            : 0xe4000000-0xe5ffffff M: (I,R,W) S/U: (R,W)
Domain0 Region06            : 0x00000000-0xffffffff M: () S/U: (R,W,X)
Domain0 Next Address        : 0x00400000
Domain0 Next Arg1           : 0x02200000
Domain0 Next Mode           : S-mode
Domain0 SysReset            : yes
Domain0 SysSuspend          : yes

Boot HART ID                : 0
Boot HART Domain            : root
Boot HART Priv Version      : v1.11
Boot HART Base ISA          : rv32imafdcnx
Boot HART ISA Extensions    : zihpm,xandespmu,sdtrig
Boot HART PMP Count         : 16
Boot HART PMP Granularity   : 2 bits
Boot HART PMP Address Bits  : 31
Boot HART MHPM Info         : 4 (0x00000078)
Boot HART Debug Triggers    : 8 triggers
Boot HART MIDELEG           : 0x00000222
Boot HART MEDELEG           : 0x0000b109

```
