��ƪ��Ҫ����TF�ķֲ�ʽ����ʱ�Ļ������Ϊ�˶�TF�ķֲ�ʽ���л�����һ�����µ��˽⣬�����Ƚ��/tensorflow/core/protobuf�е��ļ�������TF�ֲ�ʽ��Ⱥ�ĳ�����⣬Ȼ�����/tensorflow/core/distributed_runtime·���µĺ��ĸ��

***

## TF�ֲ�ʽ��Ⱥ
### ��Ⱥ��������
���ж�TF�ķֲ�ʽ����ʱ����֮ǰ��������Ҫ�ȿ���TF�ֲ�ʽ���еĻ����ܹ���TF�ļ�Ⱥ��cluster������ҵ��job�����ɣ���ҵ������task�����ɡ��ٸ����ӣ�һ����������ҵ���ɵļ�Ⱥ����ҵ1��Ϊ��worker����������3��������ҵ2��Ϊ��ps����������2���������£�
```
Cluster:
    job { name:'worker'
            tasks {key:0 value:'worker1:2222'}
            tasks {key:1 value:'worker2:2222'}
            tasks {key:2 value:'worker3:2222'}
    }
    job { name:'ps'
            tasks {key:0 value:'ps0:2222'}
            tasks {key:1 value:'ps1:2222'}
    }
```
�����ٿ�TF���ڼ�Ⱥ�Ķ��壬��һĿ��Ȼ�ˣ�
```
message JobDef {
    string name = 1;//��ҵ������
    
    //��ҵ����������id��hostname:port�ַ�����ӳ�䣬Ҳ��������ı�ŵ���������ݴ���ӿ�
    map<int32, string> tasks = 2;
}

message ClusterDef {
    repeated JobDef job = 1;
}
```
�������ǻ�ֱ����Master�����Worker����ע�⣬Master��������Master�ṩ�����ͻ���ʹ�õģ���Worker��������Worker�ṩ����Masterʹ�õġ�

### master
������Master����Master������һ�ֱ��ͻ���������ֲ�ʽ��TF���㽻���ķ���

һ��Master����ͨ��������˶��master�Ự��ÿһ���Ự������һ�ż���ͼ�Լ���֮��ص�״̬����Щmaster�Ựͨ�����Ӧͬһ��client�Ự��

һ��Master�Ự��ְ�������
- �ڵ���ã�
- ����ǡ���Ľڵ���ʵ�ֿ��豸�Ϳ���̵�����������Դ����
- ���������worker��ʹ֮���з�������ļ�����ͼ��

ͨ�����ͻ��˿���ͨ��RPC����ʽ��һ��Master֮�䱣��һ������ʽ�ļ��㡣�ͻ������Ƚ���һ���ͻ��˵ĻỰ�����ӵ�һ���ض���Master�����Master���Ŵ���һ����Ӧ��Master�Ự�������ڿͻ��˵ĵ���֮��ά��״̬��

Master�Ự����֮��Master�᷵��һ��������ͻ��ˣ����������Ա��������пͻ��˺�Master�Ự֮��Ľ�����

�ͻ��˿�����CreateSession�����д���һ����ʼ��ͼ��Master������ʹ��ExtendSession��ͼ����ӽڵ㡣

����һ��Master��˵����õĲ�����RunStep����ʵ����һ��Session::Run()��API����֧���ṩ���룬ִ��ͼ���㣬���������

��󣬵��ͻ��˲�����ҪMaster�Ự��ʱ������Ҫͨ��CloseSession�ر�����Ự��Master���Ի��ո��Ự��ص���Դ��Master�ڹرջỰ�ڼ���Ի���Ϊ�������ն�����һ��ʱ�䡣

�������ܽ���MasterService���������ݣ�
```
service MasterService {
    rpc CreateSession(CreateSessionRequest) returns (CreateSessionResponse);
    rpc ExtendSession(ExtendSessionRequest) returns (ExtendSessionResponse);
    rpc PartialRunStep(PartialRunStepRequest) returns (PartialRunSetupResponse);
    rpc RunStep(RunStepRequest) returns (RunStepResponse);
    rpc CloseSession(CloseSessionRequest) returns (CloseSessionResponse);
    rpc ListDevices(ListDevicesRequest) returns (ListDeviceResponse);
    rpc Reset(ResetRequest) returns ( ResetResponse);
}
```
�������ᵽ��xxxRequest��xxxResponse�����ж�Ӧ�Ľṹ�����/tensorflow/core/protobuf/master.proto��

### woker
Worker��������һ��TF�ķ��������Դ���MasterService����һЩ�ֲ����豸��ִ��������ͼ��

һ��Worker�������˶��ע��ͼ��ÿһ��ע��ͼ���ǿͻ�������ͼ��һ����ͼ�������˽���Ҫ�ڵ�ǰworker�ϼ���Ľڵ㡣
```
service WorkerService {
    rpc GetStatus(GetStatusRequest) returns (GetStatusResponse);
    rpc CreateWorkerSession(CreateWorkerSessionRequest) returns (CreateWorkerSessionResponse);
    rpc RegisterGraph(RegisterGraphRequest) returns (RegisterGraphResponse);
    rpc DeregisterGraph(DeregisterGraphRequest) returns (DeregisterGraphResponse);
    rpc RunGraph(RunGraphRequest) returns (RunGraphResponse);
    rpc CleanupGraph(CleanupGraphRequest) returns (CleanupGraphResponse);
    rpc CleanupAll(CleanupAllRequest) returns (CleanupAllResponse);
    rpc RecvTensor(RecvTensorRequest) returns (RecvTensorResponse) {}
    rpc Logging(LoggingRequest) returns (LoggingResponse);
    rpc Tracing(TracingRequest) returns (TracingResponse);
}
```

***
������������/tensorflow/core/protobuf����ҪΪ�˽���TF�м�Ⱥ�Ļ�����������й��̣�������������/tensorflow/core/distributed_runtime������TF�зֲ�ʽ����ʱ�����еĺ��ĸ��
## worker
Worker������ִ�м����ʵ�壬��Client��Master���Ӧ�������������Ĺ�ϵͼ��
```
graph TB
    WorkerCacheInterface-->|���ڲ���|WorkerInterface
    WorkerCache-->|���ڲ���|Worker
    WorkerCacheInterface-->|����|WorkerCache
    WorkerInterface-->|����|Worker
    WorkerCacheLogger-->|�ṩ��־��¼����|WorkerCache
    Worker-->WorkerEnv
    Worker-->WorkerSession
```

## tensor_coding
������TensorResponse�࣬�����������ǣ���һ��RPC����������ʱ��ͨ���������԰ѷ��ؽ���е����ݽ���Ϊ�������Լ�������Ԫ������Ϣ��

## session_mgr
������SessionMgr�࣬��������Worker�ϣ�ΪWorker����Ự�������˻Ự�Ĳ��������٣�ͬʱ��ά����һ����ǰWorker�ϵĻỰ������Ự��ӳ�䡣
```
class SessionMgr {
  public:
    Status CreateSession(...);
    Status DeleteSession(...);
  private:
    const WorkerEnv* const worker_env_;
    const WorkerCacheFactory worker_cache_factory_;
    std::map<string, std::unique_ptr<WorkerSession>> sessions_ GUARDED_BY(mu_);
};
```

## server_lib
TF�е�server�����Ա���Ϊ������ʽ��һ����Worker��һ����Master��������Ϊ�����߶��Ƕ����ṩ�ˡ����񡱣�ֻ���������ֲ�ͬ����ʽ��ServerInterfaceΪ�����ṩ��ͳһ�Ľӿڣ�
```
class ServerInterface {
  public:
    virtual Status Start() = 0;
    virtual Status Stop() = 0;
    virtual Status Join() = 0;
};
```
�����е�Server���������Ӧ�Ĺ���������������໹�ṩ�˶��������ע��ӿڣ�
```
class ServerFactory {
  public:
    virtual Status NewServer(...);
    
    //�κ�һ������������࣬�������������������һ������ע�ᵽ����
    static void Register(const string& server_type, ServerFactory* factory);
    
    //����server_def��Ѱ��һ���ܲ���ָ��server�Ĺ���
    static Status GetFactory(const ServerDef& server_def, ServerFactory** out_factory);
};
```

## scheduler
����Graph��CostModel����Ϣ�����㲻ͬ���Ȳ����£�ÿ���ڵ�����翪ʼʱ�������ʼʱ�䣬������SlackAnalysis��GreedyScheduler��PriorityScheduler�ֱ�������ɳڲ��ԡ�̰�ĵ��Ȳ��Ժ����ȼ����Ȳ��ԡ�

## rendezvous_mgr_interface
��RendezvousMgr������һ���ֲ�rendezvous����ļ��ϡ����б���ǰ��Worker���͵��������ڽ���֮ǰ�������RendezvousMgr�б����š�ÿһ��ȫ�ֵ�step_id����Ӧ��һ����RendezvousMgr�����һ���ֲ���rendezvousʵ����

## remote_device
������һ��������NewRemoteDevices�������Է���remote_worker�ϵĿ����豸��

## partial_run_mgr
PartialRunMgr������δ��ɵľֲ����е���������ֻ֤�е���Ӧ��ִ�����������ʱ�����Żᱻ���Ϊ��ɡ�

