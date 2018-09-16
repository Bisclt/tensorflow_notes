��common_runtime��ʣ������ݣ������ļ�����������˼򵥵Ľ�����ʱ��ԭ��д�ĺִܲ٣�����ռ���ӣ����������µ�����������䡣

## allocator_retry
��ʱ���ڴ���䲻����һ����ɣ�Ϊ�����ڴ����ʧ��ʱ�ܹ����ϳ��ԣ�TF������һ�����ڴ�����γ��ԵĽӿڣ�
```
class AllocatorRetry {
  public:
    AllocatorRetry();
    void* AllocateRaw(std::function<void*(size_t alignment, size_t num_bytes, bool verbose_failure)> alloc_func, int max_millis_to_wait, size_t alignment, size_t bytes);
    void NotifyDealloc();
  private:
    Env* env_;
    mutex mu_;
    condition_variable memory_returned_;
    
}
```
���У�ͨ������alloc_func����ȡ�ڴ档�״ε���ʱ��verbose_failure�ᱻ����Ϊfalse���������ָ��Ϊ�գ���ô�ȴ�max_millis_to_wait���룬֮��ÿ����⵽DeallocateRaw()����������ʱ���ͳ������������ڴ棬ֱ���ڴ�����ɹ������ߵ���deadline�����deadline���ˣ���verbose_failure����Ϊtrue���ٳ���һ�Ρ�

## bfc_allocator
�������TF��ʹ�õ�BFC��best-fit with coalescing���ڴ�����㷨������Doug Lea���ڴ�����㷨��dlmalloc���ļ򻯰汾��

��ͼ����Ĺ����У���Ҫ������ͷŴ������ڴ棬�Ӷ������ܶ��ڴ���Ƭ�����ʹ��Ĭ�ϵ��ڴ�����㷨���������ڴ���Ƭ�ή���ڴ�ʹ�õ�Ч�ʡ����BFC�㷨��һ��Ŀ����ǣ����ڴ����ͻ��չ����в�������Ƭ���л��ա�

���ĺ������ݽṹ���������ֱ���chunk��bin��chunk����ÿһ�������ڴ棬Ϊ����׷�ٵ��κ�һ���ڴ��ʹ��״̬����һ��˫������chunk������������Щchunk��Щ���Ѿ���ʹ�õģ���Щ��δ��ʹ�õġ����ڴ����ʱ��Ϊ����δ��ʹ�õ��ڴ�����ѡ�����ʴ�С���ڴ�飬ʹ��bin�ṹ��ָ����С�ڴ���п��ټ�����

����ģ�ÿ��bin�����˴�С����ĳ�������ڵ�����δʹ�õ��ڴ�飬���ǰ����ڴ���С�����˳���γ���һ�����������ÿ��binά�����ڴ��Ĵ�С����2^n��2^n+1�ķ�Χ�ڣ����ֽṹ�����˶�δʹ���ڴ��������ɾ���Ͳ��롣

���е�chunk���ӳ�һ��˫�������������ͨ��bin�ṹ�ҵ���һ�����ʴ�С��chunkʱ�������Ĵ�С�����жϣ��������������Ҫ���ڴ��С����������ô�ͶԸ�chunk����split������һ�鷵��ʹ�ã�����һ�����bin�ṹ��ͬʱ��chunk�ṹ���е������ָ�˫����ṹ��������ڴ汻���գ���ô�жϻ��յ�chunk��ǰ���ͺ����Ƿ�Ҳ�п��ڴ棬����ǣ���ô������chunk�ϲ���ά��˫����ṹ��ͬʱ����chunk����bin�ṹ�ڡ�

�����ֱ��������ޣ���������ͼ��

## build_graph_options
��executor���ᵽ����ʱ�����ǲ�����Ҫ����������ͼ����ֻ��Ҫ�������е�һ���֡������Ҫ���ݸ����������������������ͼ������Ҫ���е���ͼ�����������ͼ����ͼ�Ĺ��̣�����Ҫ�ο����BuildGraphOptions��Ϣ������Ҫ���������������Ľڵ㡣
```
struct BuildGraphOptions {
    std::vector<string> feed_endpoints;
    std::vector<string> fetch_endpoints;
    
    //���Ϊtrue��ʹ��FunctionCallFrame�ṹ��Ӧ����������������ʹ��Redezvous�ṹ��Ӧ��
    bool use_function_convention = false;
}
```

## constant_folding
�����۵�����TFͼ�Ż��е�һ���ֶΡ�����˵�����ͼ�е�ĳ���ڵ�������ڳ�������ô����ڵ��ֵ��ͼ����֮ǰ����ȷ����������ǿ�����ͼ����֮ǰ������CPU�豸�Ͻ���Щ�ڵ��ֵ���������

## copy_tensor
ͨ����������ע�����豸��������ݿ����ĺ���������ͨ��DMA�ķ�ʽ��

## costmodel_manager
����Ϊ�Ի�����cost model���ڲ�������һ����Graph�����Ӧ��CostModel��ӳ�䡣
```
class CostModelManager {
  public:
    typedef std::unordered_map<const Graph*, CostModel*> CostModelMap;
  private:
    CostModelMap cost_models_ GUARDED_BY(mu_);
}
```

## debugger_state_interface
��������Ҫ�Լ���ͼ����debug��ʱ����Ҫ�ڼ���ͼ�в���һЩ��debugΪĿ�ĵĶ���ڵ㣬Ȼ������Ҫʱ���ͼ�ļ�ʱ��Ϣ������TF�Ƴ��������ṹ��һ����DebugGraphDecorator������Ϊ�˶�ԭͼ�����޸ģ�����һЩdebug�ڵ㣬��һ����DebuggerState������Ϊ�˴洢debug��Ϣ�����ṩ�����ṹ�����debug��Ϣ���м����������׽ṹʵ����һ�ֲ��еĽṹ��ϵ��������Interface��Registry��Factory�������ݡ�����Factory��һ�����������ڲ���Interface����Registry����������Factory��ע����������DebuggerState��DebugGraphDecorator��˵�����ǵ�ע������ֻ��һ����̬��ע�����
```
class DebuggerStateRegistry {
  public:
    //...
  private:
    static DebuggerStateFactory* factory_;
};
class DebugGraphDecoratorRegistry {
  public:
    //...
  private:
    static DebugGraphDecoratorFactory* factory_;
}
```
���ǵĽṹͼ���£�
```
graph TB
    DebuggerStateRegistry-->|����|DebuggerStateFactory
    DebuggerStateFactory-->|����|DebuggerStateInterface
    DebugGraphDecoratorRegistry-->|����|DebugGraphDecoratorFactory
    DebugGraphDecoratorFactory-->|����|DebugGraphDecoratorInterface
```

## dma_helper
��TF�ڲ�ʹ�õ�һЩDMA����������

## eigen_thread_pool
�̳߳أ����������ǵ�������һ�����������������Ƕ�CPU��һ������Eigen���а�����һ���̳߳ؽӿڣ�����TF�����һ����Eigen���̳߳ؽṹ�ķ�װ�࣬EigenThreadPoolWrapper���ṹ���£�
```
class EigenThreadPoolWrapper : public Eigen::ThreadPoolInterface {
  public:
    void Schedule(std::function<void()> fn) override {
        pool_->Schedule(std::move(fn));
    }
    int NumThreads() const override { return pool_->NumThreads(); }
    int CurrentThreadId() const override { return pool_->CurrentThreadId(); }
  private:
    thread::ThreadPool* pool_ = nullptr;
};
```

## gpu/gpu_device
������Ҫ�����������ࣺ
```
graph TB
    LocalDevice-->|����|BaseGPUDevice
    DeviceFactory-->|����|BaseGPUDeviceFactory
```

