# Video call on two Laptop Ubuntu
Laptop A: 192.168.30.3
###
```bash
gst-launch-1.0 v4l2src ! videoconvert ! x264enc tune=zerolatency bitrate=512 speed-preset=ultrafast ! rtph264pay config-interval=1 pt=96 ! udpsink host=192.168.30.7 port=5000
```
