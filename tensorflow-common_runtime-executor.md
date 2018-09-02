# Ŀ¼
1. ���ĸ���
2. executor.h
    1. Executor
    2. NewLocalExecutor
    3. ExecutorBarrier
3. executor.cc
    1. structs
    2. GraphView
    3. ExecutorImpl
    4. ExecutorState
    5. details

# 1. ���ĸ���
ִ������TF�ĺ����еĺ����ˣ�ǰ��������ô���׼�����������Ҫ�����Ｏ����ˣ����뻹�е�С���������������������ȴ��Ԥ���룬ִ�����ĸ���ࡢ�ṹ���ӣ���Ҫ͸����Ⲣ�����ף�Ϊ�˱������µ��׶��ԣ�����Ҳ�Ǿ�����ϸ֦ĩ����������������Ӧִ�����ĺ��ı��ʣ�������ִ�����漰��������ʵ��̫�࣬��˱�ƪ��ƪ�����ܻ��е㳤���������׼����
ִ�����ĸ�����Ȼ���ӣ�������ϵ����ȴ�ܼ�𣬸���һ�Ŵ�ִ�е�ͼ�������������룬�������ռƻ�ִ�У��������ͺ��ˡ�������߶�֮ǰ�������ϵ�е����������˽⣬����ִ������ִ�й��̣�Ӧ������һ����ŵ�ӡ���ˡ�Ϊ�˼���ͼ�ܹ�ִ�У�TF�����op�ĸ�������ʵ��ִ��op��kernel���������ܹ����������ݵ�node��graph�����ڴ桢�豸������ר�ŵĹ����࣬��Щ�����һ��Ϊ����ͼ��ִ���ṩ����ȫ���֧�֡��������ִ���У����зǳ����ϸ����Ҫ�������������ǽ������ڽ��н��ܣ���һ�ڽ���executor.hͷ�ļ�����������ִ�����ṩ�Ķ���ӿڣ��ڶ����ֽ���executor.ccԴ�ļ�����������ִ������ִ��ԭ��

# 2. executor.h
��һ�ڽ�����ִ���������API���ڲ鿴����Ľṹ֮ǰ�������ȿ�һ��ִ��������α�Ӧ�õġ�
```
Graph* graph = ...;//����ͼ
Executor* executor;
NewSimpleExecutor(my_device, graph, &executor);//����ִ����
Rendezvous* rendezvous = NewNaiveRendezvous();//����ͨ��ͨ��
rendezvous->Send("input", some_input_tensor);//�ṩ����
executor->Run({ExecutorOpts, rendezvous, nullptr});
rendezvous->Recv("output",&output_tensor);//������
```
���̷ǳ��ļ��׶���TFͨ������������ṩ�����õ��ⲿAPI�����������������Եײ㸴�ӵ��ڲ��ṹ��Ϊ֧�ֵģ����������ǾͿ�һ�£�����API����TF��������Щ����
## 2.1 Executor
���ȣ���Ȼ��ִ��������ִ���������ṩ�Ľӿںܼ򵥣����£�
```
class Executor {
  public:
    typedef std::function<void(const Status&)> DoneCallback;
    virtual void RunAsync(const Args& args, DoneCallback done) = 0;
    
    //RunAsync()������ͬ���汾
    Status Run(const Args& args){
        Status ret;
        Notification n;
        RunAsync(args, [&ret, &n](const Status& s) {
            ret = s;
            n.Notify();
        });
        n.WaitForNotification();
        return ret;
    }
};
```
ִ����������Ӧ�����첽ִ�еģ�������ǿ�����⣬��Ϊͼ������һ���ǳ������������Ĺ��̣��첽����Ч�ʸ��ߡ���ͬʱִ����Ҳ�ṩ���첽�����ͬ����װ�����û�������ͬ���ķ�ʽ��ִ�С�
ִ�����ӿڵļ���ԣ���ִ�����ĸ��ӹ���֮���γ��˾޴�ķ�����������ǲ��ò����ɣ�ִ�����ڲ��ǲ���������ʲô�ṹ����Ȼ�����Ƿ���ִ�к����ĵ�һ��������Args���������������Ľṹ��
```
struct Args {
    int64 step_id = 0;
    Rendezvous* rendezvous = nullptr;
    StepStatsCollector* stats_collector = nullptr;
    FunctionCallFrame* call_frame = nullptr;
    CancellationManager* cancellation_manager = nullptr;
    SessionState* session_state = nullptr;
    TensorStore* tensor_store = nullptr;
    ScopedStepContainer* step_container = nullptr;
    
    //���Ϊ�棬���豸�ϵ���Sync����
    bool sync_on_finish = false;
    
    typedef std::function<void()> Closure;
    typedef std::function<void(Closure)> Runner;
    Runner runner = nullptr;
    
    //ÿ��һ���ڵ����ִ�е�ʱ�򣬶����������ص�����
    typedef std::function<Status(const string& node_name, const int output_slot, const Tensor* tensor, const bool is_ref, OpKernelContext* ctx)> NodeOutputsCallback;
};
```
�������еĲ��������Ǹ���һЩ˵����
- step_id��һ�����̼����Ψһ��ʶ����������ʶִ�еĲ��衣��һ������������һ����Ҫ�ڶ���豸��ִ�е�opʱ����Щ��ͬ�豸�ϵ�ִ���������յ���ͬ��step_id��step_id�Ǳ�����׷��һ���������õ�����Դ�ġ�
- RunAsync()����ʹ��rendezvous����Ϊ�����ͼ֮�乵ͨ���������Ļ��ƣ�
- RunAsync()����stats_collector���ռ�ͳ����Ϣ�������������ܸ��������ռ�ͳ�ƺ�traces��Ϣ��
- ���ִ����������ִ��һ����������ôRunAsync()����ʹ��call_frame�������ڵ����ߺͱ�������֮�䴫�ݲ����ͷ���ֵ��
- RunAsync()����ʹ��cancellation_manager��ע��һЩ���ڼ���ͼִ�б�ȡ����Ļص�������
- RunAsync()��ִ�еıհ������runner��ͨ����˵��runner������һ���̳߳�֧�֡�

