---
layout: post
title:  "The flow for session initial"
date:   2019-01-25 16:47:53
categories: webrtc
---

## Pin hole 过程

              +------------+             +--------------+
              | ice-server |             | stun server  |
              +------------+             +--------------+
                   ^    /|\                 /|\  /|\
                   |  +---\------------------+    |
                   |  |    \+-----------------+   | stun req/rsp
                   |  |<--stun req/rsp        |   | 
                   |  |                       |   |
              +----+----+  stun req/rsp  +----------+
              | offerer |<-------------->| answerer |
              +---------+                +----------+

1. offerer  create offer 成功后，会通过信令server，将offer转给另一端

1.1 offerer同时会发送StunRequestBinding message给stun server

1.2 offerer收到stun server的response for StunRequestBinding message，会解析XOR_MAPPED_ADDRESS or MAPPED_ADDRESS得到 reflex addr；
    
    创建一个reflex candidate对象，接着通过信令服务器转发给answerer。

2. answerer recv offer成功后，会根据offer生成一个answer，也是通过信令服务器转给offerer

2.1 answerer 会发送StunRequestBinding message给stun server

2.1 offerer收到stun server的response for StunRequestBinding message，会解析XOR_MAPPED_ADDRESS or MAPPED_ADDRESS得到 reflex addr；
    
    创建一个reflex candidate对象，接着通过信令服务器转发给offerer。
    
3. offerer/answerer 收到信令服务器转发的对方reflex candidate，向对方发起StunRequestBinding；
   
   对于非对称NAT，A->B 发送一个数据包，B的NAT会丢弃，但是同时在A的NAT戳了一个洞，当B->A发送数据包的时候，A就可以收到;
   
   此时A再次向B发送数据包，这样就完成了P2P穿越。

### offer/answer 报文格式

下面是通过抓包获取的报文格式

#### offer 报文格式

