---
title: Using a raw USB device to unlock a LUKS volume on NixOS
date: 2018-10-28
---

# Using a raw USB device to unlock a LUKS volume on NixOS

## Background (feel free to skip that part)

Yesterday, I learned about th
[boot.initrd.luks](https://nixos.org/nixos/options.html#boot.initrd.luks) option
in NixOS, and quickly made use of it to setup a new system.

But among the available options, one caught my eye: the
[`keyFile`](https://nixos.org/nixos/options.html#boot.initrd.luks+keyfile) one.
Its description reads "The name of the file (**can be a raw device or a
partition**) that should be used as the decryption key for the encrypted device.
If not specified, you will be prompted for a passphrase instead."

The part in bold caught my attention, as I hadn't heard about manipulating raw
devices on Linux before. Unfortunately, the Internet has very few information
about using raw devices with LUKS, and none at all when it comes to NixOS. After
some research, I found this article though:
[Unlocking a luks volume with a USB key](https://possiblelossofprecision.net/?p=300),
that explained how to use `dd` to copy a decryption key to a raw device, and
then to use it with LUKS.

Adapted to NixOS, here's what has to be done.

**Writing to any raw device may corrupt it and can make you lose your data very
easily. Double-check everything you do, and avoid using drives that haven't been
backed up. You've been warned.**

## 1. Set up the LUKS drive

Many tutorials
[already](https://gist.github.com/martijnvermaat/76f2e24d0239470dd71050358b4d5134)
[cover](https://qfpl.io/posts/installing-nixos/)
[this](https://chris-martin.org/2015/installing-nixos), so I'll skip that part.
From now on I'll just assume your NixOS configuration looks like this one.

```nix
boot.initrd.luks.devices = [
  { name   = "root";
    device = "$LUKS_DEVICE";
    preLVM = true;
  }
];
```

## 2. Create a random key and add it to your LUKS volume

A 4,096 bit key can be generated using, for instance,
`dd if=/dev/random of=key.bin bs=1 count=4096`. Once this is done, it should be
added to the LUKS volume: `cryptsetup luksAddKey $LUKS_DEVICE key.bin`.

## 3. Find somewhere to write your key

Assuming a USB drive `$USB_DEVICE`, you have to find some part of the device
that will not impact the existing partitions or partition table, **and** that is
large enough to store your key.

In my case, I used `dd if=$USB_DEVICE bs=1 skip=$OFFSET count=4096` (with
`$OFFSET` being the size of my USB drive minus a few kilobytes) to make sure
there wasn't anything of importance where I was going to write.

> Note that your USB device must have a "constant" name. It won't always be
`/dev/sda`, and unlike a partition, it has no UUID. In my case, I used the ID of
the device, and thus `USB_DEVICE = /dev/disk/by-id/<device-id>`.

## 4. Copy your key there

The following command copies the content of `key.bin` in the `$USB` device, at
the given offset. Please pay attention to the fact that unlike the previous
command, [we're using](https://unix.stackexchange.com/a/307188)
[`seek=$OFFSET`](https://unix.stackexchange.com/a/307188)
[here](https://unix.stackexchange.com/a/307188) (instead of `skip=$OFFSET`).

```bash
dd if=key.bin of=$USB bs=1 count=4096 seek=$OFFSET
```

If you want to make sure your key is there, you can always compare the output of
`dd if=key.bin` and `dd if=$USB_DEVICE bs=1 skip=$OFFSET count=4096`.

Now that your key is both known by your LUKS device and copied to your USB
device, you may remove it from your file system:
`shred --remove --zero key.bin`.

## 5. Update your configuration for these changes

Now NixOS has to know about the offset you chose, and the device that contains
that key.

```diff
boot.initrd.luks.devices = [
  { name          = "root";
    device        = "$LUKS_DEVICE";
+   keyFile       = "$USB_DEVICE";
+   keyFileOffset = $OFFSET;
+   keyFileSize   = 4096;
    preLVM        = true;
  }
];
```

## 6. (Optional) Allow fallback to password

In some cases (reformatting the USB key, error in setup, ...), the key written
to the USB drive might be lost. In this case, you won't be able to decrypt your
LUKS device anymore, which is undesirable.

To avoid this, you may set `fallbackToPassword` to `true` in your configuration.
That way, if the key is lost, you may still decrypt your partition using a
password.

```diff
boot.initrd.luks.devices = [
  { name          = "root";
    device        = "$LUKS_DEVICE";
    keyFile       = "$USB_DEVICE";
    keyFileOffset = $OFFSET;
    keyFileSize   = 4096;
    preLVM        = true;
+   fallbackToPassword = true;
  }
];
```