## 2.2 NewLocalExecutor
��������TF��������������һ�����ص�ִ����������Ҫ�õ��������������
```
::tensorflow::Status NewLocalExecutor(const LocalExecutorParams& params, const Graph* graph, Executor** executor);
```
�������ֳ�����һ��������δ�������Ľṹ��LocalExecutorParams������˼�壬�����������ɱ���ִ������Ҫ�Ĳ����������Ľṹ���£�
```
struct LocalExecutorParams {
    Device* device;
    FunctionLibraryRuntime* function_library = nullptr;
    std::function<Status<const NodeDef&, OpKernel**)> create_kernel;
    std::function<void(OpKernel*)> delete_kernel;
    Executor::Args::NodeOutputsCallback node_outputs_cb;
};
```
���������豸�������⡢kernel�����ɾ�����̡��ڵ�ִ����ϵĻص��������������ǽ��ῴ�����ں�����ʵ�����棬������������Щ��Ϣ����ִ�����ġ�

## 2.3 ExecutorBarrier
��ʵ�ʵ�Ӧ���У����ǿ�����Ҫ�õ���ֹһ��ִ������Ϊ��ʹ���ִ�����ܲ������У�������Ҫ����Щͬʱִ�е�ִ�������й����ͳ����ǾͲ�����ExecutorBarrier�ࡣ���£�
```
class ExecutorBarrier {
  public:
    typedef std::function<void(const Status&)> StatusCallback;
    
    //Ϊnum����ͬ��ִ��������ͳ��͹���r��һ����������ݴ���ͨ�����������һ��ִ����ʧ�ܣ�rendezvous�������һ�Ρ������һ��ִ����ִ�����ʱ�������done������ExecutorBarrier����ᱻɾ����
    ExecutorBarrier(size_t num, Rendezvous* r, StatusCallback done);
    
    //����һ��ִ������ִ�����֮�������õĺ����հ���ִ������ʹ�����ǽ���ʱ��״̬��Ϊִ�бհ��Ĳ���
    StatusCallback Get() {
        return std::bind(&ExecutorBarrier::WhenDone, this, std::placeholders::_1);
    }
  private:
    Rendezvous* rendez_ = nullptr;
    StatusCallback done_cb_ = nullptr;
    
    mutable mutex mu_;
    int pending_ GUARDED_BY(mu_) = 0;//��ʣ����ִ����ûִ����
    Status status_ GUARDED_BY(mu_);
    
    void WhenDone(const Status& s){
        //...
    }
};
```

# 3. executor.cc
��һ�����ǽ�̽��ִ������ʵ�֡�����������ǰ��һ����������ܣ��������߸�������⡣��һ�������ջ��������Ϣ���е���Ƿ���һ�����õĽ��ܷ���������˵���������ĺ���ʵ�ֱȽϸ��ӣ�ǰ��Ľṹ����������⣬������ǾͰ���Դ�ļ����Ⱥ�˳������ˣ��ȱ����ҵ����õĳ��ַ�ʽ�������޸������˳��

## 3.1 structs
��ͼ������ʱ��Ϊ�˷���������ṩ����Ĺ����أ����ǰѺܶ�ṹ��ƵıȽϸ��ӣ�����graph, node�ȣ�����ִ�е�ʱ��һ����Щ���ӵĽṹ���ǲ�һ���õ��ϣ��������ǵĴ���Ҳ��Ӱ��ִ��Ч�ʣ����TF������˺ܶ��֮ǰ���ӽṹ�ļ򻯣�����������һ�ڽ�Ҫ���ܵ�EdgeInfo��NodeItem���Լ���һ�ڽ�Ҫ���ܵ�GraphView��
��������������EdgeInfo��
```
struct EdgeInfo {
    int dst_id;
    int output_slot:31;
    bool is_last:1;
    int input_slot;
};
```
��Ȼ������ʾ���Ǽ���ͼ�еıߣ�������Ŀ�Ľڵ㣨dst_id����Ŀ�Ľڵ�Ķ˿ںţ�output_slot����Դ�ڵ�Ķ˿ںţ�input_slot����֮����û�а���Դ�ڵ㣬���ǲ²�����Ϊ����ṹ����Ǳ�������Դ�ڵ��ڲ��ġ�
���⣬is_last��ʾ�������߶�Ӧ���ǲ���Ŀ�Ľڵ�����һ���˿ڡ�
���```int output_slot:31```����ṹ����ʾ�����������ĸ��ֽڣ�int����32��bit��output_slot��ռ���е�31�����������������```bool is_last:1```��ռ�����һ��bitλ�����ֶ��巽ʽ��c++11֮����еģ����Ը���Ч�����ô洢�ռ䡣
���������ǿ�һ��NodeItem����ṹ������ʾ����ͼ�е�һ���ڵ㣺
```
struct NodeItem {
    const Node * node = nullptr;//��ʾһ������ͼ�еĽڵ�
    
    OpKernel* kernel = nullptr;//����ڵ��Ӧ��OpKernel
    
    bool kernel_is_expensize:1;
    bool kernel_is_async:1;
    bool is_merge:1;
    bool is_enter:1;
    bool is_exit:1;
    bool is_exit:1;
    bool is_control_trigger:1;
    bool is_sink:1;
    bool is_enter_exit_or_next_iter:1;
    
    int num_inputs;
    int num_outputs;
    
    int input_start = 0;//�������ʼ����
    
    size_t num_output_edges;//����ߵ�����
    
    PendingCounts::Handle pending_id;
    
    const EdgeInfo* output_edge_list() const { return output_edge_base(); }
    
    const EdgeInfo& output_edge(int i);
    
    DataType input_type(int i);
    DataType output_type(int i);
    
    const AllocatorAttributes* output_attrs();
  
  private:
    char* var();
    EdgeInfo output_edge_base();
    AllocatorAttributes* output_attr_base();
    uint8* input_type_base();
    uint8* output_type_base();
}
```
�ɼ���NodeItem�ṩ�˶��ڼ���ͼ�ڵ�ľ�̬��Ϣ�ķǳ���ϸ��������

