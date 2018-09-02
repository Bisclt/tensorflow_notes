## Ŀ¼
1. ���ĸ���
2. device_attributes
3. device_base

## 1. ���ĸ���
���豸����һ����������������ĸ����TF�У��豸deviceרָ�ܹ�ִ��ʵ�ʼ���ļ����豸������CPU��GPU��SYNC�豸�ȵȡ���ˣ�һ��Ҫ�������ĸ������ֿ���һ̨�������԰�������豸��

## 2. device_attributes
���豸����һ�������ĸ���֮�����ǿ���TFΪ�������豸����׼����proto��DeviceAttributes��
```
message DeviceAttributes {
    string name = 1;
    string device_type = 2;//�豸����
    int64 memory_limit = 4;//�ڴ��С
    DeviceLocality locality = 5;//ƽ̨��صģ�Ϊ֧��������Ч�����׼��������
    fixed64 incarnation = 6;//ÿ���豸�ڳ�ʼ����ᱻ����һ��ȫ��Ψһ�ı�ţ������ű��벻��Ϊ0
    string physical_device_desc = 7;//���豸����Ӧ�������豸���ַ�������
};
```
��������locality�ֶε���ϸ���壬���߻�û�ҵ������Ӧ�ã���֪���Ķ��߻����֪��

## 3. device_base
DeviceAttributesֻ�Ƕ��豸���Ե�һЩ���������������豸������DeviceBase����������һ�����Ľṹ��
```
class DeviceBase {
  public:
    explicit DeviceBase(Env* env) : env_(env){}
    //...
  private:
    Env* const env_;
    CpuWorkerThreads* cpu_worker_threads_ = nullptr;
    GpuDeviceInfo* gpu_device_info_ = nullptr;
    Eigen::ThreadPoolDevice* eigen_cpu_device_ = nullptr;
};
```
���ǿ�����DeviceBase���캯���Ĳ�����Envָ�룬�����Env��lib�ļ����ж��壬�ǶԲ���ϵͳ��ع��ܵ�ͳһ��װ���������ļ�ϵͳ�ȹ��ܣ�ʹframework��ʵ�־����Բ���ϵͳ��͸����
���⣬DeviceBase��˽�г�Ա�У�������������û�������࣬�ֱ���CpuWorkerThreads��GpuDeviceInfo�����Ƿֱ������ǵĶ��壺
```
struct CpuWorkerThreads {
    int num_threads = 0;
    thread::ThreadPool* workers = nullptr;
}
struct GpuDeviceInfo {
    perftools::gputools::Stream* stream = nullptr;
    DeviceContext* default_context = nullptr;
    EventMgr* event_mgr = nullptr;
    int gpu_id = -1;
};
```
���Կ�����ǰ����һ�����̳߳صļ򵥷�װ��������������GPU��ص���Ϣ�����е�stream����ִ�����������ں������ϸ���ܡ�EventMgr��һ���¼���������������Ӧ������¼�����ôDeviceContext��ʲô�أ�
```
class DeviceContext : public core::RefCounted {
  public:
    //...
    virtual void CopyCPUTensorToDevice(const Tensor* cpu_tensor, Device* device, Tensor* device_tensor, StatusCallback done) const;
    virtual void CopyDeviceTensorToCPU(const Tensor* device_tensor, StringPiece tensor_name, Device* device, Tensor* cpu_tensor, StatusCallback done);
};
```
����һ���������ü������࣬��Ҫ��API����CPU���豸֮�������������Ҳ����˵���κ�һ��GPU�豸��������CPU�������豸����֮���໥������API�ӿڡ�
�����������ܽ�һ�£�DeviceBase���CPU�豸��������һ��CPU���̳߳أ���һ��eigen_cpu_device��������GPU�豸��������һ��GpuDeviceInfo������ṹ�г��˰���GPUִ�������¼���Ӧ��֮�⣬��������һ��DeviceContext������ṹ�а�����CPU��GPU֮���໥����������API��