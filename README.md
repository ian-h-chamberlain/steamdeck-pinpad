# Steam Deck PIN Pad

Helper systemd service to make the Steam Deck (Big Picture) PIN pad work with
keyboards.

## Installation

1. First install `evsieve` (see [their README](https://github.com/KarsMulder/evsieve)
   for instructions). If you have an AUR helper (e.g. `yay`, `paru`) installed already,
   you can also install it from [the AUR](https://aur.archlinux.org/packages/evsieve)
   directly.

   > If you use e.g. [distrobox](https://github.com/89luca89/distrobox) it should be
   > possible to install and run inside a container, but might take some
   > tweaking to get the service to work properly.

1. Copy or symlink `steamdeck-pinpad@.service` from this repo into `~/.config/systemd/user/`:

   ```sh
   cp ./steamdeck-pinpad@.service ~/.config/systemd/user/
   # OR
   systemctl link --user ./steamdeck-pinpad@.service
   ```

1. Ensure your user account has access to `/dev/input/event*` devices.
   See [the Arch Wiki page](https://wiki.archlinux.org/title/Udev#Allowing_regular_users_to_use_devices)
   for details on `udev` if you want to set this up automatically and securely.

## Usage

First, identify the keyboard(s) you want to use for the PIN pad.

1. See if the keyboard is listed by a usable ID:

   ```sh
   ls /dev/input/by-id/*-kbd
   ```

1. Using `evtest`:

   1. If needed, install `evtest`:

      ```sh
      sudo steamos-readonly disable
      sudo pacman -Sy evtest --needed
      sudo steamos-readonly enable
      ```

   1. Find the path in the first column for the device you want to use. For example,
      `/dev/input/event21` in the output below:

      ```console
      $ evtest
      No device specified, trying to scan all of /dev/input/event*
      Not running as root, no devices may be available.
      Available devices:
      /dev/input/event10:     Microsoft X-Box 360 pad 0
      /dev/input/event21:     Keychron Keychron K1
      /dev/input/event22:     Keychron Keychron K1
      /dev/input/event28:     py-evdev-uinput
      /dev/input/event4:      AT Translated Set 2 keyboard
      Select the device event number [0-29]: ^C
      ```

      > NOTE: `/dev/input/event*` paths are not guaranteed to remain consistent, so
      > so `/by-id/` paths are preferable when possible.

Then, for each device you want to use, enable the service for that device:

```sh
# Space-separated device paths in one string here
devices="$(systemd-escape --path "/dev/input/by-id/usb-Keychron_Keychron_K1-event-kbd /dev/input/event28")"
systemctl --user daemon-reload
systemctl --user enable --now steamdeck-pinpad@$devices.service
```

To see if it's, working, quit and restart Steam and open controller settings.
You should see a controller called "Steam Deck PIN Pad" listed there.
Open up Big Picture and you should be able to use your keyboard for the PIN pad now!

## Caveats / Notes

One thing to keep in mind is that the Steam Deck PIN pad is laid out opposite
to a typical keyboard number pad:

| PIN Pad | Numpad  |
|:-------:|:-------:|
| `1 2 3` | `7 8 9` |
| `4 5 6` | `4 5 6` |
| `7 8 9` | `1 2 3` |
|   `0`   |   `0`   |

This "fake controller" binds the keys according to their number, not their physical
position on the pads.
