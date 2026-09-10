# V4-hal

Hardware abstraction API and C++17 implementation for V4.
Updated 2026-09-10 against local source; build and hardware behavior were not revalidated during this documentation update.

## Implementation

This repository contains both public headers and compiled implementation:

| Location | Role |
|---|---|
| [include/v4/hal.h](include/v4/hal.h) | Unified C-linkage API: initialization, GPIO, UART, timers and system operations |
| [include/v4/v4_hal.h](include/v4/v4_hal.h) | Legacy API declarations |
| [src/bridge](src/bridge/) | C API bridges |
| [src/internal](src/internal/) | C++17 CRTP implementations |
| [src/common](src/common/) | Core, errors and capabilities |
| [ports/posix](ports/posix/) | POSIX platform source |
| [ports/esp32](ports/esp32/) | ESP32 platform source |
| [tests](tests/) | Mock and unit tests |

The headers, bridges and platform files above are present, not merely planned.
CMake declares version 0.2.2. That is the source version, not a claim about a published release.

## Build

CMake provides a header-only interface target `v4-hal` and a compiled static target `v4-hal-lib`.
Link the latter when the implementation is required.

From this repository:

```bash
cmake -S . -B build -DHAL_PLATFORM=posix -DV4_HAL_BUILD_TESTS=ON
cmake --build build -j
ctest --test-dir build --output-on-failure
```

For integration into a CMake application:

```cmake
add_subdirectory(path/to/V4-hal v4-hal-build)
target_link_libraries(your_target PRIVATE v4-hal-lib)
```

C++17 implementation builds disable exceptions and RTTI.
ESP32 source requires its platform SDK; V4-runtime directly compiles selected HAL sources in its ESP-IDF component.
CMake also offers ch32v203, but the referenced platform source does not exist in this checkout.
RP2350 support is not implemented here.

## SYS is a separate integration layer

The current engine uses `( arg0 arg1 arg2 sys_id -- result )` and a global callback registered with `v4_register_sys_handler()`.
HAL linkage alone does not map IDs to GPIO/UART/timer calls.
The application/runtime must dispatch the ID in its callback and invoke the appropriate HAL API.

V4-runtime currently lacks that callback registration, and its V4-std initialization is disabled.
The old `SYS <id:u8>` blink examples are historical and do not work unchanged with the current engine.

- [Current SYS contract](docs/sys-opcodes.md)
- [Legacy SYS mappings](docs/sys-opcodes-legacy.md) — migration reference only
- [HAL API guide](docs/hal-api.md) — legacy API documentation; use public headers and compiled sources to check current implementation

New embedded runtime integration belongs in V4-runtime. V4-ports is deprecated.

## License

MIT ([LICENSE-MIT](LICENSE-MIT)) OR Apache-2.0 ([LICENSE-APACHE](LICENSE-APACHE)).
