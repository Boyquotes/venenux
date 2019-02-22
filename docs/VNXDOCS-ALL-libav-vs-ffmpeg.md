Libav and ffmpeg
===============

FFmpeg and Libav are libraries for multimedia decoding (and more). Both
libraries expose almost the same API and features, the last are more quite 
stable and clean, the firts are quite more extended and backguard compat but not clean API.

# Proyect artifacts

## Artifacs tools

(WIP)

avconv | ffmpeg: ffmpeg tool
avplay | ffplay: ffplay tool
avprove | ffprobe: ffprobe tool
avserver | N/A

## API interfaces and libraries

Both come as a set of libraries with quite same library names.
Both have equivalent libraries, and in few cases they are completely different, 
but the way to implement them is totally different between both. As a special note, 
the libav API are very clean an easy to implement but less features respect ffmpeg.

| project | Management    | API         | documentation |
| ------- | ------------- | ------------ | ------------------------ |
| libav   | participative | estable/clean | https://libav.org/documentation/doxygen/master/index.html |
| ffmpeg  | hierarchical  | dynamic/dirty | https://www.ffmpeg.org/doxygen/trunk/index.html |

#### file muxing/demuxing library

Libav defines as a library for dealing with various media container formats, while 
ffmpeg defines as generic framework for multiplexing and demultiplexing (muxing and demuxing) 
audio, video and subtitle streams. It encompasses multiple muxers and demuxers for multimedia container formats.
Both supports several input and output protocols to access a media resource. 

| project | artifac name | observations | documentation |
| ------- | ------------ | ------------ | ------------------------ |
| libav   | libavformat  | excelent docs | https://libav.org/documentation/doxygen/master/group__libavf.html |
| ffmpeg  | libavformat  | lack of docs | https://www.ffmpeg.org/doxygen/trunk/group__libavf.html |

#### general encoding/decoding library

Generic encoding/decoding framework and contains multiple decoders and encoders 
for audio, video and subtitle streams, and several bitstream filters.
Both are with sahred architechture that provides various services ranging from 
bit stream I/O to DSP optimizations.

| project | artifac name | observations | documentation |
| ------- | ------------ | ------------ | ------------------------ |
| libav   | libavcodec   | excelent docs | https://libav.org/documentation/doxygen/master/group__libavc.html |
| ffmpeg  | libavcodec   | lack of docs | https://www.ffmpeg.org/doxygen/trunk/group__libavc.html |

#### common/multimedia library

Libav defines as set of code shared across all the other Libav libraries, while 
ffmpeg defines as not a library for code needed by both libavcodec and libavformat.
This are a complete differents set of libraries, libav uses as a common piece, 
while in ffmpeg cases are a great set of shared portable multimedia programming interface, 

| project | artifac name | observations | documentation |
| ------- | ------------ | ------------ | ------------------------ |
| libav   | libavutil  | shared set of code library | https://libav.org/documentation/doxygen/master/group__lavu.html |
| ffmpeg  | libavutil  | portable multimedia programming | https://www.ffmpeg.org/doxygen/trunk/group__libavu.html |

#### editing frame/filter library

Libav defines as graph-based frame editing library while 
ffmpeg defines as generic audio/video filtering framework containing sources and sinks.
This library are the less used in both set.

| project | artifac name | observations | documentation |
| ------- | ------------ | ------------ | ------------------------ |
| libav   | libavfilter  | limited but very well done | https://libav.org/documentation/doxygen/master/group__lavfi.html |
| ffmpeg  | libavfilter  | great features how can use it? | https://www.ffmpeg.org/doxygen/trunk/group__libavfi.html |

#### device muxing/demuxing library

Libav defines as complementary library to libavformat, while 
ffmpeg defines as generic framework for grabbing from and rendering to devices.
Both have various "special" platform-specific muxers and demuxers for grabbing devices, 
audio capture and playback supporting several input and output devices, including Video4Linux2, VfW, DShow, and ALSA. 

| project | artifac name | observations | documentation |
| ------- | ------------ | ------------ | ------------------------ |
| libav   | libavdevice  | clean api, excelent docs | https://libav.org/documentation/doxygen/master/group__lavd.html |
| ffmpeg  | libavdevice  | same as libav,lack of docs | https://www.ffmpeg.org/doxygen/trunk/group__libavd.html |

#### audio resampling, format conversion and mixing

Libav defines as is a library that handles audio resampling, sample format conversion and mixing. 
ffmpeg defines as rematrixing and sample format conversion operations library.
Both are complety differents and done many operations specifics, the ffmpeg are 
more complete but less documented and implements may need a very close reading and 
asking to developers, while libav ones are quite easy to done and have good examples.

| project | artifac name | observations | documentation |
| ------- | ------------ | ------------ | ------------------------ |
| libav   | libavresample  | new lib, clean api, excelent docs | https://libav.org/documentation/doxygen/master/group__lavr.html |
| ffmpeg  | libswresample  | just a vulgar copy renamed but improved! | https://www.ffmpeg.org/doxygen/trunk/group__lswr.html |

