# CHOUPI - An open-source secure-oriented Java Card Virtual Machine

## Copyright and license

Copyright (C) 2020

This software is licensed under the MIT license. See [LICENSE](LICENSE) file at
the root folder of the project.

## Author

  * Guillaume BOUFFARD (<mailto:guillaume.bouffard@ssi.gouv.fr>)

## Description

CHOUPI is a Java Card Virtual Machine security-oriented for small footprint
device. Currently, this implementation provides a proof-of-concept of Java Card
3.0.5.

## Dependencies

### [Java Card & Global Platform API](https://github.com/choupi-project/javacard-api)

* ant

### [Java Card RomMask generator](https://github.com/choupi-rommask/project)

* maven

### [Java Card Operating System](https://github.com/choupi-project/choupi-os)

 * The `arm-none-eabi` toolchain
 * openocd
 * rlwrap
   
### Java Card Virtual Machine

  * Boost is required for building the computer version
  * CMake 

## Fetching this repository

To clone the repository and its dependency, you should execute the following
command:

```
git clone --recurse-submodules https://github.com/choupi-project/choupi
```

## Toolchain setup

 The nightly version of [Rust](https://www.rust-lang.org/) is required. To
 install it you should:

1. Install `rustup`:
   ``` sh
      curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   ```
   
2. Adding rust to your `PATH` file:
   ``` sh
   source ~/.cargo/env
   ```

3. Install the nightly version of the rust compiler:
   ```sh
   rustup default nightly-2020-03-10
   ```

   To have a *ready-to-build* version of the Java Card OS. The last checked
   version of rust nightly compiler is `1.43.0-nightly`.

4. Install `xargo`:
   ``` sh
   cargo install xargo
   ```

5. Add `rust-src` component:
   ``` sh
   rustup component add rust-src
   ```

6. To build to ARM Cortex M4 board, you should run:
   ``` sh
   rustup target add thumbv7em-none-eabi
   ```

## Building

The Java Card Virtual Machine has two main targets: computer and embedded board. The
Java Card OS computer version emulates the MPU behavior. The MPU emulation on computer blocks
the usage of a debugger (but debugging works on embedded version :).

### Building options

| Option                | Default value | Description                                                                                                                          |
|-----------------------|---------------|--------------------------------------------------------------------------------------------------------------------------------------|
| `CHOUPI_TARGET_PC`    | ON            | Building for PC                                                                                                                      |
| `CHOUPI_TARGET_STM32` | OFF           | Building for STM32 board, currently only Nucleo [STM32f401](https://www.st.com/en/evaluation-tools/nucleo-f401re.html) is supported. |
| `CHOUPI_ENABLE_LTO`   | ON            | Enable Link Time Optimization (LTO)                                                                                                  |
| `CHOUPI_OS_DEBUG`     | ON            | Enable OS debug output                                                                                                               |
| `CHOUPI_JCVM_DEBUG`   | OFF           | Enable JCVM debug output                                                                                                                                     |

### CHOUPI for PC

#### Building 

  ``` sh
  mkdir -p build/pc
  cd build/pc
  cmake -DCHOUPI_TARGET_PC=ON -DCHOUPI_OS_DEBUG=ON -DCHOUPI_JCVM_DEBUG=ON ../../
  make
  ```
  
#### Running

To run the PC version of the JCVM, run `choupi` as:

``` sh
./choupi -m flash
```

The `flash` file is the initial memory state generated by the
[rommask](https://github.com/choupi-project/rommask).

The other options are the following:

``` sh
Options:
  -h [ --help ]                   Print help messages
  -m [ --memory ] MEMORY_FILENAME Flash Memory
  -s [ --save ]                   Save modifications on MEMORY_FILENAME
```

#### Debugging the computer version

Note that for tests, it may be hard to debug some issues, like when the child in
an emulation has an unexpected behaviour at runtime that is not a panic!, as it
would require a debugger, which is not possible given the emulator is already a
pseudo-debugger.

For this reason, sending `SIGILL` to the child process will make the emulator
(ie. parent process) to make it dump core.
The core dump can then be used to try to debug the issue.

### CHOUPI for STM32f401 board

#### Building

Currently, the only target is a Nucleo board which embeds a
[STM32f401](https://www.st.com/en/evaluation-tools/nucleo-f401re.html). The
OS can easily be extended to support new target.

  ``` sh
  mkdir -p build/stm32
  cd build/stm32
  cmake -DCHOUPI_TARGET_STM32=ON -DCHOUPI_OS_DEBUG=ON -DCHOUPI_JCVM_DEBUG=OFF ../../
  make
  ```
  
  Due to a missing of flash memory space, the STM32f401 version cannot embeds
  JCVM debug output.


#### Loading on board

Currently, the only target is ST Nucleo STM32f401. To load firmware on board, we
use `rlwrap` though OpenOCD. 

1. start OpenOCD in a terminal:
   ``` sh
   sudo make ocd
   ```

2. in another terminal, run `rlwrap`:
   ``` sh
   sudo make load
   ```
   
   As indicate, to install the firmware, you should execute:
   
   ``` sh
   reset halt; flash write_image erase loader.hex; flash write_image erase code.hex; flash write_image flash.hex; reset run
   ```

#### Access to debug messages

To see the debug message, in case of the OS is built in debug mode. The
[Makefile](Makefile) may be modified to built the OS in debug mode.

``` sh
make screen
```

#### Debugging

To running the OS on board though a debugger, `arm-none-eabi-gdb` must be
install.

1. Starting openOCD in a terminal:
   ``` sh
   sudo make ocd
   ```

2. Starting GDB in another terminal :
   ``` sh
   make debug
   ```

3. Enjoy debugging

# Roadmap

For the version 1.0, CHOUPI must implement:

* [ ] Java exceptions handling
* [ ] Input/Output buffer (like ISO7816 buffer)
* [ ] Object serialisation in Flash memory
* [ ] Shareable interface, implemented in [OS](https://github.com/choupi-project/choupi-os) but not yet implemented in the
  VM layout
* [ ] Transient objects
* [ ] Atomicity
* [ ] Implemented cryptographic functions
