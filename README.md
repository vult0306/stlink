
STM8
Jump to navigationJump to search
 Share 

Contents
1	STM8
2	Code Resources
3	Features
4	Tutorial, Resources
5	Functions
6	ToolChain
6.1	Guide for IAR STM8
6.2	Upload code
6.3	Quick Guide in Linux SDCC
6.3.1	Install
6.3.2	Demo code example
6.3.3	Flash
6.3.4	Alternative official demo code
6.3.5	Error fix: Unlock
7	Use
8	Schematic
8.1	Project
9	Documents
STM8
[SDCC guide], commands
sdcc -mstm8 --std-c99 led.c
Important compiler options for STM8 developers include:

-c to compile into object files to be linked later
--std-c99 for compilation in C99 mode (some C99 features, e.g. variable-length arrays are not yet supported in sdcc though)
--opt-code-size for optimization for code size
--max-allocs-per-node to select the optimization level. the default value is 3000. Higher values result in more optimized code, longer compiler runtime, and higher memory usage during compilation.
Code Resources
SDCC supported, arduino based firmware - Sduino.
STM8MXcube - http://pan.baidu.com/s/1bpFyXxt pass: 3pgm
Features
Type	Core	Speed	Interface	peripheral resources	I/O	Flash	EEPROM	RAM	VCC/VDD	ADC	Crystal	Operational Temperature
STM8S103F3P6	8bit	16mhz	I2C, IrDA, LIN, SPI, UART/USART	undervoltage-check/reset, POR, PWM, WDT	16	8KB（8K x 8）	640 x 8	1K x 8	2.95 V ~ 5.5 V	A/D 5x10b	Internal	-40°C ~ 85°C（TA）
Tutorial, Resources
IAR Tutorial 1
STM8 Standard lirarary from ST
FX STM8
Functions
IO Registers

Data Direction Register – DDR, DDR to 1 for output pin, or 0 for input pin
Control Register 1 – C1
Control Register 2 – C2
Output Data Register – ODR - Reset the pin by setting ODR to 0
Input Data Register – IDR
Register	Mode	Value	Description
CR1	Input	0	Floating input
CR1	Input	1	Input with pull-up
CR1	output	0	Open drain
CR1	output	1	Push-Pull
CR2	Input	0	Interrupt disabled
CR2	Input	1	Interrupt enabled
CR2	output	0	Output up to 2 MHz.
CR2	output	1	Output up to 10 MHz
ToolChain
Guide for IAR STM8
Download standard stm8 library via google search, on st website.
Install st toolset (STVD + programmer ). Current version we provided is sttoolset_pack29 (pass electrodragon0428)
Alternative install ST-link driver
Upload code
Stvp guide.png
Install ST toolset, which include ST visual programmer
Connect your ST-LINK, SWIM, RST, VCC better 3.3V, GND.
select and open hex file, select menu program -> current tap and done.
Quick Guide in Linux SDCC
Install
Install ubuntu on virtualbox, Install sdcc: apt-get install sdcc
update sdcc for stm8, manually remove and install sdcc
sudo apt-get update
sudo apt-get remove sdcc sdcc-libraries
sudo apt-get install sdcc
Install stm8flash
git clone https://github.com/vdudouyt/stm8flash.git
cd stm8flash
make
sudo make install
Demo code example
git clone stm8 blink example:
git clone https://github.com/vdudouyt/sdcc-examples-stm8.git
cd sdcc-examples-stm8
edit code for stm8s103f3:
#define PB_ODR *(unsigned char*)0x5005
#define PB_IDR *(unsigned char*)0x5006
#define PB_DDR *(unsigned char*)0x5007
#define PB_CR1 *(unsigned char*)0x5008
#define PB_CR2 *(unsigned char*)0x5009

int main()
{
    int d;
    // Configure pins
    PB_DDR = 0x20;
    PB_CR1 = 0x20;
    // Loop
    do {
        PB_ODR ^= 0x20;
        for(d = 0; d < 15000; d++) { }
    } while(1);
}
Or use
 
#define PB_ODR *(unsigned char*)0x5005

// Port B data direction register, for setting pins as INPUT or OUTPUT 
#define PB_DDR *(unsigned char*)0x5007

// Port B control register 1, 
#define PB_CR1 *(unsigned char*)0x5008

int main() {
    int d;
    // Configure pins
    PB_DDR = 0x20; // 0x20(00100000) pin 5 set to 1 -> setting it as OUTPUT 
    PB_CR1 = 0x20; // 0x20(00100000) pin 5 set to 1 -> setting it as PUSH-PULL Mode (only when configured as output)
    // Loop
    do {
        PB_ODR ^= 0x20; // 0x20(00100000) pin 5 XOR/toggle between HIGH and LOW
        for(d = 0; d < 29000; d++) {
        }
    } while(1);
}
change the makefile:
SDCC=sdcc
SDLD=sdld
OBJECTS=blinky.ihx

