# GStreamer plugin for Rockchip MPP and RGA

# Build

```bash
meson setup build --prefix=/usr --libdir=lib/aarch64-linux-gnu
ninja -C build
sudo ninja -C build install
```

Running `gst-inspect-1.0 rockchipmpp` should show 7 elements:
```
Plugin Details:
  Name                     rockchipmpp
  Description              Rockchip Mpp Video Plugin
  Filename                 /usr/lib/aarch64-linux-gnu/gstreamer-1.0/libgstrockchipmpp.so
  Version                  1.14.4
  License                  LGPL
  Source module            gst-rockchip
  Binary package           GStreamer Rockchip Plug-ins
  Origin URL               Unknown package origin

  mpph264enc: Rockchip Mpp H264 Encoder
  mpph265enc: Rockchip Mpp H265 Encoder
  mppjpegdec: Rockchip's MPP JPEG image decoder
  mppjpegenc: Rockchip Mpp JPEG Encoder
  mppvideodec: Rockchip's MPP video decoder
  mppvp8enc: Rockchip Mpp VP8 Encoder
  mppvpxalphadecodebin: VP8/VP9 Alpha Decoder

  7 features:
  +-- 7 elements
```

## Troubleshooting

If some of the plugins do not show up after installing(usually only 3 showing up out of 7), there is likely a permission issue.
To fix, create a new group and add a udev rule to change the group permission of the affected files.

```bash
sudo groupadd -f rockchip-accel
sudo usermod -aG rockchip-accel $USER
```

Create `/etc/udev/rules.d/70-rockchip-accel.rules`:
```
SUBSYSTEM=="mpp_class", KERNEL=="mpp_service", GROUP="rockchip-accel", MODE="0660"
SUBSYSTEM=="misc", KERNEL=="rga", GROUP="rockchip-accel", MODE="0660"
```

```bash
sudo udevadm control --reload-rules && sudo udevadm trigger -s mpp_class && sudo udevadm trigger -s misc
```

It might be needed to clear the cache of GST if the plugins are still not showing up:
```bash
rm -f ~/.cache/gstreamer-1.0/registry.*.bin
```