## gpu_device_context
GPU�������ģ�����DeviceContext�࣬�����˸���stream��Ϊ˽�����ݳ�Ա��
```
class GPUDeviceContext : public DeviceContext {
  public:
    //...
  private:
    int stream_id_;
    //��ǰ������ʹ�õ�Ĭ����stream�����е��ڴ涼�������stream
    gpu::Stream* stream_;
    //��host�������ݵ�GPU��stream
    gpu::Stream* host_to_device_stream_;
    //��GPU��host�������ݵ�stream
    gpu::Stream* device_to_host_stream_;
    //��GPU֮�俽�����ݵ�stream
    gpu::Stream* device_to_device_stream_;
};
```
Ŀǰ���ǰ�stream���Ϊ���豸֮�俽�����ݵĹ����壬���������µ�����ٹ������䡣

## graph_runner
����һ��ͼ��ָ������������GraphRunner����ִ������ͼ������õ���������������ڲ�ʹ�ã����������ֲ��ļ���ͼ�еķǸ��ӽڵ㣬������״�ƶϻ��߳����۵����������ľ����ԣ������еļ��㶼����CPU�Ͻ��У����Ҳ��߱������������ٺ͸�Ч���ص㡣
```
class GraphRunner {
  public:
    GraphRunner(Env* env);
    typedef std::vector<std::pair<string, Tensor>> NamedTensorList;
    Status Run(Graph* graph, FunctionLibraryRuntime* function_library, const NamedTensorList& inputs, const std::vector<string>& output_names, std::vector<Tensor>* outputs);
  private:
    std::unique_ptr<Device> cpu_device_;
};
```

## local_device
LocalDevice�౻ThreadPoolDevice��GPUDevice��������ʼ����һ�������Eigen�����豸�������߹��á���������ս��ᱻɾ�������ǻὫThreadPoolDevice��GPUDevice�ع�Ϊ���̼���ĳ���
```
class LocalDevice : public Device {
  public:
    //...
  private:
    static bool use_global_threadpool_;
    static void set_use_global_threadpool(bool use_global_threadpool);
    
    struct EigenThreadPoolInfo;
    std::unique_ptr<EigenThreadPoolInfo> owned_tp_info_;
};
```
������ת��deviceƪ���£������һ���ṹͼ��

## memory_types
�ڴ�������صĸ���������
```
//����һ�Ž�������device_type�豸�����ϵ�ͼg��˵���������ĳ���ߵ�Դ�ڵ����Ŀ�Ľڵ�����˷Ǹ��豸�ϵ��ڴ棬�򷵻ش���
Status ValidateMemoryTypes(const DeviceType& device_type, const Graph* g);

//ͨ��������ʵ�HostSend/Recv��Send/HostRecv�ķ�ʽ����ͼg���е�����ʹ������ÿ���ߵ�Դ��Ŀ�Ķ���device_type����
Status EnsureMemoryTypes(const DeviceType& device_type, const string& device_name, Graph* g);

//��ȡ�ڵ�n�ĵ�index��������ڴ�����
Status MemoryTypeForOutput(const DeviceType& device_type, const Graph* g, const Node* n, int index, MemoryType* memory_type);
```

## mkl_cpu_allocator
һ���򵥵�CPU�ڴ�����������򵥵Ľ�����MKL����ڴ�����/�ͷ������ض����TF��BFC�ڴ��������

## renamed_device
���ض�������£��豸�����ƻᷢ���仯��TF�����һ�����������豸�࣬��Ӧ�����ֱ仯��������һ���µ���������װһ���豸�����ҽ����еĹ���������������豸��
```
class RenamedDevice : public Device {
  public:
    //...
  private:
    RenamedDevice(Device* underlying, const DeviceAttributes& attributes, bool owns_underlying);
    Device* const underlying_;
    const bool owns_underlying_;
};
```

