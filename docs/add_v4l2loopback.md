# 1. Add v4l2loopback module

Create module v4l2loopback

```bash
cd project-spec/meta-user/recipes-module
petalinux-create -t modules -n v4l2loopback --enable
```
This create file: 
```
project-spec/meta-user/recipes-modules/v4l2loopback/v4l2loopback.bb
```
Edit bitbake file:
```
SUMMARY = "Recipe for  build an external v4l2loopback Linux kernel module"
SECTION = "PETALINUX/modules"
LICENSE = "GPLv2"
LIC_FILES_CHKSUM = "file://COPYING;md5=b234ee4d69f5fce4486a80fdaf4a4263"

inherit module

INHIBIT_PACKAGE_STRIP = "1"

SRC_URI = "git://github.com/v4l2loopback/v4l2loopback.git;branch=main"

SRCREV = "c394f8fb2c168932055c2577247c42390198d7c9"

S = "${WORKDIR}/git"

# Build only the kernel module, skip userspace utilities
EXTRA_OEMAKE = "'KERNELRELEASE=${KERNEL_VERSION}' -C ${STAGING_KERNEL_DIR} M=${S} modules"

KERNEL_MODULE_AUTOLOAD += "v4l2loopback"
```
# 2. Enable Ffmpeg
```bash
petalinux-config -c rootfs
```
Location: -> Filesystem Packages -> libs -> ffmpeg
# 3. Flash on board
Due to setup autoload: KERNEL_MODULE_AUTOLOAD += "v4l2loopback"
We already have: /dev/video0
If you want create a new device:
```
rmmod v4l2loopback
modprobe v4l2loopback devices=1 video_nr=3 card_label="VirtualCam" debug=1
dmesg | tail -n 50
```
This create: /dev/video3
Create script: stream_yuv.sh
```bash
#!/bin/bash

VIDEO_DEV="/dev/video0"
INPUT_FILE="/tmp/test_video_trim_raw.yuv"
WIDTH=1280
HEIGHT=720
FRAMERATE=30

# Check device exists
if [ ! -e "$VIDEO_DEV" ]; then
    echo "Error: $VIDEO_DEV not found!"
    exit 1
fi

# Infinite loop
while true; do
    echo "Streaming $INPUT_FILE to $VIDEO_DEV..."
    ffmpeg -re -f rawvideo \
        -pix_fmt yuv420p \
        -s ${WIDTH}x${HEIGHT} \
        -r ${FRAMERATE} \
        -i "$INPUT_FILE" \
        -f v4l2 "$VIDEO_DEV"

    echo "File ended — restarting stream..."
    sleep 1
done
``` 
Change mode:
```bash
chmod +x /tmp/stream_yuv.sh
```
Run:
```bash
nohup /tmp/stream_yuv.sh &
```
Stream video from /dev/video0 encode h264 and transfer to PC
```bash
gst-launch-1.0 v4l2src device=/dev/video0 io-mode=2 ! \
  "video/x-raw,format=I420,width=1280,height=720,framerate=30/1" ! \
  x264enc tune=zerolatency bitrate=2000 speed-preset=ultrafast ! \
  rtph264pay config-interval=1 pt=96 ! \
  udpsink host=192.168.30.3 port=5000
```
On PC receive:
```bash
gst-launch-1.0 udpsrc port=5000 caps="application/x-rtp, media=video, encoding-name=H264, payload=96" ! rtph264depay ! h264parse ! avdec_h264 ! autovideosink
```