#Create offer

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
                            
          
```
