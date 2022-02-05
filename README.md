<div align="center">

# :alembic: toy-arms
Windows game hack helper utilities in rust.
This crate has some useful macros, functions and traits.

[![Crates.io](https://img.shields.io/crates/v/toy-arms?style=for-the-badge)](https://crates.io/crates/toy-arms)
[![Docs.rs](https://img.shields.io/badge/docs.rs-66c2a5?style=for-the-badge&labelColor=555555&logoColor=white&logo=data:image/svg+xml;base64,PHN2ZyByb2xlPSJpbWciIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyIgdmlld0JveD0iMCAwIDUxMiA1MTIiPjxwYXRoIGZpbGw9IiNmNWY1ZjUiIGQ9Ik00ODguNiAyNTAuMkwzOTIgMjE0VjEwNS41YzAtMTUtOS4zLTI4LjQtMjMuNC0zMy43bC0xMDAtMzcuNWMtOC4xLTMuMS0xNy4xLTMuMS0yNS4zIDBsLTEwMCAzNy41Yy0xNC4xIDUuMy0yMy40IDE4LjctMjMuNCAzMy43VjIxNGwtOTYuNiAzNi4yQzkuMyAyNTUuNSAwIDI2OC45IDAgMjgzLjlWMzk0YzAgMTMuNiA3LjcgMjYuMSAxOS45IDMyLjJsMTAwIDUwYzEwLjEgNS4xIDIyLjEgNS4xIDMyLjIgMGwxMDMuOS01MiAxMDMuOSA1MmMxMC4xIDUuMSAyMi4xIDUuMSAzMi4yIDBsMTAwLTUwYzEyLjItNi4xIDE5LjktMTguNiAxOS45LTMyLjJWMjgzLjljMC0xNS05LjMtMjguNC0yMy40LTMzLjd6TTM1OCAyMTQuOGwtODUgMzEuOXYtNjguMmw4NS0zN3Y3My4zek0xNTQgMTA0LjFsMTAyLTM4LjIgMTAyIDM4LjJ2LjZsLTEwMiA0MS40LTEwMi00MS40di0uNnptODQgMjkxLjFsLTg1IDQyLjV2LTc5LjFsODUtMzguOHY3NS40em0wLTExMmwtMTAyIDQxLjQtMTAyLTQxLjR2LS42bDEwMi0zOC4yIDEwMiAzOC4ydi42em0yNDAgMTEybC04NSA0Mi41di03OS4xbDg1LTM4Ljh2NzUuNHptMC0xMTJsLTEwMiA0MS40LTEwMi00MS40di0uNmwxMDItMzguMiAxMDIgMzguMnYuNnoiPjwvcGF0aD48L3N2Zz4K)](https://docs.rs/toy-arms)

[Usage](#Usage) | [Examples](#fire-minimal-examples) | [Document](https://docs.rs/toy-arms)

</div>

# What's toy-arms?
This is a toolkit for those who are fed up with coding game hack with C++ but still wanna make it in an elegant way.
Since this library wraps many Windows API which frequently used, you can build hack without having to struggle with it.
By using this, many part of your stress while coding low level fashion won't come in.

But be informed that this library is primitive and still could contain some buggy code which causes errors.
I'd be pleased if you spot them and make PR or issue.

# :two_hearts: support me 
**Donating me through GitHub sponsors** would be the best way to support me and this project.
You can also support me by **starring this project**, or any kind of **PR** that either refactoring this project, or adding new feature would pump me up!

# :fire: Get started
In this section I'll showcase you various examples for different situations. Find one fits your purpose.

But before actually test the example, I'll show you some preparation steps you're supposed to know.

## preparation step1
Firstly, include `toy-arms` in your dependencies' table in `Cargo.toml`.

As of now toy-arms has 2 features which are `internal` and `external`.
`internal` feature flag is on by default so you have to specify `external` when you wanna use it.

**for internal use:**
```toml
[dependencies]
toy-arms = "0.9.2"

# This annotation below is to tell the compiler to compile this into dll. MUST.
[lib]
crate-type = ["cdylib"]
```

**for external use:**
```toml
[dependencies]
toy-arms = {version = "0.9.2", features = ["external"]}
```

## preparation step2

Secondly, sicne most of those tests are targeting the game "csgo.exe(x86)", you may have to build the code in x86 architecture depending on the example.
You can either specify in `.cargo/config.toml` as following:
```toml
[build]
target = "i686-pc-windows-msvc"
```

Or put `--target i686-pc-windows-msvc` flag everytime when you build the code.


## internal
Welcome to the examples of internal hack. 
A dll file will be generated by build these examples, you inject it with whatever dll injector you possess.

With this crate, making an injectable dll is simple as this:

```rust
// A neat macro which defines entry point instead of you.
// Also, you dont have to alloc/free console by yourself, console will show up only when debug compile.
toy_arms::create_entrypoint!(hack_main_thread);

// Main thread
fn hack_main_thread() {
    // YOUR STUNNING CODE'S SUPPOSED TO BE HERE;
    for i in 0..30000 {
        println!("printing with toy-arms {}", i);
    }
}
```

While this code below will retrieve health value of LocalPlayer object in csgo.exe.
Note that you have to update the offset of `DW_LOCAL_PLAYER`.

```rust
/*
This example is the demonstration of getting player health with toy-arms internal memory analysis feature.
Make sure that you inject this image to csgo.exe.
also, the offset of DW_LOCAL_PLAYER works as of the day i wrote this but it might not be up to date in your case.
*/
use toy_arms::GameObject;
use toy_arms::Module;
use toy_arms::{cast, create_entrypoint, VirtualKeyCode};
use toy_arms_derive::GameObject;

create_entrypoint!(hack_main_thread);

// This macro provides from_raw() func that ensures the base address is not null.
#[derive(GameObject)]
struct LocalPlayer {
    pointer: *const usize, // Denote the base address of LocalPlayer to use it later in get_health() function.
}

impl LocalPlayer {
    unsafe fn get_health(&self) -> u16 {
        *cast!(self.pointer as usize + 0x100, u16)
    }
}

// This offset has to be up to date.
const DW_LOCAL_PLAYER: i32 = 0xDB35EC;

fn hack_main_thread() {
    let module = Module::from_module_name("client.dll").unwrap();
    unsafe {
        //let dw_local_player = memory.read_mut::<LocalPlayer>(0xDA244C);
        loop {
            if let Some(i) = LocalPlayer::from_raw(module.read(DW_LOCAL_PLAYER)) {
                println!("health = {:?}", (*i).get_health());
            };
            if toy_arms::detect_keypress(VirtualKeyCode::VK_INSERT) {
                break;
            }
        }
    }
}
```

## external

On the other hand, following code is how tamper with memory externally is like.

This is the code that overwrites the value at `DW_FORCE_ATTACK` to 0x5 over and over in csgo.exe.
Note that you have to check if the address of `DW_FORCE_ATTACK` is up-to-date.

```rust
use toy_arms::{Process, VirtualKeyCode};

fn main() {
    // This offset must be up to date.
    const DW_FORCE_ATTACK: usize = 0x31FF054;
    
    // Getting process information
    let memex = Process::from_process_name("csgo.exe");
    println!("process id = {}, \nprocess handle = {:?}", memex.process_id, memex.process_handle);

    // You can get module information by using get_module_info
    let module_info = memex.get_module_info("client.dll").unwrap();
    println!("{}", module_info.module_name);

    // read fetches the value at where the address is pointing.
    // U have to specify the type of the value with turbofish
    println!("default value of dwforce_attack is: {:?}", memex.read::<u32>(memex.get_module_base("client.dll").unwrap() + DW_FORCE_ATTACK as usize).unwrap());

    loop {
        // write helps you tamper with the value.
        memex.write::<u32>(memex.get_module_base("client.dll").unwrap() + DW_FORCE_ATTACK as usize, &mut 0x5).unwrap();

        // Exit this loop by pressing INSERT
        // You can pass as many hotkeys as you want.
        if toy_arms::detect_keydown!(VirtualKeyCode::VK_INSERT, VirtualKeyCode::VK_HOME) {
            break;
        }
    }
}
```

# :card_file_box: Other examples?
Yes you have! Take a look at [examples directory](https://github.com/s3pt3mb3r/toy-arms/tree/master/examples), you'll see more examples!

However, you may need to update offsets which some examples contain with your own hands.

Refer to [hazedumper](https://github.com/frk1/hazedumper/blob/master/csgo.hpp) as always for latest offsets of CSGO.

To build examples in x86 arch:
```shell
cargo build --example EXAMPLE_NAME --target i686-pc-windows-msvc
```