## rendezvous_mgr
������һ������IntraProcessRendezvous������ʾһ�����е������ߺ������߶���ͬһ�������ڲ���Rendezvous��Ҳ����˵���������ߺ�������֮�����ͨ�ţ�����ҪRPC������ֵ�Ĵ洢������һ���ֲ���Rendezvous���������ֻ��������һЩЭ�����н����ڲ��豸�����ݴ���ĸ����ӿڡ�
```
class IntraProcessRendezvous : public Rendezvous {
  public:
    explicit IntraProcessRendezvous(const DeviceMgr* device_mgr);
    Status Send(...);
    void RecvAsync(...);
  private:
    const DeviceMgr* device_mgr_;
    Rendezvous* local_;
    //...
};
```

## shape_refiner
ShapeRefinerΪTFͼ������״�ƶϡ�������Ϊͼ�е�ÿһ���ڵ�ʵ����һ��InferenceContext����Ȼ���ṩ/�洢input_tensor����״�ƶϺ���ʹ�ã�������̷�����ͼ����ʱ��

## simple_graph_execution_state
һ����ִ�е�ͼ����GraphDef���������ڣ�ǰ�ߵĽڵ��Ǳ������˵ģ�����ִ�е�ͼ�Ľڵ��������ȷ�ķ����豸��SimpleGraphExecutionState������þ��ǣ����Ȱ���ԭͼ�жԽڵ����λ�õ�����Ҫ�󣬶�ͼ�еĽڵ������ȫ�ķ��ã�Ȼ�󣬸���BuildGraphOption�ṹ�����ݣ���ԭͼ�в��г�һ����Ҫ���е���ͼ�������ͼ����SimpleClientGraph��ʾ�������µ�����ͼ��ʾ��
```
graph TB
    CompleteGraph-->|�ڵ�����㷨|PlacedGraph
    PlacedGraph-->|����BuildGraphOption��Ϣ|SimpleClientGraph
```
���濴����Ҫ�ṹ��
```
struct SimpleGraphExecutionStateOptions {
    const DeviceSet* device_set = nullptr;
    const SessionOptions* session_options = nullptr;
    //һ���ӽڵ����Ƶ��豸���Ƶ�ӳ�䣬�����˲��ܱ��ı�Ľڵ����ѡ��
    std::unordered_map<string, string> stateful_placements;
};

//SimpleClientGraph��ȫͼ��һ����ͼ����ͼ������ͨ��BuildGraphOptions�ƶ�
struct SimpleClientGraph {
    explicit SimpleClientGraph(...);
    //ÿ���ͻ���ͼ����һ���Լ��ĺ����⣬��Ϊ�Ż������п��ܻ��ͼ������д���Ӷ�����һЩ�µĺ���
    std::unique_ptr<FunctionLibraryDefinition> flib_def;
    Graph graph;
    DataTypeVector feed_types;
    DataTypeVector fetch_types;
};

class SimpleGraphExecutionState {
  public:
    //...
  private:
    std::unordered_map<string, string> stateful_placements_;
    Status OptimizeGraph(const BuildGraphOptions& options, std::unique_ptr<Graph>* optimized_graph);
    GraphDef original_graph_def_;
    const DeviceSet* device_set_;
    const SessionOptions* session_options_;
    mutable mutex mu_;
    CostModel costs_ GUARDED_BY(mu_);
    
    //�ڵ����Ƶ�ȫͼ�еķ���λ�õ�ӳ��
    NodeNameToCostIdMap node_name_to_cost_id_map_;
    
    //flib_def_ʹ��ԭʼͼ�е�def��ʼ������������ͼ�Ż������Ľ��У����ܼ����µĺ���
    std::unique_ptr<FunctionLibraryDefinition> flib_def_;
    
    //����ǰ����ӵ�е�������ͼ
    Graph* graph_;
};
```