#### color conversion and scaling library

The libswscale library performs highly optimized image scaling and colorspace and 
pixel format conversion operations. For both is usually a lossy process when the 
source and destination generally differ in colospace and audio/video channels/sizes. 

| project | artifac name | observations | documentation |
| ------- | ------------ | ------------ | ------------------------ |
| libav   | libswscale  | good documented but minimal | https://libav.org/documentation/doxygen/master/group__libsws.html |
| ffmpeg  | libswscale  | no documented but featured! | https://www.ffmpeg.org/doxygen/trunk/group__libsws.html |

#### post procesing library

Video postprocessing library. Libav does not have a specific library for this

| project | artifac name | observations | documentation |
| ------- | ----------- | ------------ | ------------------------ |
| libav   |   -         | there's no equivalent | -  |
| ffmpeg  | libpostproc | no documented: how we used? | https://www.ffmpeg.org/doxygen/trunk/group__lpp.html |



FFmpeg and Libav history
========================

In 2011, parts of the FFmpeg developers were unhappy about the FFmpeg
leadership, and decided to take over. This didn't quite work out. Apparently
Fabrice Bellard, original FFmpeg developer and owner of the ffmpeg.org
domain name, decided not to hand over the domain name to the _new_
maintainers. So they followed Plan B, and forked FFmpeg, resulting in Libav.


**The reason for the fork is most likely a limited participative and organized 
infraestructure, with a dictatorship way of management of the original project**.
As a note, there is little to no cooperation between the two projects, but the 
firts one (ffmpeg) already copies, fork and includes all that libav does already 
in ther one. So after the situation, ffmpeg project changes a lot.
Since then, Libav did its own development, and completely ignored whatever
FFmpeg did. FFmpeg, on the other hand, started to merge literally everything
Libav did.

Situation today
===============

FFmpeg has more features and slightly more active development than Libav,
going by mailing list and commit volume. In particular, FFmpeg's features are
a superset of Libav's features. This is because **FFmpeg merges Libav's git
master on a daily basis.** Libav on the other hand seems to prefer to ignore
FFmpeg development (with occasional cherry-picking of bug fixes and features).

Some Linux distributions, especially those that had Libav developers as FFmpeg
package maintainers, replaced FFmpeg with Libav, while other
distributions stick with FFmpeg. Application developers typically have to
make sure their code works with both libraries. This can be trivial to hard,
depending on the details. One larger problem is that the difference between
the libraries makes it hard to keep up a consistent level of the user experience,
since either library might silently or blatantly be not up to the task. It
also encourages library users to implement some features themselves, rather
than dealing with the library differences, or the question to which project
to contribute.

Deep notes comparison beetween
===================================

This is pretty superficial, and should give only a general impression over
differences. Also, this might be biased by the personal views of the
author of this text.

FFmpeg disadvantages / Libav advantages
---------------------------------------
- the insane amount and volume of merging from a codebase that has been
  diverging for several years most likely has some negative effects on
  FFmpeg
- FFmpeg is relatively paranoid about being compatible to Libav, which adds
  bloat, and also allows Libav to dictate the API, somewhat stifling FFmpeg
  development and adding stupid artifacts to the FFmpeg ABI
- Libav is going somewhat clean directions in API development (although
  everything from this ends up in FFmpeg anyway)
- FFmpeg seems to have an "anything goes" attitude, and merges/accepts just
  about anything, sometimes only for the purpose to have it before Libav. (E.g. see
  HEVC decoder: developed mainly on the Libav side, though separate from the
  Libav repository, it was hastily merged by FFmpeg shortly before Libav was
  done with the merge that was prepared for months.)
  (See also VP9 incident.)
- FFmpeg never cleans up anything. The shit keeps piling up.
- Libav does major work cleaning up the API and internals.

FFmpeg advantages / Libav disadvantages
---------------------------------------
- merges everything from Libav, so almost all Libav features and bug fixes are
  part of FFmpeg
- seems to be more popular (more patches, more activity on the mailing lists
  and the bug tracker)
- FFmpeg definitely has more features and more complete implementations of at
  least some file formats
- Libav has very long release cycles, which force us to be backwards compatible
  with very old code. Libav also likes to miss Debian/Ubuntu deadlines.
- Sometimes FFmpeg adds APIs we want to use, and Libav doesn't have them. This
  causes us pain, and the blame goes to Libav for not providing them.
- Really messy backporting of major FFmpeg features. (E.g. see VP9
  decoder: after it was developed and merged in FFmpeg, Libav picked
  up the VP9 patches, made their own changes, both functional and
  cosmetic, squashed it into one commit. The original authors of the
  decoder had to figure out what was changed and possibly improved,
  and what was purely cosmetic.)
  (See also HEVC incident.)

External Reading
================

Take in consideration that the present in wikipedia are not objetive in fact: http://en.wikipedia.org/wiki/FFmpeg#History
