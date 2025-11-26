# 1. Send separate
```bash
# Video
gst-launch-1.0 -v v4l2src device=/dev/video3 io-mode=2 ! \
  "video/x-raw,format=I420,width=1280,height=720,framerate=30/1" ! \
  x264enc tune=zerolatency bitrate=2000 speed-preset=ultrafast ! \
  rtph264pay pt=96 ! udpsink host=192.168.30.7 port=5000 sync=false async=false &

# Audio
gst-launch-1.0 -v alsasrc device=hw:Loopback,1,2 ! \
  audioconvert ! audioresample ! opusenc ! rtpopuspay pt=97 ! \
  udpsink host=192.168.30.7 port=5002 sync=false async=false &
```

```bash
# Video receive
gst-launch-1.0 -v udpsrc port=5000 caps="application/x-rtp,encoding-name=H264,payload=96" ! \
  rtph264depay ! avdec_h264 ! videoconvert ! autovideosink sync=false &

# Audio receive
gst-launch-1.0 -v udpsrc port=5002 caps="application/x-rtp,media=audio,encoding-name=OPUS,payload=97" ! \
  rtpopusdepay ! opusdec ! audioconvert ! audioresample ! autoaudiosink sync=false &
```

# 2. Send through RTPbin
Send:
```bash
gst-launch-1.0 -v rtpbin name=rtpbin v4l2src device=/dev/video3 io-mode=2 ! \
"video/x-raw,format=I420,width=1280,height=720,framerate=30/1" ! x264enc \
 tune=zerolatency bitrate=2000 speed-preset=ultrafast ! rtph264pay pt=96 ! \
 rtpbin.send_rtp_sink_0 rtpbin.send_rtp_src_0 ! udpsink host=192.168.30.7 \
 port=5000 sync=false async=false         rtpbin.send_rtcp_src_0 ! \
 udpsink host=192.168.30.7 port=5001 sync=false async=false udpsrc port=5005 \
 ! rtpbin.recv_rtcp_sink_0 alsasrc device=hw:Loopback,1,2 ! audioconvert ! \
 audioresample ! opusenc ! rtpopuspay pt=97 ! rtpbin.send_rtp_sink_1 \
 rtpbin.send_rtp_src_1 ! udpsink host=192.168.30.7 port=5002 sync=false async=false         rtpbin.send_rtcp_src_1 ! udpsink host=192.168.30.7 port=5003 sync=false async=false         udpsrc port=5007 ! rtpbin.recv_rtcp_sink_1
 ```
 Receive:
 ```bash
gst-launch-1.0 -v rtpbin name=rtpbin \
v4l2src device=/dev/video3 io-mode=2 ! \
"video/x-raw,format=I420,width=1280,height=720,framerate=30/1" ! \
x264enc tune=zerolatency bitrate=2000 speed-preset=ultrafast ! \
rtph264pay pt=96 ! rtpbin.send_rtp_sink_0 \
rtpbin.send_rtp_src_0 ! udpsink host=192.168.30.7 port=5000 bind-address=192.168.30.1 sync=false async=false \
rtpbin.send_rtcp_src_0 ! udpsink host=192.168.30.7 port=5001 bind-address=192.168.30.1 sync=false async=false \
udpsrc port=5005 ! rtpbin.recv_rtcp_sink_0 \
alsasrc device=hw:Loopback,1,2 ! audioconvert ! audioresample ! opusenc ! rtpopuspay pt=97 ! rtpbin.send_rtp_sink_1 \
rtpbin.send_rtp_src_1 ! udpsink host=192.168.30.7 port=5002 bind-address=192.168.30.1 sync=false async=false \
rtpbin.send_rtcp_src_1 ! udpsink host=192.168.30.7 port=5003 bind-address=192.168.30.1 sync=false async=false \
udpsrc port=5007 ! rtpbin.recv_rtcp_sink_1 
 ```

