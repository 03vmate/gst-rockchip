# GStreamer plugin for Rockchip MPP and RGA

# Prerequisites
The code was tested to be working with nyanmisaka's MPP and RGA forks on an RK3588. Your mileage may vary.

To build and install MPP and RKA:
```bash
git clone -b jellyfin-mpp --depth=1 https://gitee.com/nyanmisaka/mpp.git rkmpp
cd rkmpp && mkdir build && cd build
cmake -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=ON -DBUILD_TEST=OFF ..
make
sudo make install
```
```bash
git clone -b jellyfin-rga --depth=1 https://gitee.com/nyanmisaka/rga.git rkrga
cd rkrga
meson setup build --prefix=/usr --libdir=lib --buildtype=release --default-library=shared -Dcpp_args=-fpermissive -Dlibdrm=false -Dlibrga_demo=false
sudo ninja -C build install
```

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
