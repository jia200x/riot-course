class: center, middle

# Getting started

---

## Optional prerequisites: setup a local environment

1. Install and setup Git:
```bash
$ sudo apt-get install git
$ git config --global user.name "Your name"
$ git config --global user.email "Your email"
```
2. Get the code:

  - Latest version:
```bash
$ git clone --depth=1 https://github.com/RIOT-OS/RIOT.git
```

  - Latest stable branch:
```bash
$ git clone -b 2018.01-branch https://github.com/RIOT-OS/RIOT.git
```

3. Getting the workshop code:
```
$ git clone https://gitlab.inria.fr/riot-workshop-samples.git
```

---

## Optional prerequisites: setup your build environment

First possibility: install a toolchains and development tools locally:
  - Build essential tools (make, gcc, etc):
```bash
$ sudo apt-get install build-essential g++-multilib python-serial
```
  - Install toolchains (ARM):
```bash
$ sudo add-apt-repository ppa:team-gcc-arm-embedded/ppa
$ sudo apt-get update
$ sudo apt-get install gcc-arm-embedded
```
  - Install flasher tools, OpenOCD (version >= 0.10 required)
```bash
$ sudo apt-get install openocd
```
Otherwise, build OpenOCD from sources:<br>https://github.com/RIOT-OS/RIOT/wiki/OpenOCD

---

## Prerequisites: setup your build environment

- Using Docker
```bash
$ docker pull riot/riotbuild
$ cd <application directory>
$ make BUILD_IN_DOCKER=1
```

- Using a VM, with vagrant
```bash
$ vagrant up
$ vagrant ssh
```

- Using the provided VM in virtualbox &#x21d2; **our choice**

  - Toolchains, debugger, flasher tools already installed

  - RIOT code already downloaded

  - Sample applications provided

- More info on the Wiki:

.right[&#x21d2; &nbsp;&nbsp;https://github.com/RIOT-OS/RIOT/wiki/Setup-a-Build-Environment
]

---

## Writing your first application

A minimal RIOT application consists in:
- A `Makefile`

```mk
APPLICATION = example

BOARD ?= native

RIOTBASE ?= $(CURDIR)/../../RIOT

DEVELHELP ?= 1

include $(RIOTBASE)/Makefile.include
```

- A C-file containing a main function:

```c
#include <stdio.h>

int main(void)
{
    puts("My first RIOT application");
    return 0;
}
```

---

## Build the application (native)

Simply run **make** from the application directory:

```sh
$ cd ~/riot-workshop-samples/getting-started
$ make
Building application "example" for "native" with MCU "native".

"make" -C /home/user/RIOT/boards/native
"make" -C /home/user/RIOT/boards/native/drivers
"make" -C /home/user/RIOT/core
"make" -C /home/user/RIOT/cpu/native
"make" -C /home/user/RIOT/cpu/native/periph
"make" -C /home/user/RIOT/cpu/native/vfs
"make" -C /home/user/RIOT/drivers
"make" -C /home/user/RIOT/drivers/periph_common
"make" -C /home/user/RIOT/sys
"make" -C /home/user/RIOT/sys/auto_init
 text   data  bss    dec    hex   filename
 20206  568   47652  68426  10b4a .../getting-started/bin/native/example.elf
```

_Trick:_ use `-C` option with `make`
```
$ cd ~/riot-workshop-samples
$ make -C getting-started
```

---

## Run the application (native)

**native** target runs RIOT application as **Linux process**

Use the **term** target of `make`:

```sh
$ make -C getting-started term
/home/user/riot-workshop-sources/getting-started/bin/native/example.elf
RIOT native interrupts/signals initialized.
LED_RED_OFF
LED_GREEN_ON
RIOT native board initialized.
RIOT native hardware initialization complete.

main(): This is RIOT! (Version: vm-riot)
My first RIOT application
```
_Tricks:_<br>
**term** depends on **all** (build) target. Use the following command to force a rebuild:
```sh
$ make -C getting-started all term
```
**RIOTBASE** variable can be overriden to use a different RIOT location

---

## Build on hardware

- Use the ST B-L072Z-LRWAN1 board

- Use `BOARD` variable to select the target at build time

