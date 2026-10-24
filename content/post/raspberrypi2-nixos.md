+++
title = "Installing NixOS on a Raspberry Pi 2 Model B"
description = ""
author = 'viticlin'
date = 2026-10-24T16:00:00+01:00
tags = ["raspberry pi", "nixos", "nix"]
summary = "The tales of a Nix newbie trying to install NixOS on old hardware."
+++

When I was 16, my older cousin gifted me a Raspberry Pi to tinker with. I used it for some time as a media server, then
moved on to other things. Some months ago I rescued it from my parents' house and I started to think of a challenge that
would encourage me to do something useful with it.

## Raspberry Pi 2 Model B == Old hardware

In 2026 terms, ARM 32bit architecture is old, because it has limited support on current day software versions.
Raspberry Pi 2 Model B, released February 2015, comes with an Arm Cortex-A7 processor in its 1.1 revision [^1]. The
first revision for this CPU came out in October 2011 [^2], 15 years ago.

## My path to get here

Ok so I have "old" hardware and I want to install some new software on it. I've been intrigued by NixOS for some time,
and I saw that ARM 32bit architecture (from now on I will use `armv7l`, which is the exact architecture of the
Cortex-A7 processor) is not easy to install, there are no ISOs to download. I found the challenge I was looking for.

I got to work. I read the [NixOS wiki page for Raspberry Pi][nixos-rasp-wiki], figured out I needed to build the ISO
using Nix, and I started banged my head against the keyboard trying to create a NixOS build from my M1 MacBookPro,
obviously failing to do so.

Having understood this was not going to be a one evening project for me, I decided to first understand Nix and NixOS, at
least to a point where I could grok the incantations included in the mentioned wiki page.
To do so I read some parts of the Nix manual, but the content that served my purpose best was [Nix Pills][nix-pills]. If
you have curiosity and interest about learning Nix, add that to your reading list.

## Now we can build up

With my newly acquired knowledge, I set out to build the nixos derivation from nixpkgs using an SD card installer. 

### 1st try

Nix supports cross builds, so I can target a `armv7l` host from my `aarch64-darwin` build platform. Easy. But not for
me. Some packages needed by NixOS were not available for `aarch64-darwin`.

### 2nd try

Scratch cross building. Let's setup an emulated `armv7l` virtual machine with QEMU, install whatever linux distribution
is easiest (debian) and use Nix there to build the nixos derivation. This can't possibly fail :). It's been going on for
some time, looks good. 20 hours in, the build process gets OOM'd (32bit arch means at most 4GB memory).

### 3rd try

I set out to find a middle point between my first two tries. `aarch64-linux` is fully supported by NixOS, and it shares
the same CPU architecture as my MacBook. Let's try an `aarch64-linux` VM on top of `vz` (apple's native virtualization).
This time, the build was flying, but I still got an error on the first run.

```
CMake Error at /nix/store/39cq2g3d3flq2lmb6hc09grxs4xaax8p-cmake-4.4.3/share/cmake-4.4/Modules/FindPackageHandleStandardArgs.cmake:290 (message):
  Could NOT find Python (missing: Python_EXECUTABLE Interpreter) (found
  suitable version "3.14.7", minimum required is "3")

      Reason given by package:
          Interpreter: Wrong architecture for the interpreter "/nix/store/81ag9nxz1flw3xjkv1vvqg8y2ipx2ins-python3-3.14.7/bin/python"
```

The library `rdma-core` was getting confused with Python versions, it was using the _built_ (`armv7l`) Python instead of
the _build_ (`aarch64-linux`) Python [^3]. In order to fix that, I had to add some overlays to tell `cmake` which Python
version to use.

After another similar problem with `Pandoc`, I finally got to a `.nix` file that would build an `armv7l` NixOS ISO file
to put in an SD card!

### My `rpi2-sd-image.nix`.


```
{ 
  config,
  lib,
  pkgs,
  ...
}:

{
  imports = [
    <nixpkgs/nixos/modules/installer/sd-card/sd-image-armv7l-multiplatform.nix>
    <nixos-hardware/raspberry-pi/2>
  ];

  system.stateVersion = "26.11";
  nixpkgs.hostPlatform.system = "armv7l-linux";
  nixpkgs.buildPlatform.system = "aarch64-linux";
  nixpkgs.config.allowUnsupportedSystem = true;
  boot.supportedFilesystems.zfs = lib.mkForce false;
  hardware.raspberry-pi.firmware.uboot = {
    enable = true;
    package = pkgs.ubootRaspberryPi2;
  };

  users.users.root.openssh.authorizedKeys.keys = [
    "ssh-ed25519 [...]"
  ];

  nixpkgs.overlays = [
    (final: prev: {
      efivar = final.emptyDirectory;
      efibootmgr = final.emptyDirectory;
      pandoc = final.emptyDirectory;

      rdma-core = prev.rdma-core.overrideAttrs (old: {
        nativeBuildInputs = (old.nativeBuildInputs or []) ++ [
          final.buildPackages.python3
          final.buildPackages.python3Packages.docutils
          final.buildPackages.pandoc
        ];
        cmakeFlags = (old.cmakeFlags or []) ++ [
          "-DPython_EXECUTABLE=${final.buildPackages.python3}/bin/python3"
          "-DPANDOC_EXECUTABLE=${final.buildPackages.haskellPackages.pandoc-cli}/bin/pandoc"
        ];
      });
    })
  ];
}
```

## Next post

Now I just had to put the SD card into the Raspberry Pi and watch NixOS boot. Because, it booted without problems...
right?

## Music that helped write this post

- Beach House - Once Twice Melody
- Gurriers - Come and See

[^1]: https://www.raspberrypi.com/products/raspberry-pi-2-model-b/
[^2]: https://support.arm.com/documentation/ddi0464/latest
[^3]: This confusion has nothing to do with _built_ vs _build_, but it's funny that it got me confused twice, once
    because I did not know which version it should use and then because of the build verb conjugations.

[nixos-rasp-wiki]: https://wiki.nixos.org/wiki/NixOS_on_ARM/Raspberry_Pi
[nix-pills]: https://nixos.org/guides/nix-pills/
