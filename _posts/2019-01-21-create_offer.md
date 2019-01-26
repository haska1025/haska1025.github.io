# Create offer

## object relation

```c
                      +-----------------------------+   +------------------------+     +-----------------------+
                      | SessionDescriptionInterface |   | IceCandidateCollection |     | IceCandidateInterface |
                      +-----------------------------+   +------------------------+     +-----------------------+
 +--------------------+           /||\                            /||\                            /||\         +-----------+   
 | SessionDescription |            ||                              ||                              ||          | Candidate |
 +--------------------+            ||                              ||                              ||          +-----------+
         /|\          +------------------------+         +-------------------------+        +------------------+      /|\
          +-----------| JsepSessionDescription |-1----N->| JsepCandidateCollection |-1---N->| JsepIceCandidate |-------+
                      +------------------------+         +-------------------------+        +------------------+

+---------------------------+     +---------------------+   +------------------+                            +----------------------+
| StreamCollectionInterface |     | MediaStreamInterface|   | MediaStreamTrack |                            | MediaSourceInterface |
+---------------------------+     +---------------------+   +------------------+                            +----------------------+
            /||\                         /||\      +--------N-/|\  /||\                                            /||\
             ||                           ||       |          +-----++---------+                    +---------------++-------+
             ||                           ||       |          |                |                    |                        |
+------------------+             +-------------+   |  +------------+   +------------+  +----------------------+ +--------------------+ 
| StreamCollection |-1--------N->| MediaStream |-1-+  | AudioTrack |   | VideoTrack |  | AudioSourceInterface | |VideoSourceInterface|
+------------------+             +-------------+      +------------+   +------------+  +----------------------+ +--------------------+
                                                                                               /||\                     /||\
                                                                                +---------------++----+                  ||
                                                                                |                     |                  ||
                                                                      +-----------------+   +------------------+      +------------+
                                                                      | LocalAudioSource|   | RemoteAudioSource|      | VideoSource|
                                                                      +-----------------+   +------------------+      +------------+
                                                                                           





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
        
                     
