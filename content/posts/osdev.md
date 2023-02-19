---
title: "Rust Higher Half UEFI Bootloader"
date: 2023-02-11T13:15:41-05:00
draft: true
---

Operating system development has been something of a fascination of mine since
I took a course on it in college and I really enjoyed the experience of writing
a kernel, but there were lots of things we didnt have to do for ourselves and
also our OS never ran on real hardware, instead running on a very simplified
emulated computer created specifically for the course. I've tried a few times
to go further with the knowledge the course gave me and implement a system for
real hardware and with features like filesystem and device drivers, but
none of those attempts got very far. At the same time, I've been learning rust
and finally feel like im getting comfortable with the language. So I decided to
try and use my newfound love of rust as motivation to finally make something I
can call a real OS.

## Hardware

The hardware I'm developing this for is a 64 bit arm (aka aarch64) board that
is basically a clone of the raspberry pi 3. So far nothing has been device
specific except the u-boot image I had to use so I won't bother giving any more
details for now.

For when I don't feel like working with the real board over a serial connection
I'm also working with QEMU on my development machine.

## Why I Need a Bootloader

So, the first step in this project is writing a bootloader. At first I thought
this wouldn't be necessary, since the board already supports u-boot, but in
fact u-boot (as far as I can tell) only supports booting the Linux kernel and
booting other EFI binaries. If I'm wrong about this feel free to correct me. I
could hack support for booting my OS into u-boot, but I don't want to give up
the convenience of using a prebuilt binary of u-boot, especially since there is
no configuration for building u-boot for this board from source in the main
u-boot repository and the documentation for it is rather lacking. Thankfully we
don't actually need to do much in UEFI land to get our kernel binary loaded and
mapped into memory at the right place, so the EFI bootloader that we have
u-boot load can be very simple.

## Project Setup

For keeping the kernel, the loader, and any future things I may write that I
want to keep as separate packages (e.g. driver modules) together I created
a single [git repo](https://github.com/emm218/osdev) that is a cargo workspace. It contains 2 files, a very short
`Cargo.toml` for the entire workspace and a `rust-toolchain.toml` which ensures
that cargo will grab the required toolchain for our 2 slightly unusual targets

```toml
# Cargo.toml

[workspace]
members = ["loader", "kernel"]

[workspace.package]
authors = ["Emmy <emm218@proton.me>"]

# rust-toolchain.toml

[toolchain]
channel = "nightly"
targets = ["aarch64-unknown-uefi", "aarch64-unknown-none"]

```