## 3.2 GraphView
�ղ�Ҳ�ᵽ�ˣ�Ϊ��ִ�е�Ч�ʣ�ִ������һЩ�����ṹ�����˼򻯣��޳��˲���Ҫ����Ϣ�����磬���ڼ���ͼ��˵��������ִ�й����У�����Ҫ��ͼ�ṹ���и��ģ����ԭ����Graph���кܶ��޸�ͼ�Ľӿڶ�û���ˣ�����TF�ṩ��һ�����ɸı����ͼ������ʹͼ��ִ�и��Ӹ�Ч��
�������������������Ľӿں����ݣ�
```
class GraphView {
  public:
    GraphView(): space_(nullptr) {}
    void Initialize(const Graph* g);//GraphView��ʼ��
    Status SetAllocAttrs(const Graph* g, const Device* device);
    NodeItem* node(size_t id) const;//����ָ���Ľڵ���Ϣ
  private:
    char* InitializeNode(char* ptr, const Node* n);//��ʼ���ڵ���Ϣ
    size_t NodeItemBytes(const Node* n);
    
    int32 num_nodes_ = 0;
    uint32* node_offsets_ = nullptr;//�ڵ��ƫ�ã�node_offsets_[id]�����˽ڵ�id��space_�е�ƫ����
    char* space_;//������ָ��NodeItem����Ĵ洢��ַ��ָ��
};
```
���ԣ�����������˵�ͺ�����ˣ�GraphView֮������Graph��һ�����ɸı����ͼ������Ϊ��������һ���ڴ�ռ䣬Ȼ���ͼ�����нڵ����Ϣ��NodeItem�������δ�������ռ��У����ṩ�˶Կռ�����Ϣ���м����Ľӿڣ����ǣ�û���ṩ����Щ��Ϣ�����޸ĵĽӿڣ����ԣ�������Ȼ�ܹ����ʵ�Graph�е��κξ�̬��Ϣ�������޷���������޸ġ�

## 3.3 ExecutorImpl
�ղ������Ѿ�������Executor��ֻ��һ�����࣬������ִ����ʵ�֣���Ҫ���������࣬TF�ṩ��һ��ʵ����ExecutorImpl�����Ľṹ��Ȼ�Ƚϼ򵥣�
```
class ExecutorImpl : public Executor {
  public:
    ExecutorImpl(const LocalExecutorParams& p, const Graph* g) : params_(p), graph_(g), gview_(){
        CHECK(p.create_kernel != nullptr);
        CHECK(p.delete_kernel != nullptr);
    }
    ~ExecutorImpl() override {
        for(int i=0;i<graph_->num_node_ids();i++){
            NodeItem* item = gview_.node(i);
            if(item != nullptr){
                params_.delete_kernel(item->kernel);
            }
        }
        for(auto fiter : frame_info_){
            delete fiter.second;
        }
        delete graph_;
    }
    
    Status Initialize();
    
    //����ǰͼ�е�ÿһ���ڵ㣬���Է����������ڷ����ڴ�ʱ���ڴ��������
    Status SetAllocAttrs();
    
    void RunAsync(const Args& args, DoneCallback done) override;

  private:
    //������������Ϣ
    static Status BuildControlFlowInfo(const Graph* graph, ControlFlowInfo* cf_info);
    
    //��ʼ����ִ�м�����Ϣ
    void InitializePending(const Graph* graph, const ControlFlowInfo& cf_info);
    
    //ȷ��ÿһ��FrameInfo����׼����
    FrameInfo* EnsureFrameInfo(const string& fname){
        auto slot = &frame_info_[fname];
        if(*slot == nullptr){
            *slot = new FrameInfo;
        }
        return *slot;
    }
    
    //����ǰ�Ķ���ӵ��
    LocalExecutorParams params_;
    const Graph* graph_;
    GraphView gview_;
    
    //����params_�Ļ���
    bool device_record_tensor_accesses_ = false;
    
    //û���κ�����ߵĸ��ڵ㣬����Ӧ����ɳ�ʼԤ������
    std::vector<const Node*> root_nodes_;
    
    //��֡���Ƶ�֡��Ϣ��ӳ��
    gtl::FlatMap<string, FrameInfo*> frame_info_;
};
```
Ϊ��˵��ϸ�ڣ�������������˲��ֺ�����ʵ�ַ�ʽ���������е��ص��������˵����
- ����������������һ�������������飬��һ������GraphView�ҵ�ÿ��node������OpKernel�����ҽ���ɾ�����ڶ��������е�֡��Ϣɾ������������GraphView����ɾ����
- ��ǰִ����ʵ��ӵ�еĶ�����������һ��LocalExecutorParamsִ��������ʱ�Ĳ���������Graph*����Ӧͼ��ָ�룬ע��ִ������ӵ�����ָ�룬����ӵ������ͼ��������GraphView������ִ������ȫӵ�еĽṹ��
- ����root_nodes_���������Ӧ�û������һЩ������ͼ��ִ�й��̣��Ǵ�һЩ����Ҫ����ĸ��ڵ�����ģ����ݽڵ�֮���������ϵ����ִ�У�������̻��õ����е����ݽṹ��һ��һ��������ĳ���ڵ��ǰ���ڵ㶼׼�����ˣ�����ڵ�Ϳ��Ա�ִ���ˡ�
- frame_info_��һ��֡ӳ�䣬ͼִ�й����е�֡��Ϣ��Ҫ��Ϊ�˿��ƽṹ׼���ģ����ƽṹ�ļ���ʹ��TF������һ����Ч�ļ�����������Ϊһ���������ԣ���������˵�����������и�����

���⣬�������Ҳ����������֮ǰû�м��������ֽṹ��ControlFlowInfo��FrameInfo���������ν������ǵĽṹ��
```
struct ControlFlowInfo {
    gtl::FlatSet<string> unique_frame_names;
    std::vector<string> frame_names;
};
struct FrameInfo {
    //֡����������
    int input_count;
    
    //֡�ĸ��ڵ����������������ܺ�
    int total_inputs;
    
    //���������������մ�����pending_counts���ݽṹ�У���������Ҫ�������ڴ��λ��
    PendingCounts::Layout pending_counts_layout;
    
    //ÿ��֡�����������Լ���PendingCounts��Ϣ��ֻΪ�˵�ǰ֡�еĽڵ�
    PendingCounts* pending_counts;
    
    //֡�еĽڵ㣬���ڵ���ʱʹ��
    std::vector<const Node*>* nodes;
};
```
ControlFlowInfoֻ������֡�����ƣ�ֻ�����ṩ��set��vector���ַ�ʽ��set��Ϊ�˸�����Ĳ���ĳ��֡�������Ƿ񱻰������ڡ���FrameInfo�������֡����ϸ��Ϣ����Ҫ�������������Լ�δ��ɵĽڵ��������Ϣ��
��������һЩ�����ľ���ʵ�֣�������Ӧ�þ�����ϸ�ڣ�����Щ���ݶ������ִ����������ִ��ԭ��ǳ���Ҫ������������ֱ�۽��ͣ����������롣����Ȥ�Ķ��߿���ȥ�Ķ�Դ�롣
```
//GraphView�����

//�����������ÿ��NodeItem����������������������ɾ�����ָ���Ӧ���ڴ�
GraphView::~GraphView();

//����ĳ��Node��Ӧ��NodeItem����Ҫ���ڴ��С
size_t GraphView::NodeItemBytes(cost Node *n);

//��ʼ���ڵ�
char* GraphView::InitializeNode(char* ptr, const Node* n);

//��ʼ��GraphView����Ҫ�ǳ�ʼ����node_offsets_��space_����ָ��
void GraphView::Initialize(const Graph* g);

//�����ڴ���������
Status GraphView::SetAllocAttrs(const Graph* g, const Device* device);

//ExecutorImpl�����

//��ʼ��ִ���������ȳ�ʼ��GraphView��Ȼ�󹹽�֡����Ϣ��Ԥ����ͼ��ÿ���ڵ��Ա�Ϊop����OpKernel������ʼ��PendingCounts��Ϣ
Status ExecutorImpl::Initialize();
```

## 3.4 ExecutorState
��ִ������ִ��ͼ�����ʱ����Ҫһ���ṹ�����浱ǰ����ļ�ʱ��Ϣ��TFΪ���������ExecutorState��������������ÿһ����ExecutorImpl::Run���õ�״̬��Ϣ��������һ���ڵ��Ѿ�׼����֮���������ڵ㣬���ұ���ÿ���ڵ���δ��ɵ�������Ϣ��
����������������һ�������Ľṹ��
```
class ExecutorState {
  public:
    ExecutorState(const Executor::Args& args, ExecutorImpl* impl);
    void RunAsync(Executor::DoneCallback done);
  private:
    DeviceContextMap device_context_map_;
    
    typedef gtl::InlinedVector<TaggedNode, 8> TaggedNodeSeq;
    typedef gtl::InlinedVector<Entry, 4> EntryVector;
    
    const bool vlog_;
    const bool log_memory_;
    int64 step_id_;
    
    //δӵ��
    Rendezvous* rendezvous;
    SessionState* session_state_;
    TensorStore* tensor_store_;
    //ÿ��ִ�в����������
    ScopedStepContainer* step_container_;
    StepStatesCollector* stats_collector_;
    
    checkpoint::TensorSliceReaderCacheWrapper* slice_reader_cache_;
    FunctionCallFrame* call_frame;
    const ExecutorImpl* impl_;
    CancellationManager* cancellation_manager_;
    Executor::Args::Runner runner_;
    bool sync_on_finish_;
    
    //ӵ��
    bool dumped_on_error_ = false;
    //��ǰִ�в��迪ʼ��֡
    FrameState* root_frame_;
    //ִ��������ʱ��Ҫ���õĻص�����
    Executor::DoneCallback done_cb_;
    std::atomic_int_fast32_t num_outstanding_ops_;
    mutex mu_;
    Status status_ GUARDED_BY(mu_);
    
    //��֡���Ƶ�ʵ��֡��ӳ�䡣�ڵ�ǰ֡��ĳ�����������ڣ����ܻ����һ���µ�֡���µ���֡��Ψһ��ֵ�����ɸ�֡�����ơ�������š��Լ���nodedef�ƶϳ�������֡��������϶���
    gtl::FlatMap<string, FrameState*> outstanding_frames_ GUARDED_BY(mu_);
    
    //һ��֡������
    inline string MakeFrameName(FrameState* frame, int64 iter_id, const string& name);
    
    //�ҵ�һ���ִ��֡�����ߴ���һ����֡����֡frame��iter��������
    void FindOrCreateChildFrame(FrameState* frame, int64 iter, const Node* node, FrameState** child);
    
    //ɾ��һ��֡����֡���ý���ʱʹ��
    void DeleteFrame(FrameState* frame, TaggedNodeSeq* ready);
    
    //�����Щ��Դ��֡frame�͵���iter��֡����һ����֡����ʱ����
    void CleanupFramesIterations(FrameState* frame, int64 iter, TaggedNodeSeq* ready);
    
    //�ڵ�ǰ���߳��д���һ����׼���õĽڵ�
    void Process(TaggedNode node, int64 scheduled_usec);
    
    //�ڵ���item->kernel֮ǰ��������������
    Status PrepareInputs(const NodeItem& item, Entry* first_input, TensorValueVec* inputs, DeviceContextVec* input_device_contexts, AllocatorAttributeVec* input_alloc_attrs, bool* is_input_dead);
    
    //��item->kernel�������֮�󣬴������
    Status ProcessOutputs(const NodeItem& item, OpKernelContext* ctx, EntryVector* outputs, NodeExecStats* stats);
    
    //�ڴ��������֮�󣬽����봫�ݸ���һ������
    void PropagateOutputs(const TaggedNode& tagged_node, const NodeItem* item, EntryVector* outputs, TaggedNodeSeq* ready);
    
    //�ڵ��������󣬽ӹ�stats�����ִ������򷵻�true
    bool NodeDone(const Status& s, const Node* node, const TaggedNodeSeq& ready, NodeExecStats* stats, TaggedNodeReadyQueue* inline_ready);
    
    //����ready�е����и��ӽڵ㣬Ȼ��ready�еķǸ��ӽڵ����inline_ready
    void ScheduleReady(const TaggedNodeSeq& ready, TaggedNodeReadyQueue* inline_ready);
    
    //���������Ի��¼
    inline void MaybeMarkCompleted(FrameState* frame, int64 iter, int64 id);
    
    //���һ��δ��ɻ��߻�Ծ�ڵ����Ϣ
    void DumpPendingNodeState(const int node_id, const Entry* input_vector, bool show_nodes_with_no_ready_inputs);
    void DumpActiveNodeState(const FrameState* frame, IterationState* iteration);
    
    //�ṩִ������״̬��Ϣ
    void DumpState();
    const Tensor* GetTensorValueForDump(const Entry& input);
    
    //��ִ��������ִ��ʱ������
    void Finish();
};
```
��API��������ExecutorState����������ִ������ְ�𣬴Ӻ���Ľ���Ҳ���Կ�����ʵ����ȷʵ��ˡ�ִ�����ڲ�ʵ�ʵ��õľ���ExecutorState�ڲ���API������Ľṹ�У����ǻ��ǿ��������δ����ʶ�Ľṹ������������һһ������Щ�������ͽṹ��

��������Entry��EntryҪô��һ������ָ�룬Ҫô��һ������ֵ��Ϊ����ͼ�еĽڵ�����������ṩ��һ��ͳһ�����͡�
```
struct Entry {
    Entry(const Entry& other);
    Entry& operator=(const Entry& other);
    
    void ClearVal();//���val�ֶ�
    ManualConstructor<Tensor> val;//һ��������ֵ�����val_filed_is_set��true�Ļ�
    Tensor* ref = nullptr;//һ����������
    mutext* ref_mu = nullptr;//Ϊ�����������õĻ�����
    bool has_value = false;//ֵ�Ƿ���ڣ�������val����ref
    bool val_filed_is_set = false;//val�ֶ��Ƿ�����
    
    AllocatorAttributes alloc_attr;//Ϊ��ǰ�����������ڴ���ڴ������������
    
    DeviceContext* device_context = nullptr;//�����˹������������δ������豸��ص���Ϣ
};
```

����������IterationState����������һ�ֵ�����״̬��
```
struct IterationState {
  public:
    //һ�ֵ�����״̬��ÿ�������ִζ���һ�������Ŀ��������ڵ�k�ֵ�������i���ڵ�ĵ�j��������input_tensors[k][impl_->nodes[i].input_start+j]��ע�⣬û�б�Ҫ��input_tensors�������������е�����ֻ�ᱻ�ߵ�ǰһ���ڵ�д�룬���ߵĺ�һ���ڵ��������ÿ���ߵ�ǰ�������ڵ��ǲ�����ͬʱ���е�
    Entry* input_tensors;
    
    //ÿһ�ֵ�����δ��ɵ�op����
    size_t outstanding_ops;
    
    //ÿһ�ֵ�����δ��ɵ�֡����
    int outstanding_frame_count;
    int pending(PendingCounts::Handle h);
    int decrement_pending(PendingCounts::Handle int v);
    
    //���һ��merge�ڵ�Ϊlive
    void mark_live(PendingCounts::Handle h);
    //���һ���ڵ�Ϊ����ʼ
    void mark_started(PendingCounts::Handle h);
    //���һ���ڵ�Ϊ�������
    void mark_completed(PendingCounts::Handle h);
    //��ȡ�ڵ�״̬
    PendingCounts::NodeState node_state(PendingCounts::Handle h);
    int dead_count(PendingCounts::Handle h);
    void increment_dead_count(PendingCounts::Handle h);
    void adjust_for_activation(PendingCounts::Handle h, bool increment_dead, int* pending_result, int* dead_result);
  
  private:
    PendingCounts counts_;
};
```
��������FrameState��������һ��֡��״̬������֡�͵����ִΣ������¼�����Ҫ˵����
- ���ڼ���ͼ�е�ѭ����˵��ÿ��ѭ������Ҫ����һ���µ�֡��ִ�дӵ�0��������ʼ������0��������ĳ����ֵͨ����һ��NextIteration�ڵ�ʱ����1�ֵ����ͱ���������ʼ�����ˡ�ע����ʱ��0�ֵ����������ڽ��У����Զ��ֵ������ܻ�ͬʱ�����С�֡�����˶������ݽṹ������ÿ�ֵ�����״̬������0�ֵ������������Ƕ����Ӧ��״̬�����������ա�
- һ��֡���������������붼�Ѿ������룬���еĵ��������������ʱ�����֡�ͱ���Ϊ������ˣ����Ա��������������ˡ�
- һ��֡����������ÿһ�ֵ�����״̬������������������������㣬��ô��i�ֵ����ͻᱻ��Ϊ���Ѿ�����ˣ���һ����i�ֵ����Ѿ�û��δ��ɵĽڵ��ˣ��ڶ������и��ֵĽ��ղ������Ѿ�����ˣ���������i-1������ɡ����ڵ�0�ֵ�������֡���������붼����ɣ����Ǿ���Ϊ���Ѿ������ˡ�
- ֡�͵����ִ��ڽ����󣬶�������������ա�������Ҫ�����״̬����������������Ĳ��жȸ߶���ء�����ϣ���������ܹ���̬�Ŀ���δ��ɵĲ���֡�͵�����������Ϊ�˼����ڴ����ģ�������������Ҫ���ȵ����ڲ��֡�ͽϵ͵ĵ����ִΡ�
- ֡��״̬һ����������Ҫ��ʱ��Żᱻ��ʼ�����������û������������ġ�

�������������忴��FrameState�Ľṹ��
```
struct FrameState {
    const ExecutorImpl* executor = nullptr;//֡���ڵ�ִ����
    string frame_name;//��ǰ֡�����ƣ��Ǹ�֡�������ִΣ���frame_name�ֶ�ƴ�������õ���
    uint64 frame_id;//��ǰ֡��Ψһ��ʶ
    int64 parent_iter = -1;//��֡�ĵ����ִΣ�frame_name��parent_iter��ͬ��ʶ�˵�ǰ��FrameState
    FrameState* parent_frame = nullptr;//��֡��FrameState
    const int 
    
    max_parallel_iterations;//�������Ĳ��е�������
    int num_pending_inputs = 0;//��ǰ֡��Ȼ�ڵȴ�����������
    int64 iteration_count GUARDED_BY(mu) = 0;//��ǰ֡�е���������ĵ�������
    int num_outstanding_iterations GUARDED_BY(mu) = 1;//δ��ɵĵ�������
    
    gtl::InlinedVecotr<IterationState*,12> iterations;//��ǰ֡��Ծ�ĵ���״̬
    std::vector<std::pair<const Node*, Entry>> next_iter_roots GUARDED_BY(mu);
    std::vector<std::pair<const Node*, Entry>> inv_values GUARDED_BY(mu);
    std::vector<const Node*> dead_exits GUARDED_BY(mu);
    
    //���ڵ�ǰ֡�ľ�̬��Ϣ
    PendingCounts* pending_counts = nullptr;
    int total_input_tensors = 0;
    std::vector<const Node*>* nodes = nullptr;
    
    void InitializeFrameInfo(const string& enter_name);
    inline IterationState* GetInteration(int64 iter);
    inline void SetIteration(int64 iter, IterationState* state);
    
    //����δ��ɵĲ�������������֡�еĵ�����Ϣ�����ִ֡�н����򷵻�true
    inline bool DecrementOutputstandingOps(const GraphView* gview, int64 iter, TaggedNodeSeq* ready);
    inline bool DecrementOutstandingOpsLocked(const GraphView* gview, int64 iter, TaggedNodeSeq* ready);
    
    //���֡�еļ��㶼�Ѿ�����򷵻�true
    inline bool IsFrameDone();
    //��������ļ����Ѿ������򷵻�true
    bool IsIterationDone(int64 iter);
    //���ӵ����ı�ţ������һ���µ������ͳ�ʼ����
    void IncrementIteration(const GraphView* gview, TaggedNodeSeq* ready);
    //����һ���µĵ����ִ������е�NextIteration�ڵ�
    void ActivateNexts(const GraphView* gview, int64 iter, TaggedNodeSeq* ready);
    void ActivateLoopInvs(const GraphView* gview, int64 iter, TaggedNodeSeq* ready);
    void AddLoopInv(const NodeItem* item, const Entry& value, TaggedNodeSeq* ready);
    void ActivateNodes(const NodeItem* item, const bool is_dead, int64 iter, EntryVector* outputs, TaggedNodeSeq* ready);
    bool CleanupIterations(const GraphView* gview, int64 iter, TaggedNodeSeq* ready);
};
```
������������������������ṹ�壬TaggedNode��TaggedNodeReadyQueue������TaggedNode�ǳ��򵥣�����һ��<frame*, iter, node*>�Ľṹ�壬�����߾���ǰ�ߵ�һ��Queue��������ʾ�Ѿ�׼���õĽڵ�Ķ��С�
```
struct TaggedNode {
    const Node* node = nullptr;
    FrameState* input_frame = nullptr;
    int64 input_iter = -1;
    bool is_dead = false;
    
    TaggedNode(const Node* t_node, FrameState* in_frame, int64 in_iter, bool dead);
};
class TaggedNodeReadyQueue {
  public:
    void push_back(TaggedNode node);
    void pop_front();
    bool empty();
    const TaggedNode* begin();
    const TaggedNode* end();
  private:
    gtl::InlinedVector<TaggedNode, 16> ready_;
    int front_index_;
};
```
����TaggedNodeReadyQueue������Ҫ˵��һ�£������������Ȼ�Ŀ���ʹ��std::deque�����׼��˫���б���ʵ�ֵģ�����Ϊ�ڴ���������������ͨ����û��̫��Ľڵ㣬����Ϊ��Ч������ֻʹ����һ��vector��ʵ�֣�����ʡȥ�˽ڵ����ĺ��ͷſռ���鷳��

