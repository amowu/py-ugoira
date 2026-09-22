# py-ugoira

Requires [Python 3.6.x](https://www.python.org/) and [FFmpeg](https://www.ffmpeg.org/).

```sh
$ python py_ugoira.py -h | fold -sw 80
usage: py_ugoira.py [-h] [--pixiv_id PIXIV_ID] [--frames_path FRAMES_PATH]
                    [--process {all,getframes,convertframes}]
                    [--video_output VIDEO_OUTPUT] [--interpolate]
                    [--ffmpeg_path FFMPEG_PATH] [--ffmpeg_args FFMPEG_ARGS]
                    [--cookie COOKIE] [-v]

Python script to download and convert an ugoira animation on Pixiv, and
convert it to a video via FFmpeg.

optional arguments:
  -h, --help            show this help message and exit
  --pixiv_id PIXIV_ID   The pixiv ID for the ugoira illustration. Required if
                        the --process argument is "all" or "getframes".
  --frames_path FRAMES_PATH
                        The path to where the image frames and ffconcat.txt
                        is. Required if the --process argument is
                        "convertframes".
  --process {all,getframes,convertframes}
                        The process that should take place. "all" will execute
                        both "getframes" and "convertframes". "getframes" will
                        only obtain the ugoira frames, and generate a FFmpeg
                        concat demuxer file. "convertframes" will only convert
                        the ugoira frames into a video type of your choice
                        through FFmpeg.
  --video_output VIDEO_OUTPUT
                        The output filename for the converted video. Defaults
                        to "output.webm".
  --interpolate         Attempts to interpolate the frames to 60 frames per
                        second. Note, it only works well with some ugoira, and
                        would take a longer time to finish conversion. Use
                        with care.
  --ffmpeg_path FFMPEG_PATH
                        The path to the FFmpeg executable.
  --ffmpeg_args FFMPEG_ARGS
                        The arguments for FFmpeg. Defaults to "-c:v libvpx
                        -crf 10 -b:v 2M -an", which is VP8 WEBM with a
                        variable bitrate of 2 MBit/s, with no audio.
  --cookie COOKIE       The cookie to send when fetching the ugoira data, e.g.
                        "PHPSESSID=12345_abcde". Required for ugoira that need
                        a login, such as R-18 works. A bare value without "="
                        is treated as the PHPSESSID.
  -v, --verbose         Forces the system to print out verbose process
                        messages.
```


### Example usage

```sh
# convert illustration ID 69689053 to webm
python py_ugoira.py --pixiv_id 69689053

# fetch the frames for illustration ID 69689053
python py_ugoira.py --pixiv_id 69689053 --process getframes

# convert the fetched frames earlier to mp4 with verbose info
python py_ugoira.py --frames_path ./ugoira_69689053 --process convertframes \
    --video_output cute_69689053.mp4 \
    --ffmpeg_path "C:\ffmpeg\ffmpeg.exe" \
    --ffmpeg_args "-c:v libx264 -profile:v baseline -pix_fmt yuv420p -an" \
    --verbose

# convert illustration ID 69689053 to mp4 (H.264)
python py_ugoira.py --pixiv_id 69689053 --video_output output.mp4 \
    --ffmpeg_args "-c:v libx264 -pix_fmt yuv420p -an"

# fetch a ugoira that requires login (e.g. R-18), using the PHPSESSID cookie
# from a logged-in browser session
python py_ugoira.py --pixiv_id 69689053 --cookie "PHPSESSID=12345_abcde"
```


### Output formats

The container is picked from the extension of `--video_output`, and the codec
from `--ffmpeg_args`. The default arguments encode VP8, which only fits in
`.webm` or `.mkv`, so changing the extension alone is not enough.

| Output | `--ffmpeg_args` |
|---|---|
| `.webm` (VP8, default) | `-c:v libvpx -crf 10 -b:v 2M -an` |
| `.mp4` (H.264) | `-c:v libx264 -pix_fmt yuv420p -an` |

`-pix_fmt yuv420p` is needed for the mp4 to play in browsers and QuickTime,
since the ugoira frames are usually decoded as `yuvj444p`.

Note that FFmpeg runs inside the frames folder, so a relative `--video_output`
ends up in `ugoira_<pixiv_id>/`. Pass an absolute path, such as
`"$PWD/output.mp4"`, to write it elsewhere.


### Login cookie

Some ugoira, such as R-18 works, can only be fetched while logged in. Without a
login, Pixiv responds with `HTTP Error 404: Not Found` for those works.

To fetch them, copy the `PHPSESSID` cookie from a browser that is logged in to
Pixiv (DevTools → Application → Cookies → `https://www.pixiv.net`), and pass it
with `--cookie`. Both `"PHPSESSID=<value>"` and the bare `<value>` are
accepted.

The cookie grants access to your Pixiv account, so do not commit it or share
it, and keep in mind that it is saved to your shell history.


### License

GPLv3
