# `avr-defmt-test`

This repository contains a stripped version of `defmt-test`, which is part of Ferrous Systems' [defmt](http://github.com) framework.

The only purpose of this crate is to allow easy unittesting of my other Rust project to build an embedded JVM for Atmel processors.

`defmt` targets Cortex-M and offers much more functionality than I need here (although doing a full port would be an interesting future project).

For now, this crate only contains a stripped version of the `defmt-test` part of `defmt`, modified so it can run on my AVR simulator avrora, although the implementation isn't specific to AVR in any way.

Tests can be written much in the same way as in the original `defmt-test`, see `README-ori.md`, except for the fact that the user needs to provide two implementations, and make sure an appropriate panic handler is setup in case an `assert()` call fails.

The `#[tests]` attribute now takes two required parameters: `avr_println` and `avr_exit`.
- `avr_println` should be a macro that will be used to print output, and will only be called with a string literal
- `avr_exit` should be set to a function that will be called to end the test run if all tests pass

Example (using an `avrora` module in my VM that controls the Avrora AVR simulator):
```
#[allow(unused_macros)]
macro_rules! avr_println {
    ($s:expr) => { {
        crate::avrora::print_flash_string!($s);
    } };
}

#[cfg(test)]
fn exit()  -> ! {
    avrora::print_flash_string!("Exiting");
    avrora::exit();
}

#[cfg(test)]
#[panic_handler]
fn panic(_info: &core::panic::PanicInfo) -> ! {
    print_flash_string!("TEST FAILED!");
    exit();
}

#[cfg(test)]
#[avr_defmt_test::tests(avr_exit=crate::exit, avr_println=crate::avr_println)]
mod tests {
    #[test]
    fn test1() {
        assert!(true);
    }

    #[test]
    fn test2() {
        assert_eq!(2, 1);
    }
}
```





