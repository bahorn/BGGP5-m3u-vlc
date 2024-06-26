# M3U for VLC

## TLDR

* .m3us as VLC supports them can just be a list of URLs, which VLC will send
  requests to fetch.
* When opened they will add their contents to the active playlist, with the
  filename being the name that shows up in the playlist.
* VLC has an incredibly insecure http interface, which allows you to just send
  GET requests (with the requirement of HTTP auth) to access its commands.
* The interface lets you use VLM commands, which lets you script VLC to create
  streams.
* It is possible to use run a few VLM commands to download a file via HTTPS to
  disk, and also open this file as well.
* So if you download the binary.golf/5/5, save it as a m3u and open it, the
  contents will be displayed!
