# syntax=docker/dockerfile:1.7
# workmates: the Workmates agent desktop. ghcr.io/trycua/linux:24.04 (Ubuntu
# 24.04, XFCE on Xvfb :1, Chromium, Firefox, cua-spacesd; the full tier) with
# Workmates branding on everything a viewer sees:
#
#   - the Workmates wallpaper on every monitor and workspace, no desktop icons;
#   - one centered bottom dock: the Workmates browser, files and terminal
#     launchers (files/icons) and the open windows as icons; no top panel;
#   - Greybird-dark windows and widgets, Inter as the UI font.
#
# Nothing here touches cua-spacesd, the desktop user, input, audio or
# streaming, so the base image's doctor claims hold unchanged: image.json is
# linux/image.json's full tier under the Workmates name, re-stamped into
# /etc/cua-image/manifest.json at the end. Keep BASE_IMAGE and image.json's
# tool pins in step when the base moves.
#
# Build context: libs/images (so workmates/files/ is reachable), like every
# image here. build.sh produces the rootfs for docker / gVisor and, with
# --outputs disk, the same guest as a VM disk:
#
#   libs/images/build.sh workmates --platform linux/amd64 \
#       --repo ghcr.io/dev-paxeer/workmates-linux --tag 24.04 --push
#
# Point the SDK, the CLI and host-provided Spaces at it with
# CUA_IMAGE_LINUX=ghcr.io/dev-paxeer/workmates-linux:24.04 (then `linux`,
# Image.linux() and create_space(image="linux") all resolve to it), or pass
# the reference as the image.
#
# BASE_IMAGE is the canonical full tier; for a reproducible build pin it to a
# dated tag or a digest with --build-arg BASE_IMAGE=<ref>@sha256:<digest>.
ARG BASE_IMAGE=ghcr.io/trycua/linux:24.04
FROM ${BASE_IMAGE}
ARG TARGETARCH
ARG CUA_IMAGE_SOURCE_REVISION=

# Inter, the UI font (also what the Cua Driver cursor overlay draws with).
RUN apt-get update \
 && apt-get install -y --no-install-recommends fonts-inter \
 && rm -rf /var/lib/apt/lists/*

# The wallpaper, the dock icons (hicolor: the fallback of every icon theme)
# and the launchers the dock shows. The browser launcher is cua-chromium, the
# same Chromium cua-driver's browser tools drive, so a viewer sees the
# agent's own windows.
COPY workmates/files/backgrounds/ /usr/share/backgrounds/workmates/
COPY workmates/files/icons/ /usr/share/icons/hicolor/
COPY workmates/files/applications/ /usr/share/applications/

# XFCE defaults for every user (xfconf reads /etc/xdg under the user's own
# channel files, and the base image's desktop user ships none of these):
# wallpaper and desktop icons, GTK/icon/cursor/font settings, the window
# manager theme.
COPY workmates/files/xfconf/xfce4-desktop.xml \
     workmates/files/xfconf/xsettings.xml \
     workmates/files/xfconf/xfwm4.xml \
     /etc/xdg/xfce4/xfconf/xfce-perchannel-xml/

# The dock: the desktop user's own panel profile (xfce4-panel reads
# /etc/xdg/xfce4/panel/default.xml only when the user has no panels).
COPY --chown=cua:cua workmates/files/xfconf/xfce4-panel.xml \
     /home/cua/.config/xfce4/xfconf/xfce-perchannel-xml/xfce4-panel.xml

# Session glue: once xfdesktop is up, sets the wallpaper on whatever monitor
# names the X server reports (Xvfb "screen", Xvnc "VNC-0", a VM's virtual
# output), so the defaults above never miss one.
COPY workmates/files/bin/ /opt/workmates/bin/
COPY --chown=cua:cua workmates/files/autostart/ /home/cua/.config/autostart/

RUN chmod 0755 /opt/workmates/bin/* \
 && find /usr/share/icons/hicolor -name 'workmates-*.png' -exec chmod 0644 {} + \
 && chmod 0644 /usr/share/applications/workmates-*.desktop /usr/share/backgrounds/workmates/* \
 && if command -v gtk-update-icon-cache >/dev/null 2>&1; then gtk-update-icon-cache -f -t /usr/share/icons/hicolor; fi \
 && chown -R cua:cua /home/cua/.config

# The image's own identity: image.json (linux's full tier under the Workmates
# name) and the manifest `cua-spacesd doctor` checks the running guest against.
COPY workmates/image.json /etc/cua-image/image.json
RUN /opt/cua/bin/cua-image-manifest generate --image-json /etc/cua-image/image.json \
      --variant rootfs --arch "$TARGETARCH" --spacesd-source "$(cat /etc/cua-image/spacesd-source)" \
      --source-revision "$CUA_IMAGE_SOURCE_REVISION" --out /etc/cua-image/manifest.json
