# openFPGALoader
Universal utility for programming FPGA

__Current support kits:__

* Trenz cyc1000 Cyclone 10 LP 10CL025 (memory and spi flash)
* [Colorlight 5A-75B (version 7)](https://fr.aliexpress.com/item/32281130824.html)
* Digilent arty Artix xc7a35ti (memory and spi flash)
* Lattice MachXO3LF Starter Kit LCMX03LF-6900C (memory and flash)
* [Lattice ECP5 5G Evaluation Board (LFE5UM5G-85F-EVN)](https://www.latticesemi.com/en/Products/DevelopmentBoardsAndKits/ECP5EvaluationBoard)
* [Trenz Gowin LittleBee (TEC0117)](https://shop.trenz-electronic.de/en/TEC0117-01-FPGA-Module-with-GOWIN-LittleBee-and-8-MByte-internal-SDRAM)
* [SeeedStudio Spartan Edge Accelerator Board](http://wiki.seeedstudio.com/Spartan-Edge-Accelerator-Board) (memory)
* [Sipeed Tang Nano](https://tangnano.sipeed.com/en/) (memory)

__Supported (tested) FPGA:__

* Gowin [GW1N (GW1N-1, GW1N-4, GW1NR-9)](https://www.gowinsemi.com/en/product/detail/2/) (SRAM and Flash (flash mode only tested with GW1NR-9))
* Lattice [MachXO3LF](http://www.latticesemi.com/en/Products/FPGAandCPLD/MachXO3.aspx) (SRAM and Flash)
* Lattice [ECP5 (25F, 5G 85F](http://www.latticesemi.com/Products/FPGAandCPLD/ECP5) (SRAM)
* Xilinx Artix 7 [xc7a35ti, xc7a100t](https://www.xilinx.com/products/silicon-devices/fpga/artix-7.html) (memory (all) and spi flash (xc7a35ti)
* Xilinx Spartan 7 [xc7s15](https://www.xilinx.com/products/silicon-devices/fpga/spartan-7.html) (memory)
* Intel Cyclone 10 LP [10CL025](https://www.intel.com/content/www/us/en/products/programmable/fpga/cyclone-10.html)

__Supported cables:__

* [digilent_hs2](https://store.digilentinc.com/jtag-hs2-programming-cable/): jtag programmer cable from digilent
* JTAG-HS3: jtag programmer cable from digilent
* FT2232: generic programmer cable based on Ftdi FT2232
* Tang Nano USB-JTAG interface: FT2232C clone based on CH552 microcontroler
  (with some limitations and workaround)

## compile and install

This application uses **libftdi1**, so this library must be installed (and,
depending of the distribution, headers too)
```bash
apt-get install libftdi1-2 libftdi1-dev libudev-dev cmake
```
**libudev-dev** is optional, may be replaced by **eudev-dev** or just not installed.

By default, **(e)udev** support is enabled (used to open a device by his */dev/xx*
node). If you don't want this option, use:

```-DENABLE_UDEV=OFF```

For distributions using non-glibc (musl, uClibc) **argp-standalone** must be
installed.

And if not already done, install **pkg-config**, **make** and **g++**.

To build the app:
```bash
$ mkdir build
$ cd build
$ cmake ../ # add -DBUILD_STATIC=ON to build a static version
            # add -DENABLE_UDEV=OFF to disable udev support and -d /dev/xxx
$ cmake --build .
or
$ make -j$(nproc)
```
To install
```bash
$ sudo make install
```
The default install path is `/usr/local`, to change it, use
`-DCMAKE_INSTALL_PREFIX=myInstallDir` in cmake invokation.

## Usage

```bash
openFPGALoader --help
Usage: openFPGALoader [OPTION...] BIT_FILE
openFPGALoader -- a program to flash cyclone10 LP FPGA

  -b, --board=BOARD          board name, may be used instead of cable
  -c, --cable=CABLE          jtag interface
  -d, --device=DEVICE        device to use (/dev/ttyUSBx)
      --detect               detect FPGA
  -f, --write-flash          write bitstream in flash (default: false, only for
                             Gowin devices)
      --list-boards          list all supported boards
      --list-cables          list all supported cables
      --list-fpga            list all supported FPGA
  -m, --write-sram           write bitstream in SRAM (default: true, only for
                             Gowin devices)
  -o, --offset=OFFSET        start offset in EEPROM
  -r, --reset                reset FPGA after operations
  -v, --verbose              Produce verbose output
  -?, --help                 Give this help list
      --usage                Give a short usage message
  -V, --version              Print program version

```
To have complete help

### Generic usage

#### display FPGA

With board name:
```bash
openFPGALoader -b theBoard
```
(see `openFPGALoader --list-boards`)

With cable:
```bash
openFPGALoader -c theCable
```
(see `openFPGALoader --list-cables`)

With device node:
```bash
openFPGALoader -d /dev/ttyUSBX
```

**Note:** for some cable (like *digilent* adapters) signals from the converter
are not just directly to the FPGA. For this case, the *-c* must be added.

**Note:** when -d is not provided, *openFPGALoader* will opens the first *ftdi*
found, if more than one converter is connected to the computer,
the *-d* option is the better solution

#### Reset device

```bash
openFPGALoader [options] -r
```

#### load bitstream device (memory or flash)
```bash
openFPGALoader [options] /path/to/bitstream.ext
```

### CYC1000

#### loading in memory:

sof to svf generation:
```bash
quartus_cpf -c -q -g 3.3 -n 12.0MHz p project_name.sof project_name.svf
```
file load:
```bash
openFPGALoader -b cyc1000 project_name.svf
```

#### SPI flash:
sof to rpd:
```bash
quartus_cpf -o auto_create_rpd=on -c -d EPCQ16A -s 10CL025YU256C8G project_name.svf project_name.jic
```
file load:
```bash
openFPGALoader -b cyc1000 -r project_name_auto.rpd
```

**Note about SPI flash:
svf file used to write in flash is just a bridge between FT2232 interfaceB
configured in SPI mode and sfl primitive used to access EPCQ SPI flash.**

**Note about FT2232 interfaceB:
This interface is used for SPI communication only when the dedicated svf is
loaded in RAM, rest of the time, user is free to use for what he want.**

### ARTY and Spartan Edge Accelerator Board

To simplify further explanations, we consider the project is generated in the
current directory.

**Note: Spartan Edge Accelerator Board has only pinheader, so the cable must be
provided**

#### loading in memory:

*.bit* file is the default format generated by *vivado*, so nothing special
task must be done to generates this bitstream.

__file load:__
```bash
openFPGALoader -b arty *.runs/impl_1/*.bit
```
or
```bash
openFPGALoader -b spartanEdgeAccelBoard -c digilent_hs2 *.runs/impl_1/*.bit
```

#### SPI flash (only for ARTY):
.mcs must be generates through vivado with a tcl script like
```tcl
set project [lindex $argv 0]

set bitfile "${project}.runs/impl_1/${project}.bit"
set mcsfile "${project}.runs/impl_1/${project}.mcs"

write_cfgmem -format mcs -interface spix4 -size 16 \
    -loadbit "up 0x0 $bitfile" -loaddata "" \
    -file $mcsfile -force

```
**Note:
*-interface spix4* and *-size 16* depends on SPI flash capability and size.**

The tcl script is used with:
```bash
vivado -nolog -nojournal -mode batch -source script.tcl -tclargs myproject
```

__file load:__
```bash
openFPGALoader -b arty *.runs/impl_1/*.mcs
```
### MachXO3 Starter Kit

#### Flash memory:

*.jed* file is the default format generated by *Lattice Diamond*, so nothing
special must be done to generates this file.

__file load__:
```bash
openFPGALoader -b machXO3SK impl1/*.jed
```
#### SRAM:

To generates *.bit* file **Bitstream file** must be checked under **Exports Files** in *Lattice Diamond* left panel.

__file load__:
```bash
openFPGALoader -b machXO3SK impl1/*.bit
```

### Trenz GOWIN LittleBee (TEC0117) and Sipeed Tang Nano

*.fs* file is the default format generated by *Gowin IDE*, so nothing
special must be done to generates this file.

Since the same file is used for SRAM and Flash a CLI argument is used to
specify the destination.

#### Flash SRAM:

with **-m**

__file load (Trenz)__:
```bash
openFPGALoader -m -b littleBee impl/pnr/*.fs
```
__file load (Tang Nano)__:
```bash
openFPGALoader -m -b tangnano impl/pnr/*.fs
```

#### Flash (only with Trenz board):

with **-f**

__file load__:
```bash
openFPGALoader -f -b littleBee impl/pnr/*.fs
```