```c
   v=0\r\n
   o=- 5251259112467435424 2 IN IP4 127.0.0.1\r\n
   s=-\r\n
   t=0 0\r\n
   a=group:BUNDLE audio video\r\n
   a=msid-semantic: WMS stream_label\r\n
   
   m=audio 9 UDP/TLS/RTP/SAVPF 111 103 104 9 102 0 8 106 105 13 127 126\r\n
   c=IN IP4 0.0.0.0\r\n
   a=rtcp:9 IN IP4 0.0.0.0\r\n
   a=ice-ufrag:jZBwD+MPnR3diyqI\r\n
   a=ice-pwd:O8TzzmpyhZdZP115LPQSccsm\r\n
   a=fingerprint:sha-256 46:BE:94:B5:1B:48:B7:43:2E:52:15:17:C0:74:7F:89:C5:68:11:EB:8D:C2:54:7E:1F:9B:63:7F:EA:C0:F0:BA\r\n
   a=setup:actpass\r\n
   a=mid:audio\r\n
   a=extmap:1 urn:ietf:params:rtp-hdrext:ssrc-audio-level\r\n
   a=extmap:3 http://www.webrtc.org/experiments/rtp-hdrext/abs-send-time\r\n
   a=sendrecv\r\n
   a=rtcp-mux\r\n
   a=rtpmap:111 opus/48000/2\r\n
   a=rtcp-fb:111 transport-cc\r\n
   a=fmtp:111 minptime=10; useinbandfec=1\r\n
   a=rtpmap:103 ISAC/16000\r\n
   a=rtpmap:104 ISAC/32000\r\n
   a=rtpmap:9 G722/8000\r\n
   a=rtpmap:102 ILBC/8000\r\n
   a=rtpmap:0 PCMU/8000\r\n
   a=rtpmap:8 PCMA/8000\r\n
   a=rtpmap:106 CN/32000\r\n
   a=rtpmap:105 CN/16000\r\n
   a=rtpmap:13 CN/8000\r\n
   a=rtpmap:127 red/8000\r\n
   a=rtpmap:126 telephone-event/8000\r\n
   a=maxptime:60\r\n
   a=ssrc:1211355640 cname:h34m6GOBfUdKeDXI\r\n
   a=ssrc:1211355640 msid:stream_label audio_label\r\n
   a=ssrc:1211355640 mslabel:stream_label\r\n
   a=ssrc:1211355640 label:audio_label\r\n
   
   m=video 9 UDP/TLS/RTP/SAVPF 100 101 116 117 96 97 98\r\n
   c=IN IP4 0.0.0.0\r\n
   a=rtcp:9 IN IP4 0.0.0.0\r\n
   a=ice-ufrag:jZBwD+MPnR3diyqI\r\n
   a=ice-pwd:O8TzzmpyhZdZP115LPQSccsm\r\n
   a=fingerprint:sha-256 46:BE:94:B5:1B:48:B7:43:2E:52:15:17:C0:74:7F:89:C5:68:11:EB:8D:C2:54:7E:1F:9B:63:7F:EA:C0:F0:BA\r\n
   a=setup:actpass\r\n
   a=mid:video\r\n
   a=extmap:2 urn:ietf:params:rtp-hdrext:toffset\r\n
   a=extmap:3 http://www.webrtc.org/experiments/rtp-hdrext/abs-send-time\r\n
   a=extmap:4 urn:3gpp:video-orientation\r\n
   a=sendrecv\r\n
   a=rtcp-mux\r\n
   a=rtpmap:100 VP8/90000\r\n
   a=rtcp-fb:100 ccm fir\r\n
   a=rtcp-fb:100 nack\r\n
   a=rtcp-fb:100 nack pli\r\n
   a=rtcp-fb:100 goog-remb\r\n
   a=rtcp-fb:100 transport-cc\r\n
   a=rtpmap:101 VP9/90000\r\n
   a=rtcp-fb:101 ccm fir\r\n
   a=rtcp-fb:101 nack\r\n
   a=rtcp-fb:101 nack pli\r\n
   a=rtcp-fb:101 goog-remb\r\n
   a=rtcp-fb:101 transport-cc\r\n
   a=rtpmap:116 red/90000\r\n
   a=rtpmap:117 ulpfec/90000\r\n
   a=rtpmap:96 rtx/90000\r\n
   a=fmtp:96 apt=100\r\n
   a=rtpmap:97 rtx/90000\r\n
   a=fmtp:97 apt=101\r\n
   a=rtpmap:98 rtx/90000\r\n
   a=fmtp:98 apt=116\r\n
   a=ssrc-group:FID 1926690562 3444384719\r\n
   a=ssrc:1926690562 cname:h34m6GOBfUdKeDXI\r\n
   a=ssrc:1926690562 msid:stream_label video_label\r\n
   a=ssrc:1926690562 mslabel:stream_label\r\n
   a=ssrc:1926690562 label:video_label\r\n
   a=ssrc:3444384719 cname:h34m6GOBfUdKeDXI\r\n
   a=ssrc:3444384719 msid:stream_label video_label\r\n
   a=ssrc:3444384719 mslabel:stream_label\r\n
   a=ssrc:3444384719 label:video_label\r\n
```

#### answer

