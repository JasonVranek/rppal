# RPPAL - Raspberry Pi Peripheral Access Library

[![Build Status](https://travis-ci.org/golemparts/rppal.svg?branch=master)](https://travis-ci.org/golemparts/rppal)
[![crates.io](https://meritbadge.herokuapp.com/rppal)](https://crates.io/crates/rppal)
[![MIT licensed](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Minimum rustc version](https://img.shields.io/badge/rustc-v1.31.0-lightgray.svg)](https://blog.rust-lang.org/2018/12/06/Rust-1.31-and-rust-2018.html)


RPPAL is a Rust library that provides access to the Raspberry Pi's GPIO, I2C, PWM and SPI peripherals. Support for [additional peripherals](https://github.com/golemparts/rppal/projects/1) will be added in future updates. The library is compatible with the Raspberry Pi A, A+, B, B+, 2B, 3A+, 3B, 3B+, CM, CM 3, CM 3+, Zero and Zero W.

Backwards compatibility for minor revisions isn't guaranteed until the library reaches v1.0.0.

RPPAL is currently under active development on the [master branch](https://github.com/golemparts/rppal/tree/master) of the repository on GitHub. If you're looking for the `README.md` or the `examples` directory for the latest release or any of the earlier releases, visit [crates.io](https://crates.io/crates/rppal), download an archived release from the GitHub [releases](https://github.com/golemparts/rppal/releases) page, or clone and checkout the relevant release tag.

## Documentation

Online documentation is available for the latest release, older releases, and the version currently in development.

* Latest release: [docs.golemparts.com/rppal](https://docs.golemparts.com/rppal)
* Older releases: [docs.rs/rppal](https://docs.rs/rppal)
* In development: [docs.golemparts.com/rppal-dev](https://docs.golemparts.com/rppal-dev)

## Usage

Add a dependency for `rppal` to your `Cargo.toml`.

```toml
[dependencies]
rppal = "0.10"
```

Call `new()` on any of the peripherals to construct a new instance.

```rust
use rppal::gpio::Gpio;
use rppal::i2c::I2c;
use rppal::pwm::{Channel, Pwm};
use rppal::spi::{Bus, Mode, SlaveSelect, Spi};

let gpio = Gpio::new()?;
let i2c = I2c::new()?;
let pwm = Pwm::new(Channel::Pwm0)?;
let spi = Spi::new(Bus::Spi0, SlaveSelect::Ss0, 16_000_000, Mode::Mode0)?;
```

Some peripherals may need to be enabled first through `sudo raspi-config` or by editing `/boot/config.txt`. Refer to the relevant module's documentation for any required steps.

## Examples

This example demonstrates how to blink an LED connected to a GPIO pin. Remember to add a resistor of an appropriate value in series, to prevent exceeding the maximum current rating of the GPIO pin and the LED.

```rust
use std::error::Error;
use std::thread;
use std::time::Duration;

use rppal::gpio::Gpio;
use rppal::system::DeviceInfo;

// Gpio uses BCM pin numbering. BCM GPIO 23 is tied to physical pin 16.
const GPIO_LED: u8 = 23;

fn main() -> Result<(), Box<dyn Error>> {
    println!("Blinking an LED on a {}.", DeviceInfo::new()?.model());

    let mut pin = Gpio::new()?.get(GPIO_LED)?.into_output();

    // Blink the LED by setting the pin's logic level high for 500ms.
    pin.set_high();
    thread::sleep(Duration::from_millis(500));
    pin.set_low();

    Ok(())
}
```

Additional examples can be found in the `examples` directory.

## Supported peripherals

### [GPIO](https://docs.golemparts.com/rppal/latest/gpio)

To ensure fast performance, RPPAL controls the GPIO peripheral by directly accessing the registers through either `/dev/gpiomem` or `/dev/mem`. GPIO interrupts are configured using the `gpiochip` character device.

#### Features

* Get/set pin modes
* Read/write pin logic levels
* Activate built-in pull-up/pull-down resistors
* Configure synchronous and asynchronous interrupt handlers

### [I2C](https://docs.golemparts.com/rppal/latest/i2c)

The Broadcom Serial Controller (BSC) peripheral controls a proprietary bus compliant with the I2C bus/interface. RPPAL communicates with the BSC using the `i2cdev` device interface.

#### Features

* Single master, 7-bit slave addresses, transfer rates up to 400kbit/s (Fast-mode)
* I2C basic read/write, block read/write, combined write+read
* SMBus protocols: Quick Command, Send/Receive Byte, Read/Write Byte/Word, Process Call, Block Write, PEC

### [PWM](https://docs.golemparts.com/rppal/latest/pwm)

RPPAL controls the Raspberry Pi's PWM peripheral through the `/sys/class/pwm` sysfs interface.

#### Features

* Up to two hardware PWM channels
* Configurable frequency, duty cycle and polarity

### [SPI](https://docs.golemparts.com/rppal/latest/spi)

RPPAL controls the Raspberry Pi's main and auxiliary SPI peripherals through the `spidev` device interface.

#### Features

* SPI master, mode 0-3, Slave Select active-low/active-high, 8 bits per word, configurable clock speed
* Half-duplex reads, writes, and multi-segment transfers
* Full-duplex transfers and multi-segment transfers
* Customizable options for each segment in a multi-segment transfer (clock speed, delay, SS change)
* Reverse bit order helper function

## Caution

Always be careful when working with the Raspberry Pi's peripherals, especially if you attach any external components to the GPIO pins. Improper use can lead to permanent damage.

## Cross compilation

If you're not working directly on a Raspberry Pi, you'll likely need to cross compile your code for the appropriate ARM architecture. Check out [this guide](https://github.com/japaric/rust-cross) for more information, or try the [cross](https://github.com/japaric/cross) project for "zero setup" cross compilation.

## Copyright and license

Copyright (c) 2017-2019 Rene van der Meer. Released under the [MIT license](LICENSE).
