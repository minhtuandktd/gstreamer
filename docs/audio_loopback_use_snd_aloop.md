# Petalinux config

```bash
petalinux-config -c kernel
```
```bash
│ Symbol: SND_ALOOP [=n]                                                               
  │ Type  : tristate                                                                                                                                                            │
  │ Defined at sound/drivers/Kconfig:92                                                                                                                                         │
  │   Prompt: Generic loopback driver (PCM)                                                                                                                                     │
  │   Depends on: SOUND [=y] && !UML && SND [=y] && SND_DRIVERS [=n]                                                                                                            │
  │   Location:                                                                                                                                                                 │
  │     -> Device Drivers                                                                                                                                                       │
  │       -> Sound card support (SOUND [=y])                                                                                                                                    │
  │         -> Advanced Linux Sound Architecture (SND [=y])                                                                                                                     │
  │ (1)       -> Generic sound devices (SND_DRIVERS [=n])                                                                                                                       │
  │ Selects: SND_PCM [=y] && SND_TIMER [=y]  
```

# Check module loaded on linux:
```bash
lsmod | grep snd_aloop
```
# Check module exists in kernel:
```bash
modinfo snd_aloop
```
# 1. Insmod snd_aloop

Insmod snd_aloop
```bash
sudo modprobe snd_aloop
```

List device audio: 
```bash
arecord -l
```

Send audio file to loopback interface:
```bash
aplay -D hw:Loopback,0,0 audio_test.wav
```

Loop bash aplay_loop.sh:
```bash
#!/bin/bash
while true; do
    aplay -D hw:Loopback,0,0 /tmp/audio_test.wav
done
```

Run bash aplay_loop.sh with nohup (daemon):
```bash
nohup ./aplay_loop.sh > ./aplay_loop.log 2>&1 &
```

On sender send audio from loopback interface without encode:
```bash
gst-launch-1.0 alsasrc device=hw:Loopback,1,2 ! audioconvert ! audioresample ! \
audio/x-raw,rate=44100,channels=2,format=S16LE ! udpsink host=192.168.30.7 port=5000
```

On Receiver:
```bash
gst-launch-1.0 udpsrc port=5000 \
caps="audio/x-raw,rate=44100,channels=2,format=S16LE,layout=interleaved" ! \
audioconvert ! audioresample ! autoaudiosink
```

Send audio + video: audio from loopback, video from camera on PC to autosink (not through network) 
```bash
gst-launch-1.0 v4l2src device=/dev/video0 ! "video/x-raw,format=YUY2,width=1280,height=720,framerate=10/1" ! \
 videoconvert ! queue ! x264enc tune=zerolatency bitrate=2000 speed-preset=ultrafast ! h264parse ! avdec_h264 ! \
 videoconvert ! autovideosink sync=false alsasrc device=hw:Loopback,1,0 ! queue ! audioconvert ! audioresample ! autoaudiosink sync=false
```