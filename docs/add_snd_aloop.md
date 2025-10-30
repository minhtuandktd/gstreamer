# 1. Insmod snd_aloop

Insmod snd_aloop
```bash
sudo modprobe snd_aloop
```

List device audio: 
```bash
arecord -l
```

Stream audio file to interface loopback:
```bash
aplay -D hw:Loopback,0,0 audio_test.wav
```

Stream audio from loopback to speaker
```bash
gst-launch-1.0 alsasrc device=hw:Loopback,1,0 ! audioconvert ! audioresample ! autoaudiosink
```

Stream audio from loopback + video from camera
```bash
gst-launch-1.0 v4l2src device=/dev/video0 ! "video/x-raw,format=YUY2,width=1280,height=720,framerate=10/1" ! \
 videoconvert ! queue ! x264enc tune=zerolatency bitrate=2000 speed-preset=ultrafast ! h264parse ! avdec_h264 ! \
 videoconvert ! autovideosink sync=false alsasrc device=hw:Loopback,1,0 ! queue ! audioconvert ! audioresample ! autoaudiosink sync=false
```