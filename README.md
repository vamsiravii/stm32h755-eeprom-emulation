# EEPROM Emulation for STM32H755 (dual-core)

Flash-based EEPROM emulation for the **STM32H755ZI** (Cortex-M7 + Cortex-M4), adapted from STMicroelectronics' EEPROM emulation library to the H7's 128 KB flash sectors. It lets firmware keep small values, such as calibration data or counters, across power cycles without an external EEPROM chip.

## How it works

- Two flash sectors (**Bank 2, Sectors 6 and 7**, starting at `0x081C0000`) are used as alternating "pages". When one fills up, the latest values are copied to the other and the old sector is erased. This spreads wear across both sectors.
- Up to **216 virtual 16-bit variables** are defined in `eeprom_vars_list.h`. Adjust the list to the number of variables you need.
- The emulation runs on the **Cortex-M7** core (`CM7/`). The `CM4/` project is the standard CubeMX dual-core scaffold and has no application code of its own.

## Layout

| Path | Purpose |
|---|---|
| `CM7/Middleware/Eeprom/eeprom.c`, `eeprom.h` | The emulation library (flash layout, init, read and write) |
| `CM7/Middleware/Eeprom/eeprom_vars*.{c,h}` | Virtual address table for the stored variables |
| `CM7/Middleware/Eeprom/eeprom_test.{c,h}` | On-target tests (see below) |
| `CM7/Core/Src/main.c` | Calls `EEPROM_RunTests()` at startup |
| `eeprom emulation.ioc` | CubeMX configuration |

## Built-in tests

`EEPROM_RunTests()` runs four checks and records the outcome in the global `eeprom_test_results`:

| Field | Checks that |
|---|---|
| `init` | `EEPROM_Init()` succeeds |
| `save_load` | a saved pattern reads back unchanged |
| `newest_wins` | after two saves, a load returns the second value |
| `ping_pong` | 10 consecutive save/load cycles are all correct |
| `all_passed` | all of the above are 1 |

To read the results, start a debug session in STM32CubeIDE and add `eeprom_test_results` to *Live Expressions*.

## Build

Open the project in **STM32CubeIDE**, build the `CM7` configuration (and `CM4`, which dual-core debugging expects), then flash and debug as usual for the board. Build output goes in `Debug/` and is git-ignored.

## Credits

The flash emulation algorithm is from STMicroelectronics' EEPROM emulation library (Copyright STMicroelectronics, see the licence headers in the source files). This repository adapts it for the STM32H755 sector layout and adds the variable table and tests.
