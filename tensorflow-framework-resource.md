## Ŀ¼
1. core/framwork
    1. resource

## ���ĸ���
��Դ����������ָ�����ڴ棬����������������ص����л��ṹ��ResourceHandleProto
```
message ResourceHandleProto {
    //��������Դ���豸Ψһ����
    string device = 1;
    //��������Դ����������
    string container = 2;
    //����Դ��Ψһ����
    string name = 3;
    //����Դ�������͵�Ψһ��ϣֵ�����ڵ�ǰ�豸�͵�ǰִ������Ч
    uint64 hash_code = 4;
    //������Ի�ȡ�Ļ�����ʾ��ǰ���ָ����Դ�����ͣ�������ʹ��
    string maybe_type_name = 5;
}
```
����˵������һ����Դ�ľ���������������������Դ��λ�á����ԵĹؼ���Ϣ��

## resource_handle
ϸ��һ��resource_handle.h���ͻᷢ�����е�ResrouceHandle���������ǰ��proto��C++ʵ�֡�֮����Ҫ������ʵ��һ��C++�汾����Դ�������Ϊ�˱�����kernels������protos��

## resource_mgr
ϵͳ�еĴ�����Դ��Ҫ����Ч�Ĺ�����Щ��Դ���ͷ��࣬�ô��ָ��в�ͬ�����TF����˰�����������Դ�����ࡪ��ResourceMgr�����ߵĹ�ϵ���£�
```mermaid
graph LR
ResourceMgr-->Container
Container-->Resource
```
����ResourceMgr���ж���������Щ˽�����ݳ�Ա��
```
class ResourceMgr {
    //...
private:
    //...
    typedef std::pair<uint64,string> Key;
    typedef std::unordered_map<Key,ResourceBase*,KeyHash,KeyEqual> Container;
    const string default_container_;
    mutable mutext mu_;
    std::unordered_map<string,Container*> containers_ GUARDED_BY(mu_);
    std::unordered_map<uint64,string> debug_type_names GUARDED_BY(mu_);
}
```
���У�����Container��������һ��ӳ�䣬��Key��ResourceBase*��ӳ�䣬ǰ�߰�����Դ���͵Ĺ�ϣֵ������ResourceHandleProto�е�hash_code�ֶΣ�����Դ�����ƣ������߾���������Դ�Ļ�������ָ�룬�������������Ǻ���ὲ������Դ������������ĵ�˽��������containers_����Ҳ��һ��ӳ�䣬������������ӳ��Ϊ����ָ�롣ͨ������һ�������ӳ�䣬TFʵ������Դ����Ĺ��ܡ�
�ղ��ᵽ��ResourceBase�࣬���ǿ�һ������ʵ�֣�
```
class ResourceBase : public core::RefCounted {
public:
    virtual string DebugString() = 0;
    virtual int64 MemoryUsed() const {return 0;};
};
```
��ˣ�ResourceBaseͽ��������������ֻ��һ���ṩ�����ü������ܵĶ�����Դ��ʹ��һ��Ҫ��֮�������ṩ���ü�������Ҳ��Ϊ�˷������Դ�����ա�
�ٻص�ResourceMgr�࣬Ϊ��͸��������Ĺ��ܣ������ٿ�������������Ҫ�ӿڣ�
```
class ResourceMgr {
public:
    //��container�����д���һ����Ϊname����Դ
    Status Create(const string& container, const string& name, T* resource);
    //��container�в���һ����Ϊname����Դ
    Status Lookup(const string& container, const string& name, T** resource) const;
    //���container�а�����Ϊname����Դ����䵽*resource�У�����ʹ��creater()����һ����Դ
    Status LookupOrCreate(const string& container, const string& name, T** resource, std::function<Status(T**)> creater);
    //ɾ��container�е���Ϊname����Դ
    Status Delete(const string& container, const string& name);
    //ɾ�����handleָ�����Դ
    Status Delete(const ResourceHandle& handle);
    //ɾ��container�е�������Դ����ɾ����container
    Status Cleanup(const string& container);
    //ɾ�����������е�������Դ
    void Clear();
}
```
����˼���ǣ�ResourceMgr����������Delete����������һ�����������ƺ���Դ����Ϊ��������һ����ResourceHandleΪ������������Դ��������þͺ������ˣ������Է����ָ��һ����Դ��λ�á�
Ϊ�˸������ʹ��ResourceHandle��TF���ṩ�˺ܶศ��������Ϊ�˽�ʡƪ�������г���������
```
//����ResourceHandle
MakeResourceHandle
MakeResourceHandleToOutput
MakePerStepResourceHandle
HandleFromInput
//����ResourceHandle���һ�����Դ
CreateResource
LookupResource
LookupOrCreateResource
DeleteResource
```
˵����ô�࣬resource������ʲô�أ���TF�У���Щkernel�Ǵ�״̬�ģ���ĳ��ִ����ɺ�����Ҫ����һЩ״̬��Ϣ�������´�ִ�е�ʱ��ʱ����Щ״̬��Ϣ����resource��һ�֣�Ϊ�˷��������Щ״̬��Ϣ���Ź�������Դ��ص��ࡣ����һ��kernel��˵��һ������Ȼ�������ǣ��������Ҫ���м�״̬��Ϊresource��������������Ҫѡ���ĸ�container�أ������ͬkernel����ʱ״̬������Ĵ������Դ�������У��ܲ����㣬���TF�����һ���࣬ר����������kernel�������ҵ�һ�����ʵ���������������ContainerInfo��
```
class ContainerInfo {
public:
    Status Init(ResourceMgr* rmgr, const NodeDef& ndef, bool use_node_name_as_default);
    //...
private:
    ResourceMgr* rmgr_ = nullptr;
    string container_;
    string name_;
    bool resource_is_private_to_kernel_ = false;
}
```
��������ľ�������������ģ�
- ���container�����ǿգ���ʹ����������ʹ����Դ��������Ĭ��container
- ���shared_name�����Ƿǿյģ���ʹ�������������use_node_name_as_defaultΪ�棬���kernel�Ľڵ����Ʊ�������Դ���ƣ����򣬴���һ��ר���ڵ�ǰ���̵�����
ע�⣬�����container��shared_name����NodeDef������ֵ��
���Ǿ������������������kernel��ctx->resource_manager()�л�ȡ��Դ�ĺ�����
```
Status GetResourceFromContext(OpKernelContext* ctx, const string& input_name, T** resource);
```
���⣬resource_mgr.h�����������µ����ݣ�
- һ���ж���Դ�Ƿ񱻳�ʼ����OpKernel
- һ������ע��op�ĺ꣬���op��������ĳ�����͵���Դ���
- һ����������ָ����Դ�ľ������
- һ������ע��kernel�ĺ꣬���kernel��Ե��ǿ�������ĳ��������Դ�����op