## 3.5 details
���ڿ�Ҫ�ӽ��յ��ˡ���ǰ�������ǽ�����ô��ṹ�����ռ���ͼ��ִ�й��̾����������ģ�������Ȼ���ö�֪����Ϊ�����ʵ��ϸ�ڶ������ں�����ʵ���У�������������ȫ������̽�ֽӿڡ��������Ǿ������£������ʵ�ַ�����
���ȣ�ִ�����������Run��������������ExecutorImpl�е�Run���������ʵ�ֵİɡ�
```
void ExecutorImpl::RunAsync(const Args& args, DoneCallback done){
    (new ExecutorState(args,this))->RunAsync(std::move(done));
}
```
����֤�������������ᵽ�ģ�ExecutorImpl��Ȼֻ��һ���ӿڣ�������ִ���Ǳ��Ƶ�ExecutorState������ɵġ������������У��������ȶ�����һ��ExecutorState����Ȼ�����������RunAsync�������ڹ��캯���У����ȳ�ʼ����root_frame��iteration 0�����Ǿ��忴��RunAsync�����ʵ�ֵģ�
```
void ExecutorState::RunAsync(Executor::DoneCallback done){
    const Graph* graph = impl_->graph_;//��ȡ����ͼָ��
    TaggedNodeSeq ready;//����ready�ڵ�����
    
    //���豸����豸������ӳ��
    Device* device = impl_->params_.device;
    Status fill_status = device->FillContextMap(graph, &device_context_map_);
    if(!fill_status.ok()){
        done(fill_status);
        return;
    }
    
    //��ʼ��ready����
    for(const Node* n : impl_->root_nodes){
        DCHECK_EQ(n->in_edges().size(),0);
        ready.push_back(TaggedNode{n,root_frame_,0,false});
    }
    if(ready.empty()){
        done(Status::OK());
    } else {
        num_outstanding_ops = ready.size();
        root_frame_->iterations[0]->outstanding_ops = ready.size();
        done_cb_ = std::move(done);
        ScheduleReady(ready,nullptr);
    }
}
```
�ɼ�����Ҫ���������£���һ�ǳ�ʼ����ready queueΪ���ڵ���У��ڶ���������ScheduleReady������
��������������һ��SheduleReady���������л��ƣ�
```
void ExecutorState::ScheduleReady(const TaggedNodeSeq& ready, TaggedNodeReadyQueue* inline_ready){
    if(ready.empty()) return ;
    
    int64 scheduled_usec = 0;
    if(stats_collector_){
        scheduled_usec = nodestats::NowInUsec();
    }
    if(inline_ready == nullptr){
        //���̳߳��е��������Ѿ�׼���õ�op
        for(auto& tagged_node : ready){
            runner_([=]() {Process(tagged_node, scheduled_usec);});
        }
        return;
    }
    const GraphView& gview = impl_->gview_;
    const TaggedNode* curr_expensive_node = nullptr;
    for(auto& tagged_node : ready){
        const NodeItem& item = *gview.node(tagged_node.node->id());
        if(tagged_node.is_dead || ! item.kernel_is_expensive){
            //����������Ǹ��ӽڵ�
            inline_ready->push_back(tagged_node);
        } else {
            if(curr_expensive_node){
                //�����ӽڵ㶪�������߳�ȥ������Ϊ��ǰ�̻߳��кܶ�����Ҫ��
                runner_(std::bind(&ExecutorState::Process, this, *curr_expensive_node, scheduled_usec));
            }
            curr_expensive_node = &tagged_node;
        }
    }
    if(curr_expensive_node){
        if(inline_ready->empty()){
            //β�ݹ��Ż�
            inline_ready->push_back(*curr_expensive_node);
        } else {
            //������Ȼ�������ڵ���Ҫ���У���˰�������ӽڵ㶪�������߳�ȥ����
            runner_(std::bind(&ExecutorState::Process, this, *curr_expensive_node, scheduled_usec));
        }
    }
}
```
��������������������룬һ���Ǵ�ִ�еĽڵ���У�һ���Ǵ�ִ�е������ڵ����С�һ���������������
- ��һ�������inline_ready����Ϊ�գ���������£����ǻ�Ϊready�����е�ÿһ���ڵ㣬��������һ��ִ���̣߳���Ҳ��ExecutorState::RunAsync�������õ��Ⱥ���ʱ��ִ�з�ʽ��Ҳ����˵��ִ�е�����ǣ����ڸ�ִ�ж����еĽڵ㣬�ֱ�����һ���߳���ִ�У�
- �ڶ��������inline_ready���зǿգ���������£�������Ҫ��ȷһ�㣬���Ⱥ�����������κ�ʵ�ʵ�ִ�У�ֻ���ִ�н��з��䡣�������ready�е�ÿ���ڵ㣬�������ڵ��ǷǸ��ӽڵ���߽ڵ����������ͷ���inline_ready���д�ִ�У�����͵�������һ���߳�ִ������ͬʱ������������̽�����֮�󣬻ᱣ�����һ�����ӽڵ㣨curr_expensive_node������ʱ�����inline_ready�����ǿյģ��Ͱ�������ӽڵ�����������У�����Ϳ���һ���߳�ִ�С�

����������Process��������������ִ�еĺ��ģ�������������Ĵ������Ƚϴ���Ϊ���ǵĺ���Ŀ����˵��ִ�й��̣�����ϸ֦ĩ����ʱ��ȥ����������ܣ�
```
void ExecutorState::Process(TaggedNode tagged_node, int64 scheduled_usec){
    const GraphView& gview = impl_->gview_;
    TaggedNodeSeq ready;
    TaggedNodeReadyQueue inline_ready;
    
    //ΪOpKernel::Compute׼������
    
    inline_ready.push_back(tagged_node);
    while(!inline_ready.empty()){
        //��Ϣ��ȡ����¼����
        
        //������ڵ��Ƿ������ڵ㣬��������ڵ���send/recv���������ݴ���ڵ�ʱ����ִ������ڵ㣬���ڴ���ڵ㣬������Ҫ��dead����ֽڴ�����ȥ
        bool launched_asynchronously = false;
        if(tagged_node.is_dead & !IsTransferNode(node)){
            outputs.resize(item.num_outputs);
        } else {
            //����PreparedInputs׼������
            //���ü������
            if(item.kernel_is_async){
                AsyncOpKernel* async = item.kernel->AsAsync();
                launched_asynchronously = true;
                AsyncState* state = new AsyncState(params, tagged_node, &item, first_input, stats);
                
                auto done = [this, state](){
                    //����ProcessOutputs�������
                    //��������
                    //����PropagateOutputs�������
                    //����NodeDone����ս��
                };
                device->ComputeAsync(async, &state->ctx, done);
            } else {
                device->Compute(op_kernel,&ctx);
                //����ProcessOutputs�������
            }//ͬ���������
        }//���������Ǵ���ڵ㴦�����
        if(!launched_asynchronously){
            //��������
            //����PropagateOutputs�������
            //����NodeDone��ɨս��
        }//�����������
    }//whileѭ������
}
```
��������ѽڵ�����Ϊͬ�����첽����ֱ�����ͬ����ѭ���µĴ���ʽ��
```
graph LR
PrepareInput-->Compute
Compute-->ProcessOutput
ProcessOutput-->PropagateOutput
PropagateOutput-->NodeDone
```
�������Ƿֱ�����һ�����ĸ�������ʵ�֣�������׼�����뺯�������������
```
Status ExecutorState::PrepareInputs(const NodeItem& item, Entry* first_input, TensorValueVec* inputs, DeviceContextVec* input_device_contexts, AllocatorAttributeVec* input_alloc_attrs, bool* is_input_dead);

Status ExecutorState::ProcessOutputs(const NodeItem& item, OpKernelContext* ctx, EntryVector* outputs, NodeExecStats* stats);
```
����������û��ʲô�������ӣ�ֻ�Ƕ��������������Խ������ú���䣬�Ƚ����飬����Ȥ�Ķ��߿���ȥ����Դ�롣
�Ƚ�Ҫ�������ƹ�����ͽڵ���������������������ȿ�һ���ƹ�����ĺ�����
```
void ExecutorState::PropagateOutputs(const TaggedNode& tagged_node, const NodeItem* item, EntryVector* outputs, TaggedNodeSeq* ready){
    //��������ߴ������������׼���õĽڵ����ready����
    
    //�жϵ�ǰ�ڵ�����ͣ�ѡ����ʵĴ��������ڼ�����ActivateNodes��DecrementOutstandingOpsLocked��AddLoopInv��DecrementOutstandingOps��IncrementIteration�Ⱥ������д���
    
    //�ڵ㴦����ɺ��жϵ�ǰ֡�Ƿ�ִ����ϣ��Լ��ݹ���жϸ�֡��û��ִ�����
}
```
���У���ActivateNodes�����У�������һ���ṹ��
```
for(size_t out_index=0; out_index<num_output_edges;out_index++){
    //��������
    if(dst_ready){
        if(dst_item->is_control_trigger)
            dst_dead = false;
        ready->push_back(TaggedNode(dst_item->node, this, iter, dst_dead));
        iter_state->outstanding_ops++;
    }
}
```
Ҳ����˵���ڼ���ڵ��ʱ�򣬻�˳�ưѸýڵ�����ִ�ж���ready����һ�����Ҫ����Ϊ��Process�����У���������һ��whileѭ����inline_ready���У�Ҳ���Ǽ���ڵ㺯���е�ready���У�������ģ����տ�ʼ���������ֻ��һ���ڵ㣬��ʽ��Ϊ��PropagateOutputs�����л����ActivateNodes������������ready����ӽڵ㣬��ʹ�����whileѭ���ܹ���������
����ٿ�һ�£��ڵ�������������
```
bool ExecutorState::NodeDone(const Status& s, const Node* node, const TaggedNodeSeq& ready, NodeExecStats* stats, TaggedNodeReadyQueue* inline_ready){
    //��������
    if(s.ok()){
        ScheduleReady(ready, inline_ready);
    }
    return completed;
}
```
�о��ֻص���ͷ�ˣ����ǵøտ�ʼ�������ģ�```ExecutorState::RunAsync```�����о͵�����```ScheduleReady```������������������������ʲô���������أ�
���������б�Ҫ�������Ĺ����ܽ�һ���ˣ��������Ҳ�Ǳ�ƪ����ĵ������ˣ�
- ExecutorImpl::RunAsync��Ϊִ��������ڣ���ʵ���ǰ�ʵ��ִ�еĹ���������ExecutorState::RunAsync�����������һ��������ScheduleReady����������ִ�У���ס����������������룬ready��inline_ready�������inline_ready�ǿյġ�Ҳ����˵���������ScheduleReady�������ǣ������ڵ������Ľڵ㣬�ֱ����һ���߳�ִ�У�ִ�еĹ��̵��õ���Process������
- ��Process�����ڲ������ǻ�Ҫ��סһ�㣬�������������ֻ�нڵ㣬û��ready��inline_ready������������������Process�����ڲ��´����ġ�Ҳ����˵��һ����һ���ڵ㽻��Process����ȥ��������ڵ����ڵĶ��и�Process������û���κι�ϵ�ˡ�����Ĺ��̷�Ϊ����׼����ʵ�ʼ��㡢���׼����������ݡ��ڵ����������衣����ֻ��������ݺͽڵ���ɻ��ready��inline_ready�ṹ����Ӱ�졣
- ���ǰѽڵ�ְ����첽�ڵ��ͬ���ڵ�ֿ����������첽�ڵ㣬NodeDone���������һ������inline_ready�ǿգ�Ҳ����˵�����첽ִ��ʱ������NodeDone�е�ScheduleReadyʱ����RunAsync�е�������һ���ģ�ֱ�ӵ���ready�еĽڵ�ͺ��ˣ�����Ҫ����inline_ready�����������ͬ���ڵ㣬NodeDone���������һ������inline_ready�ǵ�ǰProcess�������´�����inline_ready��Ҳ����˵�����ݸ�ScheduleReady��inline_ready�Ƿǿյģ���Ҳ���п��ܶ�inline_ready�Ľṹ���޸ģ�ע�������inline_ready�Ǵ�Process�����д����ģ�ÿ��Process��������Ӧһ��ȫ�µ��̣߳�Ҳ����˵��ÿ��ȫ�µ��߳�����ֻ��һ��inline_ready�ṹ�����еĺ������ϵ��޸��������ݣ�Ȼ�󲻶ϵĶ������е���ִ�С�ע��Process�е�while��ѭ�������inline_ready����ִ�еġ