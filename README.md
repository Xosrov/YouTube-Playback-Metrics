# YouTube-Playback-Metrics

YouTube communicates with its server in 3 important ways before, during and after playback.
## Loading and Buffering the Video

YouTube buffers the video even before the user watches it. The video is loaded in chunks with the `application/vnd.yt-ump` content type and from the `/videoplayback` URL. 
Post parameters and headers have not been inspected for these requests.
## Sending Watch Time Metrics

YouTube sends watch time metrics gradually during playback. This data seems to include additional information that could be inferred from the video ID already, signaling that each request made to the metric handling servers are processed completely independently of each other and of other processes at YouTube.

**Request headers have not been inspected**, but request parameters have given a lot of valuable information into what metrics YouTube sends and how often it does so. This information is still subject to change since testing done was limited and actual javascript code (mainly `base.js`) have not been inspected, and all parameters below are guessed based on experimentation.
Requests are of method GET and sent to `/api/stats/watchtime`. A typical request has parameters that look like this:  
"ns=yt&el=detailpage&cpn=\[REDACTED\]&ver=2&cmt=6.642&fmt=248&fs=0&rt=453.007&euri&lact=2318&cl=\[REDACTED\]&state=playing&volume=50%2C50&cbr=\[REDACTED\]&cbrver=\[REDACTED\]&c=\[REDACTED\]&cver=\[REDACTED\]&cplayer=UNIPLAYER&cos=\[REDACTED\]&cosver=\[REDACTED\]&cplatform=\[REDACTED\]&hl=en_US&cr=\[REDACTED\]&len=757.701&rtn=463&feature=related&afmt=251&idpj=\[REDACTED\]&ldpj=\[REDACTED\]&rti=453&st=0%2C2.149&et=2.042%2C6.642&muted=0%2C0&docid=\[REDACTED\]&ei=\[REDACTED\]&plid=\[REDACTED\]&referrer=\[REDACTED\]&sdetail=\[REDACTED\]&sourceid=yw&of=\[REDACTED\]&vm=\[REDACTED\]"
### Logging Strategy

Metrics are logged differently for ads; Detailed look on these is not expanded upon at the moment.

Other than any ad-related metrics, request for video watch time are logged with an interesting strategy.

YouTube gathers logs in two ways:
- Logs are gathered in a timer that runs for the duration of the video. Once the timer hits, logs are sent. Actions like pausing the video may eventually stop sending logs, but the timer will still be running.
- Events may also trigger logs. Any event that leaves the webpage, like closing the tab or clicking off the video, will also send logs.

The ticker initially fires every 10 seconds, for 30 seconds. Afterward, it transitions to firing every 40 seconds. This continues for the rest of the video.
### Log Data

Below are a list of important parameters sent:

- `ns`. Namespace, set to `yt`
- `el`. Triggering element, either from an ad or the video itself (`detailpage`) ``
- `fmt`. Video format, and internal ID probably containing info about resolution, encoding, etc.
- `afmt`. Audio format, similar to `fmt`.
- `fs`. Is video full-screen (0,1)
- `st`. List of starting timestamps for events
- `et`. List of ending timestamps for events
- `rt`. Absolute ticker start time from the start of video page loading.
- `rti`. When video is playing, rt=rti,rtn = st,et in absolute time. When video is paused rt,rti=st,et in absolute time
- `rtn`. refer to `rti`
- `cl`. Client session ID
- `state`. Video session \[playing, paused\]
- `volume`. Video volume
- `cbr`, `cbrver`, `c`, `cver`, `cplayer`, `cos`, `cosver`, `cplatform`. Browser info
- `hl`, `cr`. Region and language info
- `len`. Full length of active video
- `feature`. Set to `related`. I'm not sure what this is
- `muted`. Is video muted (0,1)
- `docid`. Same as video ID.
- `lact`. Last activity in ticker duration. Activity is like clicking, moving mouse, etc.
- `vis`. Video visibility. `0` for when fully visible (even if mouse focus was on another window), `3` for when not visible at all (focused on another window that covers video or in another workspace). Not sure about any other value

State changes cause these values to have multiple parameters, separated by commas:
`et`, `st` always even if another state was changed
`muted` always even if another state was changed
`volume` always even if another state was changed
`vis` only if visibility was changed
## Sending QoE Metrics

YouTube needs to know how the user handles buffering and playback, and this is done through metrics sent as POST requests to `/api/stats/qoe`. 
Post parameters and headers have not been inspected for these requests, but they definitely do include information about buffering issues due to network.