.PHONY: all clean flash

all: $(OBJECTS)

clean:
        rm -f $(OBJECTS)

flash: $(OBJECTS)
        stm8flash -cstlinkv2 -pstm8s103f3 -w $(OBJECTS)

%.ihx: %.c
        $(SDCC) -lstm8 -mstm8 --out-fmt-ihx $(CFLAGS) $(LDFLAGS) $<
run make again and run commands to convert c file to ihx and write flash:
make
Flash
stm8flash -c stlinkv2 -p stm8s103 -w blinky.ihx
or convert first by
sdcc -lstm8 -mstm8 --out-fmt-ihx blinky.c
read flash
stm8flash -c stlinkv2 -p stm8s003?3  -r aa.hex
./stm8flash -c stlink -p stm8s103k3 -w led.ihx
Alternative official demo code
// Source code under CC0 1.0
#include <stdint.h>

#define CLK_DIVR	(*(volatile uint8_t *)0x50c6)
#define CLK_PCKENR1	(*(volatile uint8_t *)0x50c7)

#define TIM1_CR1	(*(volatile uint8_t *)0x5250)
#define TIM1_CNTRH	(*(volatile uint8_t *)0x525e)
#define TIM1_CNTRL	(*(volatile uint8_t *)0x525f)
#define TIM1_PSCRH	(*(volatile uint8_t *)0x5260)
#define TIM1_PSCRL	(*(volatile uint8_t *)0x5261)

#define PB_ODR	(*(volatile uint8_t *)0x5005)
#define PB_DDR	(*(volatile uint8_t *)0x5007)
#define PB_CR1	(*(volatile uint8_t *)0x5008)


unsigned int clock(void)
{
	unsigned char h = TIM1_CNTRH;
	unsigned char l = TIM1_CNTRL;
	return((unsigned int)(h) << 8 | l);
}

void main(void)
{
	CLK_DIVR = 0x00; // Set the frequency to 16 MHz

	// Configure timer
	// 1000 ticks per second
	TIM1_PSCRH = 0x3e;
	TIM1_PSCRL = 0x80;
	// Enable timer
	TIM1_CR1 = 0x01;

	PB_DDR = 0x20;
	PB_CR1 = 0x20;

	for(;;)
		PB_ODR = (clock() % 1000 < 500) << 5;
}
Error fix: Unlock
The issue written in pdf datasheet pg 45.
sudo stm8flash -c stlinkv2 -p "stm8s103f3" -w blinky.ihx
Determine FLASH area
Writing Intel hex file 182 bytes at 0x8000... Tries exceeded
Run command
echo "00 00 ff 00 ff 00 ff 00 ff 00 ff" | xxd -r -p > factory_defaults.bin
stm8flash -c stlinkv2 -p stm8s103f3 -s opt -w factory_defaults.bin
Use
programming via SWIM port, ST link programmer can be found on our store
When power up, LED should flashing, this is programmed for testing purpose
Schematic
STM8S general


STM8S103F3 - current selling .

 

STM8S103F3P6

 

STM8S103F3P6-2

 

STM8S003F3P6

 

STM8S103K3T6

 

STM8S105K-005K

 

STM8S207R

STM8S Application


delay relay control board

STM8L Low power series


STM8L051, LDO use ME6211

 

STM8L152, SCH

Vcap must connect with 1UF cap.
Project

STM8 Delay Relay with LCD

 

STM 8 Clock

Demo code delay relay - File:STM8 delay relay w-lcd.zip
IAR Demo code STM8 Clock - File:LED CLOCK2.0.zip
Documents
STM8S103 Datasheet link from ST website, if not work please try search it
STM8 Family
STM8 and STM32 product and tool selection guide
STM8S003F3 (P6) - Direct IC datasheet from St.com here.
Demo code back up page.

File:STM8S207xx.PDF
Category: STM
Navigation menu
Log inPageDiscussionReadView historySearch
Search ElectroDragon
Electrodragon Home
Electrodragon Store
Wiki Home
Community portal
Current events
Recent changes
Random page
Mediawiki
Help
Tools
What links here
Related changes
Special pages
Printable version
Permanent link
Page information
Share
This page was last edited on 24 December 2018, at 03:44.
Privacy policyAbout ElectroDragonDisclaimersPowered by MediaWiki



Open source version of the STMicroelectronics Stlink Tools
==========================================================

