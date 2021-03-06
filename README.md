# QrssPiG

[![License: GPL v3](https://img.shields.io/badge/License-GPL%20v3-blue.svg)](http://www.gnu.org/licenses/gpl-3.0)
[![Build Status](https://travis-ci.org/MartinHerren/QrssPiG.svg?branch=master)](https://travis-ci.org/MartinHerren/QrssPiG)

QrssPiG is short for QRSS (Raspberry)Pi Grabber.

Haven't found a headless standalone grabber for my Pi. So I try to create my own.
If i find one working for my needs I might well using the existing one and stop this.

## Functionality
 - Headless standalone daemon
 - Able to process I/Q stream from an rtl-sdr, HackRF or other sdr devices
 - Optionally control the sdr device
 - Optionally process audio from stream or audio input
 - Generate pretty horizontal or vertical waterfall graphs
 - Upload them via scp
 - Ftp upload is not planned, maybe ftps

## Build
To build QrssPiG you need cmake

It depends on following dev libs:
 - libboost-program-options-dev
 - libgd-dev
 - libssh-dev
 - libfftw3-dev
 - libyaml-cpp-dev
 - libfreetype6-dev
 - (librtlsdr-dev)

To build:
```
$ mkdir build
$ cd build
$ cmake ..
$ make
```

## Run
### Piping from audio device using alsa including resampling and remixing using sox
You need arecord and sox to be installed
```
arecord -q -t raw -f S16_LE -r 48000 | sox -t s16 -c 1 -r 48000 - -t s16 -c 2 -r 6000 - remix 1 0 | ./src/QrssPiG -c qrss.yaml
```
arecord is used to record from the standard audio input as raw audio data without header in signed 16 bit little-endian format with 48kHz samplerate. Output is sent to standard out.
sox is used to resample audio from 48kHz to 6kHz to lower processing and create a stereo signal with the audio data on the left channel and the right one silent.
In the qrss.yaml config file you must set the sample rate to 6000 and the format to s16iq like in the following example:
```
input:
  format: s16iq
  samplerate: 6000
  basefreq: 10138500
processing:
  fft: 16384
  fftoverlap: 3
output:
  orientation: horizontal
  minutesperframe: 10
  freqmin: 300
  freqmax: 2700
  dBmin: -30
  dBmax: 0
upload:
  type: local
```

### Piping from rtl_sdr through sox for resampling
You need rtl_sdr and sox installed. From your build directory
```
rtl_sdr -f 27999300 -s 1000000 - | sox -t u8 -c 2 -r 1000000 - -t u8 -c 2 -r 6000 - | ./src/QrssPiG -c qrss.yaml
```
rtl_sdr is used as receiver and produces unsigned 8 bit I/Q data at a samplerate of 1MSample/s from 27999300Hz and send them to stdout.
The frequency is choosen so that with a frequency accurate receiver qrss data should appear around 1.5kHz which will be in the middle of the plot.
sox is used to resample the data from 1MSample/s to 6kSample/s.
In the qrss.yaml config file you must set the sample rate to 6000 and the format to rtlsdr (or u8iq) like in the following example:
```
input:
  format: rtlsdr
  samplerate: 6000
  basefreq: 27999300
processing:
  fft: 16384
  fftoverlap: 3
output:
  orientation: horizontal
  minutesperframe: 10
  freqmin: 300
  freqmax: 2700
  dBmin: -40
  dBmax: -15
upload:
  type: local
```

### Piping from hackrf through sox for resampling
You need a recent version of hackrf_transfer (first version couldn't stream to stdout) and sox installed. From your build directory
```
hackrf_transfer -f 10138500 -s 4000000 -b 1.75 -a 1 -g 60 -l 24 -r - | sox -t s8 -c 2 -r 4000000 - -t s8 -c 2 -r 6000 - | ./src/QrssPiG -c qrss.yaml
```
hackrf_transfer is used as receiver and produces signed 8 bit I/Q data at a samplerate of 4MSample/s from 10138500Hz and send them to stdout.
The frequency is choosen so that with a frequency accurate receiver qrss data should appear around 1.5kHz which will be in the middle of the plot.
sox is used to resample the data from 4MSample/s to 6kSample/s.
In the qrss.yaml config file you must set the sample rate to 6000 and the format to hackrf (or s8iq) like in the following example:
```
input:
  format: hackrf
  samplerate: 6000
  basefreq: 10138500
processing:
  fft: 16384
  fftoverlap: 3
output:
  orientation: horizontal
  minutesperframe: 10
  freqmin: 300
  freqmax: 2700
  dBmin: -40
  dBmax: -30
upload:
  type: local
```