��TF��worker�У�ִ�������첽��ִ�У�ֱ�������������ܹ����������Ĳ���������Ŀ�꣨���᷵�������Ĳ�������ɡ�Ҳ����˵������ͼ��������ڵ㶼������Ϊworkerִ�е�Ŀ�꣬һ���Ƿ��������Ĳ�����Ӧ�Ľڵ㣬һ���ǲ��������Ĳ�����Ӧ�Ľڵ㡣һ���ֲ����а�����������һ������������Ҫ�������Ŀ�꣬�ڶ������������ڵڶ���ʱ�����ܴ���һ�������������ͼ�����������Ѿ�������ɣ��������Ŀ�����ڼ��㡣��ʱ��PartialRunMgr�ͷ��������ˣ���Ȼ��ʱ�����Ͽ��Է����ˣ���Ϊ����������������������ˣ�ʣ�������Ŀ�겢��Ӱ�췵�صĽ������TF��ȻҪ�����ȵ����е�Ŀ�궼��ɼ�����У���Ϊ��Ŀ����ɼ���֮ǰ�����ǲ���֪���м������Ƿ�ᷢ���仯��

## message_wrappers
��Master��Worker֮���໥ͨ�ŵ�Request/Response�İ�װ�ࡣ
```
// Wrapper classes for the `MasterService.RunStep` request message.
class RunStepRequestWrapper {}
class MutableRunStepRequestWrapper : public RunStepRequestWrapper {}
class InMemoryRunStepRequest : public MutableRunStepRequestWrapper {}
class MutableProtoRunStepRequest : public MutableRunStepRequestWrapper {}
class ProtoRunStepRequest : public RunStepRequestWrapper {}

// Wrapper classes for the `WorkerService.RunGraph` request message.
class RunGraphRequestWrapper {}
class MutableRunGraphRequestWrapper : public RunGraphRequestWrapper {}
class InMemoryRunGraphRequest : public MutableRunGraphRequestWrapper {}
class MutableProtoRunGraphRequest : public MutableRunGraphRequestWrapper {}
class ProtoRunGraphRequest : public RunGraphRequestWrapper {}

// Wrapper classes for the `WorkerService.RunGraph` response message.
class MutableRunGraphResponseWrapper {}
class InMemoryRunGraphResponse : public MutableRunGraphResponseWrapper {}
class OwnedProtoRunGraphResponse : public MutableRunGraphResponseWrapper {}
class NonOwnedProtoRunGraphResponse : public MutableRunGraphResponseWrapper {}

// Wrapper classes for the `MasterService.RunStep` response message.
class MutableRunStepResponseWrapper {}
class InMemoryRunStepResponse : public MutableRunStepResponseWrapper {}
class OwnedProtoRunStepResponse : public MutableRunStepResponseWrapper {}
class NonOwnedProtoRunStepResponse : public MutableRunStepResponseWrapper {}
```

## master_session
�뵥������µ�DirectSession��Ӧ�ģ��ֲ�ʽ����µ�Master�Ự����������ͼ����Ļ������裬������Դ���䡢�ڵ���á�ͼִ�еȡ�

## master_interface
������TF��Master����ͨ�ŵ�����ӿڡ�����ӿڼ�֧�ֻ���RPC��masterʵ�֣�Ҳ֧�ֽ����ڲ���masterʵ�֡�

## master
TF��Master�����ʵ�֡���Worker�����Ӧ��

## master_env
Master�Ļ����࣬������һ��Master������Ļ�����Դָ�롣ע��Master����ӵ����Щָ�롣

## local_master
�ֲ�Master��ʵ�֡��ֲ�Master�ĺ����ǣ���Client��ͨ�Ų��ǿ��豸�ģ�����ֱ���ڽ����ڲ����еġ����Master��ʵ�֣���Ϊ�˸�ͬ�����ڲ���Client�ṩ����Ч��Master����

## graph_mgr
GraphMgr������ע�ᵽĳ��worker��ͼ�ļ��ϡ�ÿһ��ע���ͼ���ᱻһ�������ʶ����������GraphMgr���������ҷ��ظ������ߡ���ע��ɹ�֮�󣬵�����ͨ��һ��ͼ�����ִ��һ��ͼ��ÿһ�ε�ִ�ж���һ��ȫ�ֵ�"step_id"Ψһ��ʶ����ͬһ��ͼ�ϣ������ظ��Ͷ�����ִ�ж�Σ�ֻҪÿһ��ִ�е�"step_id"���ǲ�ͬ�ġ�

## call_options
Ϊ��ͬ��RPCϵͳ�ṩ�˿ɲ�εĵ��ýӿڡ�

## base_rendezvous_mgr
ΪRendezvousMgrInterface�ṩ�˲�ͬ��ʵ�֣�������ͼ���£�
```
graph TB
    RendezvousMgrInterface-->|����|BaseRendezvousMgr
    RemoteRendezvous-->|����|BaseRemoteRendezvous
    Rendezvous-->|����|RemoteRendezvous
    
```

[github��ַ](https://github.com/tengkz/tensorflow_notes)