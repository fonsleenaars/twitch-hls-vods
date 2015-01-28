# twitch-hls-vods
With the Twitch Video on Demand (vod) system being changed to the HLS format, the ability to retrieve and download past broadcasts isn't currently available through the documented API.

After some research and using Wireshark when loading a past broadcast page I've put together a small Java library/example that shows how this new system will get you the M3U8 files that are used by HLS that hold the vod.

## HLS / M3U8
I had never used HLS/M3U8 formats before I encountered this change, so I might be wrong on some things, just a note of warning!

The M3U8 extension (.m3u playlists with UTF-8 capabilities) basically holds a list of all the small segments of the vod.
The segments in the case of Twitch are between 0-4 seconds in length and the M3U8 index is the playlist that holds the meta information about these segments, and the link that it can be retrieved from.

Once obtained, you can write your own downloader/parser and do whatever you want with the information, the goal of this example is merely to demonstrate how you can get the link to the M3U8 file for a specific vod.

There's already a few examples of how to retrieve the information for a LIVE stream, one of which can be found at [http://johannesbader.ch/2014/01/find-video-url-of-twitch-tv-live-streams-or-past-broadcasts/](http://johannesbader.ch/2014/01/find-video-url-of-twitch-tv-live-streams-or-past-broadcasts/)
