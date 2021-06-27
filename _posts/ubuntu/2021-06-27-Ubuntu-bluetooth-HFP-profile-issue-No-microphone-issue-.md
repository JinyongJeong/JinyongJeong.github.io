---
layout: post
title: 'Ubuntu bluetooth HFP profile issue (No microphone issue)'
tags: [ubuntu]
comments: true
description: >
  Ubuntu bluetooth HFP profile issue (No microphone issue)
sitemap :
  changefreq: weekly
  priority : 1.0
---
I am using Ubuntu 18.04 on two laptop (dell XPS9700, Lenovo X1 carbon 7th). In both laptop, I was not able to use my Bluetooth earphone for online meeting because I can't find any micro phone. I tested with galaxy buds+, galaxy buds pro, liberty air 2 pro. When I tested with MDR-1RBT Bluetooth headset of Sony, I can use microphone without any issue. The reason of this issue is that default pulseAudio of ubuntu is not support HFP profile of bluetooth because any reason (reason is not sure). MDR-1RBT support HSP/HFP profile, but galaxy buds, and liberty air 2 pro don't support HSP only support HFP profile. 

The solution of this issue is replacing pulse audio to Pipewire

1. Add PPA (tested in ubuntu 18.04, 20.04)

```jsx
sudo add-apt-repository ppa:pipewire-debian/pipewire-upstream
```

2. Update packages

```jsx
sudo apt update
```

3. Install the packges

```jsx
sudo apt install pipewire
```

4. Install dependency. If you don't install this dependency, you will face the issue of "Bluetooth headset won't connect after installing pipewire"

```jsx
sudo apt install libspa-0.2-bluetooth
```

5. Install client library

```jsx
sudo apt install pipewire-audio-client-libraries
```

6. Reload daemon

```jsx
systemctl --user daemon-reload
```

7. Disable pulseAudio

```jsx
systemctl --user --now disable pulseaudio.service pulseaudio.socket
```

8. If you are on ubuntu 20.04, you also need to "mask" the pulseAudio using follow

```jsx
systemctl --user mask pulseaudio
```

When I used this command in Ubuntu 18.04, there was no issue. So you can try to run this command on other version too.

9. Enable pipewire service

```jsx
systemctl --user --now enable pipewire-media-session.service
```

10. You can ensure that Pipewire is not running through

```jsx
pactl info
```

This command will give the following output, in server name

```jsx
PulseAudio (on PipeWire 0.3.28)
```

If you don't show up, then try restart pipewire by this command.

```jsx
systemctl --user restart pipewire
```

If you can't see this server name, just try to connect your bluetooth earbuds to test it.

11. If you have installed ofono, ofono-phonesim, remove them

```jsx
sudo apt remove ofono
sudo apt remove ofono-phonesim
```

If it’s still not showing your microphone, you can try rebooting once and remove and pair your Bluetooth device again to check if it works now.

If you want to rollback all changes we  did, do this

```jsx
systemctl --user unmask pulseaudio
systemctl --user --now enable pulseaudio.service pulseaudio.socket
```

reference: 

[how to use bluetooth device with HSP/HFP profile using pulseaudio >=6 and bluez >= 5.24](https://unix.stackexchange.com/questions/341776/how-to-use-bluetooth-device-with-hsp-hfp-profile-using-pulseaudio-6-and-bluez)