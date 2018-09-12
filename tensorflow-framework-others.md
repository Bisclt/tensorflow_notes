��framework��ʣ������ݣ������ļ��������˼򵥽�����ʱ��ԭ��д�ĺִܲ٣�����ռ���ӣ����������µ�����������䡣

## allocation_description.proto
һ���Ե����ڴ������������Ϣ������proto��

## attr_value
֮ǰ�ڽ�op��ʱ���ᵽ�����������в����ġ���AttrValue��ʾ�ľ��ǲ�����ֵ���ȿ�һ������proto���壺
```
message AttrValue {
    message ListValue {
        repeated bytes s = 2;
        repeated int64 i = 3;
        repeated float f = 4;
        repeated bool b = 5;
        repeated DataType type = 6;
        repeated TensorShapeProto shape = 7;
        repeated TensorProto tensor = 8;
        repeated NameAttrList func = 9;
    }
    
    oneof value {
        bytes s = 2;
        int64 i = 3;
        float f = 4;
        bool b = 5;
        DataType type = 6;
        TensorShapeProto shape = 7;
        TensorProto tensor = 8;
        ListValue list = 1;
        
        //func����һ��������func.name����һ�����������ƣ����ߺ��Ĳ��������ƣ�func.attr.first��Ϊ��������Ĳ��������ƣ�func.attr.second������������ֵ
        NameAttrList func = 10;
        
        //placeholder���ں����ڲ��Ľڵ���ʹ�ã�����ζ�ţ��������ֱֵ����������ʼ��ʱ�Ż��ṩ�����磬���Ǽ��躯��FN����һ���ڵ�N���ڵ�Nӵ������A��A������ֵ��"foo"�����FN�ڳ�ʼ��ʱ��foo����Ϊbar����ôN�ڵ��A���Ե�ֵҲ�ѱ�����Ϊbar
        string placeholder = 9;
    }
}
message NameAttrList {
    string name = 1;
    map<string, AttrValue> attr = 2;
}
```
�ɼ����������ԵĿ�ȡֵ���ͺܷḻ���������ַ��������͡������͡������͡�Ԫ���ͣ�ָ��һ���������ͣ���������״���������б����������ֵ��placeholder���ȵȡ�

attr_value_util�а�����һЩ����Բ���ֵ�������úͱ�ʾ�ĸ���������

## bfloat16
Google������16λ���ȵĸ�����������32λ���ȵĸ�����������������ļ��㣬ģ�͵ľ��Ȳ�û�������½�����ģ�ʹ�С���Խ��ͣ����TF�Ƴ�������16λ�ĸ�������

��Ϥ�����ȸ�������ʾ������Ӧ��֪����������float��32λ��ʾ����1λ����λ+8λ����+23λβ����ɵġ�IEEEҲ�����һ��16λ���ȵĸ�����������Ŀǰ��32λ���ȸ�����֮���ת���Ƚϸ��ӣ����TF�����һ���µ�16λ��������ʾ��1λ����λ+8λ����+7λβ������float�ķ���λ�ͽ�������ͬ�ģ���β��λ����ͬ����˷������໥ת����

## cancellation
�ڼ���ͼ��ִ�й����У������Ҫ��ʱ��ֹ����ͼ��ִ�У����緢��������д���󣬻��߱�����󣬼���ͼ����������ͣ��������Ϊһ�����кܶ��첽�������ڽ��У���һ����ܶ��������Զ���豸��ִ�еģ����Ǳ���֪ͨ������ִ�е�Զ���豸����Щ��������Ҫһ��ʵ�������������CancellationManager��
```
class CancellationManager {
  public:
    //...
    void StartCancel();//�������и���ǰ��ȡ���������йصĻص�����
    bool IsCancelled();//���ҽ���StartCancel�����ú󣬷���true
    CancellationToken get_cancellation_token();//��ע��ͽ�ע��ص�����ʱ����Ҫ�õ�������
    bool RegisterCallback(CancellationToken token, CancellCallback callback);//��һ��token��ע��ȡ���ص�����
    bool DeregisterCallback(CancellationToken token);//��һ��token�Ͻ�ע��ȡ���ص�����
  private:
    bool is_cancelling_;
    std::atomic_bool is_cancelled_;
    
    mutex mu_;
    Notification cancelled_notification_;
    CancellationToken next_cancellation_token_ GUARDED_BY(mu_);
    gtl::FlatMap<CancellationToken, CancelCallback> callbacks_ GUARDED_BY(mu_);
};
```

