# mts-converter

##Table of contents

  * [Table of contents](#table-of-contents)
  * [Fast start](#fast-start)
    * [Conversion to one file](#conversion-to-one-file)
    * [Conversion to several files](#conversion-to-several-files)


## Fast start


### Conversion to one file


For call 
```bash
bulk_video_converter.bash -c config.yaml /path/to/video/file/00001.MTS
```

with `config.yaml` like this
```yaml
ffmpeg:
  bin: /usr/bin/ffmpeg
  threads: 0
  start: 00:00:10
  stop: 00:00:30
profile:
  base:
    abstract: 1     # This profile will be used only for inheritance.
    output_dir_name: ./out          # Optional parametr.
    pass_log_dir_name: ./pass_log   # Optional parametr.
  my_profile:
    extends: base   # Other options are inherited from `base`.
    passes: 2
    video:
      preset: veryslow
      width: 1280
      height: 720
      bitrate: 2000k
      codec:
        name: h264
        weightp: 2
        bframes: 3
        opts: "keyint=96:min-keyint=96:no-scenecut"
        profile: high
        level: 3.1
    audio:
      codec:
        name: aac
      channels: 5.1
      bitrate: 320k
```

It generates output log like this:
```yaml
bulk_video_converter.bash:
  /home/user/Video/00001.MTS :
    profile my_profile:
      global input: -ss '00:00:10' -threads '0' 
      video: -preset 'veryslow' -b:v '2000k' -vf 'scale=1280:720' -codec:v 'libx264' -profile:v 'high' -level:v '3.1' -weightp '2' -bf '3' -x264opts 'keyint=96:min-keyint=96:no-scenecut' 
      audio: -b:a '320k' -ac '6' -strict 'experimental' -codec:a 'aac' 
      global output: -ss '00:00:10' -to '00:00:30' 
      passes:
        pass 1: /usr/bin/ffmpeg -ss '00:00:10' -threads '0' -i '/home/user/Video/00001.MTS' -preset 'veryslow' -b:v '2000k' -vf 'scale=1280:720' -codec:v 'libx264' -profile:v 'high' -level:v '3.1' -weightp '2' -bf '3' -x264opts 'keyint=96:min-keyint=96:no-scenecut' -pass 1 -passlogfile ./pass_log/00001-my_profile.mts -b:a '320k' -ac '6' -strict 'experimental' -codec:a 'aac' -ss '00:00:10' -to '00:00:30' -f 'mp4' -y '/dev/null' 2>&1 | tee /var/log/bulk_video_converter.bash/2016-05-02_04-09-08-036081777/00001-my_profile-1-mp4.ffmpeg.log 1>&2;
        pass 2: /usr/bin/ffmpeg -ss '00:00:10' -threads '0' -i '/home/user/Video/00001.MTS' -preset 'veryslow' -b:v '2000k' -vf 'scale=1280:720' -codec:v 'libx264' -profile:v 'high' -level:v '3.1' -weightp '2' -bf '3' -x264opts 'keyint=96:min-keyint=96:no-scenecut' -pass 2 -passlogfile ./pass_log/00001-my_profile.mts -b:a '320k' -ac '6' -strict 'experimental' -codec:a 'aac' -ss '00:00:10' -to '00:00:30' -f 'mp4' -y './out/00001-my_profile.mp4' 2>&1 | tee /var/log/bulk_video_converter.bash/2016-05-02_04-09-08-036081777/00001-my_profile-2-mp4.ffmpeg.log 1>&2;
      # passes done
    # profile my_profile done
  # /home/user/Video/00001.MTS done
# bulk_video_converter.bash done

```

And perform 2 sequential commands for `ffmpeg`.

* For the first pass:
```bash
/usr/bin/ffmpeg                                             \
    -ss '00:00:10'                                          \
    -threads '0'                                            \
    -i '/path/to/video/file/00001.MTS'                      \
    -preset 'veryslow'                                      \
    -b:v '2000k'                                            \
    -vf 'scale=1280:720'                                    \
    -codec:v 'libx264'                                      \
    -profile:v 'high'                                       \
    -level:v '3.1'                                          \
    -weightp '2'                                            \
    -bf '3'                                                 \
    -x264opts 'keyint=96:min-keyint=96:no-scenecut'         \
    -pass '1'                                               \
    -passlogfile './pass_log/00001-my_profile.mts'          \
    -b:a '320k'                                             \
    -ac '6'                                                 \
    -strict 'experimental'                                  \ 
    -codec:a 'aac'                                          \
    -ss '00:00:10'                                          \
    -to '00:00:30'                                          \
    -f 'mp4'                                                \
    -y '/dev/null';
```

* And for second one:

```bash
/usr/bin/ffmpeg                                             \
    -ss '00:00:10'                                          \
    -threads '0'                                            \
    -i '/path/to/video/file/00001.MTS'                      \
    -preset 'veryslow'                                      \
    -b:v '2000k'                                            \
    -vf 'scale=1280:720'                                    \
    -codec:v 'libx264'                                      \
    -profile:v 'high'                                       \
    -level:v '3.1'                                          \
    -weightp '2'                                            \
    -bf '3'                                                 \
    -x264opts 'keyint=96:min-keyint=96:no-scenecut'         \
    -pass '2'                                               \
    -passlogfile './pass_log/00001-my_profile.mts'          \
    -b:a '320k'                                             \
    -ac '6'                                                 \
    -strict 'experimental'                                  \ 
    -codec:a 'aac'                                          \
    -ss '00:00:10'                                          \
    -to '00:00:30'                                          \
    -f 'mp4'                                                \
    -y './out/00001-my_profile.mp4';
```


### Conversion to several files


If you want to get several files from one source
you shold describe several profiles in the config.

For example, let we use profiles from 
[H.264 web video encoding tutorial with FFmpeg.](https://www.virag.si/2012/01/web-video-encoding-tutorial-with-ffmpeg-0-9/)

```yaml
ffmpeg:
  bin: /usr/bin/ffmpeg
  threads: 0
  start: 00:00:10
  stop: 00:00:30
profile:
  base:
    abstract: 1 # It will be used only for inheritance.
    source: /home/user/Video/*.MTS  # also you can set file names here.
    output_dir_name: ./out
    pass_log_dir_name: ./pass_log
  virag_h264x1_pal_sd:                  # PAL SD video.
    extends:   base
    passes: 1
    video:
      width:  0   # any
      height: 576 # SD
      preset: slower
      codec:
        profile: main
      bitrate: 1000k
    audio:
      codec:
        name: aac
      channels: 5.1
      bitrate: 196k
  virag_h264x1_480p_web:                # Standard web video.
    extends: virag_h264x1_pal_sd
    video:
      height: 480
      preset: slow
      bitrate: 500k
      maxrate: 500k
      bufsize: 1000k
    audio:
      bitrate: 128k
      channels: stereo
  virag_h264x1_480p_tablet:             # Video for iPads and tablets.
    extends: virag_h264x1_480p_web
    video:
      codec:
        profile: main
      bitrate: 400k
      maxrate: 400k
      bufsize: 800k
  virag_h264x1_360p_mobile:             # 360p video for old phones.
    extends: virag_h264x1_480p_tablet
    video:
      height: 360
      codec:
        profile: baseline
      bitrate: 250k
      maxrate: 250k
      bufsize: 500k
    audio:
      bitrate: 96k
      channels: mono
```


You can check what you will get with dry-run option (`-d`).
So for virag's profiles you will have output with four ffmpeg-commands.
All profiles will be handled in parallel way.
It is same if you run ffmpeg four times at the same moment.

```bash
[user@host ~]$ ./bulk_video_converter.bash -c config.yaml -d
bulk_video_converter.bash:
# NOTICE 25:  bulk_video_converter.bash creates directory /tmp/bulk_video_converter.bash/01TaRYSd
  /home/user/Video/00001.MTS:
    profile virag_h264x1_360p_mobile:
      global input:
        -ss '00:00:10' -threads '0' 
      # global input done
      video:
        -preset 'slow' -b:v '250k' -maxrate '250k' -bufsize '500k' -vf 'scale=0:360' -codec:v 'libx264' -profile:v 'baseline' 
      # video done
      audio:
        -b:a '96k' -ac '1' -strict 'experimental' -codec:a 'aac' 
      # audio done
      global output:
        -ss '00:00:10' -to '00:00:30' 
      # global output done
      passes:
        pass 1:
          /usr/bin/ffmpeg -ss '00:00:10' -threads '0' -i '/home/user/Video/00001.MTS' -preset 'slow' -b:v '250k' -maxrate '250k' -bufsize '500k' -vf 'scale=0:360' -codec:v 'libx264' -profile:v 'baseline' -b:a '96k' -ac '1' -strict 'experimental' -codec:a 'aac' -ss '00:00:10' -to '00:00:30' -f 'mp4' -y './out/00001-virag_h264x1_360p_mobile.mp4' 2>&1 | tee /var/log/bulk_video_converter.bash/2016-05-02_05-14-20-695774334/00001-virag_h264x1_360p_mobile-1-mp4.ffmpeg.log 1>&2;
        # pass 1 done
      # passes done
    # profile virag_h264x1_360p_mobile done
  # /home/user/Video/00001.MTS done
  /home/user/Video/00001.MTS:
    profile virag_h264x1_480p_tablet:
      global input:
        -ss '00:00:10' -threads '0' 
      # global input done
      video:
        -preset 'slow' -b:v '400k' -maxrate '400k' -bufsize '800k' -vf 'scale=0:480' -codec:v 'libx264' -profile:v 'main' 
      # video done
      audio:
        -b:a '128k' -ac '2' -strict 'experimental' -codec:a 'aac' 
      # audio done
      global output:
        -ss '00:00:10' -to '00:00:30' 
      # global output done
      passes:
        pass 1:
          /usr/bin/ffmpeg -ss '00:00:10' -threads '0' -i '/home/user/Video/00001.MTS' -preset 'slow' -b:v '400k' -maxrate '400k' -bufsize '800k' -vf 'scale=0:480' -codec:v 'libx264' -profile:v 'main' -b:a '128k' -ac '2' -strict 'experimental' -codec:a 'aac' -ss '00:00:10' -to '00:00:30' -f 'mp4' -y './out/00001-virag_h264x1_480p_tablet.mp4' 2>&1 | tee /var/log/bulk_video_converter.bash/2016-05-02_05-14-20-695774334/00001-virag_h264x1_480p_tablet-1-mp4.ffmpeg.log 1>&2;
        # pass 1 done
      # passes done
    # profile virag_h264x1_480p_tablet done
  # /home/user/Video/00001.MTS done
  /home/user/Video/00001.MTS:
    profile virag_h264x1_480p_web:
      global input:
        -ss '00:00:10' -threads '0' 
      # global input done
      video:
        -preset 'slow' -b:v '500k' -maxrate '500k' -bufsize '1000k' -vf 'scale=0:480' -codec:v 'libx264' -profile:v 'main' 
      # video done
      audio:
        -b:a '128k' -ac '2' -strict 'experimental' -codec:a 'aac' 
      # audio done
      global output:
        -ss '00:00:10' -to '00:00:30' 
      # global output done
      passes:
        pass 1:
          /usr/bin/ffmpeg -ss '00:00:10' -threads '0' -i '/home/user/Video/00001.MTS' -preset 'slow' -b:v '500k' -maxrate '500k' -bufsize '1000k' -vf 'scale=0:480' -codec:v 'libx264' -profile:v 'main' -b:a '128k' -ac '2' -strict 'experimental' -codec:a 'aac' -ss '00:00:10' -to '00:00:30' -f 'mp4' -y './out/00001-virag_h264x1_480p_web.mp4' 2>&1 | tee /var/log/bulk_video_converter.bash/2016-05-02_05-14-20-695774334/00001-virag_h264x1_480p_web-1-mp4.ffmpeg.log 1>&2;
        # pass 1 done
      # passes done
    # profile virag_h264x1_480p_web done
  # /home/user/Video/00001.MTS done
  /home/user/Video/00001.MTS:
    profile virag_h264x1_pal_sd:
      global input:
        -ss '00:00:10' -threads '0' 
      # global input done
      video:
        -preset 'slower' -b:v '1000k' -vf 'scale=0:576' -codec:v 'libx264' -profile:v 'main' 
      # video done
      audio:
        -b:a '196k' -ac '6' -strict 'experimental' -codec:a 'aac' 
      # audio done
      global output:
        -ss '00:00:10' -to '00:00:30' 
      # global output done
      passes:
      # NOTICE 527:  bulk_video_converter.bash creates directory /var/log/bulk_video_converter.bash/2016-05-02_05-14-20-695774334
        pass 1:
          /usr/bin/ffmpeg -ss '00:00:10' -threads '0' -i '/home/user/Video/00001.MTS' -preset 'slower' -b:v '1000k' -vf 'scale=0:576' -codec:v 'libx264' -profile:v 'main' -b:a '196k' -ac '6' -strict 'experimental' -codec:a 'aac' -ss '00:00:10' -to '00:00:30' -f 'mp4' -y './out/00001-virag_h264x1_pal_sd.mp4' 2>&1 | tee /var/log/bulk_video_converter.bash/2016-05-02_05-14-20-695774334/00001-virag_h264x1_pal_sd-1-mp4.ffmpeg.log 1>&2;
        # pass 1 done
      # passes done
    # profile virag_h264x1_pal_sd done
  # /home/user/Video/00001.MTS done
# NOTICE 835:  bulk_video_converter.bash deletes directory /tmp/bulk_video_converter.bash
# bulk_video_converter.bash done
[user@host ~]$ 
```