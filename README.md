# MediaTools
#### an assortment of BASH scripts to automate media-related tasks
#### TODO:
  - [ ] maybe add some YAD interfaces?
  - [ ] general cleanup
## footdump
#### a tool for dumping footage
`footdump` is a CLI-driven tool for transferring media files from recording devices.
```
Usage:
  footdump --job <JOBCODE> --from <FROM> --to <TO>

  -j|--job        <Unique job code for project>
  -f|--from       <Directory or device to copy media FROM>
  -t|--to         <Directory to copy media TO>
  -T|--toplevel   Copies all files to the top level of the "from" directory
                  rather than maintaining the existing directory structure

Example:
  footdump -j MYPROJ -f /media/user/camera/ -t /media/user/dumped_footage/

"footdump" with no arguments runs in interactive mode
```
Example of interactive use:
```
$ footdump
   ____         _____
  | __ \       /    /|
  ||  || ===> /____/ /
  ||__||      |____|/

  ===|Footage Dumper|===

  Job Code: EXAMPLE

  NAME LABEL        MOUNTPOINT                SIZE TYPE
  sda1              /                       931.5G part
  sdb1 FootageDrive /media/user/footage     931.5G part
  sdc1 EOS_DIGITAL  /media/user/EOS_DIGITAL 127.4G part

  From: /media/user/EOS_DIGITAL
  To: /media/user/footage/example
  (1/1) "MVI_0000.MOV" --> "EXAMPLE_2020-01-01_12-00-00_0000.MOV"
```
`footdump` can accept a `FROM` location in the form of:
 - a partition, mounted or not, e.g. `/dev/sdc1` or just `sdc1`
 - an existing directory, such as a pre-mounted device, e.g. `/media/user/EOS_DIGITAL`

Media files copied via `footdump` will be automatically renamed as follows:<br>
`<job code>_<year-month-day>_<hour-minute-second>_<file number>.<file type>`<br>
For example, the original file `MVI_0000.MOV` recorded on January 1, 2020 at 12:00 and given a job code of `EXAMPLE` would be renamed to `EXAMPLE_2020-01-01_12-00-00_0000.MOV`.<br><br>
If available, `footdump` will make use of `pv` to copy files with a progress bar.  Otherwise, it will fall back to `cp`.<br><br>
By default, all media files will retain their directory structure from the removable media, e.g.<br>`/media/user/EOS_DIGITAL/DCIM/100CANON/MVI_0000.MOV` will be copied to `/media/user/footage/example/DCIM/100CANON/EXAMPLE_2020-01-01_12-00-00_0000.MOV`.<br>To instead copy all media to a single directory, use the `-T` or `--toplevel` flag.  With this enabled,<br> `/media/user/EOS_DIGITAL/DCIM/100CANON/MVI_0000.MOV` will now be copied to<br>`/media/user/footage/example/EXAMPLE_2020-01-01_12-00-00_0000.MOV`.<br><br>
Following the transfer, a log file will be left in the `TO` directory. `footdump` will NOT remove any original files; this will need to be done separately.<br><br>
To ensure accurate date/time information, make sure it is set properly on all recording devices prior to using them.

## mkproxy
#### a tool for batch generating video proxies/dailies with timecode
`mkproxy` uses `ffmpeg` to transcode video to either H.264 (default) or ProResProxy and the `drawtext` video filter to generate either drop-frame (default) or non-drop-frame timecode. Any audio streams present in the original video files will be copied to the new proxy files without transcoding.
```
Usage:
  mkproxy  <options>  /path/to/originals  /path/to/proxies

Options:
  h|help|-h|-help|--help
  tcpos|-tcpos|--timecode-position  <l|left|c|center|r|right>
  tcrate|-tcrate|--timecode-rate  <framerate>
  res|-res|--resolution  <width>x<height>
  f|-f|--format <h264|prores>
  df|-df|--dropframe
  ndf|-ndf|--nondropframe

Example:
  mkproxy -tcpos left -tcrate 29.97 -res 854x480 footage/original footage/proxy

"mkproxy" with no directory paths runs in interactive mode

Default values are
  --timecode-position left
  --timecode-rate 29.97
  --resolution 854x480
  --format h264
  --dropframe
```
Example of interactive use:
```
$ mkproxy
  Path to originals: /media/user/footage/original
  Path to proxies: /media/user/footage/proxy
  Generating proxies from "/media/user/footage/original" to "/media/user/footage/proxy" ...

  EXAMPLE_2020-01-01_12-00-00_0000.MOV
    Separating audio ...
    Processing video ...
    Concatenating audio and video ...
    Removing temp files ...  
```
#### TODO:
  - [ ] make framerate default to original media's, not `29.97`
  - [ ] add LUT support
  - [ ] expand timecode functionality
  - [ ] add flag to bypass timecode
## lufsnorm
#### a tool for normalizing audio based on LUFS
`lufsnorm` is a simple frontend for `ffmpeg`'s `loudnorm` audio filter. More information can be found [here](https://k.ylo.ph/2016/04/04/loudnorm.html). It may be superfluous alongside digital audio workstations that support loudness normalization, such as Ardour 5.
```
Usage:
  lufsnorm  <options>  <files to process>

Options:
  -I|--loudness <value>    Defaults to -16 LUFS
  -LRA|--range <value>     Defaults to 11 LU
  -TP|--truepeak <value>   Defaults to -1.5 dBTP
  -ow|--overwrite          Overwrite existing output files
  -n|--dryrun              Scans, but doesn't process, input files

Example:
  lufsnorm -I -16 -LRA 11 -TP -1.5 audio-1.wav audio-2.wav audio-n.wav
```
The output file of `lufsnorm` will inherit the input filename with the loudness value appended before the file extension, e.g. `example-audio.wav` will output `example-audio-16LUFS.wav` by default, or `example-audio-22LUFS.wav` if given the option `-I -22`.  There will also be an accompanying log file, `example-audio-16LUFS.wav.txt` or `example-audio-22LUFS.wav.txt`, respectively.
#### TODO:
  - [ ] get sample rate from input file, don't always output 192k (*.wav)
  - [ ] support appropriate non-WAV filetypes
  - [ ] support video files
