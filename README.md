# rusty-pico-1


🔧 1. Set up your Rust toolchain for embedded

You need nightly Rust and the right target:

rustup install nightly
rustup target add thumbv6m-none-eabi


The RP2040 uses Cortex-M0+ cores, which is thumbv6m-none-eabi.

📦 2. Create a new project
cargo new --bin pico_baremetal
cd pico_baremetal


In Cargo.toml, add dependencies:

[dependencies]
cortex-m = "0.7"
cortex-m-rt = "0.7"
rp2040-hal = "0.9"   # HAL for Raspberry Pi Pico
panic-halt = "0.2"

⚙️ 3. Configure your project for no-std

In src/main.rs:

#![no_std]
#![no_main]

use cortex_m_rt::entry;
use panic_halt as _;

use rp2040_hal as hal;

#[entry]
fn main() -> ! {
    // get access to hardware peripherals
    let mut pac = hal::pac::Peripherals::take().unwrap();

    // configure watchdog (required by rp2040-hal)
    let mut watchdog = hal::watchdog::Watchdog::new(pac.WATCHDOG);

    // set up clocks
    let clocks = hal::clocks::init_clocks_and_plls(
        hal::rosc::rosc_reset(&mut pac.RESETS).initialize(),
        pac.XOSC,
        pac.CLOCKS,
        pac.PLL_SYS,
        pac.PLL_USB,
        &mut pac.RESETS,
        &mut watchdog,
    ).ok().unwrap();

    // configure GPIO pin 25 (onboard LED)
    let sio = hal::Sio::new(pac.SIO);
    let pins = hal::gpio::Pins::new(pac.IO_BANK0, pac.PADS_BANK0, sio.gpio_bank0, &mut pac.RESETS);
    let mut led_pin = pins.gpio25.into_push_pull_output();

    loop {
        led_pin.set_high().unwrap();
        cortex_m::asm::delay(500_000);
        led_pin.set_low().unwrap();
        cortex_m::asm::delay(500_000);
    }
}


That’s a blinky program for the Pico.

🏗 4. Build it
cargo build --release --target thumbv6m-none-eabi


The binary will be in:

target/thumbv6m-none-eabi/release/pico_baremetal


But the Pico expects a UF2 file.

🔄 5. Convert ELF → UF2

Install elf2uf2-rs:

cargo install elf2uf2-rs


Convert:

elf2uf2-rs target/thumbv6m-none-eabi/release/pico_baremetal pico.uf2

⬆️ 6. Flash to Pico

Hold down the BOOTSEL button on the Pico.

Plug it in via USB.

It will appear as a USB drive (RPI-RP2).

Copy pico.uf2 onto it.

Your code will start running immediately.

✅ That’s the minimal path: Rust → ELF → UF2 → Pico.
You can expand with debugging (probe-rs), RTOS-like crates (embassy), or use the higher-level rp-pico crate for easier setup.

Do you want me to show you the rp-pico crate version (simpler boilerplate for the Pico), or stick with bare rp2040-hal?
