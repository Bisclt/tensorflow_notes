# Ŀ¼
1. ʲô��allocator
2. �ڴ�������Ĺ���
3. �ڴ����׷��
4. �����ṹ
5. ��ϵͼ
5. �漰���ļ�
6. ������¼

# 1. ʲô��allocator
Allocator�������ڴ�������Ļ��࣬���������ڴ��������Ҫʵ�ֵĽӿڡ�
```
class Allocator {
public:
    //�ڴ�����뷵��
    virtual void* AllocateRaw(size_t alignment, size_t num_bytes) = 0;
    virtual void DeallocateRaw(void* ptr) = 0;
    T* Allocate(size_t num_elements);
    T* Allocate(size_t num_elements, const AllocationAttributes& allocation_attr);
    void Deallocate(T* ptr, size_t num_elements);
    
    //׷���ڴ������Ϣ
    virtual bool TracksAllocationSizes();
    virtual bool ShouldAllocateEmptyTensors();
    virtual size_t RequestedSize(void* ptr);
    virtual size_t AllocatedSize(void* ptr);
    virtual int64 AllocationId(void* ptr);//�����ڴ����ı��
    virtual size_t AllocatedSizeSlow(void* ptr);
    virtual void GetStats(AllocatorStats* stats);
};
```
��ЩAPI���Ա���Ϊ���࣬һ���ڴ�����뷵���������ڴ������Ϣ׷�١�ǰ�����ڴ�������ı�ְ�����������ṩ�˶��ڴ��������������ڴ����׷�ٺ͹���Ĺ��ܡ�����ʵ�ֺ��ߵĹ��ܣ���Ҫ�ṩһ�������ݽṹ֧�֣������еġ��ڴ����׷�١�����ϸ������

���⻹������һ���µĽṹAllocatorStats��������Ƕ��ڴ��������ǰ�����ڴ�����һ�����ͳ�ƣ����Ķ������£�
```
struct AllocatorStats {
    int64 num_allocs;//�ڴ�������
    int64 bytes_in_use;//������ڴ��У���ǰ����ʹ�õĴ�С
    int64 max_bytes_in_use;//ʹ���е��ڴ��С�ķ�ֵ
    int64 max_alloc_size;//���ĵ����ڴ�����С
    int64 bytes_limit;//��ǰ�ڴ�������ܷ��������ڴ�������������ڴ��С���������ֵ������0
    //...
}
```
Allocator�����ṩ�ڴ�����Ľӿ�֮�⣬���ṩ��Ϊ����õ��ڴ����Ĭ�Ϲ�������������Ľӿڡ�����������ʱ��ָ���˶�������ͣ��Ϳ���ѡ����ö���������Ĺ��������������Allocator�ṩ��������ֳ�����Ĺ��췽�����ֱ���String��ResourceHandle��Variant��
```
class Allocator {
public:
    //...
private:
    void RunCtor(T* p, size_t n);
    virtual void RunStringCtor(string* p, size_t n);
    virtual void RunStringDtor(string* p, size_t n);
    virtual void RunResourceCtor(ResourceHandle* p, size_t n);
    virtual void RunResourceDtor(ResourceHandle* p, size_t n);
    virtual void RunVariantCtor(Variant* p, size_t n);
    virtual void RunVariantDtor(Variant* p, size_t n);
};
```
���˳�����ڴ�������ӿ�֮�⣬TF��Ϊ��õ�CPU�ڴ���������ṩ��һ��Ĭ��ʵ�֣�
```
class CPUAllocator : public Allocator {
  public:
    //...
    void GetStats(AllocatorStats* stats) override {
        mutex_lock l(mu_);
        *stats = stats_;
    }
    size_t AllocatedSizeSlow(void *ptr) override {
        return port::MallocExtension_GetAllocatedSize(ptr);
    }
  private:
    mutex mu_;
    AllocatorStats stats_ GUARDED_BY(mu_);
};
```
��Ҫע�����㣬��һ����Ϊ�ڴ������ͳ�Ƽ�����һ�����ݳ�Աstats_������Ҫʱֱ�ӽ��䷵�أ��ڶ���Ϊ׷���ڴ�����С�ṩ��һ���������汾��ʵ�֣����ʵ�ֵ��������ڣ������Ҫ֪��ĳ��ָ���Ӧ�ķ����ڴ�Ĵ�С����������û��ר��Ϊ��׼�����������ݼ�¼ʱ������ֱ�ӵ��ò���ϵͳ����ĺ���������ȡָ���Ӧ�ķ����ڴ�Ĵ�С�������������Ժ�ʱ������ʤ���ޡ�

