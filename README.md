# twitch-hls-vods
With the Twitch Video on Demand (vod) system being changed to the HLS format, the ability to retrieve and download past broadcasts isn't currently available through the documented API.

After some research and using Wireshark when loading a past broadcast page I've put together a small Java library/example that shows how this new system will get you the M3U8 files that are used by HLS that hold the vod.

## HLS / M3U8 / VOD
I had never used HLS/M3U8 formats before I encountered this change, so I might be wrong on some things, just a note of warning!

First note: the new vods can be recognised by their 'v' prefix in the ID: v123456. The old system used 'a' for past broadcasts and 'c' for highlights.

The M3U8 extension (.m3u playlists with UTF-8 capabilities) basically holds a list of all the small segments of the vod.
The segments in the case of Twitch are between 0-4 seconds in length and the M3U8 index is the playlist that holds the meta information about these segments, and the link that it can be retrieved from.

Once obtained, you can write your own downloader/parser and do whatever you want with the information, the goal of this example is merely to demonstrate how you can get the link to the M3U8 file for a specific vod.

There's already a few examples of how to retrieve the information for a LIVE stream, one of which can be found at [http://johannesbader.ch/2014/01/find-video-url-of-twitch-tv-live-streams-or-past-broadcasts/](http://johannesbader.ch/2014/01/find-video-url-of-twitch-tv-live-streams-or-past-broadcasts/)

## API Calls
As I'm working on the Java example I can outline the calls that should be made in order to get the M3U8 file for your desired HLS vod. These are the steps I currently take

**Retrieve an access token for a specific HLS VOD**  
[https://api.twitch.tv/api/vods/{vodId}/access_token](https://api.twitch.tv/api/vods/{vodId}/access_token)

The {vodId} should be the ID without the prefix letter, so 'v1234567' becomes '1234567'

Result
```
{
"token": "{\"user_id\":xxxxxxxx,\"vod_id\":xxxxxxx,\"expires\":xxxxxxxxxx,\"chansub\":{\"restricted_bitrates\":[]},\"privi leged\":false}",
"sig":"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
}
```
The `token` and `sig` values can be put straight into the next API call.

**Retrieve M3U8 file containing the qualities and respective links**  
[https://usher.twitch.tv/vod/{vodId}?nauthsig={sig}&nauth={token}](https://usher.twitch.tv/vod/{vodId}?nauthsig={sig}&nauth={token})

Where the `{vodId}`, `{sig}` and `{token}` are the vod id and the exact values you obtained from the previous API call.

Result
A file looking something like
```
#EXTM3U
#EXT-X-MEDIA:TYPE=VIDEO,GROUP-ID="chunked",NAME="Source",AUTOSELECT=YES,DEFAULT=YES
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=3752371,CODECS="avc1.4D4029,mp4a.40.2",VIDEO="chunked"
http://vod.ak.hls.ttvnw.net/v1/AUTH_system/vods_xxxx/{channel}_xxxxxxxxxxx_xxxxxxxxx/chunked/index-dvr.m3u8
#EXT-X-MEDIA:TYPE=VIDEO,GROUP-ID="high",NAME="High",AUTOSELECT=YES,DEFAULT=YES
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=1616504,CODECS="avc1.42C01F,mp4a.40.2",VIDEO="high"
http://vod.ak.hls.ttvnw.net/v1/AUTH_system/vods_xxxx/{channel}_xxxxxxxxxxx_xxxxxxxxx/high/index-dvr.m3u8
#EXT-X-MEDIA:TYPE=VIDEO,GROUP-ID="medium",NAME="Medium",AUTOSELECT=YES,DEFAULT=YES
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=917811,CODECS="avc1.42C01E,mp4a.40.2",VIDEO="medium"
http://vod.ak.hls.ttvnw.net/v1/AUTH_system/vods_xxxx/{channel}_xxxxxxxxxxx_xxxxxxxxx/medium/index-dvr.m3u8
#EXT-X-MEDIA:TYPE=VIDEO,GROUP-ID="low",NAME="Low",AUTOSELECT=YES,DEFAULT=YES
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=656905,CODECS="avc1.42C01E,mp4a.40.2",VIDEO="low"
http://vod.ak.hls.ttvnw.net/v1/AUTH_system/vods_xxxx/{channel}_xxxxxxxxxxx_xxxxxxxxx/low/index-dvr.m3u8
#EXT-X-MEDIA:TYPE=VIDEO,GROUP-ID="mobile",NAME="Mobile",AUTOSELECT=YES,DEFAULT=YES
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=300036,CODECS="avc1.42C00D,mp4a.40.2",VIDEO="mobile"
http://vod.ak.hls.ttvnw.net/v1/AUTH_system/vods_xxxx/{channel}_xxxxxxxxxxx_xxxxxxxxx//mobile/index-dvr.m3u8
```

You can write your own parser for this file and the index-dvr.m3u8 will contain the actual links to the segments!

## FFMPEG
It's worth noting that the FFMPEG command line program is able to take a M3U8 file as a value for its input (-i) parameter, which will automatically download and concatenate the pieces together if, for example, run like this

`ffmpeg -i "input.m3u8" -bsf:a aac_adtstoasc -c copy output.mp4`

However, the links to the segments in the input.m3u8 file must be absolute paths (not just `index-xxxxxxxxxx-xxxx.ts?start_offset=xxxxxxx&end_offset=xxxxxxx` like you would obtain from the API calls.)

A solution for this is to parse the index.m3u8 yourself, add the prefix, save it to a file and then run FFMPEG on it.

