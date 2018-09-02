## Ŀ¼
1. ���ĸ���
2. device
3. device_factory
4. device_mgr
5. device_set

## 1. ���ĸ���
��framework���֣����ǽ�����DeviceAttributes��DeviceBase�����ṹ����Щ��ʵ��Ϊ�����ǽ���Ҫ���ܵ�Device����׼���ġ�����Ȥ�Ķ��߿���ȥ�ع���ǰ�潲�������ݡ�Device��ֻ�Ƕ�DeviceBase��ļ̳У�û����Ӹ����µ����ݳ�Ա�����ṩ��Compute����ӿڡ�DeviceSet��һ���豸�����࣬��DeviceMgr��DeviceSet�Ĳ�ͬ�����ڣ����ṩ���豸����Ĺ��ܣ�Ϊ�豸���Һͼ����ṩ�˱��������ݽṹ�����DeviceFactory��Ϊ�˲���ĳ�����͵��豸׼���Ĺ����࣬ͬ�����豸���ͣ�����CPU�����Ӧ��ͬ�Ĺ�������ζ�Ų�ͬ��ʵ�֣�����ͬ�Ĺ������Ų�ͬ��Ȩ�ء������Ȩ����Ϊ�˸�������ѡ��ĳ�����͵��豸�õġ�

## 2. device
Device�࣬���˰������ڲ�˽�����ݵķ���API֮�⣬�������˺��ĵļ���API Compute������������һ�����Ľṹ��
```
class Device : public DeviceBase {
  public:
    virtual void Compute(OpKernel* op_kernel, OpKernelContext* context){
        op_kernel->Compute(context);
    }
    virtual void ComputeAsync(AsyncOpKernel* op_kernel, OpKernelContext* context, AsyncOpKernel::DoneCallback done){
        op_kernel->ComputeAsync(context, std::move(done));
    }
    //...
  private:
    const DeviceAttributes device_attributes_;
    DeviceNameUtils::ParsedName parsed_name_;
    OpSegment op_seg_;
    ResourceMgr* rmgr_ = nullptr;
}
```
TF�����豸��������Ҫ��ģ��������������ָ�ʽ��```/job:_/replica:_/task:_/(gpu|cpu):_```���ٸ����ӣ�```/job:train/replica:0/task:3/gpu:2```�����У�Device������ݳ�Աparsed_name_���Ƕ������豸���ƵĲ�⣬����Ȥ�Ķ��߿������п���ParsedName�Ķ��塣ResourceMgr��OpSegment����֮ǰ��framework����Ҳ�����ܹ��ˡ����Դ����ݽǶȽ���Deviceû��ʲô���ʵģ�ֻ�Ƕ�ԭ�еĹ����豸����������һ�����ϡ�����API�ĽǶȽ�����������һ������ӿ�Compute��ʵ����Ҳ���Ƕ�OpKernel�е�Compute�ӿڵķ�װ��

## 3. device_set
DeviceSet��һ�������࣬���ڹ���һ��ģ��ʹ�õĲ�ͬ�豸���������ԱȽϼ򵥣����ǿ����Ľṹ��
```
class DeviceSet {
  public:
    //...
  private:
    std::vector<Device*> devices_;
    std::unordered_map<string, Device*> device_by_name_;
    Device* client_device_ = nullptr;
}
```
���У�device_by_name_��һ�����豸ȫ�Ƶ��豸ָ���ӳ�䣬��client_device_�����Ǵ�devices_����ѡ�ģ�Ĭ�ϵĿͻ����豸��

## 4. device_mgr
DeviceMgr����˼����һ���豸�����࣬��ʵ����Ҫ���ṩ��һϵ�����ݽṹ�����API��Ч�ʣ����磬����Ҫ����һ�������豸�����豸ָ�룬����Ҫ��ĳ�����͵��豸������������ָ�Ƶ������DeviceMgrΪ��׼���˸�Ч�����ݽṹ����Ľṹ���£�
```
class DeviceMgr {
  public:
    //...
  private:
    typedef gtl::InlinedVector<Device*, 8> DeviceVec;
    DeviceVec devices_;
    std::unordered_map<StringPiece, Device*, StringPiece::Hasher> device_map_;
    core::Arena name_backing_store_;
    std::unordered_map<string, int> device_type_counts_;
}
```
device_map_��Ϊ����߲���ָ�����Ƶ��豸��Ч�ʣ�device_type_counts_��Ϊ����߲���ָ�����͵��豸����Ч�ʡ�

## 5. DeviceFactory
����ղ��ᵽ���ģ�DeviceFactory������ĳ���豸������CPU����ĳ��ʵ�ֵĹ����ࡣ�������ǿ���DeviceFactory��Ľṹ��
```
class DeviceFactory {
  public:
    static void Register(const string& device_type, DeviceFactory* factory, int priority);
    static DeviceFactory* GetFactory(const string& device_type);
    static Status AddDevices(const SessionOptions& options, const string& name_prefix, std::vector<Device*>* devices);
    static Device* NewDevice(const string& type, const SessionOptions& options, const string& name_prefix);
    virtual Status CreateDevices(const SessionOptions& options, const string& name_prefix, std::vector<Device*>* devices) = 0;
    static int32 DevicePriority(const string& device_type);
};
```
��������࣬���Ǹо����ɻ����ṩ�˺ܶ��API������û�����ݳ�Ա������ע�����Щ�������洢�������أ�
��ţ�������device_factory.cc�ļ��У��ҵ��������Ķ��壺
```
struct FactoryItem {
    std::unique_ptr<DeviceFactory> factory;
    int priority;
};
std::unordered_map<string, FactoryItem>& device_factories(){
    static std::unordered_map<string, FactoryItem>* factories = new std::unordered_map<string, FactoryItem>;
    return *factories;
}
```
���ڵڶ������������ڲ�������һ����̬��Ա������൱���ṩ��һ��ȫ�ֵĴ��豸�������Ƶ�������������ӳ�䡣ÿ��������Ҫ���ӳ��ʱ���͵������������ʵ���ϣ�DeviceFactory�ĺܶ��Ա��������������ʵ�ֵġ�
���⣬TF���ṩ��һ��Registrar�࣬ΪDeviceFactory�ṩ��ע�����ڣ�
```
template<class Factory> class Registrar {
  public:
    explicit Registrar(const string& device_type, int priority=50){
        DeviceFactory::Register(device_type, new Factory(), priority);
    }
};
```
��ʵ������Ϊĳ���豸����ע�����豸������
�����豸�����࣬�����ڴ����о�������priority������Ȩ�أ�������ϸ˵��һ�£�
- ����ͬ��һ���豸���ͣ���ͬ��ע������ɲ�ͬ��Ȩ�أ���ͬһ���豸���͵Ĳ�ͬʵ�֣�����ӵ�в�ͬ��Ȩ�ء�Ȩ����Ҫ��Ӧ���������������棺
- �����ϣ���һ����������ҪΪĳһ���ض����豸����ѡ�񹤳�ʱ��ӵ�����Ȩ�صĹ������ᱻѡ�����磬��������µ�����ע����Ϣ
```Registrar<CPUFactory1>("CPU", 125);```��
```Registrar<CPUFactory2>("CPU", 150);```
��ô������```DeviceFactory::GetFactory("CPU")```ʱ��CPUFactory2���ᱻ���ء�
- �����ϣ��ڶ�������Ҫ��DeviceSet��ѡ��һ���豸����ʱ��ѡ���˳����Ȩ��priority���������磬�������µ�����ע�᣺
```Registrar<CPUFactory>("CPU",100);```��
```Registrar<GPUFactory>("GPU",200);```
��DeviceType("GPU")���ᱻ����ѡ��
- ��ͬ�豸��Ĭ��Ȩ�����£�```GPU:200��SYCL:200��GPUCompatibleCPU:70��ThreadPoolDevice:60��Default:50```��