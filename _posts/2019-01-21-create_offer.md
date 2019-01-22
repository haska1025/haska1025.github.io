# Create offer

## object relation

```c

                                                                                            +------------------------------+
                                                                                        +-->| TransportDescriptionFactory  |     
+----------------+    +---------------+           +----------------------------------+  |   +------------------------------+
| PeerConnection |--->| WebRtcSession |---------->| WebRtcSessionDescriptionFactory  |--|   +--------------------------------+ 
+----------------+    +---------------+           +----------------------------------+  +-->| MediaSessionDescriptionFactory |
                                                                                            +--------------------------------+
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