## common_shape_fns
������һЩͨ�õģ���״�ƶϺ����п��ܻ��õ��Ĺ��ܺ��������磬������㣬����ͨ��������״���˴�С��padding��С��stride��С���ж��������״��

## control_flow
��������һ�������⣬TF�ٷ���һƪ���½��͵ĺܺã�������ר��Ϊ��дһƪ���ġ���ǰ��Ҫ�����ǣ���TF��Ϊ��ʵ�ֿ�������������ѭ���ȣ�����Ҫ���������һЩ���ӵ����ԡ����磬�������µĴ��룺
```
int fun(int t){
    int s = 0;
    for(int i=1;i<=100;++i){
        s += t;
    }
    return s;
}

int x(){
    return fun(2);
}

int y(){
    return fun(3);
}
```
��δ�������ŵ�TF����ͼ��ʵ�֣�ͬ����һ������s�����ж����ͬ��ֵ����fun�����ڲ�����ͬ�����ִ���s��ֵ�ǲ�ͬ�ģ���x�����е��ã�����y�����е��ã���ͬ�ִ�s��ֵҲ�ǲ�ͬ�ġ�Ϊ�˶Բ�ͬ���ã���ͬ�����ִ��ڵ�s�����֣�TF�����һ���ṹ�壺
```
struct FrameAndIter {
    uint64 frame_id = kIllegalFrameId;
    int64 iter_id = kIllegalIterId;
    
    FrameAndIter(){}
    FrameAndIter(uint64 frame, int64 iter){
        frame_id = frame;
        iter_id = iter;
    }
    
    bool operator==(const FrameAndIter& other) const {
        return (frame_id == other.frame_id && iter_id == other.iter_id);
    }
};
```
��ϸ����֮ǰ���ĵ������Ѿ�������������ṹ��ִ�����е�TaggedNode����ֻ����FrameAndIter���������TaggedNode��Խڵ㣬����Ȥ�Ķ��߿���ȥ�ع���[executor-��](https://www.cnblogs.com/jicanghai/p/9572217.html)��

## cost_graph
�ڶԼ���ͼ�����Ż�ʱ��һ������Ҫ����Ϣ���ǶԼ���ͼ�и��ڵ�ļ������Ľ��й��ơ�TFר�������һ��ͳ�Ƽ���ͼ���ģ��ڴ����ġ�����ʱ�����ģ���ģ�ͣ����濴һ�����Ľṹ��
```
message CostGraphDef {
    message Node {
        string name = 1;
        string device = 2;
        int32 id = 3;
        
        message InputInfo {
            int32 preceding_node = 1;
            int32 preceding_port = 2;
        }
        repeated InputInfo input_info = 4;
        
        message OutputInfo {
            int64 size = 1;
            int64 alias_input_port = 2;
            TensorShapeProto shape = 3;
            DataType dtype = 4;
        }
        repeated OutputInfo output_info = 5;
        
        int64 temporary_memory_size = 6;//��ʱ�ڴ����
        int64 host_temp_memory_size = 10;//host��ʱ�ڴ����
        int64 device_temp_memory_size = 11;//device��ʱ�ڴ����
        int64 host_persistent_memory_size = 12;//host�����ڴ����
        int64 device_persistent_memory_size = 16;//device�����ڴ����
        
        int64 compute_cost = 9;//�ýڵ����ʱ���Ĺ��ƣ���λ����
        int64 compute_time = 14;//��������ģ��������ڴ�������
        int64 memory_time = 15;//�ڴ������ģ��������������
        bool is_final = 7;//��ǰ�ڵ��������Ƿ�����������ͼ�����������ǣ������������ܱ�����
        
        repeated int32 control_input = 8;//��ǰ�ڵ�Ŀ�������
    }
    repeated Node node = 1;
}
```
�ɼ����Լ���ͼ��ĵ�ͳ�ƣ����ǶԽڵ����ͳ�Ƶļ��ϡ������ڽڵ㣬���˻�����������Ϣ�����������Ϣ֮�⣬��Ҫ�������ڴ����ĺ�ʱ�����ĵ���Ϣ��

## fake_input
Ϊ�˲���NodeDefBuiler�Ĺ��ܣ�������ҪΪ�ڵ�׼��һЩ���룬����ʵ�󲿷ֹ��ܵĲ��Բ�����Ҫ��ʵ�����ݣ�����ֻ��Ҫ��һ���������ʽ�ڡ����TF�Ƴ���FakeInput�ṹ������һ���ڲ�ʹ�õĽṹ������ʵ����FakeInputImpl������Ȥ�Ķ��߿���ȥ����Դ�롣

## load_library
������ʱ������ʼ����ʱ����Ҫ��op��kernel�Ķ��������ڴ档�״�����ʱ����Щ��Դ�ᱻ����һ��ȫ�ֵ����ݽṹ�У�������Ҫʱ���Դ��м�����
```
Status LoadLibrary(const char* library_filename, void** result, const void** buf, size_t* len){
    static mutex mu;
    static std::unordered_map<string, Library> loaded_libs;
    //...
}
```
Ҳ����˵��������Ŀ���ʵ������һ��ȫ�ֵ�map�У�����keyΪ�����ڵ��ļ�����valueΪһ��Library�ṹ�����濴�����Ķ��壺
```
struct Library {
    void* handle = nullptr;
    OpList op_list;
};
```
��������Ŀ�ʵ���ϰ�������һ��ָ������ľ�����Լ�һ�������ļ��ϡ�

## log_memory
�ڳ�������ʱ�����Ǿ�����Ҫ�����ڴ�ռ䣬�ڴ�ķ���ͨ������Ϊ�����������һ����OpKernel����ʱ�����ڴ棬��Щ�������һ�����̼���ı�ţ�step_id������ʶ���ڶ��Ǹ���������ڴ���䳡����������
- �����м�ʱ�ĳ����۵��Ż�ʱ��
- ������OpKernel�Ĺ���ʱ��
- ��ʹ���ⲿ���룬����C API��������ʱ��
- ��Ϊ���紫������ڴ�ʱ��
- ��ΪGPU���������proto�����ڴ�ʱ��
- �������߲�û��ָ��step_idʱ��

�˽�����Щ�����Ҫ��¼�ڴ���䣬����Ҫ֪�����ڲ�ͬ���������Ҫ��¼��Щ��Ϣ��Ϊ�������ڴ������Ϣ��¼�Ĳ�ͬ������TF���������·��ࣺ
- ��¼��ͨ�������ڴ���䣬��ͨ��������OpKernel����ʱ������ڴ棬�Լ��������������
- ��¼��ͨ�������ڴ���գ�
- ����ĳ��������Ϊ���ʱ����Ҫ��¼��
- ԭʼ�ڴ�ķ��䣬�����ĳ�����Eigen�ڴ�ķ��䣬�ڴ濽����
- ԭʼ�ڴ�Ļ��գ�

������Щ�����������ڴ�������ؽṹ�������ˣ��������ǿ��£�TFΪ5���ڴ�����¼���������Ƶ�proto��
```
//����һ���������ڴ����
message MemoryLogStep {
    int64 step_id = 1;//���̼���Ĳ���id�������ڲ���ͬ������֮�䲻ͬ
    string handle = 2;//������ǰ�����������ľ��
};

//�����ڴ�������Ϣ
message MemoryLogTensorAllocation {
    int64 step_id = 1;
    
    //�����ڴ�����kernel���ƣ�����"/affine2/weights/Assign"
    string kernel_name = 2;
    TensorDescription tensor = 3;//�����������ϸ��
};

//�����ڴ���յ���Ϣ
message MemoryLogTensorDeallocation {
    int64 allocation_id = 1;
    string allocator_name = 2;
};

//��������Ϊ�������Ϣ
message MemoryLogTensorOutput {
    int64 step_id = 1;
    string kernel_name = 2;
    int32 index = 3;//�����õ����������
    TensorDescription tensor = 4;
};

//ԭʼ���ڴ������Ϣ
message MemoryLogRawAllocation {
    int64 step_id = 1;
    string operation = 2;
    int64 num_bytes = 3;
    uint64 ptr = 4;
    int64 allocation_id = 5;
    string allocator_name = 6;
};

//ԭʼ���ڴ������Ϣ
message MemoryLogRawDeallocation {
    int64 step_id = 1;
    string operation = 2;
    int64 allocation_id = 3;
    string allocator_name = 4;
    bool deferred = 5;
};
```
��Σ����ǿ����ڴ����Ĺ�����LogMemory��
```
class LogMemory {
  public:
    static bool IsEnabled();
    static void RecordStep(int64 step_id, const string& handle);
    static void RecordTensorAllocation(const string& kernel_name, int64 step_id, const Tensor& tensor);
    static void RecordTensorDeallocation(int64 allocation_id, const string& allocator_name);
    static void RecordTensorOutput(const string& kernel_name, int64 step_id, int index, const Tensor& tensor);
    static void RecordRawAllocation(const string& operation, int64 step_id, size_t num_bytes, void* ptr, Allocator* allocator);
    static void RecordRawDeallocation(const string& operation, int64 step_id, void* ptr, Allocator* allocator, bool deferred);
};
```
���Կ�������Ҫ��API�����ϸ�ǰ���protoһһ��Ӧ��
��ЩAPI�ڲ����ʵ�ֵ��أ�������һ����򵥵������£�
```
void LogMemory::RecordStep(const int64 step_id, const string& handle){
    MemoryLogStep step;
    step.set_step_id(step_id);
    step.set_handle(handle);
    OutputToLog(step);
}
```
�ɼ�����ЩAPI����Ҫ���þ��ǰ��������������Ӧ��proto��Ȼ������־����ʽ����Щproto���������ĺ������£�
```
template <typename T>
void OutputToLog(const T& proto){
    string type_name = proto.GetTypeName();
    const size_t index = type_name.find_last_of(".");
    if (index != string::npos) type_name = type_name.substr(index + 1);
    LOG(INFO) << LogMemory::kLogMemoryLabel << " " << type_name << " { " << ProtoShortDebugString(proto) << " }";
}
```

## memory_types
�ṩ��һ��������NodeDef����ȡ�ڵ���������ڴ����͵ĺ������ӿ����£�
```
Status MemoryTypesForNode(const OpRegistryInterface* op_registry, const DeviceType& device_type, const NodeDef& ndef, MemoryTypeVector* input_memory_types, MemoryTypeVector* output_memory_types);
```

## numeric_op
�����������������ֵ�������ͣ�
- �����뵥��������������������ͬ�������������㣻
- ˫���뵥�������ͬ���ͣ���������ӷ���
- ��������ӵ����ͬ����״�����������һһ��Ӧ���������Ԫ�ط������㣻
- ��������ӵ����ͬ����״�������������Ӧһ�������������������������㣻

�����У�����������Ķ������£�
```
class UnaryOp : public OpKernel;
class BinaryOp : public OpKernel;
template <class T, class CHILD> class UnaryElementWiseOp : public UnaryOp<T>;
template <class T, class CHILD> class BinaryElementWiseOp : public BinaryOp<T>;
```

## numeric_types
�����˳��õ���ֵ���͡�

## register_types
�ڶ���һ��������ʱ�������������ṩһ�����Ͳ�������һ���治һ�����еĲ�����֧�����е��������ͣ���һ���棬��ǰ��Ӳ��Ҳ��һ��֧�����е��������͡�����б�ҪΪ�������һ�����Կ���ʵ����Ϊ���������;�������ĺ꣬Ҳ�б�ҪΪ��ͬ��Ӳ����Ʋ�ͬ�Ŀ��úꡣ

��ˣ�����ĺ��Ϊ���࣬һ����TF_CALL_float������Ծ����������͵ľ���꣬��һ����TF_CALL_ALL_TYPES��������꣬�ڶ����ͨ�����õ�һ��������������磺
```
#define TF_CALL_INTEGRAL_TYPES(m) \
  TF_CALL_int64(m) TF_CALL_int32(m) TF_CALL_uint16(m) TF_CALL_int16(m) TF_CALL_uint8(m) TF_CALL_int8(m)
```

## register_types_traits
����ļ��Ĺ����ǣ�����CPU��GPU��������ʱ��ΪPOD���������ṩ�������͡�ʲôPOD���������أ�POD��ȫ����Plain Old Data������˵��һ������߽ṹ�壬�ھ��������ƿ������ܱ������ݲ��䣬�����POD�������͡�����CPU��GPU��������ʱ��ʵ���Ͽ������Ƕ����Ƶ����ݣ��������ʵ�ʵ��������ͣ��ڴ���ʱ���Ժ��Ե���ֱ�ӿ������������ݾͺ��ˡ�

## rendezvous
һ��Rendezvous��һ�����������������ߴ��������ĳ�������һ��ͨ��ӳ�����ɡ�ÿһ��ͨ����һ��Rendezvous��Ψһ��ʶ���������"producer,consumer"��ɣ������ߺ������߶���TF�е��豸��

������ͨ������Send()��������һ������ͨ��ͨ�����ݸ������ߣ�������ͨ������Recv()��������һ��ͨ���н��մ��ݹ����������������߰���������������˳����մ����������

�����߿����������߽�������������֮ǰ��֮����Ҫ��������������߿���ѡ�����һ���������ã������ṩһ���ص����������κ�һ������£�ֻҪ�������������������߶����һʱ��õ����������ߴӲ�������

���ڱȽϼ򵥣����ǽ��г�API�����ƣ�����ǩ����ʵ�֣���ҿ��Բο�Դ���룺
```
class Rendezvous : public core::RefCounted {
  public:
    static string CreateKey(...);//����һ������ļ�
    static Status ParseKey(...);//����һ������ļ�
    virtual Status Send(...) = 0;
    virtual void RecvAsync(...) = 0;
    Status Recv(...);
    virtual void StartAbort(...) = 0;
}
```

## session_state
����������������࣬SessionState������������Ҫ�ڲ�ͬ�����б�����������������ѧϰ��Ҫ������ѵ����ÿ��ѵ��֮����Ҫ����һЩ���ݣ��ͱ����������TensorStore�����������ڵ�ǰ��������Ҫ������������������е�op_kernel����
```
class SessionState {
  public:
    Status GetTensor(const string& handle, Tensor* tensor);
    Status AddTensor(const string& handle, const Tensor& tensor);
    Status DeleteTensor(const string& handle);
    int64 GetNewId();
    static const char* kTensorHandleResourceTypeName;
  private:
    mutex state_lock_;
    int64 tensor_id_ = 0;
    std::unordered_map<string, Tensor> tensors_;
};
class TensorStore {
  public:
    struct TensorAndKey {
        Tensor tensor;
        int64 id;
        string device_name;
        string GetHandle(const string& tensor_name){
            return strings::StrCat(tensor_name, ";", id, ";", device_name);
        }
    };
    Status AddTensor(const string& name, const TensorAndKey& tk);
    Status SaveTensors(const std::vector<string>& output_names, SessionState* session_state);
  private:
    mutex lock_;
    std::unordered_map<string, TensorAndKey> tensors_ GUARDED_BY(lock_);
};
```

## step_stats
��Ҫ�ǹ�������ʱ����ڴ�ʹ�õ�ͳ�ơ�����ͳ�����豸ͳ�ƹ��ɣ����豸ͳ���ɽڵ�ͳ�ƹ��ɡ����濴�����ǵĺ��Ľṹ��
```
message NodeExecStats {
    string node_name = 1;
    int64 all_start_micros = 2;
    int64 op_start_rel_micros = 3;
    int64 op_end_rel_micros = 4;
    int64 all_end_rel_micros = 5;
    repeated AllocatorMemoryUsed memory = 6;
    repeated NodeOutput output = 7;
    string timeline_label = 8;
    int64 scheduled_micros = 9;
    uint32 thread_id = 10;
    repeated AllocationDescription referenced_tensor = 11;
    MemoryStats memory_stats = 12;
};

message DeviceStepStats {
    string device = 1;
    repeated NodeExecStats node_stats = 2;
};

message StepStats {
    repeated DeviceStepStats dev_stats = 1;
};
```
ʣ���AllocatorMemoryUsed��NodeOutput��AllocationDescription��MemoryStats���Ƚϼ򵥣���Ҹ���Ȥ����ֱ��ȥ��Դ�롣

## summary.proto
����Tensorboard�У�������ʾÿ���ڵ�Ԫ�صĻ�����Ϣ��

## type_index
c++�е�std::type_index�ṩ��RTTI��������ʱ����ʶ��Ĺ��ܡ��������ϰ�����һ�����͵Ĺ�ϣֵ��������Ϊһ������ӳ��ļ���cppreference�ϵ����ʾ������ǳ������
```
#include <iostream>
#include <typeinfo>
#include <typeindex>
#include <unordered_map>
#include <string>
#include <memory>
 
struct A {
    virtual ~A() {}
};
 
struct B : A {};
struct C : A {};
 
int main()
{
    std::unordered_map<std::type_index, std::string> type_names;
 
    type_names[std::type_index(typeid(int))] = "int";
    type_names[std::type_index(typeid(double))] = "double";
    type_names[std::type_index(typeid(A))] = "A";
    type_names[std::type_index(typeid(B))] = "B";
    type_names[std::type_index(typeid(C))] = "C";
 
    int i;
    double d;
    A a;
 
    // ע���������ڴ洢ָ������ A ��ָ��
    std::unique_ptr<A> b(new B);
    std::unique_ptr<A> c(new C);
 
    std::cout << "i is " << type_names[std::type_index(typeid(i))] << '\n';
    std::cout << "d is " << type_names[std::type_index(typeid(d))] << '\n';
    std::cout << "a is " << type_names[std::type_index(typeid(a))] << '\n';
    std::cout << "b is " << type_names[std::type_index(typeid(*b))] << '\n';
    std::cout << "c is " << type_names[std::type_index(typeid(*c))] << '\n';
}
```
���std::type_index(typeid(i))ʵ������ͨ��std::type_index�Ĺ��캯����������һ��TypeIndex������Ȼb��c������Ϊָ��A��ָ�룬��ͨ��typeid������Ȼ�ܱ�ʶ����ʵ����ָ�����ʲô���͡�

���ָ��ӵ�������Ϣ���д��۵ġ���ĳЩƽ̨�ϣ�����ϣ��ͨ�����������������Ϣ������ø�С�Ķ����ƴ洢�ռ䡣���TF�����һ���򻯰��TypeIndex�࣬��ģ����std::type_index�Ĺ��ܣ�����û��ʹ��RTTI��Ϣ�����Ҳ�Ͳ����ṩ������������Ϣ��ֻ�Ƿ���һ����־����ʾRTTI�ѱ����á����а����Ĺ�ϣ���ÿ������Ψһ�ģ�Ȼ������������ʱ�����ģ���������ϣֵ�����л���û�����壬��Ϊÿ�����е�ʱ�������ϣֵ����һ����ͬ��

��������������TF�Զ����TypeIndex��ʵ�֣�
```
class TypeIndex {
  public:
    TypeIndex(const TypeIndex& src) : hash_(src.hash_){}
    //...
  private:
    TypeIndex(const uint64 hash) : hash_(hash){}
    uint64 hash_;
};
```

## types
�����ڴ�����MemoryType���豸����DeviceType�Ķ��壬����һЩ�������Ϣ�����Դ�롣

## type_traits
������һЩ���������жϵ�ģ�壬��Ҫ�����������֣�
- is_quantized���Ƿ���quantized type��
- is_complex���Ƿ��Ǹ������ͣ�
- is_simple_type���Ƿ��Ǽ����ͣ�
- is_signed���Ƿ��Ǵ����ŵ����ͣ�

## unique_tensor_references
һ��Ψһ��tensor references�ļ��ϡ�������������м���tensor��ʱ����жϣ�ָ�����tensor�ڲ���buffer�������Ƿ��Ѿ����ڵģ�����Ѵ��ڣ������κβ�������������ڣ�����롣

��������һ��СС���Ż�����Ϊ������д洢�Ĳ�ͬ���������ò���̫�࣬����һ��ʼ������һ����󳤶�Ϊ4�����������洢��Щ���á�����ͬ����������������4ʱ��ʹ��һ��set���洢�������ڴ󲿷�����£����ܱ�֤�ϺõĲ���Ͳ�ѯ���ܡ������Ż���ʽ�Ƚ��ձ飬��TF��Ҳ������ʹ�á�

## variant
һ�����Ͳ����������������ڴ洢�����������͡�ʵ�ַ�ʽ��std::any���񣬵����ڴ洢���������������ƣ�ֻ�ܴ洢���޼������͵����ݡ����ܴ洢�����ݱ����������µļ���������
- ������ǿ��Ը��ƹ���ģ�
- ����Ĭ�Ϲ��캯����
- ��Ҫô��һ��protobuf��Ҫô��һ��TF��tensor��Ҫô���������µ����ֺ���
```
    string TypeName() const;
    void Encode(VariantTensorData* data) const;
    void Decode(const VariantTensorData& data);
```
ʹ��get<T>�����ǻ�ȡVariant�д洢���ڲ����ݵ���Ҫ��ʽ�����ַ�ʽ�����Ͱ�ȫ�ģ�����ڲ��洢���������Ͳ���T��������null��

Variant�������л��ͷ����л��Ĺ��������ڲ�����������ȥ��������һЩδʵ�����л��ͷ����л����ܵ�POD���ͣ�plain old data����TF��tensor���ͣ��Լ�protobuf���ͣ�TF���ļ����ṩ��һЩ���������������Ҫ��Variant�д�������������ͣ���Ҫ�����ṩEncode��Decodeʵ�֡�

��Variant�д洢���������ͣ�ͨ�������ָ������TF��tensor�����á�Ϊ����Ч��֧�����������TF�����л��Ľṹ�ṩ����ȷ�Ľṹ��Ҳ����˵�����뽫���ǵĽṹ���л�Ϊһ��VariantTensorData������ṹ���£�
```
struct VariantTensorData {
    string type_name;
    string metadata;
    std::vector<Tensor> tensors;
};
```
��������ڰ���ָ���������������ã����԰����ǰ�����tensors�У���������Ԫ�������ݷ���metadata�С�

## versions
�ṩ�˰汾��Ϣ���Ĺ��ܡ