# 2. �ڴ�������Ĺ���
��ͬ���͵��豸��������Ҫ��ͬ���ڴ�����������������ͬ���͵��豸�����ǵ�Ч�����⣬Ҳ���ܻ��ṩ��ͬ���ڴ�������汾����ˣ���Ҫһ�����ڴ���������м��й���ĵط���TFΪ�����ṩ����AllocatorRegistry��
```
class AllocatorRegistry {
  public:
    //�ڴ������ע��
    void Register(const string& name, int priority, Allocator* allocator);
    //����������ȼ����ڴ������
    Allocator* GetAllocator();
    //����һ��ȫ�ֵ��ڴ��������ע����
    static AllocatorRegistry* Global();
  private:
    //�ڴ�������洢λ��
    std::vector<AllocatorRegistryEntry> allocators_;
    //...
}
```
���е��ڴ����������������allocators_���������������б���Ĳ������ڴ�������������Ƕ�����һ����װ�����ǿ��������װ�Ľṹ��
```
typedef struct {
    string name;
    int priority;
    Allocator* allocator;
} AllocatorRegistryEntry;
```
�����ڴ������֮�⣬���entry�ﻹ������ڴ�����������ƺ����ȼ�������AllocatorRegistry����һ���ڴ������ʱ�������ص��Ǿ���������ȼ��ķ�����������������������ͬ�����ȼ����ͷ������е�һ����

AllocatorRegistryʵ������һ��������������Global�ӿڷ���һ��ȫ�־�̬��ע��������Ϊ�˷������ע�ᣬTF�������һ��ͳһ��ע������ࣺ
```
class AllocatorRegistration {
  public:
    AllocatorRegistration(const string& name, int priority, Allocator* allocator){
        AllocatorRegistry::Global()->Register(name,priority,allocator);
    }
};
```
���⣬TF�������һ��������ע����̣�����Ȥ�Ķ��߿��Բο�Դ���롣

# 3. �����ڴ�׷��
�ղ��ᵽ�����ڴ�������Ĺ���API�У���һ��ר������׷���ڴ���䣬��Ҫ��һЩר�õ����ݽṹ��������ÿһ���ڴ�������Ϣ����Щ������TrackingAllocator��ʵ�֣�
```
class TrackingAllocator : public Allocator {
  public:
    std::tuple<size_t, size_t, size_t> GetSizeAndUnRef();
    //...
  private:
    bool UnRef() EXCLUSIZE_LOCKS_REQUIRED(mu_);
    Allocator* allocator_;
    mutex mu_;
    int ref_ GUARDED_BY(mu_);
    //��ǰ����ʹ�õķ����ڴ��С�����allocator_��֧���ڴ�׷�٣���Ϊ0
    size_t allocated_ GUARDED_BY(mu_);
    //allocated_�ķ�ֵ
    size_t high_watermark_ GUARDED_BY(mu_);
    //��ǰ�ڴ�������ܹ�������ڴ��С
    size_t total_bytes_ GUARDED_BY(mu_);
    
    const bool track_sizes_locally_;
    struct Chunk {
        size_t requested_size;
        size_t allocated_size;
        int64 allocation_id;
    };
    std::unordered_map<void*, Chunk> in_use_ GUARDED_BY(mu_);
};
```
�����ṩallocated_��high_watermark_��total_bytes_�������ݳ�Ա��¼�ڴ�����ͳ����Ϣ֮�⣬����Ҫ���Ǽ�����in_use_������ݳ�Ա������һ��ָ��ӳ�䵽һ��Chunk�������Chunk�б�����ÿ���ڴ������Ҫ���ڴ��С��ʵ�ʷ�����ڴ��С�������ڴ�����Ψһ��ʶ��������һ���ṹ������ÿ���ڴ�������ϸ��Ϣ��

