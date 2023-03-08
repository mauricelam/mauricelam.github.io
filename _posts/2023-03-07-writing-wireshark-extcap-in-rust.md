Recently I have been working on a project (what is now [btsnoop-extcap](https://crates.io/crates/btsnoop-extcap)) that calls for an extcap extension. Having recently learned Rust and really liking that language, I decided to try what it would take to write it in Rust.

# What is extcap?

[Wireshark's documentation](https://www.wireshark.org/docs/man-pages/extcap.html) has the following description of extcap:

> The extcap interface is a versatile plugin interface that allows external binaries to act as capture interfaces directly in Wireshark. It is used in scenarios, where the source of the capture is not a traditional capture model (live capture from an interface, from a pipe, from a file, etc). The typical example is connecting esoteric hardware of some kind to the main Wireshark application.

In other words, you can think of extcap as an adapter program, that bridges between an "estoeric hardware" to Wireshark. For normal uses, Wireshark can either use built-in interface capturing for things like `eth0`, or it can read from a pcap (or pcapng) file. It turns out, Wireshark also accepts a pipe to its stdin that writes pcap packets to it, so something like `cat mycapture.pcap | wireshark` will also work. This becomes more interesting when you consider the program doesn't have to be `cat`, but can be any program that writes `pcap` packets to stdout.

In my `btsnoop-extcap` project, what I want is to read the BTSnoop logs from the log file on an Android phone (the "esoteric hardware" in this case), and writes the output in pcap format for Wireshark.

Extcap takes the pipe model one step further, and introduce additional steps to allow the capture to feel more natively integrated into Wireshark, with the following 4 steps:

1. Interfaces – listing the supported "virtual" capture interfaces, for Wireshark to show in its UI.
2. DLT - indicating the data link type of all the supported interfaces.
3. Config - optional, a list of configurations that the user can change.
4. Capture - perform the capture and writes the packet to a given fifo.

# How to write extcap in Rust?

I ended up creating the [`r-extcap`](https://docs.rs/r-extcap/latest/r_extcap/) crate for this purpose. Getting started on this is simple, you simply create a `clap` dervied struct, and put `ExtcapArgs` as one of its field with the `#[command(flatten)]` attribute. Then in the `main` function, call `let my_args = MyArgs::parse()`, and then call `my_args.extcap.run()`. The return value of `run()` will be one of the 4 steps described above (actually 5, with "Reload config") being an optional 3.5th step.

Here's a complete example:

```rs
use clap::Parser;
use r_extcap::{cargo_metadata, ExtcapArgs, ExtcapStep, interface::*, controls::*, config::*};

#[derive(Debug, Parser)]
struct AppArgs {
    #[command(flatten)]
    extcap: ExtcapArgs,
}

lazy_static! {
    // static ref CONFIG_FOO: SelectorConfig = ...;
    // static ref CONFIG_BAR: StringConfig = ...;

    // static ref CONTROL_A: BooleanControl = ...;

    // static ref INTERFACE_1: Interface = ...;
}

fn main() -> anyhow::Result<()> {
    match AppArgs::parse().extcap.run()? {
        ExtcapStep::Interfaces(interfaces_step) => {
            interfaces_step.list_interfaces(
                &cargo_metadata!(),
                &[
                    // &*INTERFACE_1,
                ],
                &[
                    // &*CONTROL_A,
                    // &*CONTROL_B,
                ],
            );
        }
        ExtcapStep::Dlts(dlts_step) => {
            dlts_step.print_from_interfaces(&[
                // &*INTERFACE_1,
            ])?;
        }
        ExtcapStep::Config(config_step) => config_step.list_configs(&[
            // &*CONFIG_FOO,
            // &*CONFIG_BAR,
        ]),
        ExtcapStep::ReloadConfig(reload_config_step) => {
            reload_config_step.reload_from_configs(&[
                // &*CONFIG_FOO,
                // &*CONFIG_BAR,
            ])?;
        }
        ExtcapStep::Capture(capture_step) => {
            // Run capture
        }
    }
    Ok(())
}
```

# What are the alternatives in Rust?

When I initially attempted the project, I looked at the [`extcap`](https://docs.rs/extcap/latest/extcap/) crate, which does the same thing. However, when I attempted to compile the example on its docs, the example was panicking and I didn't go much further to debug the issue, since looking at the [documentations for extcap](https://www.wireshark.org/docs/wsdg_html_chunked/ChCaptureExtcap.html), it didn't look to hard to write a binary with the given requirements, especially with the excellent command line argument parsing functionality provided by the `clap` crate.

In hindsight, that was probably a rash decision, since I ended up spending a lot more time cleaning up the APIs and writing the documentations for the library. But by the time I realize how much work this is going to take, I was already knee-deep in the water, so I figured I might as well use this as a learning experience, finish it and publish it to crates.io.

Perhaps as a justification for spending the time, I tried to differentiate my crate in the following two places:

1. Having extensive documentation in all modules and functions, complete with `#[warn(missing_docs)]` to force my hand even when I am feeling lazy.
2. Allowing access at any level. While the simplest way would be to use the high level API `ExtcapArgs.run()`, you can also choose to read the individual fields in `ExtcapArgs` should you need that low-level access. Similarly, while convenient methods are provided for controls to send messages, like `ButtonControl.set_label()`, you can also build your own `ControlPacket` and send it directly using `ExtcapSender`.
