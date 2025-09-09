# 1. Two Laptop Ubuntu
Laptop A: 192.168.30.3
###
```bash
gst-launch-1.0 v4l2src ! videoconvert ! x264enc tune=zerolatency bitrate=512 speed-preset=ultrafast ! rtph264pay config-interval=1 pt=96 ! udpsink host=192.168.30.7 port=5000
```
# 2. ZynqMP and Laptop Ubuntu
## 2.1 Only Video stream
ZynqMP: 192.168.30.22 sender a MP4 file
###
```bash
gst-launch-1.0 filesrc location=/tmp/test_video.mp4 ! qtdemux name=demux demux.video_0 ! queue ! h264parse ! rtph264pay config-interval=1 pt=96 ! udpsink host=192.168.30.3 port=5000
```
Laptop: 192.168.30.3 receiver and display
###
```bash
gst-launch-1.0 udpsrc port=5000 caps="application/x-rtp,media=video,encoding-name=H264,payload=96" ! rtph264depay ! avdec_h264 ! videoconvert ! autovideosink
```
Laptop: 192.168.30.3 sender camera
###
```bash
gst-launch-1.0 v4l2src ! videoconvert ! x264enc tune=zerolatency bitrate=2000 speed-preset=ultrafast ! rtph264pay config-interval=1 pt=96 ! udpsink host=192.168.30.22 port=5000
```
ZynqMP: 192.168.30.22 receiver and write to MP4
###
```bash
gst-launch-1.0 -e udpsrc port=5000 caps="application/x-rtp, media=video, encoding-name=H264, payload=96" ! rtph264depay ! h264parse ! mp4mux ! filesink location=/tmp/output.mp4
```