[![GitHub release](https://img.shields.io/github/release/texane/stlink.svg)](https://github.com/texane/stlink/releases/latest)
[![BSD licensed](https://img.shields.io/badge/license-BSD-blue.svg)](https://raw.githubusercontent.com/hyperium/hyper/master/LICENSE)
[![GitHub commits](https://img.shields.io/github/commits-since/texane/stlink/1.5.1.svg)](https://github.com/texane/stlink/compare/1.5.1...master)
[![Downloads](https://img.shields.io/github/downloads/texane/stlink/total.svg)](https://github.com/texane/stlink/releases)
[![Linux Status](https://img.shields.io/travis/texane/stlink/master.svg?label=linux)](https://travis-ci.org/texane/stlink)
[![Build Status](https://jenkins.ncrmnt.org/buildStatus/icon?job=GithubCI/stlink)](https://jenkins.ncrmnt.org/job/GithubCI/job/stlink/)

## HOWTO

First, you have to know there are several boards supported by the software.
Those boards use a chip to translate from USB to JTAG commands. The chip is
called stlink and there are two versions:

* STLINKv1, present on STM32VL discovery kits,
* STLINKv2, present on STM32L discovery and later kits.

Two different transport layers are used:

* STLINKv1 uses SCSI passthru commands over USB
* STLINKv2 and STLINKv2-1 (seen on nucleo boards) uses raw USB commands.

## Installation

Windows users can [download v1.3.0](https://github.com/texane/stlink/releases/tag/1.3.0) from the releases page.

Mac OS X users can install from [homebrew](http://brewformulas.org/Stlink) or [download v1.3.0](https://github.com/texane/stlink/releases/tag/1.3.0) from the releases page.

For Debian Linux based distributions there is no package available
 in the standard repositories so you need to install [from source](doc/compiling.md) yourself.

Arch Linux users can install from the [repository](https://www.archlinux.org/packages/community/x86_64/stlink)

Alpine Linux users can install from the [repository](https://pkgs.alpinelinux.org/packages?name=stlink)

Fedora users can install from [repository](https://src.fedoraproject.org/rpms/stlink)

RedHat/CentOS 7 users can install from EPEL [repository](https://src.fedoraproject.org/rpms/stlink/branch/epel7)

Gentoo Linux users can install from the official portage [repository](https://packages.gentoo.org/packages/dev-embedded/stlink)

FreeBSD users can install from [freshports](https://www.freshports.org/devel/stlink)

OpenBSD users need to install [from source](doc/compiling.md).

## Installation from source (advanced users)

When there is no executable available for your platform or you need the latest
 (possible unstable) version you need to compile yourself. This is explained in
 the [compiling manual](doc/compiling.md).

## Using the gdb server

To run the gdb server:

```
$ make && [sudo] ./st-util

There are a few options:

./st-util - usage:

  -h, --help		Print this help
  -vXX, --verbose=XX	Specify a specific verbosity level (0..99)
  -v, --verbose		Specify generally verbose logging
  -s X, --stlink_version=X
			Choose what version of stlink to use, (defaults to 2)
  -1, --stlinkv1	Force stlink version 1
  -p 4242, --listen_port=1234
			Set the gdb server listen port. (default port: 4242)
  -m, --multi
			Set gdb server to extended mode.
			st-util will continue listening for connections after disconnect.
  -n, --no-reset
			Do not reset board on connection.
```

The STLINKv2 device to use can be specified in the environment
variable `STLINK_DEVICE` in the format `<USB_BUS>:<USB_ADDR>`.

Then, in your project directory, someting like this...
(remember, you need to run an _ARM_ gdb, not an x86 gdb)

```
$ arm-none-eabi-gdb fancyblink.elf
...
(gdb) tar extended-remote :4242
...
(gdb) load
Loading section .text, size 0x458 lma 0x8000000
Loading section .data, size 0x8 lma 0x8000458
Start address 0x80001c1, load size 1120
Transfer rate: 1 KB/sec, 560 bytes/write.
(gdb)
...
(gdb) continue
```

## Resetting the chip from GDB

You may reset the chip using GDB if you want. You'll need to use `target
extended-remote' command like in this session:

```
(gdb) target extended-remote localhost:4242
Remote debugging using localhost:4242
0x080007a8 in _startup ()
(gdb) kill
Kill the program being debugged? (y or n) y
(gdb) run
Starting program: /home/whitequark/ST/apps/bally/firmware.elf
```

Remember that you can shorten the commands. `tar ext :4242` is good enough
for GDB.

If you need to send a hard reset signal through `NRST` pin, you can use the following command:

```
(gdb) monitor jtag_reset
```


## Running programs from SRAM

You can run your firmware directly from SRAM if you want to. Just link
it at 0x20000000 and do

```
(gdb) load firmware.elf
```

It will be loaded, and pc will be adjusted to point to start of the
code, if it is linked correctly (i.e. ELF has correct entry point).

## Writing to flash

The GDB stub ships with a correct memory map, including the flash area.
If you would link your executable to `0x08000000` and then do

```
(gdb) load firmware.elf
```

then it would be written to the memory.

## Writing Option Bytes

Example to read and write option bytes (currently writing only supported for STM32G0)
```
./st-flash --debug --reset --format binary --flash=128k read option_bytes_dump.bin 0x1FFF7800 4
./st-flash --debug --reset --format binary --flash=128k write option_bytes_dump.bin 0x1FFF7800
```

## FAQ

Q: My breakpoints do not work at all or only work once.

A: Optimizations can cause severe instruction reordering. For example,
if you are doing something like `REG = 0x100;' in a loop, the code may
be split into two parts: loading 0x100 into some intermediate register
and moving that value to REG. When you set up a breakpoint, GDB will
hook to the first instruction, which may be called only once if there are
enough unused registers. In my experience, -O3 causes that frequently.

Q: At some point I use GDB command `next', and it hangs.

A: Sometimes when you will try to use GDB `next` command to skip a loop,
it will use a rather inefficient single-stepping way of doing that.
Set up a breakpoint manually in that case and do `continue`.

Q: Load command does not work in GDB.

A: Some people report XML/EXPAT is not enabled by default when compiling
GDB. Memory map parsing thus fail. Use --enable-expat.

## Currently known working combinations of programmer and target

See [doc/tested-boards.md](doc/tested-boards.md)

## Known missing features

Some features are missing from the `texane/stlink` project and we would like you to
 help us out if you want to get involved:

* Control programming speed (See [#462](https://github.com/texane/stlink/issues/462))
* OTP area programming (See [#202](https://github.com/texane/stlink/issues/202))
* EEPROM area programming (See [#318](https://github.com/texane/stlink/issues/218))
* Protection bits area reading (See [#346](https://github.com/texane/stlink/issues/346))
* MCU hotplug (See [#449](https://github.com/texane/stlink/issues/449))
* Writing options bytes (region) (See [#458](https://github.com/texane/stlink/issues/458))
* Instrumentation Trace Macro (ITM) Cell (See [#136](https://github.com/texane/stlink/issues/136))
* Writing external memory connected to an STM32 controller (e.g Quad SPI NOR flash) (See [#412](https://github.com/texane/stlink/issues/412))

## Known bugs

### Sometimes flashing only works after a mass erase

There is seen a problem sometimes where a flash loader run error occurs and is resolved after mass-erase
of the flash:

```
2015-12-09T22:01:57 INFO src/stlink-common.c: Successfully loaded flash loader in sram
2015-12-09T22:02:18 ERROR src/stlink-common.c: flash loader run error
2015-12-09T22:02:18 ERROR src/stlink-common.c: run_flash_loader(0x8000000) failed! == -1
```

Issue related to this bug: [#356](https://github.com/texane/stlink/issues/356)

### Flash size is detected as zero bytes size

It is possible that the STM32 flash is write protected, the st-flash tool will show something like this:

```
st-flash write prog.bin 0x8000000
2017-01-24T18:44:14 INFO src/stlink-common.c: Loading device parameters....
2017-01-24T18:44:14 INFO src/stlink-common.c: Device connected is: F1 High-density device, id 0x10036414
2017-01-24T18:44:14 INFO src/stlink-common.c: SRAM size: 0x10000 bytes (64 KiB), Flash: 0 bytes (0 KiB) in pages of 2048 bytes
```

As you can see, it gives out something unexpected like
```
Flash: 0 bytes (0 KiB) in pages of 2048 bytes
```

```
st-info --probe
Found 1 stlink programmers
 serial: 303030303030303030303031
openocd: "\x30\x30\x30\x30\x30\x30\x30\x30\x30\x30\x30\x31"
  flash: 0 (pagesize: 2048)
   sram: 65536
 chipid: 0x0414
  descr: F1 High-density device
```

Try to remove the write protection (probably only possible with ST Link Utility from ST itself).

Issue related to this bug: [#545](https://github.com/texane/stlink/issues/545)

## Contributing and versioning

* The semantic versioning scheme is used. Read more at [semver.org](http://semver.org)
* When creating a pull request, please open first a issue for discussion of new features. Bugfixes don't need a discussion.

## License

The stlink library and tools are licensed under the [BSD license](LICENSE).
The flashloaders/stm32l0x.s and flashloaders/stm32lx.s source files are licensed under the GPL-2+.
