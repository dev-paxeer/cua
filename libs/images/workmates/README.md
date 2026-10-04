# libs/images/workmates

The Workmates agent desktop: `ghcr.io/trycua/linux:24.04` (the full tier:
Ubuntu 24.04, XFCE on Xvfb `:1`, Chromium, Firefox, cua-spacesd) with
Workmates branding on everything a viewer sees. Nothing about cua-spacesd,
input, audio or streaming changes, so the base image's doctor claims hold.

| On screen | Comes from | Lands in the guest at |
| --- | --- | --- |
| Wallpaper, no desktop icons, one backdrop for every workspace | `files/backgrounds/workmates.jpg`, `files/xfconf/xfce4-desktop.xml` | `/usr/share/backgrounds/workmates/`, `/etc/xdg/xfce4/xfconf/xfce-perchannel-xml/` |
| The dock: browser, files, terminal, then the open windows; no top panel | `files/xfconf/xfce4-panel.xml`, `files/applications/*.desktop`, `files/icons/<size>/apps/*.png` | `/home/cua/.config/xfce4/xfconf/xfce-perchannel-xml/`, `/usr/share/applications/`, `/usr/share/icons/hicolor/` |
| Greybird-dark windows and widgets, Inter UI font, greyscale antialiasing | `files/xfconf/xsettings.xml`, `files/xfconf/xfwm4.xml` | `/etc/xdg/xfce4/xfconf/xfce-perchannel-xml/` |
| The wallpaper on any monitor name the X server reports | `files/bin/workmates-apply-theme`, `files/autostart/workmates-theme.desktop` | `/opt/workmates/bin/`, `/home/cua/.config/autostart/` |

`image.json` is `linux/image.json`'s full tier under the Workmates name; the
Dockerfile re-stamps `/etc/cua-image/manifest.json` from it. When the base
moves, keep `BASE_IMAGE` and the tool pins in `image.json` in step.

## Build and publish

```bash
# rootfs for docker / gVisor (add --outputs rootfs,disk for the VM disk too)
libs/images/build.sh workmates --platform linux/amd64 \
    --repo ghcr.io/dev-paxeer/workmates-linux --tag 24.04 --push

# gate it the way the linux image is gated, before anything runs on it
scripts/images/image-doctor-lane.sh --lane runc --image ghcr.io/dev-paxeer/workmates-linux:24.04
```

## Use it

Set `CUA_IMAGE_LINUX=ghcr.io/dev-paxeer/workmates-linux:24.04` wherever
sandboxes are created (the SDK, `cua`, a host that provides Spaces): `linux`,
`Image.linux()` and `create_space(image="linux")` then all resolve to it. Or
pass the reference as the image.

## Changing the branding

- **Wallpaper**: replace `files/backgrounds/workmates.jpg` (any size; it is
  zoomed to fill). Keep the name, or change it in `xfce4-desktop.xml` and
  `workmates-apply-theme` too.
- **Dock icons**: replace the PNGs under `files/icons/<size>x<size>/apps/`
  (square, transparent background, every size listed). A new launcher is one
  `.desktop` in `files/applications/` plus a `launcher` plugin in
  `xfce4-panel.xml`.
- **Theme and font**: `xsettings.xml` and `xfwm4.xml`. A GTK theme other than
  Greybird must be installed by the Dockerfile.
