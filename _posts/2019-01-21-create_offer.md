# Create offer

## object relation

```c

                                                                                            +------------------------------+
                                                                                        +-->| TransportDescriptionFactory  |     
+----------------+    +---------------+           +----------------------------------+  |   +------------------------------+
| PeerConnection |--->| WebRtcSession |---------->| WebRtcSessionDescriptionFactory  |--|   +--------------------------------+ 
+----------------+    +---------------+           +----------------------------------+  +-->| MediaSessionDescriptionFactory |
                          |  |                                                              +--------------------------------+
                          |  +--1------3-+                                                         
    +---------------------+             \|/
    |                            +-------------+                                 +--------------+
    |     +----------------------| BaseChannel |--1--------------------------1-->| MediaChannel |
    |     |                      +-------------+                                 +--------------+
    |     |                          /||\                                              /||\
    |     |             +-------------++-------+-------------+                  +-------++-------------+
    |     |             |                      |             |                  |                      |
    |     |      +--------------+  +--------------+ +-------------+    +-------------------+  +-------------------+
    |     |      | VoiceChannel |  | VideoChannel | | DataChannel |    | VoiceMediaChannel |  | VideoMediaChannel |
    |     |      +--------------+  +--------------+ +-------------+    +-------------------+  +-------------------+
    |     |                                                                   /||\                     /||\
    |     |                                                         +-------------------------+ +---------------------+
    |     |                                                         | WebRtcVoiceMediaChannel | | WebRtcVideoChannel2 |
    |     +---------------------------------------------------+     +-------------------------+ +---------------------+
    |                                                         |  
    |                                                         |
+---------------------+          +-----------+         +------------------+
| TransportController |-1-----N->| Transport |-1----N->| TransportChannel |
+---------------------+          +-----------+         +------------------+
                                       /||\                      /||\
                                        ||                        ||                                                         
                               +--------------+       +----------------------+
                               | P2PTransport |       | TransportChannelImpl |
                               +-----+--------+       +----------------------+
                            |--------|                         /||\
                   +---------------+                  +---------++------------+
                   | DtlsTransport |                  |                       |
                   +---------------+     +---------------------+ +-----------------------------+
                                         | P2PTransportChannel | | DtlsTransportChannelWrapper |
                                         +---------------------+ +-----------------------------+
                                                    |
                                                   \|/                  +------------+            +--------------------+
                                         +----------------------+       | Candidate  |<--------+->| AllocationSequence |
                                         | PortAllocatorSession |       +------------+         |  +--------------------+
                                         +----------------------+              +----------+    |  +-------+ 
                                                   /||\                  +---->| PortData |----+->|  Port |      +--------------------+
                                         +---------------------------+   |     +----------+       +-------+      | StunRequestManager |
                                         | BasicPortAllocatorSession |---+                           /||\        +--------------------+
                                         +---------------------------+              +-----------------++-------+             /|\
                                                                                    |            |             |              |
                                                                                +---------+  +----------+  +---------+        |
                                                                                | TCPPort |  | TurnPort |  | UDPPort |--------+
                                                                                +---------+  +----------+  +---------+
                                                                                                               /||\
                                                                                                           +----------+
                                                                                                           | StunPort |
                                                                                                           +----------+
 BaseChannel持有2个TransportChannel实例，一个是rtp，另一个是rtcp。
 
 WebRtcSession 中有三个BaseChannel实例，分别是audio，video，data
 
 StunRequestManager 主要职责是：
 1、维护了所有的StunRequest, key是transcactionid
 2、定时检查是否收到响应
 
```                                                                                            
## offer 报文格式

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

## answer

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

## initial

```c
PeerConnectionFactory::Initialize -----初始化
--> new RefCountedDtlsIdentityStore() //// RefCountedDtlsIdentityStore是rtc::RefCountedObject<DtlsIdentityStoreImpl>
    --> RefCountedDtlsIdentityStore::GenerateIdentity
        --> worker_thread_->Post(task, MSG_GENERATE_IDENTITY, msg) //// 投递消息
            --> DtlsIdentityStoreImpl::WorkerTask::GenerateIdentity_w()
               --> signaling_thread_->Post(this, MSG_GENERATE_IDENTITY_RESULT, msg); //// 投递消息
                   --> DtlsIdentityStoreImpl::WorkerTask::OnMessage()
                      --> DtlsIdentityStoreImpl::OnIdentityGenerated
                         -->  WebRtcIdentityRequestObserver::OnSuccess
                            --> SignalCertificateReady ===发信号
                            --> WebRtcSessionDescriptionFactory::SetCertificate
                               --> WebRtcSessionDescriptionFactory::InternalCreateOffer
                                  --> MediaSessionDescriptionFactory::CreateOffer
                                     --> MediaSessionDescriptionFactory::AddAudioContentForOffer///增加audio媒体描述，主要是填充AudioContentDescription
                                        --> CreateMediaContentOffer /// 将Option等结构中的数据填充到AudioContentDescription
                                        --> AddTransportOffer /// 增加和传输相关的内容信息
                                     --> MediaSessionDescriptionFactory::AddVideoContentForOffer///增加video相关媒体描述，具体流程类似音频  
                                  --> WebRtcSessionDescriptionFactory::PostCreateSessionDescriptionSucceeded///结构创建完成，投递消息，会通知给CreateSessionDescriptionObserver，这个结构需要Application实现的。                                  
          
```