```c
v=0\r\n
o=- 473499626881440663 2 IN IP4 127.0.0.1\r\n
s=-\r\n
t=0 0\r\n
a=group:BUNDLE audio video\r\n
a=msid-semantic: WMS stream_label\r\n
m=audio 9 UDP/TLS/RTP/SAVPF 111 103 104 9 102 0 8 106 105 13 127 126\r\n
c=IN IP4 0.0.0.0\r\n
a=rtcp:9 IN IP4 0.0.0.0\r\n
a=ice-ufrag:suBkY3uWqx/Auins\r\n
a=ice-pwd:CK7slog4ign5HQCIgLf0IMPU\r\n
a=fingerprint:sha-256 C5:F0:47:0B:65:41:05:4E:34:07:B8:41:F9:60:98:06:8E:87:10:1D:A2:BB:90:A1:D4:82:46:2F:03:19:CF:2F\r\n
a=setup:active\r\n
a=mid:audio\r\n
a=extmap:1 urn:ietf:params:rtp-hdrext:ssrc-audio-level\r\n
a=extmap:3 http://www.webrtc.org/experiments/rtp-hdrext/abs-send-time\r\n
a=sendrecv\r\n
a=rtcp-mux\r\n
a=rtpmap:111 opus/48000/2\r\n
a=rtcp-fb:111 transport-cc\r\n
a=fmtp:111 minptime=10; useinbandfec=1\r\n
a=rtpmap:103 ISAC/16000\r\n
a=rtpmap:104 ISAC/32000\r\n
a=rtpmap:9 G722/8000\r\n
a=rtpmap:102 ILBC/8000\r\n
a=rtpmap:0 PCMU/8000\r\n
a=rtpmap:8 PCMA/8000\r\n
a=rtpmap:106 CN/32000\r\n
a=rtpmap:105 CN/16000\r\n
a=rtpmap:13 CN/8000\r\n
a=rtpmap:127 red/8000\r\n
a=rtpmap:126 telephone-event/8000\r\n
a=maxptime:60\r\n
a=ssrc:4039702718 cname:/2LmE6VojgltLKZr\r\n
a=ssrc:4039702718 msid:stream_label audio_label\r\n
a=ssrc:4039702718 mslabel:stream_label\r\n
a=ssrc:4039702718 label:audio_label\r\n
m=video 9 UDP/TLS/RTP/SAVPF 100 101 116 117 96 97 98\r\n
c=IN IP4 0.0.0.0\r\n
a=rtcp:9 IN IP4 0.0.0.0\r\n
a=ice-ufrag:suBkY3uWqx/Auins\r\n
a=ice-pwd:CK7slog4ign5HQCIgLf0IMPU\r\n
a=fingerprint:sha-256 C5:F0:47:0B:65:41:05:4E:34:07:B8:41:F9:60:98:06:8E:87:10:1D:A2:BB:90:A1:D4:82:46:2F:03:19:CF:2F\r\n
a=setup:active\r\n
a=mid:video\r\n
a=extmap:2 urn:ietf:params:rtp-hdrext:toffset\r\n
a=extmap:3 http://www.webrtc.org/experiments/rtp-hdrext/abs-send-time\r\n
a=extmap:4 urn:3gpp:video-orientation\r\n
a=sendrecv\r\n
a=rtcp-mux\r\n
a=rtpmap:100 VP8/90000\r\n
a=rtcp-fb:100 ccm fir\r\n
a=rtcp-fb:100 nack\r\n
a=rtcp-fb:100 nack pli\r\n
a=rtcp-fb:100 goog-remb\r\n
a=rtcp-fb:100 transport-cc\r\n
a=rtpmap:101 VP9/90000\r\n
a=rtcp-fb:101 ccm fir\r\n
a=rtcp-fb:101 nack\r\n
a=rtcp-fb:101 nack pli\r\n
a=rtcp-fb:101 goog-remb\r\n
a=rtcp-fb:101 transport-cc\r\n
a=rtpmap:116 red/90000\r\n
a=rtpmap:117 ulpfec/90000\r\n
a=rtpmap:96 rtx/90000\r\n
a=fmtp:96 apt=100\r\n
a=rtpmap:97 rtx/90000\r\n
a=fmtp:97 apt=101\r\n
a=rtpmap:98 rtx/90000\r\n
a=fmtp:98 apt=116\r\n
a=ssrc-group:FID 3930323380 2600335466\r\n
a=ssrc:3930323380 cname:/2LmE6VojgltLKZr\r\n
a=ssrc:3930323380 msid:stream_label video_label\r\n
a=ssrc:3930323380 mslabel:stream_label\r\n
a=ssrc:3930323380 label:video_label\r\n
a=ssrc:2600335466 cname:/2LmE6VojgltLKZr\r\n
a=ssrc:2600335466 msid:stream_label video_label\r\n
a=ssrc:2600335466 mslabel:stream_label\r\n
a=ssrc:2600335466 label:video_label\r\n
```