## simple_placer
һ���򵥵Ľڵ�����㷨���ڸ���ͼ�Լ��ɷ��õ��豸֮��ȷ��ÿ���ڵ���õ��豸�����ǵ��������£�
- �Ѿ������Ľڵ����Լ�����ܸı䣻
- �ڵ�����ģ��������������ȫ���󣩵��豸�������ƣ��������㣻
- ���������͵ı�������һ��Ľڵ㣬���뱻������ͬһ���豸�ϣ�
- �����ڵ�A��B������ڵ�B��һ�����õ���@loc:A����ô�ڵ�A��B���뱻������һ̨�豸�ϣ�
```
class SimplePlacer {
  public:
    typedef std::unordered_map<string, int> NodeNameToIdMap;
    //...
  private:
    Graph* const graph_;
    const DeviceSet* const devices_;
    const SessionOptions* options_;
    const bool log_device_placement_;
};
```

## stats_publisher_interface
StatsPublisherInterface������һ�������Ự������Ϣ�Ķ��󡣵�ǰ����������״̬��
```
class StatsPublisherInterface {
  public:
    //����step_stats
    virtual void PublishStatsProto(const StepStats& step_stats) = 0;
    
    //����ÿ���ָ���ͼ��graph_defs
    virtual void PublishGraphProto(const std::vector<const GraphDef*>& graph_defs) = 0;
    
    //����execution_count��RunOptionsΪһ��������step����һ��profile handler
    virtual std::unique_ptr<ProfileHandler> GetProfileHandler(uint64 step, int64 execution_count, const RunOptions& ropts) = 0;
};
```

## step_stats_collector
StepStatsCollector�������һ��StepStats����ļ��ϡ����ǵ�֮ǰ��StepStats�ķ�����StepStats�����˶��DeviceStats����ÿ��DeviceStats�����ְ����˶��NodeExecStats��
```
class StepStatsCollector {
  public:
    void BuildCostModel(CostModelManager* cost_model_manager, const std::unordered_map<string, const Graph*>& device_map);
    void Save(const string& device, NodeExecStats* nt);
    void Swap(StepStats* ss);
  private:
    StepStats* step_stats_ GUARDED_BY(mu_);
    uint64 collectedNodes GUARDED_BY(mu_) = 0;
};
```

## threadpool_device
ThreadPoolDevice�����CPU�豸��ʵ�֡�
```
class ThreadPoolDevice : public LocalDevice {
  public:
    ThreadPoolDevice(...);
    void Compute(OpKernel* op_kernel, OpKernelContext* context) override;
    Allocator* GetAllocator(AllocatorAttributes attr) override;
    Status MakeTensorFromProto(const TensorProt& tensor_proto, const AllocatorAttributes alloc_attrs, Tensor* tensor) override;
    Status Sync() override { return Status::OK(); }
  private:
    Allocator* allocator_;
};
```

## visitable_allocator
���һ���ڴ��������Ҫ�Է�����ڴ����һЩע��/��ע��Ĳ�������ô���������̳�VisitableAllocator��������ֱ�Ӽ̳�Allocator����Ϊǰ��Ϊע��/��ע���ṩ�˽ӿڡ�
```
class VisitableAllocator : public Allocator {
  public:
    typedef std::function<void(void*, size_t)> Visitor;
    virtual void AddAllocVisitor(Visitor visitor) = 0;
    virtual void AddFreeVisitor(Visitor visitor) = 0;
};
```
���ǵ�֮ǰ�����TrackingAllocator�������Ա���ÿһ�η�����ڴ����Ϣ��������Ǽ���Ҫע��/��ע���ڴ棬����Ҫ����������ѵ��Ϣ�أ������Ҫһ�����ϵ���TrackingVisitableAllocator�������õ��˶��ؼ̳�����C++���е����ԣ����ؼ̳��������ǿ���ʹ�õģ���ΪVisitableAllocator����һ����Ľӿڣ�ֻ��TrackingAllocatorӵ��Ĭ�ϵ�ʵ�֡�
```
class TrackingVisitableAllocator : public TrackingAllocator, public VisitableAllocator {
  public:
    //...
  protected:
    VisitableAllocator* allocator_;
};
```
����һ���ڴ������ƪ�£����һ���ܹ�ͼ��