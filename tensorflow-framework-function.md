## Ŀ¼
1. ���ĸ���
2. FunctionDef
3. function related classes

## 1. ���ĸ���
�ڽ���function�ĸ���֮ǰ������Ҫ�Ȼع���op��op�ǹ涨�����������Ĳ������������о�node��ʱ������Ҳ������NodeDef�ǰ���OpDef�ģ���ô�ǲ���op��ֻ���ǽڵ㼶��Ĳ����أ�������ˣ������ǿ���Ƕ�׵ģ�Ҳ����˵������A�����ڲ������˲���BCD��������Ƕ����function�������ˣ�function��ʵ����һЩ���op�������ı����Ǹ������룬��������������������op�Ķ�λ��ͬ������һЩ��op�����ǿ��Զ��庯����֮��Ӧ����Щ�����ڲ������OpDef����ʾ���������ǩ�������롢�������Ҳ�����һϵ��NodeDef�����ڱ�ʾ�����ڲ������л��ơ�

## 2. FunctionDef
�����������⣬����������һ��FunctionDef�Ľṹ��
```
message FunctionDef {
    OpDef signature = 1;
    map<string, AttrValue> attr = 5;//�����е��ڲ�����
    repeated NodeDef node_def = 3;
    map<string, string> ret = 4;//һ����signature������������ƣ���node_def�������ӳ��
}
message GradientDef {
    string function_name = 1;//ԭ��������
    string gradient_func = 2;//�ݶȺ�������
}
message FunctionDefLibrary {
    repeated FunctionDef function = 1;
    repeated GradientDef gradient = 2;
}
```
�����¼�����Ҫ˵����
- �������Ǹղ���˵�ģ�node_def��һϵ�нڵ㣬��Щ�ڵ������һ���γ��˺����ڲ��Ľṹ��OpDef�а�����������������ƣ���function�����ǵ�����Ǳ�������node_def�еģ�������Ҫһ����OpDef�е�������Ƶ�������ڽڵ����ƺͶ˿ںŵ�ӳ�䣬���Ǿ�����ret��
- TF֧���ݶȼ��㣬����ΪTF���ÿ�����������������ݶȺ������ݶȺ�������Ҳ��һ���������������������Ϊ���ܽ�ԭ���������ݶȺ�����ϵ��һ�𣬾�����GradientDef����ṹ������ṹ�а�����ԭ���������ƣ�Ҳ������ԭ��������Ӧ���ݶȺ��������ơ�
- TF������ʱ������һ����������⣬��Ҫʹ��ĳ������ʱ������ȥ�����ң��������������������˶����ͨ���������ݶȺ�����

## 3. function related class
### 3.1 FunctionDefHelper
Ϊ�˷����FunctionDef�Ķ��壬�����FunctionDefHelper�࣬���������Է���Ķ��庯�������£�
```
FunctionDef my_func = FunctionDefHelper::Create(
  "my_func_name",
  {"x:T", "y:T"},//ÿ�����������һ���ַ�����ʾ
  {��z:T"},//ÿ�������һ���ַ�����ʾ
  {"T: {float, double}"},//ÿ������һ���ַ���
  {
      {{"o"},"Mul",{"x","y"},{{"T","$T"}}}
  },//ÿ���ڵ��Ӧһ��Ԫ��
  {{"z", "o:z"}}//����������ڵ������ӳ��
);
```
������ʵ�ֱȽϼ򵥣��������ǾͲ���׸���ˡ�

### 3.2 FuncionCallFrame
��TFͼ�У����Ҫ����һ��function����֪�����������ǲ����ģ����ǻ�ҪΪ�����д������ݣ��Լ��Ӻ����з������ݣ��ṩ�ṹ�͹����ϵ�֧�֡����ǵ�OpKernel���Compute������ÿ��kernel�ļ��㺯����ʹ����ͬ��һ���ӿڣ�����ͬһ���ӿ�ʵ���˲�ͬ�����㣬���ܾ����ں������������OpKernelContext�����൱��Compute�������õ������ģ���ͬ���Ľӿڣ�����Ϊ��ȫ��ͬ�������ṩ֧�֡���Ҳ����FunctionCallFrame���ڵ����壬����������һ��������תվ���Ѻ�������������������ṹ���ں�������������ٰ�����������룬�ú��������߻�ȡ��Ҫ�����ݡ���ĳ�������Ͻ������������������ڵ�ջ֡����Ҳ����FunctionCallFrame������ֵ�������
```
class FunctionCallFrame {
    //...
  private:
    DataTypeVector arg_types_;
    DataTypeVector ret_types_;
    gtl::InlinedVector<Tensor, 4> args_;
    struct Retval {
        bool has_val = false;
        Tensor val;
    };
    gtl::InlinedVector<Retval, 4> rets_;
}
```
���Կ�����������˽�����ݳ�Աֻ������������͡����������ֵ�������࣬�����Ͼ��Ǻ������õ�һ����תվ��

### 3.3 FunctionLibraryDefinition
�ղ������ڿ��������proto��ʱ�򿴵�һ���ṹ��FunctionDefLibrary����������Ҫ���������Definition�౾������һ��ע�������ṩ�˺���ע�ᡢ���ҵȹ��ܣ���Library��������һ����������ļ��ϣ����߱����ҹ��ܡ�������������һ�£���Ľṹ��
```
class FunctionLibraryDefinition : public OpRegistryInterface {
  private:
    struct FunctionDefAndOpRegistration {
        FunctionDefAndOpRegistration(const FunctionDef& fdef_in);
        FunctionDef fdef;
        OpRegistrationData op_registration_data;
    };
    const OpRegistryInterface* const default_registry_;
    gtl::FlatMap<string, std::unique_ptr<FunctionDefAndOpRegistration>> function_defs_;
    gtl::FlatMap<string, string> func_grad_;
};
```
�����������ṩ��һ�������function���м��й���ĵط���

### 3.4 FunctionLibraryRuntime
����˼�壬�Ǻ����������ʱ�ࡣΪ������ִ���ṩ�˺ܶ�����Ľӿڡ��������ǰ�����FunctionLibraryDefinition�����֮�ϵģ��ṩAPI֧�֣�������û���κ����ݳ�Ա�ġ����Ǽ򵥿��������ṩ����ЩAPI��
```
class FunctionLibraryRuntime {
  public:
    //...
    virtual Status Instantiate(const string& function_name, AttrSlice attrs, Handle* handle) = 0;//�ò���ʵ����һ������
    virtual const FunctionBody* GetFunctionBody(Handle h) = 0;//��ȡһ���Ѿ�ʵ�����˵ĺ����ĺ�����
    virtual void Run(const Option& opts, Handle handle, gtl::ArraySlice<Tensor> args, std::vector<Tensor>* rets, DoneCallback done) = 0;//�첽�ĵ���һ��ʹ��handle��ʶ�ĺ���
    virtual Status CreateKernel(const NodeDef& ndef, OpKernel** kernel) = 0;//����ndef������һ��kernel
    virtual bool IsStateful(const string& function_name) = 0;//�ú����Ƿ��Ǵ���״̬��
    virtual Device* device() = 0;//�����������ڵ��豸
    virtual const FunctionLibraryDefinition* GetFunctionLibraryDefinition() const = 0;
    virtual Env* env() = 0;
};
```