```sh
$ make BOARD=b-l072z-lrwan1 -C getting-started
Building application "example" for "b-l072z-lrwan1" with MCU "stm32l0".

"make" -C /home/user/RIOT/boards/b-l072z-lrwan1
"make" -C /home/user/RIOT/core
"make" -C /home/user/RIOT/cpu/stm32l0
"make" -C /home/user/RIOT/cpu/cortexm_common
"make" -C /home/user/RIOT/cpu/cortexm_common/periph
"make" -C /home/user/RIOT/cpu/stm32_common
"make" -C /home/user/RIOT/cpu/stm32_common/periph
"make" -C /home/user/RIOT/cpu/stm32l0/periph
"make" -C /home/user/RIOT/drivers
"make" -C /home/user/RIOT/drivers/periph_common
"make" -C /home/user/RIOT/sys
"make" -C /home/user/RIOT/sys/auto_init
"make" -C /home/user/RIOT/sys/isrpipe
"make" -C /home/user/RIOT/sys/newlib_syscalls_default
"make" -C /home/user/RIOT/sys/pm_layered
"make" -C /home/user/RIOT/sys/tsrb
"make" -C /home/user/RIOT/sys/uart_stdio
 text   data    bss    dec    hex filename
 7596    140   2740  10476   28ec .../getting-started/bin/b-l072z-lrwan1/example.elf
```

---

## Run on hardware

Use the **flash** and **term** targets:
- **flash** calls the flasher tool automatically (OpenOCD)
- **term** opens a serial terminal on the board (using pyterm by default)

```sh
$ make BOARD=b-l072z-lrwan1 -C getting-started flash term
[...]
### Flashing Target ###
Open On-Chip Debugger 0.10.0+dev-00290-g5a98ff78 (2018-01-31-14:50)
[...]
INFO # Connect to serial port /dev/ttyACM0
Welcome to pyterm!
Type '/exit' to exit.
INFO # main(): This is RIOT! (Version: vm-riot)
2018-03-23 18:28:48,474 - INFO # My first RIOT application
```

Useful variables:
- **PORT**: specify a specific serial port (`/dev/ttyACM1`)
- **TERMPROG**: specify another terminal application (`gtkterm`, etc)
- **TERMFLAGS**: override terminal application parameters

```sh
make BOARD=b-l072z-lrwan1 TERMPROG=gtkterm \
TERMFLAGS="-s 115200 -p /dev/ttyACM0 -e" flash term
```
---

## Debugging an application

Use **debug** targets

**DEVELHELP**

Useful `CFLAGS` options:<br>
-DLOG_LEVEL=LOG_DEBUG : enable debug output globally
-DDEBUG_ASSERT_VERBOSE : catch `FAILED ASSERTION` errors and give the line number

---

## Extending the application

In the `Makefile` or from the command line:

- Add extra modules with **USEMODULE**<br>
    &#x21d2; `xtimer`, `fmt`, `shell`, `ps`, etc

- Include packages with **USEPKG**<br>
    &#x21d2; `lwip`, `semtech-loramac`, etc

- Use MCU peripherals with **FEATURES_REQUIRED**:<br>
    &#x21d2; `periph_gpio`, `periph_uart`, `periph_spi`, `periph_i2c`

Example:
```mk
USEMODULE += xtimer shell

USEPKG += semtech-loramac

FEATURES_REQUIRED += periph_gpio
```

---

## Writing an application with a shell

Modify the `getting-started` application:

- Add the **shell** module to the `Makefile`

```mk
USEMODULE += shell
```

- Modify the `main.c`:

```c
#include "shell.h"
```

```c
/* in main */
char line_buf[SHELL_DEFAULT_BUFSIZE];
shell_run(NULL, line_buf, SHELL_DEFAULT_BUFSIZE);
return 0;
```

Build and run:
```sh
$ make all term
> help
help
Command              Description
---------------------------------------
> 
```

---

## Adding commands to the shell

- Add `shell_commands` module &#x21d2; add default commands (`reboot`)

- Include extra modules with predefined commands: `ps`, `random`

- Define your own handler:

```c
int cmd_handler(int argc, char **argv);
```

- Add it to the shell initialization:

```c
#include "shell.h"

static int cmd_handler(int argc, char **argv)
{
    /* Your code */
}

static const shell_command_t shell_commands[] = {
    { "command", "description", cmd_handler },
    { NULL, NULL, NULL }
};
[...]
shell_run(shell_commands, line_buf, SHELL_DEFAULT_BUFSIZE);
```

---

## An example of command handler

```c
static int cmd_handler(int argc, char **argv)
{
    if (argc < 3) {
        printf("usage: %s <arg1> <arg2>\n", argv[0]);
        return 1;
    }

    printf("Using arguments %s and %s\n", argv[1], argv[2]);
    return 0;
}
```

```c
$ make all term
> command
usage: command <arg1> <arg2>
> command arg1 arg2
Using arguments arg1 and arg2
```

It works the same on hardware without modifications:
```c
$ make BOARD=b-l072z-lrwan1 flash term
```