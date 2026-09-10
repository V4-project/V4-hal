# V4 SYS Instruction

Updated 2026-09-10 against the local V4-engine source.

## Current contract

SYS has no immediate operand. The ID and all arguments are stack values:

```forth
( arg0 arg1 arg2 sys_id -- result )
```

The engine pops sys_id, arg2, arg1 and arg0, invokes the registered handler, then pushes its result.
The ID is 32 bits (v4_i32 in the C API). All calls consume four cells and produce one result, regardless of the operation selected.
Unused arguments must still be supplied. Insufficient input produces a VM stack error.

## Registration

The public API in V4-engine/include/v4/vm_api.h is:

```cpp
typedef v4_i32 (*v4_sys_handler_fn)(struct Vm *vm, v4_i32 sys_id,
                                  v4_i32 arg0, v4_i32 arg1, v4_i32 arg2);
void v4_register_sys_handler(v4_sys_handler_fn handler);
```

There is one global callback shared by all VM instances. Register it before execution; passing NULL unregisters it.
With no callback, SYS consumes the four inputs and pushes -1. That result is a handler-level unsupported value, not by itself a VM execution error.
The callback owns ID dispatch, argument interpretation and any HAL calls. Including or linking V4-hal does not install it.

## Minimal example

This illustrates an application-specific operation and does not assign a standard hardware ID:

```cpp
#include "v4/vm_api.h"

static v4_i32 example_sys(Vm *, v4_i32 id, v4_i32 arg0,
                         v4_i32 arg1, v4_i32 arg2)
{
    (void)arg1;
    (void)arg2;
    return id == 0x10000 ? arg0 : -1;
}

// Call before executing bytecode:
// v4_register_sys_handler(example_sys);
```

After registering this example callback:

```forth
42 0 0 0x10000 SYS  \ leaves 42
```

The snippet documents the callback contract; it was not compiled or run during this documentation update.

## Hardware integration status

The current engine no longer directly dispatches the legacy GPIO/UART/timer IDs in its SYS case.
V4-runtime has not registered the new callback yet. Existing hardware Forth examples therefore require integration work.

V4-std has a separate registry using uint16_t IDs and callbacks without a VM pointer.
It is not automatically connected to this API, and runtime V4-std initialization is currently disabled.
Its ID definitions describe that partial library, not an implemented universal engine mapping.

## Legacy reference

The previous 8-bit immediate form `SYS <id:u8>`, per-operation stack layouts and hardware examples are preserved in
[sys-opcodes-legacy.md](sys-opcodes-legacy.md) for migration reference. They are not valid current-engine usage.