��ϸ˵һ��ref_��Ա�����á�������ݳ�Ա���ڵ��������ڣ����浱ǰ�ڴ�����������ڴ�Ĵ����������з�����ڴ�ȫ������֮�󣬾�ɾ������ǰ���ڴ������������ᷢ�֣����ref_��Ա��TrackingAllocator�����ʼ����ʱ�򣬱����Ѿ���ֵΪ1�ˣ��Ǽ�������ÿ�η����ڴ�ʱ������UnRef����������һ�����ղ����ǲ���Ϊ0��ԭ�����ڣ�����ϣ������GetSizeAndUnRef��������ڶ��������������ֻ����һ�Σ��������������ú�Ὣref_��һ����������GetSizeAndUnRef�������ǵ���UnRef��ֻҪref_ֵ����0����ɾ�����������Ҫ�����Ǳ��������GetSizeAndUnRef����һ�Σ�����ͻ�����ڴ�й©��

# 4. �����ṹ
��TF�У������Ƿ����ڽڵ��ϵģ����ڵ㱻�����ھ�����豸�ϡ�����һ��GPU�豸�������������еĽڵ㣬�ǲ��ǾͲ���ҪCPU�ڴ����أ���Ȼ���ǣ����磬Ϊ��ʹ��DMA��ĳЩ�豸�������ݣ�������GPU�ϵĽڵ���Ȼ��Ҫ����CPU�ڴ档��ˣ����ڵ���һ���豸��Ҫ�ڴ������ʱ����Ҫ�����ṩһЩ��Ϣ�������豸������Ҫ�����������͵��ڴ棬��Щ��Ϣ�ʹ洢��AllocatorAttributes���С�
```
struct AllocatorAttributes {
    void set_on_host(bool v);
    bool on_host() const;
    void set_nic_compatible(bool v);
    bool nic_compatible() const;
    void set_gpu_compatible(bool v);
    bool gpu_compatible() const;
    void set_track_sizes(bool v);
    bool track_sizes() const;
    void Merge(AllocatorAttributes other);
    bool IsEqualOrLessRestrictiveThan(const AllocatorAttributes& other);
    uint32 value = 0;//�����ֵ�ĸ�8λ������Ϊ�豸��ص����á����豸��ʵ�ֿ��Ը�����Ҫ���н�����Ϊ��Щ�豸ʵ�ֵĲ���Ҳ��Ҫ��ȷ�Ľ�����
}
```
AllocatorAttributes������������һ����������Ǿ���AllocationAttributes�����߼�¼����Ϊ�ڴ��������ĳһ�ξ�����ڴ�����������Ϣ��ʹ��ʱ����ȫ��һ����
```
class AllocationAttributes {
    bool no_retry_on_failure = false; //����״��ڴ����ʧ���ˣ����ٳ��ԡ�
    bool allocation_will_be_logged = false;//�����ڴ�����Ƿ�ᱻ��¼
}
```
���⣬��ʱ���������ĳ���ڴ���������з�װ���Ա���ĳ��API��ʵ�ֶ��ƻ���TFΪ��׼������AllocatorWrapper�࣬�������Ͼ��Ƕ�Allocator���ֱ�ӷ�װ������Ȥ�Ķ��߿���ȥ����Դ�롣

# 5. ��ϵͼ
```mermaid
graph TB
    A(Allocator)-->|����|B(CPUAllocator)
    A(Allocator)-->|����|C(AllocatorWrapper)
    A(Allocator)-->|����|D(TrackingAllocaotr)
    E(AllocatorRegistration)-.��װ.->F(AllocatorRegistry)
    A(Allocator)-.ע��.->F(AllocatorRegistration)
```

# 6. �漰���ļ�
- allocator
- allocator_registry
- tracking_allocator

# 7. ������¼
- v1.0 2018-08-25 �ĵ�����
- v2.0 2018-09-08 �ĵ��ع�

[github��ַ](https://github.com/tengkz/tensorflow_notes)