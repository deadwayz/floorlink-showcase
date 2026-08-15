# FloorLink

Live: https://deadwayz.github.io/hyle-showcase/

![HYLE showcase preview](floorlink.gif)

Internal web app for low-latency live screen and system-audio streaming
between two office floors. One PC broadcasts its display via
`getDisplayMedia()`, the stream travels **peer-to-peer over WebRTC**, and
a PC on another floor views it in the browser. Cloudflare is used only
for signaling - it never sees the video/audio.

```
Floor 1 PC (broadcaster)                Floor 2 PC (viewer)
  getDisplayMedia()                        <video>
        │                                     ▲
        ▼                                     │
  RTCPeerConnection  ── direct WebRTC media ───┘
        │                                     ▲
        │ signaling (SDP/ICE over WSS)        │
        ▼                                     │
        └────────► Cloudflare Worker ─────────┘
                   (Durable Object per room)
```
- **Media path**: browser-to-browser, direct, over WebRTC. Never touches
  a server.
- **Control path**: a Cloudflare Worker (one Durable Object per room)
  relays SDP offers/answers and ICE candidates between the two browsers,
  and tracks room membership (broadcaster present? how many viewers?).
- No RTMP/HLS, no media server, no transcoding, no video storage. If a
  requirement can't be met with browser APIs alone, that limitation is
  documented rather than faked.
