## Ŀ¼
1. ���ĸ���
2. allocator
    1. Allocator
    2. AllocatorAttributes
    3. AllocationAttributes
    4. AllocatorWrapper
    5. AllocatorStats
3. allocator_registry
    1. AllocatorRegistry

## 1. ���ĸ���
allocator������ֻ���ڴ�������Ľӿڣ�û�и�������ʵ�֡�allocator_registry�õ���ģʽʵ����һ��ȫ�ֵ��ڴ������ע���࣬��������ע�����е��ڴ��������

## 2. allocator
### 2.1 Allocator
Allocator��һ���ڴ�������Ľӿ��࣬���涨��һ���ڴ��������Ҫ������ЩAPI�����忴���룺
```
class Allocator {
public:
    virtual void* AllocateRaw(size_t alignment, size_t num_bytes) = 0;
    virtual void DeallocateRaw(void* ptr) = 0;
    T* Allocate(size_t num_elements);
    T* Allocate(size_t num_elements, const AllocationAttributes& allocation_attr);
    void Deallocate(T* ptr, size_t num_elements);
    virtual bool TracksAllocationSizes();
    virtual bool ShouldAllocateEmptyTensors();
    virtual size_t RequestedSize(void* ptr);
    virtual size_t AllocatedSize(void* ptr);
    virtual int64 AllocationId(void* ptr);//�����ڴ����ı��
    virtual size_t AllocatedSizeSlow(void* ptr);
    virtual void GetStats(AllocatorStats* stats);
}
```
���⣬Allocator�����ṩ�����ڴ�Ľӿ�֮�⣬���ṩ��Ϊ����õ��ڴ����Ĭ�Ϲ�������������Ľӿڡ�����������ʱ��ָ���˶�������ͣ��Ϳ���ѡ����ö���������Ĺ��������������Allocator�ṩ��������ֳ�����Ĺ��췽�����ֱ���String��ResourceHandle,Variant��
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
}
```

### 2.2 AllocatorAttributes
��ͬ���豸�����ڴ�ķ���������ͬ�����ǲ��Ǹ��豸ֻ��Ҫʵ��������ڴ�������Ϳ������أ�����ڼ�����ÿ���豸ֻ��Ҫ�õ��Լ����ڴ棬��Ȼ��û������ģ�����TF�У���Щ�����Ϊ��Ч�ʣ�GPUҲ��Ҫ�õ�CPU�ڴ棬���磬Ϊ��ʹ��DMA��ĳЩ�豸�������ݣ�������Ȼ��Ҫ����CPU�ڴ档��ˣ���������һ���豸��Ҫ�ڴ������ʱ����Ҫ�����ṩһЩ��Ϣ�������豸������Ҫ�����������͵��ڴ棬��Щ��Ϣ�ʹ洢��AllocatorAttributes���С�
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

### 2.3 AllocationAttributes
AllocatorAttributes������������һ����������Ǿ���AllocationAttributes��������Ϊ�ڴ��������ĳһ�ξ�����ڴ����׼����Ϣ�ģ���ǰ����Ϊ���豸��Ҫ���ʵ��ڴ�������ṩ���豸�ģ�ʹ��ʱ����ȫ��һ����
```
class AllocationAttributes {
    bool no_retry_on_failure = false; //����״��ڴ����ʧ���ˣ����ٳ��ԡ�
    bool allocation_will_be_logged = false;//�����ڴ�����Ƿ�ᱻ��¼
}
```

### 2.4 AllocatorWrapper
��ʱ���������ĳ���ڴ���������з�װ���Ա���ĳ��API��ʵ�ֶ��ƻ�����ʱ����Ҫ�õ�AllocatorWrapper�࣬�������Ͼ��Ƕ�Allocator���ֱ�ӷ�װ��

### 2.5 AllocatorStats
Ϊ�˶�ĳ���ڴ�������ѷ�����ڴ����ͳ�ƣ�TF�������һ���ṹ��AllocatorStats��
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

## 3. allocator_registry
### 3.1 AllocatorRegistry
�ȿ�һ�£�ע�����ڲ��������洢�ڴ�������ģ�
```
class AllocatorRegistry {
    //...
private:
    std::vector<AllocatorRegistryEntry> allocators_;
    //...
}
```
��ʵ���ǰ��ڴ�������洢��һ���������������ֱ�Ӵ洢�ڴ�������������Ƕ�����һ����װ�����ǿ��������װ�Ľṹ��
```
typedef struct {
    string name;
    int priority;
    Allocator* allocator;
} AllocatorRegistryEntry;
```
�����ڴ������֮�⣬���entry�ﻹ������ڴ�����������ƺ����ȼ�����������AllocatorRegistry����һ���ڴ������ʱ�������ص��Ǿ���������ȼ��ķ�����������������������ͬ�����ȼ����ͷ������е�һ����