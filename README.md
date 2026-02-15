# SCTE35-ffmpeg-Kabuki

ffmpeg with the SuperKabuki SCTE-35 pass through patch applied.
# FFmpeg with the SuperKabuki SCTE-35 patch applied.


* The patch  allows you copy a SCTE-35 stream over as SCTE-35, when you're encoding with ffmpeg.
* The patch also adds the SCTE-35 Descriptor __(CUEI / 0x49455543)__ , just to be fancy.
* The patch adds only seven lines of code to two files, libavformat/mpegts.c and libavformat/mpegtsenc.c.
* Everything else works just like unpatched ffmpeg.


<img width="1068" height="541" alt="image" src="https://github.com/user-attachments/assets/6554cc61-18c5-4ece-8829-09c6d8f0980f" />

_(Of course it runs on OpenBSD and everything else ffmpeg supports)_
---


## Install  


__This is a tarball out of a stable ffmpeg build with the SuperKabuki SCTE35 patch applied__


1.    `git clone https://github.com/superkabuki/FFmpeg_SCTE35.git`

2.    `cd FFmpeg_SCTE35`
3.     tar -xvjf ffmpeg-scte35.tbz


4.    `./configure --enable-shared --extra-version=-SuperKabuki-patch` 
 
      you can customize configure as needed. <br>
      I use <br>`./configure --enable-gpl --enable-nonfree --enable-libx264 --enable-libx265 --extra-version=-SuperKabuki-patch`--enable-openssl

      There are a lot of ffmpeg configure options available. <br>
      __The superkabuki patch doesn't require any special configure options.__
        

5.    `make all` 

  On OpenBSD use `gmake` instead of `make`.

6.    `sudo make install` 



 
---

## How to use:

Use it just like unpatched FFmpeg.

---

# Examples

### 1.  Re-encode video to h.265, audio to aac, copy over the SCTE-35, and keep the timestamps.

* I build my ffmpeg with libx265 enabled. `--enable-libx265 --enable-nonfree`
```smalltalk
ffmpeg -copyts -i input.ts -map 0  -c:v libx265 -c:a aac -c:d copy -muxpreload 0 -muxdelay 0 output.ts
```

---


### 2. Copy all streams including SCTE-35, cut the first 200 seconds, and keep the timestamps.


```smalltalk
ffmpeg -copyts -ss 200 -i input.ts -map 0  -c copy -muxpreload 0 -muxdelay 0 output.ts
```

> Notice the start time and duration have both changed by ~200 seconds.

---

### 3. Dump binary SCTE-35 data to a file.
```smalltalk
ffmpeg -i input.ts -map 0:d -f data  -y output.bin
```