# 3. Send on Zynq
Send:
```bash
gst-launch-1.0 -v rtpbin name=rtpbin v4l2src device=/dev/video0 io-mode=2 ! \
"video/x-raw,format=I420,width=1280,height=720,framerate=30/1" ! x264enc \
 tune=zerolatency bitrate=2000 speed-preset=ultrafast ! rtph264pay pt=96 ! \
 rtpbin.send_rtp_sink_0 rtpbin.send_rtp_src_0 ! udpsink host=192.168.30.7 \
 port=5000 sync=false async=false         rtpbin.send_rtcp_src_0 ! \
 udpsink host=192.168.30.7 port=5001 sync=false async=false udpsrc port=5005 \
 ! rtpbin.recv_rtcp_sink_0 alsasrc device=hw:Loopback,1,0 ! audioconvert ! \
 audioresample ! opusenc ! rtpopuspay pt=97 ! rtpbin.send_rtp_sink_1 \
 rtpbin.send_rtp_src_1 ! udpsink host=192.168.30.7 port=5002 sync=false async=false         rtpbin.send_rtcp_src_1 ! udpsink host=192.168.30.7 port=5003 sync=false async=false         udpsrc port=5007 ! rtpbin.recv_rtcp_sink_1
```

```bash
gst-launch-1.0 -v rtpbin name=rtpbin \
v4l2src device=/dev/video0 io-mode=2 ! \
"video/x-raw,format=I420,width=1280,height=720,framerate=30/1" ! \
x264enc tune=zerolatency bitrate=2000 speed-preset=ultrafast ! \
rtph264pay pt=96 ! rtpbin.send_rtp_sink_0 \
rtpbin.send_rtp_src_0 ! udpsink host=192.168.30.7 port=5000 sync=false async=false \
rtpbin.send_rtcp_src_0 ! udpsink host=192.168.30.7 port=5001 sync=false async=false \
udpsrc port=5005 ! rtpbin.recv_rtcp_sink_0 \
alsasrc device=hw:Loopback,1,0 ! audioconvert ! audioresample ! opusenc ! rtpopuspay pt=97 ! rtpbin.send_rtp_sink_1 \
rtpbin.send_rtp_src_1 ! udpsink host=192.168.30.7 port=5002 sync=false async=false \
rtpbin.send_rtcp_src_1 ! udpsink host=192.168.30.7 port=5003 sync=false async=false \
udpsrc port=5007 ! rtpbin.recv_rtcp_sink_1
```

Receive:
```bash
gst-launch-1.0 -v rtpbin name=rtpbin udpsrc port=5000 \
caps="application/x-rtp,media=video,encoding-name=H264,clock-rate=90000,pt=96" ! \
rtpbin.recv_rtp_sink_0 rtpbin. ! rtph264depay ! avdec_h264 ! videoconvert ! \
autovideosink sync=false udpsrc port=5001 ! rtpbin.recv_rtcp_sink_0 \
rtpbin.send_rtcp_src_0 ! udpsink host=192.168.30.1 port=5005 sync=false async=false \
udpsrc port=5002 \
caps="application/x-rtp,media=audio,encoding-name=OPUS,clock-rate=48000,pt=97" ! \
rtpbin.recv_rtp_sink_1 rtpbin. ! rtpopusdepay ! opusdec ! audioconvert ! audioresample ! \
autoaudiosink sync=false udpsrc port=5003 ! rtpbin.recv_rtcp_sink_1 \
rtpbin.send_rtcp_src_1 ! udpsink host=192.168.30.1 port=5007 sync=false async=false
```
Receive from zynq
```bash
gst-launch-1.0 -v rtpbin name=rtpbin udpsrc port=5000 \
caps="application/x-rtp,media=video,encoding-name=H264,clock-rate=90000,pt=96" ! \
rtpbin.recv_rtp_sink_0 rtpbin. ! rtph264depay ! avdec_h264 ! videoconvert ! \
autovideosink sync=false udpsrc port=5001 ! rtpbin.recv_rtcp_sink_0 \
rtpbin.send_rtcp_src_0 ! udpsink host=192.168.30.22 port=5005 sync=false async=false \
udpsrc port=5002 \
caps="application/x-rtp,media=audio,encoding-name=OPUS,clock-rate=48000,pt=97" ! \
rtpbin.recv_rtp_sink_1 rtpbin. ! rtpopusdepay ! opusdec ! audioconvert ! audioresample ! \
autoaudiosink sync=false udpsrc port=5003 ! rtpbin.recv_rtcp_sink_1 \
rtpbin.send_rtcp_src_1 ! udpsink host=192.168.30.22 port=5007 sync=false async=false
```