## set local session

```c
PeerConnection::SetLocalDescription
 --> WebRtcSession::SetLocalDescription
   --> WebRtcSession::CreateChannels
      --> WebRtcSession::CreateVoiceChannel/// 音频channel的创建流程
        --> ChannelManager::CreateVoiceChannel_w
           --> CompositeMediaEngine<WebRtcVoiceEngine>::CreateChannel
              --> WebRtcVoiceEngine::CreateChannel
                 --> new WebRtcVoiceMediaChannel
           --> new VoiceChannel       
              --> VoiceChannel::Init()
                --> BaseChannel::Init()
                  --> BaseChannel::SetTransport
                     --> BaseChannel::SetTransport_w
                       == set_rtcp_transport_channel/// 创建rtcp 的transport channel
                        --> TransportController::CreateTransportChannel_w
                           --> TransportController::GetOrCreateTransport_w
                              --> TransportController::CreateTransport_w
                                 --> new DtlsTransport<P2PTransport>
                           --> Transport::CreateChannel
                              --> DtlsTransport<P2PTransport>::CreateTransportChannel
                                 --> P2PTransport::CreateTransportChannel
                                    --> new P2PTransportChannel(name(), component, port_allocator())
                        == set_transport_channel ///设置rtp的transport channel
            
          
      --> WebRtcSession::CreateVideoChannel /// 视频channel的创建流程和音频类似
  --> WebRtcSession::MaybeStartGathering
     --> TransportController::MaybeStartGathering()
     --> TransportController::MaybeStartGathering_w
        --> Transport::MaybeStartGathering
           --> Transport::CallChannels
              --> P2PTransportChannel::MaybeStartGathering()
                 --> PortAllocator::CreateSession
                    --> BasicPortAllocator::CreateSessionInternal
                       --> new BasicPortAllocatorSession
                          --> BasicNetworkManager::StartUpdating()
                             --> BasicNetworkManager::StartNetworkMonitor()
                    --> P2PTransportChannel::AddAllocatorSession
                       --> BasicPortAllocatorSession::StartGettingPorts()
                          --> BasicPortAllocatorSession::GetPortConfigurations /// 配置
                          --> BasicPortAllocatorSession::OnConfigReady ///配置ready
                          --> BasicPortAllocatorSession::OnAllocate() ///准备分配
                             --> BasicPortAllocatorSession::DoAllocate()
                                --> BasicPortAllocatorSession::GetNetworks ///获取本地机器所有网卡的地址，貌似只是获取Ipv4地址
                                --> new AllocationSequence
                                --> AllocationSequence::Init() /// 创建一个udpsocket
                                --> AllocationSequence::Start() /// 通过消息MSG_ALLOCATION_PHASE 倒一下线程
                                --> AllocationSequence::OnMessage 
                                   --> AllocationSequence::CreateUDPPorts()
                                      --> UDPPort::Create
                                         --> new UDPPort
                                         --> UDPPort::Init() /// 绑定了AsyncSocket和UDPPort两层之间的信号通信
                                      --> BasicPortAllocatorSession::AddAllocatedPort
                                         --> UDPPort::PrepareAddress()
                                            --> UDPPort::OnLocalAddressReady /// 没有获取到网卡地址列表，会尝试设置一个缺省地址
                                               --> Port::AddAddress /// 生成local candidate
                                               --> UDPPort::MaybePrepareStunCandidate()
                                                  --> UDPPort::SendStunBindingRequests()
                                                     --> UDPPort::SendStunBindingRequest
                                                        --> new StunBindingRequest /// 生成请求报文
                                                        --> StunRequestManager::Send
                                                          /// 通过消息MSG_STUN_SEND 倒一次线程到StunRequest::OnMessage完成
                                                          /// StunRequest::OnMessage只是将StunRequest序列化成ByteBuffer，然后，
                                                          /// 通过信号manager_->SignalSendPacket，最后是到在UDPPort::OnSendPacket，通过socket发送出去
                                                           --> StunRequestManager::SendDelayed 
                                                           
                                                     
 ```
 
 ## recv the response for StunRequestBinding 
 
 ```c
   ///如果是socket共享模式，在AllocationSequence调用init的时候，设置了OnReadPacket来响应AsyncSocket的read事件。
   ///否则，是在UDPPort里面创建socket，并且响应socket的read事件。
   AllocationSequence::OnReadPacket
   --> UDPPort::HandleIncomingPacket
      --> UDPPort::OnReadPacket
         /// 如果是发给Stun server的响应包，走匹配逻辑
         --> StunRequestManager::CheckResponse
            --> StunMessage::Read /// 反序列化
            --> StunRequestManager::CheckResponse
               /// 1. 获取响应报文中的STUN_ATTR_MAPPED_ADDRESS地址，其实是一个reflex addr，
               ///    然后构造一个SocketAddr结构。
               /// 2. 还会继续发送一遍StunBindingRequest，不管本次响应是否成功。
               --> StunBindingRequest::OnResponse
                  --> UDPPort::OnStunBindingRequestSucceeded
                     --> Port::AddAddress /// 构造reflex candidate结构
                        --> SignalCandidateReady(this, c)
    --> BasicPortAllocatorSession::OnCandidateReady
        
                     
