# Ŀ¼
1. ʲô��resource
2. ���ʹ��resource
3. ��ι���resource
4. ����resource
5. �����ṹ
6. ��ϵͼ
7. �漰���ļ�
8. ������¼

# 1. ʲô��resource
����֪����TF�ļ��������豸��ɵġ�ÿ���豸�������ɸ��ڵ㣬����Щ�ڵ����ʵ�ʵļ��㡣��Щʱ��������Ҫ�ڲ�ͬ�Ľڵ�֮�䣬����һЩ���ݣ����磬����ֵ��kv�洢�����У���ȡ���ȵȣ���Щ��ͬ�豸�ϵĽڵ㹲������ݣ�������Դ��
���е���Դ�඼�̳���һ�����࣬ResourceBase�����濴������ʵ�֣�
```
class ResourceBase : public core::RefCounted {
public:
    virtual string DebugString() = 0;
    virtual int64 MemoryUsed() const {return 0;};
};
```
���̳���`core::RefCounted`��Ҳ����˵����Դ����ӵ�����ü����Ĺ��ܡ�����Ϊ�˷������Դ��ʹ�ý��м�أ�����Դ���ú��ܹ���ʱ�����ͷŵ���

# 2. ���ʹ��resource
��ͨ�ڵ����޷�ֱ��ʹ�ù�����Դ�ģ�����ͨ��һ�ְ�����ResourceOpKernel������ڵ㣨����OpKernel����ϸ���壬��μ�[Kernel](https://www.cnblogs.com/jicanghai/p/9545674.html)��Ŀǰ����Ҫ֪�������ǽڵ�������ʵ��ִ�м������ͺ��ˣ������ֽڵ������ͨ�ڵ�����Դ�������в��һ򴴽�ĳ���ض����͵���Դ����ͼ��ʾ��
```
graph LR
    A(OpKernel) -->|�̳���| B(ResourceOpKernel)
    B(ResourceOpKernel) -->|���������| C(Resource)
    A(OpKernel) -->|�̳���| D(NormalOpKernel)
    D(NormalOpKernel) -->|ʹ����Դ| B(ResourceOpKernel)
```
ResourceOpKernel��Ķ������£��������ӿڣ���
```
template <typename T>
class ResourceOpKernel : public OpKernel {
  public:
    //...
    void Compute(OpKernelContext* context) override LOCKS_EXCLUDED(mu_);
  protected:
    mutex mu_;
    ContainerInfo cinfo_ GUARDED_BY(mu_);//�����˶���Դ��Ҫ��
    T* resource_ GUARDED_BY(mu_) = nullptr;
  private:
    virtual Status CreateResource(T** resource) EXECLUSIVE_LOCKS_REQUIRED(mu_) = 0;//����һ��T���������Դ�����������鵱ǰ��ResourceOpKernel��������
    virtual Status VerifyResource(T* resource);//У��resource�Ƿ��ǵ�ǰ������Ҫ������
    PersistentTensor handle_ GUARDED_BY(mu_);
}
```
���е�Compute���������״ε���ʱ�������cinfo_�а����Ķ���Դ��Ҫ�󣬴���Դ�������в��ң����ߴ���һ���µ���Դ��
����ǣ���������µĸ��һ����ContainerInfo��һ������Դ�����������ǽ������������ۡ�

# 3. ��ι���resource
��򵥵ģ��������豸��ά��һ����Դ���Ƶ���Դʵ���ӳ�䣬�����ֹ���ʽ���ڴֲڣ�TFʹ�������ĸ���ʵ���˶���Դ�ķ��飬���濴һ�¶������Ķ��壺
```
typedef std::pair<uint64,string> Key;
typedef std::unordered_map<Key,ResourceBase*,KeyHash,KeyEqual> Container;
```
���У�Key�Ƕ���Դ��������uint64������Դ���͵Ĺ�ϣֵ��string��������Դ�����ƣ���Container�����Ͼ���һ����Դ��������Դָ���ӳ�䡣
Ϊ�˶�ͬһ���豸�ϵ���Դ�ṩͳһ�Ĺ�����ڣ�TF������ResourceMgr�࣬��˽�����ݳ�Ա���£�
```
class ResourceMgr {
  private:
    //��ǰ�豸�ϵ�Ĭ���������ƣ��������ʱû��ָ������������Ĭ�������в���
    const string default_container_;
    mutable mutext mu_;
    //�������Ƶ�����ָ���ӳ��
    std::unordered_map<string,Container*> containers_ GUARDED_BY(mu_);
    //��Դ���͵Ĺ�ϣֵ����Դ�������Ƶ�ӳ��
    std::unordered_map<uint64,string> debug_type_names GUARDED_BY(mu_);
};
```

��Դ������Ĺ����ӿ����£�
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
};
```
ע�ͺ�����������Ͳ���׸���ˡ�
�ղ��ᵽ��Ϊ����ResourceOpKernel��Ķ����ܹ��ҵ���Ӧ����Դ�������ڲ�������һ��ContainerInfo���࣬���������ǣ�����һ����״̬��OpKernel����������Ҫͨ���ĸ�container/shared_name��������ʶ�Ӧ����Դ����Ķ������£�
```
class ContainerInfo {
  public:
    Status Init(ResourceMgr* rmgr, const NodeDef& ndef, bool use_node_name_as_default);
  private:
    ResourceMgr* rmgr_ = nullptr;
    string container_;
    string name_;
    bool resource_is_private_to_kernel_ = false;
};
```
���У�������߹������£�
- ������NodeDef�������У�������"container"����ʹ������������ƣ�����ʹ��rmgr��Ĭ��������
- ������NodeDef�������У�������"shared_name"����ô���Դ���Ϊ��Դ�����ƣ��������"use_node_name_as_default"Ϊ�棬��ʹ�ýڵ�������Ϊ��Դ���ƣ����Ϊ�٣�������һ��Ψһ��ʶ��Ϊ��Դ����

�ɼ���ContainerInfo�������ṩ���ǣ�**��ΰ����ڵ��ҵ���Դ��λ��**��������**��Դλ�õľ�̬����**��Ϊ�˷���ָ����Դ��TF�������Դ����ĸ������protobuf�������£�
```
message ResourceHandleProto {
    //��Դ�����豸
    string device = 1;
    //��Դ��������
    string container = 2;
    //��Դ����
    string name = 3;
    //��Դ���͵Ĺ�ϣֵ
    uint64 hash_code = 4;
    //��Դ�������ƣ���������
    string maybe_type_name = 5;
}
```
��Դ����ṩ�˶�����Դλ�ú����Ե���ϸ������ͨ��������ǿ���Ψһ��λ��һ����Դ�����������Դ����ǿ����л��ģ�������ǿ��Է���Ĵ�����������Ϊ�˱���ʹkernel������proto��TF�������һ����ResourceHandleProtoӵ����ͬ���ܵ��࣬ResourceHandle�����Դ���롣

# 4. ����resource
��ƪ�Ŀ�ͷ�ᵽ�ˣ����õ���Դ��������ֵ��kv�洢�����У���ȡ���ȵȣ���������ֵ֮�⣬TFΪ�������ṩ��Ӧ�õĽӿڣ����£�
```
class LookupInterface : public ResourceBase;
class QueueInterface : public ResourceBase;
class ReaderInterface : public ResourceBase;
```
����
- kv�洢���ӿ�LookupInterface�ṩ��һ���������ҺͲ���kv�洢���Ľӿڣ����㲻ͬ�ļ���ڵ�֮�乲��kv���ݣ�
- ���нӿ�QueueInterface�ṩ����ӡ����ӵȽӿڣ����㲻ͬ�ڵ�֮�乲����У�����ڼ���ͼ���첽ִ���Ƿǳ���Ҫ�ģ���ȡ���ӿ��ڲ�Ҳ�õ��˶��нӿڣ�
- ��ȡ���ӿ�ReaderInterface������TF֧�ֵ��ļ���ȡ����ͳһ�ӿڡ�TF����֧�ִӲ�ͬ��ʽ���ļ��ж�ȡ���ݣ�ÿһ���ļ���ȡ������һ����Դ������Ҫ�̳���ReaderInterface�ӿڣ�

���У����ڶ�ȡ���ӿڣ���ʵ�����Ķ�ȡ����Դ������ֱ�Ӽ̳��Զ�ȡ���ӿڣ����Ǽ̳�������һ�������࣬ReaderBase�������Ϊ��ȡ���ӿڵ�ÿһ��API�ṩ��һ��Ĭ�ϵ�ʵ�֣�����������ֻҪ��д��ЩĬ��ʵ�֣�������ɶԲ�ͬ�ļ���ʽ�Ķ�ȡ��
ͬʱ��Ϊ�˷����ȡ��ȡ����Դ��TF��ΪResourceOpKernel������һ��ר���ڻ�ȡ��ȡ���ӿڵ�������ReaderOpKernel��
```
class ReaderOpKernel : public ResourceOpKernel<ReaderInterface>;
```
��������ڲ��һ��ߴ����ȡ����Դ������Щ��ȡ����Դ��Ҫ������ReaderBase�ࡣ

# 5. �����ṹ
Ϊ�˷������Դ���в�����TF�������ResourceOpKernel�����֮�⣬����������ֺˣ�IsResourceInitialized��ResourceHandleOp��ǰ�����ڼ��ĳ����Դ�Ƿ��Ѿ���ʼ������������Ϊĳ����Դ������Դ��������嶨����Բ鿴Դ���롣

# 6. ��ϵͼ
```
graph TB
    A(core::RefCounted)-->|����|B(ResourceBase)
    B(ResourceBase)-->|����|C(LookupInterface)
    B(ResourceBase)-->|����|D(QueueInterface)
    B(ResourceBase)-->|����|E(ReaderInterface)
    E(ReaderInterface)-->|����|F(ReaderBase)
    F(ReaderBase)-->|����|G(�����Ķ�����Դ)
    H(OpKernel)-->|����|I(ResourceOpKernel)
    I(ResourceOpKernel)-->|����|J(ReaderOpKernel)
    J(ReaderOpKernel)-.���һ򴴽�.->G(�����Ķ�����Դ)
    H(OpKernel)-->|����|K(IsResourceInitialized)
    H(OpKernel)-->|����|L(ResourceHandleOp)
    L(ResourceHandleOp)-.����.->M(ResourceHandle)
    M(ResourceHandle)-.ָ��.->G(�����Ķ�����Դ)
    K(IsResourceInitialized)-.�ж��Ƿ��ʼ��.->G(�����Ķ�����Դ)
    I(ResourceOpKernel)-.ӵ��.->N(ContainerInfo)
    N(ContainerInfo)-.��λ.->G(�����Ķ�����Դ)
```

# 7. �漰���ļ�
- resource_handle
- resource_mgr
- resource_op_kernel
- lookup_interface
- queue_interface
- reader_interface
- reader_base
- reader_op_kernel

# 8. ������¼
- v1.0 2018-08-25 �ĵ�����
- v2.0 2018-09-08 �ĵ��ع