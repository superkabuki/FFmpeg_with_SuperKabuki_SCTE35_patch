# 02/21/2026 Fresh new tarball built from latest FFmpeg repo.

### The patch _(adds about 14 lines )_
```diff
diff -ruN FFmpeg/libavformat/mpegts.c FFmpeg-skabuki/libavformat/mpegts.c
--- FFmpeg/libavformat/mpegts.c	2026-02-21 21:47:00.235535891 -0500
+++ FFmpeg-skabuki/libavformat/mpegts.c	2026-02-21 19:46:09.479014553 -0500
@@ -879,6 +879,7 @@
     { MKTAG('I', 'D', '3', ' '), AVMEDIA_TYPE_DATA,  AV_CODEC_ID_TIMED_ID3 },
     { MKTAG('V', 'C', '-', '1'), AVMEDIA_TYPE_VIDEO, AV_CODEC_ID_VC1   },
     { MKTAG('O', 'p', 'u', 's'), AVMEDIA_TYPE_AUDIO, AV_CODEC_ID_OPUS  },
+    { MKTAG('C', 'U', 'E', 'I'), AVMEDIA_TYPE_DATA,  AV_CODEC_ID_SCTE_35 },
     { 0 },
 };
 
diff -ruN FFmpeg/libavformat/mpegtsenc.c FFmpeg-skabuki/libavformat/mpegtsenc.c
--- FFmpeg/libavformat/mpegtsenc.c	2026-02-21 21:47:00.235535891 -0500
+++ FFmpeg-skabuki/libavformat/mpegtsenc.c	2026-02-21 21:37:29.317155489 -0500
@@ -444,6 +444,11 @@
             stream_type = STREAM_TYPE_PRIVATE_DATA;
         }
         break;
+    case AV_CODEC_ID_SCTE_35:
+        stream_type = STREAM_TYPE_SCTE_DATA_SCTE_35;
+        st->codecpar->codec_type = AVMEDIA_TYPE_DATA;
+        st->codecpar->codec_id   = AV_CODEC_ID_SCTE_35;
+        break;
     default:
         av_log_once(s, AV_LOG_WARNING, AV_LOG_DEBUG, &ts_st->data_st_warning,
                     "Stream %d, codec %s, is muxed as a private data stream "
@@ -497,6 +502,11 @@
     case AV_CODEC_ID_HDMV_TEXT_SUBTITLE:
         stream_type = STREAM_TYPE_BLURAY_SUBTITLE_TEXT;
         break;
+    case AV_CODEC_ID_SCTE_35:
+        stream_type = STREAM_TYPE_SCTE_DATA_SCTE_35;
+        st->codecpar->codec_type = AVMEDIA_TYPE_DATA;
+        st->codecpar->codec_id   = AV_CODEC_ID_SCTE_35;
+        break;
     default:
         av_log_once(s, AV_LOG_WARNING, AV_LOG_DEBUG, &ts_st->data_st_warning,
                     "Stream %d, codec %s, is muxed as a private data stream "
@@ -814,7 +824,9 @@
             }
             break;
         case AVMEDIA_TYPE_DATA:
-            if (codec_id == AV_CODEC_ID_SMPTE_KLV) {
+             if (codec_id == AV_CODEC_ID_SCTE_35) {
+                put_registration_descriptor(&q, MKTAG('C', 'U', 'E', 'I'));   
+             } else if (codec_id == AV_CODEC_ID_SMPTE_KLV) {
                 put_registration_descriptor(&q, MKTAG('K', 'L', 'V', 'A'));
             } else if (codec_id == AV_CODEC_ID_SMPTE_2038) {
                 put_registration_descriptor(&q, MKTAG('V', 'A', 'N', 'C'));


```


# SCTE35-ffmpeg-Kabuki

ffmpeg with the SuperKabuki SCTE-35 pass through patch applied.
# FFmpeg with the SuperKabuki SCTE-35 patch applied.


* The patch  allows you copy a SCTE-35 stream over as SCTE-35, when you're encoding with ffmpeg.
* The patch also adds the SCTE-35 Descriptor __(CUEI / 0x49455543)__ , just to be fancy.
* The patch adds only seven lines of code to two files, libavformat/mpegts.c and libavformat/mpegtsenc.c.
* Everything else works just like unpatched ffmpeg.


<img width="1068" height="541" alt="image" src="https://github.com/user-attachments/assets/6554cc61-18c5-4ece-8829-09c6d8f0980f" />

---


## Install  


__This is a tarball out of a stable ffmpeg build with the SuperKabuki SCTE35 patch applied__


1.    `git clone https://github.com/superkabuki/FFmpeg_SCTE35.git`

2.    `cd FFmpeg_SCTE35`
3.     tar -xvjf ffmpeg-superkabuki-02-21-2026.tbz


4.    `./configure  --enable-libfreetype --enable-gpl --enable-nonfree --enable-libx264 --enable-libx265 --enable-openssl --extra-version=-SuperKabuki-patch-02-21-2026 --cc=clang --enable-libsrt --enable-shared
` 
 
      you can customize configure as needed. <br>
      I use <br>`./configure --enable-libfreetype --enable-gpl --enable-nonfree --enable-libx264 --enable-libx265 --enable-openssl --extra-version=-SuperKabuki-patch-02-21-2026 --cc=clang --enable-libsrt --enable-shared
` 
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
