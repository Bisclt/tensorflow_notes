# Ŀ¼
1. ʲô��kernel
2. kernel_def
3. op_kernel
4. kernel��ע��
4. op_segment
5. ��ϵͼ
6. �漰���ļ�
7. ������¼

# 1. ʲô��kernel
���˵op�൱�ڲ�������������ôkernel���ǲ�����ʵ�֡�ͬһ�������ڲ�ͬ���豸�ϣ����ŵ�ʵ�ַ�ʽ�ǲ�һ���ģ��������MatMul������������������CPU�Ͽ�����SSEָ���Ż����٣���GPU�Ͽ�����GPUʵ�ָ����ܼ��㡣��˾ͻ��ӦCPU��GPU���ֲ�ͬ��ʵ�֡����ԣ��ڶ���һ��kernelʱ������Ҫָ��kernel��Ӧ��op֮�⣬����Ҫָ��kernel���ڵ��豸���͡�

���⣬kernel��һ�������ڵĸ���ڶ���ͼʱ��ÿ���ڵ��op����֪�����������ĸ�kernelʵ����������ģ���Ϊ��ʱ�ڵ㻹û�б����䵽������豸�ϣ����Ҳ��û��Ϊ��ѡ����ʵ�kernel��

# 2. kernel_def
��Ȼkernel��һ�������ڵĸ����������Ȼ��Ҫ��һЩ��̬����Ϣ��kernel���������������̬����Ϣ����KernelDef���proto���������£�
```
message KernelDef {
    string op = 1;//��Ӧ����������
    string device_type = 2;//��Ӧ�豸������
    message AttrConstraint {
        string name = 1;
        AttrValue allowed_values = 2;
    }
    repeated AttrConstraint constraint = 3;//��Ӧop�жԲ���������
    repeated string host_memory_arg = 4;//�������������������У�������host�ڴ������device�ڴ��еĲ���
    string label = 5;
}
```
���е�label�ֶ����ǽ���һ�¡���ʱ���û����дһЩʵ���Ե�kernel��Ȼ��ע�ᵽĳһ��op��ȥ��������kernelĬ��������ǲ��ᱻʹ�õģ������û��û���op�ж�����һ��_kernel�ֶΣ����Ұ�����ֶθ�ֵΪĳ��kernel��label��Ӧ��ֵ���ٸ����ӣ�������Ƕ�����һ��MulKernel���Ұ�����label����Ϊ'mulkernel'���������kernel��Ӧ�Ĳ�������MulOp����ôֻ��MulOpҲ����һ���ֶ�_kernel������_kernel="mulkernel"ʱ��MulKernel�ſ��Ա�MulOpʹ�á�

���չ�����TFҲ��ΪKernelDef���һ�������࣬�����KernelDefBuilder����֮ǰ�Ĺ��������ƣ�KernelDefBuilderҲֻ���ṩ��һϵ�е���������API��˽�����ݳ�ԱҲֻ��һ��KernelDefָ�룺
```
class KernelDefBuilder {
    //...
  private:
    KernelDef* kernel_def_;
}
```

# 3. op_kernel
## 3.1 OpKernel
���˵KernelDefֻ�Ƕ�kernel�ľ�̬��Ϣ�������������Ҫ����kernel�ı����ˣ�OpKernel����������kernel�Ļ��࣬��������KernelDefһ��������ֵ����֮�⣬���ṩ��kernel�ĺ���API��compute������������������ϸ��һ��OpKernel��Ľṹ��
```
class OpKernel {
  public:
    explicit OpKernel(OpKernelConstruction* context);
    virtual void Compute(OpKernelContext* context) = 0;//ִ��ͬ������
    virtual AsyncOpKernel* AsAsync() { return nullptr; }
  private:
    const std::unique_ptr<const NodeDef> def_;
    const DataTypeVector input_types_;//�������������
    const MemoryTypeVector input_memory_types_;//������ڴ�����
    const DataTypeVector output_types_;//�������������
    const MemoryTypeVector output_memory_types_;//������ڴ�����
    const bool is_internal_;//�Ƿ����ڲ�����
    NameRangeMap input_name_map_;
    NameRangeMap output_name_map_;
    bool expensive_;//�Ƿ��Ǹ��Ӳ���
}
```
���¶���������һЩ˵����
- �ڹ��캯���У����Ƿ�������Ҫ����һ��OpKernelConstructionָ�룬���ǲ��룬���ָ��Ӧ�ð����˹���һ��OpKernel����������ݳ�Ա�͹��ܺ��������ǽ�����������ϸ���ܣ�
- ���⣬�ں���API Compute�У����յ���һ��OpKernelContextָ��Ĳ��������ǲ��룬���ָ��Ӧ�ð�����OpKernelִ��ʵ�ʼ�������Ҫ�����ݳ�Ա�͹��ܺ�����������Ҳ��ר�Ž�������ࣻ
- ����֮�⣬���ǻ�������һ��ָ��NodeDef��ָ�롣֮ǰ˵���ˣ�kernel��һ�������ڵĸ����Ȼkernel��op�ľ���ʵ�֣����������У�op�Ǹ�����Ľڵ����һ��ģ�����ÿ��kernel����Ҫ��һ������Ľڵ㣻
- kernel���Ա���Ϊͬ��kernel���첽kernel���࣬�󲿷ֵ�kernel��Ӧ����ͬ���ģ�Compute�����ڼ�������󷵻ؽ��״̬������Щ������������첽�ģ������Ҫ��ĳЩkernel���Ϊ�첽�ģ������������ݽ��ղ�������������������Ƴ�ͬ������ô����������߳���ʹ��ͬһ��������շ��񣬵�ǰ�߳̾ͻᱻ�������Ӷ������Դ���˷ѡ�

�ղŽ������첽kernel���������Ǿ�����һ���첽kernel��Ӧ���࣬AsyncOpKernel��
```
class AsyncOpKernel : public OpKernel {
  public:
    typedef std::function<void()> DoneCallback;
    virtual void ComputeAsync(OpKernelContext* context, DoneCallback done) = 0;
    //...
};
```
�ɼ����첽�����API����context֮�⣬����Ҫ�ṩһ���ص����������첽����ִ�н���֮����á�

## 3.2 OpKernelConstruction
�ղ������ᵽ��OpKernel�Ĺ��캯���У���Ҫһ������ΪOpKernelConstructionָ��Ĳ��������Ҳ����������������OpKernel��������������ݳ�Ա�͹��ܺ�����ʵ����Ҳȷʵ��ˣ������������������Ľṹ��
```
class OpKernelConstruction {
  public:
    Status allocate_temp(DataType type, const TensorShape& shape, Tensor* out_temp);//����һ����ʱ�ڴ�
    Status allocate_persistent(DataType type, const TensorShape& shape, PersistentTensor* out_persistent, Tensor** out_tensor);//����һ��ɸ����ڴ�
    //...
  private:
    const DeviceType device_type;
    DeviceBase* const device_;
    Allocator* allocator_;
    const NodeDef* def_;
    const OpDef* op_def_;
    FunctionLibraryRuntime* flib_;
    DataTypeSlice input_types_;
    MemoryTypeSlice input_memory_types_;
    DataTypeSlice output_types_;
    MemoryTypeSlice output_memory_types_;
    const int graph_def_version_;
    Status* status_;
}
```
�����м�����Ҫ˵����
- ������ʱ�ڴ�Ϳɸ����ڴ档��OpKernel�����Ĺ����У����ǿ�����Ҫ����һЩ�ڴ棬��Щ�ڴ�����ʱ�Եģ���OpKernel��������֮���û���ˣ����ǻ���������������ͷš����⣬����Ҳϣ����Щ�ڴ��ǿ�����OpKernel�Ķ��ִ��֮�乲��ģ����磬��Щkernel����״̬�ģ�����Variable�����Ǳ�����kernel����ʱ�͸���Щ���������ڴ棻
- ���ڿɸ����ڴ棬����һ����Ҫ˵����������GPU������Ŀɸ����ڴ棬����GPU����CPU�����ڴ淽������������ʱ��Ҫ��ÿһ���ڴ��ʹ���������ָ�ƣ���˶��ڿɸ��õ��ڴ棬���Ǳ������ÿһ��ʹ�ö��˽⡣TFΪ��ר�������һ��PersistentTensor�࣬������Ƕ�Tensor�ķ�װ���������ڲ���������ֻ��ͨ��һ��AccessData�Ľӿ������ʣ�ֻҪ����������ӿ�������һ��Watcher�����ܼ�����пɸ����ڴ��ʹ���ˣ�
- ����PersistentTensor����һ���ɵ㣬����֪��OpKernelConstruction���ܹ���OpKernel��ʼ��ʱ���������ڴ棬�������������У���û�з��ֶ����Ĵ洢����Ȼ�����ڴ�ʱ���Թ�kernel�ڲ�ͬ����֮�乲��ģ����ڵڶ�������ʱ�����ҵ���������أ����ǵ���resource�½��У����ǽ��ܵ�resource_op_kernel�����kernelʵ���Ͼ��Ǵ洢���������ĵط�����������һ����������֮������Ϊ֮����һ���µ�resource_op_kernel������������Ҫ�������ʱ���������kernel��ȡ������������ǣ�����Ϊresource_op_kernel����һ���µĽڵ㣬�����½ڵ���ԭkernel���ڽڵ�֮������һ���ߣ�ʹ�½ڵ���Ϊ�Ͻڵ�����룻
- ˽�������л���һ��FunctionLibraryRuntime�ṹ��ָ�룬����˼�壬����ṹ��ʾһ������ʱ�ĺ����⣬���ǽ���function�½�����ϸ������

## 3.3 OpKernelContext
���ǵ����Ǹղ��ᵽ��OpKernel�ĺ���API Compute��������Ҫһ������ΪOpKernelContextָ�����������𣿸ղ����ǲ��룬����������ִ��kernel��������Ҫ�����ݳ�Ա�͹��ܺ�����ʵ����Ҳȷʵ��ˡ��������������������Ĺ��ɣ�
```
class OpKernelContext {
  public:
    //���캯��
    explicit OpKernelContext(Params* params);
    
    //�����ȡ
    const Tensor& input(int index);//��ȡ���ɱ����������
    Status input(StringPiece name, const Tensor** tensor);//��ȡ���ɱ����������
    Status input_list(StringPiece name, OpInputList* list);//��ȡ���ɱ�����������б�
    Status input_ref_mutex(StringPiece name, mutex** out_mutex);//��ȡ�ɱ����������
    Tensor mutable_input(int index, bool lock_held);//��ȡ�ɱ����������
    Status mutable_input_list(StringPiece name, OpMultableInputList* list);//��ȡ�ɱ�����������б�
    void replace_ref_input(int index, const Tensor& tensor, bool lock_held);//�滻ĳ����������
    Status replace_ref_input(StringPiece name, const Tensor& tensor, bool lock_held);
    
    //�������������
    //����������ת��Ϊ�������
    void forward_ref_input_to_ref_output(int input_index, int output_index);
    //������ת��Ϊָ����״�����
    bool forward_input_to_output_with_shape(int input_index, int output_index, const TensorShape& output_shape, Tensor** output);
    Status forward_input_to_output_with_shape(StringPiece input_name, StringPiece output_name, const TensorShape& output_shape, Tensor** output);//ͬ��
    //���ָ������1.�����������룬2.���������������һ�£�3.�ײ��buffer�����ü���Ϊ1����ô�ͷ���һ��ָ�������ĵײ����ݵ�ָ��
    std::unique_ptr<Tensor> forward_input(int input_index, DataType dtype, const TensorShape& shape, MemoryType memory_type, const AllocatorAttributes& attr);
    //���԰�ָ�����봫�ݵ�ָ����������û���κ�һ��������Ա����ݣ���ô������һ���µ��ڴ���Ϊ���
    Status forward_input_or_allocate_output(gtl::ArraySlice<int> candidate_input_indices, int output_index, const TensorShape& output_shape, Tensor** output);
    //���԰�ָ������������ʱ���������û��������ã���ʹ��allocate_temp����һ����ʱbuffer
    Status forward_input_or_allocate_temp(gtl::ArraySlice<int> candidate_input_indices, DataType type, const TensorShape& shape, const AllocatorAttributes& allocator_attr, Tensor* out_temp);
    
    //�����ȡ
    Status output_list(StringPiece name, OpOutputList* list);
    
    //�ڴ����
    Status allocate_output(int index, const TensorShape& shape, Tensor** tensor);
    Status allocate_output(int index, const TensorShape& shape, Tensor** tensor, AllocatorAttributes attr);
    Status allocate_temp(DataType type, const TensorShape& shape, Tensor* out_temp, AllocatorAttributes allocator_attr, const AllocationAttributes& allocation_attr);
    Status allocate_persistent(DataType type, const TensorShape& shape, PersistentTensor* out_persistent, Tensor** out_tensor, AllocatorAttributes attr);
    
    //�������
    Status set_output(StringPiece name, const Tensor& tensor);
    Status set_output_ref(StringPiece name, mutex* mu, Tensor* tensor_for_ref);
    Status mutable_output(StringPiece name, Tensor** tensor);
    
    //...
    
  private:
    Status status_;
    Params* params_;
    mutable mutex mu_;
    gtl::InlinedVector<WrappedAllocator, 4> wrapped_allocators_ GUARDED_BY(mu_);
    gtl::InlinedVector<TensorValue,4> outputs_;
    ManualConstructor<UniqueTensorReferences> referenced_tensors_ GUARDED_BY(mu_);
    bool is_output_dead_ = false;
    int64 host_temp_memory_size_;
    int64 device_temp_memory_size_;
    gtl::InlinedVector<int64, 2> host_persistent_alloc_ids_;
    gtl::InlinedVector<int64, 2> device_persistent_alloc_ids_;
    int64 host_persistent_memory_allocated_;
    int64 device_persistent_memory_allocated_;
}
```
Ϊ�˿�����ṹ������������ɾ����������ˣ���������Ȼ�ܴ󡣶����API�����ˣ���ȡ���룬���봫�ݵ������������ã������ȡ���ڴ���䣬���������ص㽲�������ĸ��
- �����������������ͣ����ſ��������Ѿ���ӡ���ˣ�������Ϊ����������ƣ�������Ϊ���࣬����������������롣���������ǲ��ɸı�ģ������������ǿ��Ըı�ġ��������Ϊ���������ı��������������������ڲ����ݣ�ֻ���½�һ����������������������ݿ���������Ȼ��ı�������������ݣ��������������룬�Ϳ���ֱ�Ӷ����޸ģ���ʱ�򻹿��԰��޸ĺ����������ֱ�ӵ����������������ȡ��API����е�API�����������������������������汾���������뵽��������API�Ҳר����һ���������뵽��������Ĵ��䣻
- �����ڴ���䡣��kernelִ���ڼ䣬�����ַ����ڴ�ķ�ʽ����һ���Ƿ��������ڴ棬Ҳ�����������ᵽ�Ŀɸ����ڴ棬��ΪĳЩ��������״̬�ģ���ͬһ��kernel�Ķ�ε���֮�䣬���ǿ��Ա���һЩ�������ݡ��ڶ����Ƿ�������ڴ棬һ��kernel���ܻ�������ݣ�������ݱ����������ڴ档�������Ƿ�����ʱ�ڴ棬kernel�����п��ܻ��õ�һЩ��ʱ���ڴ棬�������֮��Ͳ����ˣ�
- ��ĳЩ����£�һ���������㲻�Ǳ�����Ϊ�����Ҳ���ܻᱻ�������ʹ�á��������������һ�����룬�����Ǵ洢��һ�����������У������ھ�������ĸ�������û��ȷ��֮ǰ�ͱ������ˡ���������£����ǿ���ʹ��set_output����set_output_ref������ָ�������������������������ǿ���ʹ���κ�֮ǰ�����������Ϊ�����������������Ǳ�������ʱ��������ġ�ʹ����Щ���Ǳ�allocate_output������������������������һ����������ģ���Ϊallocate_outputʹ���˴洢��output_attr_array�е�AllocatorAttributes�������������������ڴ棬���ʹ�õ�����������ڴ�����Ҫ�󲻷������ܻ��������������ڴ濽����

��OpKernelContext��˽�����ݳ�Ա�У�����һ��Params��������û�н��ͣ���������������Ľṹ��
```
struct Params {
    int step_id = 0;//ִ�еĲ�����
    OpKernel* op_kernel = nullptr;
    DeviceBase* device = nullptr;//��kernelִ��ʱ���ڵ��豸����
    PerOpGpuDevice* eigen_gpu_device = nullptr;
    
    //׷�����
    bool track_allocations = false;
    bool log_memory = false;
    bool record_tensor_accesses = false;
    
    const AllocatorAttributes* output_attr_array = nullptr;//������ڴ����������
    ResourceMgr* resource_manager = nullptr;//��ǰ��op_kernel���Է��ʵĹ�����Դ
    ScopedStepContainer* step_container = nullptr;//���ڵ�ǰop_kernel�ĵ�����Դ
    Rendezvous* rendezvous = nullptr;//ͨ�Ż���
    TensorStore* tensor_store = nullptr;
    CancellationManager* cancellation_manager = nullptr;
    
    const gtl::InlindecVector<TensorValue,4>* inputs = nullptr;//��ǰop_kernel������
    bool is_input_dead = false;
    const gtl::InlinedVector<AllocatorAttributes, 4>* input_alloc_attrs = nullptr;
    
    //�豸������
    const gtl::InlineVector<DeviceContext*,4>* input_device_contexts = nullptr;
    DeviceContext* op_device_context = nullptr;
    
    //�Կ�������ز�����֧��
    FrameAndIter frame_iter;
    
    //��������֧��
    FunctionCallFrame* call_frame = nullptr;
    FunctionLibraryRuntime* function_library = nullptr;
    std::function<void(std::function<void()>)>* runner = nullptr;
    StepStatsCollector* stats_collector = nullptr;
};
```
��ȻParams������ֲ������ۣ�ʵ�����ڲ�����Ǭ���������������OpKernel��������Ҫ����Դ�������кܶ���������ʱ����Դ�����ǵ�ǰ��û�н��ܵ���������Թ���������Щ���ݶ�����֮���ٻع�ͷ�������ɡ�

# 4. kernel��ע��
����ڶ��OpKernel������Ҳ��Ҫһ�����й���ĵط���������OpRegistryһ����TFҲ�����һ��OpKernelRegistrar�������ϻ���һ��ӳ�䣬�������£�
```
class OpKernelRegistrar {
  public:
    typedef OpKernel* (*Factory)(OpKernelConstruction*);
    OpKernelRegistrar(const KernelDef* kernel_def, StringPiece kernel_class_name, Factory factory){
        if(kernel_def != nullptr){
            InitInternal(kernel_def, kernel_class_name, factory);
        }
    }
  private:
    void InitInternal(const KernelDef* kernel_def, StringPiece kernel_class_name, Factory factory);
}
```
�ƺ�û���ҵ��洢���ݵ�λ�ã�ʵ���ϴ������InitInternal�����У������Ƚ������׷����Դ���õ���������һ���ṹ���壺
```
typedef std::unordered_multimap<string, KernelRegistration> KernelRegistry;
```
����ṹ����Ҫ�����������ܷ������ã�
```
void* GlobalKernelRegistry() {
    static KernelRegistry* global_kernel_registry = new KernelRegistry;
    return global_kernel_registry;
}
static KernelRegistry* GlobalKernelRegistryTyped() {
    return reinterpret_cast<KernelRegistry*>(GlobalKernelRegistry());
}
```
ͨ����GlobalKernelRegistryTyped��������Ϊstatic��ʹ�������ص�����Ψһ���������Ҳ�͵õ���һ��ȫ�ֵ�KernelRegistry��ΪOpKernel��ע�����ġ�����KernelRegistration�Ľṹ���Ƚϼ򵥣��������������Լ�ȥ̽Ѱ�ɡ�

# 5. op_segment
��ʱ�����ǻ�Ϊÿ���Ự��Session��׼��ר�õ�kernel����˾���Ҫһ���ṹ������ÿ���Ự��OpKernel����������OpSegment�࣬���ǿ������Ľṹ��
```
class OpSegment {
  public:
    void AddHold(const string& session_handle);
    void RemoveHold(const string& session_handle);
    typedef std::function<Status(OpKernel**)> CreateKernelFn;
    Status FindOrCreate(const string& session_handle, const string& node_name, OpKernel** kernel, CreateKernelFn create_fn);
  private:
    typedef std::unordered_map<string, OpKernel*> KernelMap;
    struct Item {
        int num_holds = 1;
        KernelMap name_kernel;
        ~Item();
    };
    typedef std::unordered_map<string, Item*> SessionMap;
    mutalbe mutex mu_;
    SessionMap session_ GUARDED_BY(mu_);
    //...
};
```
�ɼ������OpSegment�࣬��������һ��SessionMap������ʵ��һ��SessionHandle��Item�ṹ���ӳ�䣬����������op���Ƶ�OpKernel�ṹ��ӳ�䡣���ǿ����������ͼ����ʾ��
```mermaid
graph LR
    SessionHandle-->|����|Item
    Item-->|����|OpKernel
```
���ǿ�����Item�ṹ�У���һ��num_holds��Ա������ʾ�ж���holdָ����ĳ��SessionHandle��hold�����ÿ������Ϊ���ü�������ֹSession��ɾ������һ��SessionHandle���hold����Ϊ�˷�ֹSessionHandle��Ӧ��OpKernel��ɾ����

# 6. ��ϵͼ
```mermaid
graph TB
    KernelDefBuilder-.����.->KernelDef
    KernelDef-.����.->OpKernel
    OpKernel-.����.->OpKernel���캯��
    OpKernel���캯��-.ʹ��.->OpKernelConstruction
    OpKernel-.����.->Compute����
    Compute����-.ʹ��.->OpKernelContext
    OpKernelContext-.����.->Params
    OpKernel-.ע��.->OpKernelRegistrar
    OpSegment-.����.->OpKernel
```

# 7. �漰���ļ�
- op_kernel
- kernel_def_builder

# 8. ������¼
- v1.0 2018-08-28 �ĵ�����
- v2.0 2018-09-09 �ĵ��ع�

[github��ַ](https://github.com/tengkz/tensorflow_notes)