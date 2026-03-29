---
layout: post
title: "microSD Card Comparison"
author: Felix
date: 2024-10-14
---

In the frame of my FormationFlight project I’m using an [OpenMV Cam H7](https://openmv.io/collections/retired-openmv-cams/products/openmv-cam-h7) with a [global shutter](https://openmv.io/collections/openmv-cam-camera-modules/products/global-shutter-camera-module) sensor module, running a script that continuously takes a snapshot, and performs blob finding. It also listens on a serial port for MAVLink message through which it can be commanded to start and stop recording data. When recording is active, each snapshot is stored as an mjpeg (or raw binary) frame of a video onto an SD card, along with binary data about the frame (such as exposure, blobs found, etc.).

When recording (i.e. writing to the SD card) the script had frequent drops in FPS, from just under 20 FPS to single-digit numbers. This would occur for a single frame, after which the frame rate would recover, only to drop again some time later. These drops seem to occur randomly and independently if the camera was plugged into USB or used stand-alone.

Considering the SD card was quite old, I decided to buy more modern examples and test them. The table below summarizes the samples and their specs (Card #1 is the original card).

|Card|Manufacturer|Capacity|Filesystem|Speed Ratings|Application Performance Class|Bitrate (claimed)|
|:--:|:---|:---|:---|:---|:---|:---|
|#1|Kingston|16 GB (SDHC)|FAT32|Speed Class 10,</br>UHS-I,</br>UHS Speed Class 1|n/a|up to 45 MB/s (read)|
|#2|Kingston|16 GB (SDHC)|FAT32|Speed Class 10,</br>UHS-I,</br>UHS Speed Class 3,</br>Video Speed Class 30|A1|up to 100 MB/s (read)|
|#3|SanDisk|32 GB (SDHC)|FAT32|Speed Class 10,</br>UHS-I,</br>UHS Speed Class 3,</br>Video Speed Class 30|A1|up to 100 MB/s (read),</br>up to 90 MB/s (write)|

Wikipedia has a good [article about SD cards](https://en.wikipedia.org/wiki/SD_card), explaining the different standards.

|Card #1|Card #2|Card #3|
|:---:|:---:|:---:|
|![Card #1](/assets/sdcard1_small.jpg)|![Card #2](/assets/sdcard2_small.jpg)|![Card #3](/assets/sdcard3_small.jpg)|

Example of FPS performance drops with the original card #1 (no `sync()`):
![Card #1 Performance](/assets/Kingston_16GB_45MBps_2.png)

Formatting card #1 with exFAT (apparently this could be better for smaller chunks of data), seemed to help a tiny bit, but not consistently. Performance was very random, but always with these frequent dips to single-digit FPS.
Using `mjpeg.sync()` on the video file every frame, every other, or every n-th frame (as recommended by the OpenMV micropython documentation) made the drops a bit less frequent, but they were still occurring too often.

Using cards #2 and #3 above completely fixed this issue. Frame rate is now consistently approx. 18..19 FPS (the maximum the H7 camera can do at VGA resolution running this script), only occasionally dropping a few FPS. And while the camera only occasionally hit the peak frame rate with card #1, this is now happening consistently. This applies for both the mjpeg module (i.e compressed) as well as the ImageIO writer where the raw frame buffer is written. The latter is however not fully understood yet as the data to be written is significantly larger, and thus needs further testing (*).

Example of FPS performance with card #2 (worst case):
![Card #2 Performance](/assets/Kingston_16GB_100MBps_1.png)

Example of FPS performance with card #3 (worst case). No drops in frame rate have been noticed.
![Card #3 Performance](/assets/SanDisk_32GB_100MBps_1.png)

*(*) Edit: I figured out why there is no difference between using the recorder in `mjpeg` or `ImageIO` mode. When storing a frame using `ImageIO`, `compress()` is called, converting the frame to a JPG still. As the file overhead for an AVI RIFF (mjpeg) and `ImageIO` is marginal, the resulting file size is roughly equivalent.*