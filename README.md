# HTML5 MediaElement startOffsetTime Patches #

This repository contains patches we created to implement the `startOffsetTime` attribute of HTML 5 [`HTMLMediaElement`](http://www.whatwg.org/specs/web-apps/current-work/multipage/the-video-element.html#htmlmediaelement) objects in [Firefox](http://www.mozilla.org/projects/firefox/). The `startOffsetTime` value is implemented for [WebM](http://www.webmproject.org/) video only, using the [`DateUTC` field](http://matroska.org/technical/specs/index.html) in the Segment Information section of the WebM stream.

Note that this implementation differs slightly from the current HTML 5 specification, in that `startOffsetTime` returns a double and not a [`Date`](https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Date) object. The value contains the number of milliseconds since the Unix epoch, which is easily converted to a `Date` value in JavaScript:

    var date = new Date(videoElement.startOffsetTime);

The Firefox patch also adds some other attributes to the `HTMLMediaElement` that are not part of the HTML 5 specification, but we used during our investigation:

    // Absolute time reference of media stream, in seconds,
    // essentially currentTime + startOffsetTime
    readonly attribute double currentTimeAbsolute;

    // Time of browser joining the stream, relative to the
    // start of the stream, in microseconds
    readonly attribute double startTime;

    // The DateUTC value read from the Segment Information
    // section of the WebM stream, but converted to be milliseconds
    // relative to the Unix epoch
    readonly attribute double dateUTC;

There is also a patch for [FFmpeg](http://ffmpeg.org/) that sets the `DateUTC` value to the current system time when generating WebM output. `DateUTC` is calculated in the same way as the GStreamer Matroska plugin (refer to [matroska-mux.c](http://cgit.freedesktop.org/gstreamer/gst-plugins-good/tree/gst/matroska/matroska-mux.c?id=RELEASE-0.10.28) and [ebml-write.c](http://cgit.freedesktop.org/gstreamer/gst-plugins-good/tree/gst/matroska/ebml-write.c?id=RELEASE-0.10.28).

## Firefox ##

The file `firefox-8.patch` contains a patch for the Firefox 8 source code. The following instructions explain how to compile Firefox on Linux.

First, download and unpack the [Firefox source code](ftp://ftp.mozilla.org/pub/mozilla.org/firefox/releases/8.0/source/firefox-8.0.source.tar.bz2):

    $ mkdir firefox
    $ cd firefox
    $ curl -O ftp://ftp.mozilla.org/pub/mozilla.org/firefox/releases/8.0/source/firefox-8.0.source.tar.bz2
    $ tar jxvf firefox-8.0.source.tar.bz2

Apply the patch:

    $ cd mozilla-release
    $ patch -p2 < firefox-8.patch

Install Firefox's dependencies (refer to [this page](https://developer.mozilla.org/en/Linux_Build_Prerequisites) for details):

    $ sudo apt-get -y build-dep firefox
    $ sudo apt-get -y install mercurial libasound2-dev libcurl4-openssl-dev libnotify-dev libxt-dev libiw-dev mesa-common-dev autoconf2.13 yasm

Build:

    $ ./configure
    $ make

When the build is complete, you can find the patched binary in the `dist/bin` directory.

## FFmpeg ##

The file `ffmpeg.patch` contains a patch for FFmpeg 0.8.2.

Download and unpack the [ffmpeg source code](http://ffmpeg.org/download.html):

    $ mkdir ffmpeg
    $ cd ffmpeg
    $ curl -O http://ffmpeg.org/releases/ffmpeg-0.8.2.tar.bz2
    $ tar jxvf ffmpeg-0.8.2.tar.bz

Apply the patch:

    $ cd ffmpeg-0.8.2
    $ patch -p1 < ffmpeg.patch

Build:

    $ ./configure
    $ make

## License ##

These patches are released under the same licenses as the original projects: the Firefox patch is released under the [Mozilla Public License 1.1](http://www.mozilla.org/MPL/1.1/) and the FFmpeg patch is released under the [GNU Lesser General Public License (LGPL) version 2.1](http://www.gnu.org/licenses/old-licenses/lgpl-2.1.html).

## Author ##

This software was written by Chris Needham, chris.needham at bbc.co.uk.
