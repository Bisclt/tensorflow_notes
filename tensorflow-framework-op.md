## Ŀ¼
1. ���ĸ���
2. op_def
3. op
4. op_def_builder_wrapper
5. op_gen_lib

## 1. ���ĸ���
op��kernel��TF���������Ҫ������������һ��Ҫ��һ����ȵĻ���������Ϊop�൱�ں���������kernel�൱�ں���ʵ�֡��ٸ����ӣ����ھ�����ˣ��ҿ�������һ��op����MatMul��ָ���������ƣ����룬������������Լ��Բ��������Ƶȡ�opֻ�Ǹ������ǣ����������Ŀ����ʲô���������ṩ����ʵ�֣�������ĳ���豸�ϵľ���ʵ�ַ���������kernel�����ġ���ΪʲôҪ�Ѳ���������ʵ�ַ����أ���ΪTF��Ŀ�����ڶ��豸�����У��������ڱ����ʱ�����޷�Ԥ��֪��ĳһ�������������������豸�����еģ����磬��֪�����������GPU���и���Ч���㷨�������������δ���Ļ���û��GPU�������Ͳ��ܽ��У���TF�������Ծͻ����ۿۣ���ˣ���Ҫ�������������Ͳ�����ʵ�ַ��뿪��

## 2. op_def
���ڽ��ǲ�����������OpDef����Ҫ����̫���API������������һ��proto�С���������������Ҫ�����������������г����Ĵ��룺
```
message OpDef {
    string name = 1;//����������
    message ArgDef { //����������Ķ���
        string name = 1;
        string description = 2;
        DataType type = 3;//����4���ֶ�˵�������ݵ����ͣ��������
        string type_attr = 4;
        string number_attr = 5;
        string type_list_attr = 6;
        bool is_ref = 16;//���������Ƿ�Ϊ����
    };
    repeated ArgDef input_arg = 2;//��������
    repeated ArgDef output_arg = 3;//�������
    message AttrDef {
        string name = 1;
        string type = 2;
        AttrValue default_value = 3;
        string description = 4;
        bool has_minimum = 5;
        int64 minumum = 6;
        AttrValue allowed_values = 7;
    }
    repeated AttrDef attr = 4;
    OpDeprecation deprecation = 8;
    string summary = 5;
    string description = 6;
    bool is_commutative = 18;//�Ƿ�ɽ�������op(a,b) == op(b,a)
    bool is_aggregate = 16;//�Ƿ�ɾۼ�
    bool is_stateful = 17;//�Ƿ����״̬
    bool allows_uninitialized_input = 19;//��Ը�ֵ����
};
message OpDeprecation {
    int32 version = 1;
    string explanation = 2;
};
message OpList {
    repeated OpDef op = 1;
};
```
���ǿ�����OpDef������ĵ����ݳ�Ա�ǲ������ơ����롢��������������еĲ�����������أ�����֮ǰ�ᵽop�൱�ں������������������ʵ�Ǵ�ģ������ģ�������node_def��ʹ�øò���ʱ��������Ҫ����������ʵ�ʵ���ֵ������ڽ���������node_defʱ����ϸ������
�������еļ��������ĵ㣬����˵����
- ArgDef�е�3-6�ĸ��ֶΣ���Ϊ�������������������ͣ�������������һ������ʱ��type��type_attr������Ϊ����������������ͣ�������������һ������ͬ�������͵��������ɵ�����ʱ��number_attr������Ϊint��Ӧ�ı�ʶ��������������һ�����������ɵ��б�ʱ��type_list_attr������Ϊlist(type)��Ӧ�ı�ʶ��
- AttrDef�е�has_minimum�ֶΣ�������������Ƿ�����Сֵ���������������int����ôminimum�����������Сֵ����������������б���ôminimum�����б����̳��ȡ�
- is_aggregate����ֶΣ�������ǰ�Ĳ����Ƿ��ǿɾۼ��ģ�һ���ɾۼ��Ĳ����ǣ��ܽ�������������ͬ���ͺ���״�����룬���ұ��������ÿ����������ͺ���״��ͬ������ֶζ��ڲ������Ż��ǳ���Ҫ�����һ�������ǿɾۼ��ģ��������������Զ����ͬ���豸����ô���ǾͿ��԰Ѿۼ��Ż���һ�����εĲ����������豸�ڲ����������ۼ�������ڲ������ڵ��豸���У������������Ч�ʡ������Ż����ڷֲ�ʽ�Ļ���ѧϰģ��ѵ���ǳ��а�����Spark ML�е�TreeAggregate��ʵ�����������Ż�����ϧ��ֹ���߿�����TF1.2�汾����û��ʵ������Ż���
- is_stateful����ֶΣ�������ǰ�Ĳ����Ƿ��Ǵ���״̬�ģ�ʲô���������״̬�أ�����Variable��

Ϊ�˷������OpDef�Ĺ�����TF�������OpDefBuilder�࣬����˽�����ݳ�Ա���£�
```
class OpDefBuilder {
    //...
  private:
    OpRegistrationData op_reg_data_;
    std::vector<string> attrs_;
    std::vector<string> inputs_;
    std::vector<string> outputs_;
    string doc_;
    std::vector<string> errors_;
}
```
���Է��֣�����errors_�ֶ�֮�⣬�������ݼ������ǰ�OpDef�Ľṹԭ�ⲻ���İ��˹��������������Ƿ�����һ���µĽṹ��OpRegistrationData�����Ľṹ���£�
```
struct OpRegistrationData {
  public:
    //...
    OpDef op_def;
    OpShapeInferenceFn shape_inference_fn;
}
```
������ṹ�У�����������֪��OpDef֮�⣬��������һ��OpShapeInferenceFn�ṹ�����Ķ������£�
```
typedef std::function<Status(shape_inference::InferenceContext* c)> OpShapeInferenceFn;
```
����ṹ�Ķ����У��漰�������Ǻ���Ҫ��������״�ƶϵ����ݣ���������ֻ��Ҫ֪����OpShapeInferenceFn��һ��������������������״�������״�����ƶϵĺ������ɡ�

## 3. op
Ϊ�˷���Բ�������ͳһ����TF����˲���ע�����ĸ�����ں��Ľṹ��ͳһ�����������ͣ����ǲ���İ��������֮ǰ���ܵ�ResourceMgr��AllocatorRegistry��ԭ�����һ�ޡ���ˣ��������ע��������ã�����Ϊ���ֲ����ṩһ��ͳһ�Ĺ���ӿڡ�
����ע����ļ̳нṹ���£�
```
graph LR
OpRegistry-->OpRegistryInterface
OpListOpRegistry-->OpRegistryInterface
```
���У�OpRegistryInterface��һ���ӿ��࣬���ṩ��ע����������Ĳ��ҹ��ܣ�
```
class OpRegistryInterface {
  public:
    //...
    //�������ҷ���
    virtual Status LookUp(const string& op_type_name, const OpRegistrationData** op_reg_data) const = 0;
    Status LookUpOpDef(const string& op_type_name, const OpDef** op_def) const;
}
```
OpRegistry����������˼�����ĺ��Ľӿں��������£�
```
class OpRegistry : public OpRegistryInterface {
  public:
    typedef std::function<Status(OpRegistrationData*)> OpRegistrationFactory;
    void Register(const OpRegistrationDataFactory& op_data_factory);//����ע��
    static OpRegistry* Global();//����һ��ȫ�־�̬����
    typedef std::function<Status<const Status&, const OpDef&)> Watcher;
    Status SetWatcher(const Watcher& watcher);
  private:
    mutable mutex mu_;
    mutable std::vector<OpRegistrationDataFactory> deferred_ GUARDED_BY(mu_);
    mutable std::unordered_map<string, const OpRegistrationData*> registry_ GUARDED_BY(mu_);
    mutable bool initialized_ GUARDED_BY(mu_);
    mutable Watcher watcher_ GUARDED_BY(mu_);
}
```
�������м�������˼�ĵط���
- ע�ắ��Register�����룬��һ���������ã������������һ��OpRegistrationDataָ����Ϊ���룬��ô����������õ����þ�����ʲô�أ����¶�Ӧ��Դ���룺
```
Status OpRegistry::RegisterAlreadyLocked(const OpRegistrationDataFactory& op_data_factory) const {
    std::unique_ptr<OpRegistrationData> op_reg_data(new OpRegistrationData);
    Status s = op_data_factory(op_reg_data.get())
    //...
}
```
- �����ϣ�ԭ���������Ƚ�����һ��OpRegistrationData�Ķ���Ȼ������Ϊ��������op_data_factory������������������������������ݣ�Ȼ����������������Ϣ����ע�ᡣ
- Watcher��һ����������ÿ�ε�����ע����һ��������ʱ����ע�Ჽ������Ҫ����һ�����Watcher�����������Է������Ƕ�ע��Ĳ������м�أ����еĲ���ע�ᶯ�����Ӳ��������۾������ǿ��Ը����Լ�����Ҫ����Watcher��
- deferred��һ��������OpRegistrationDataFactory����������ע��Ĳ���һ����ȷ���������������Ҫ�Բ�������ȫ�ֲ���ʱ����ͳһע��deferred_�еĲ������������ƺ���Ϊ�����ע���Ч�ʣ��������߻�û���ҵ������ԭ����������з��֣������֪�ҡ�
- registry_����ע��Ĳ���������ŵ�λ�ã����Ľṹ�ܼ򵥣���һ�����������������ݵ�ӳ�䡣

���򵥽���һ��OpListRegistry��������������OpList��ʼ��һ��ע��������ע�⣬OpList������OpDef���б�������������״�ƶϺ��������Ϣ��������ע�����еĲ������ǲ�������״�ƶϺ����ġ��������Ҫ���ҵĲ�������Ҫ��״�ƶϺ������Ϳ���ʹ�����ע����������˽���������£�
```
class OpListOpRegistry : public OpRegistryInterface {
  public:
    //...
  private:
    std::unordered_map<string, const OpRegistrationData*> index_;
}
```

## 4. op_def_builder_wrapper
Ϊ�˷���Բ�����ע�ᣬTF�����רΪע������ĺ꣬�������£�
```
REGISTER_OP("my_op_name")
    .Attr("<name>:<type>")
    .Attr("<name>:<type>=<default>")
    .Input("<name>:<type-expr>")
    .Output("<name>:<type-expr>")
    .Doc(R"(
<1-line summary>
<rest of the description (potensitally many lines)>
...
)");
```
����д����󷽱���ע������Ĺ��̡�����Ҫʵ�����ֺ������Ŀǰ���໹���㲻�ˡ�TF�������������ʵ��������ܣ�һ����Ϊop�Ĺ����ṩ��ʽ�﷨֧�֣�����һ�������op����������ṩ����ע�Ṧ�ܡ���������ֱ���OpDefBuilderWrapper��OpDefBuilderReceiver������������ǰ�ߣ�
```
class OpDefBuilderWrapper<true> {
  public:
    OpDefBuilderWrapper(const char name[]) : builder_(name){}
    OpDefBuilderWrapper<true>& Attr(StringPiece spec){
        builder_.Attr(spec);
        return *this;
    }
    //...
  private:
    mutable ::tensorflow::OpDefBuilder builder_;
}
```
������Ƚ�����˼�����ȹ���˼�������������Ƕ�OpDefBuilder��һ����װ���ṩ�˼�����ȫһ�µ�API����Σ�����API���������ͣ��Ҷ����ض��������Ϊ��ʽ�����������ṩ�˿��ܡ�ֵ��ע����ǣ���������������һ��<true>�����ĺ������Ǵ����������
��������OpDefBuilderReceiver��
```
struct OpDefBuilderReceiver {
    OpDefBuilderReceiver(const OpDefBuilderWrapper<true>& wrapper);
    constexpr OpDefBuilderReceiver(const OpDefBuilderWrapper<false>&){}
};
```
���ṩ�Ĺ��캯������OpDefBuilderWrapper��Ϊ���������Ҳ����˵�����ǿ���ͨ����ֵ����Ѻ���ֱ�Ӹ�ֵ��ǰ�ߣ�����REGISTER_OP�ĺ궨�壺
```
//Ϊ�˺��Բ���Ҫ��ϸ�ڣ����´��������ʵ�ɾ��
#define REGISTER_OP(name) REGISTER_OP_UNIQ_HELPER(__COUNTER__, name)
#define REGISTER_OP_UNIQ_HELPER(ctr, name) REGISTER_OP_UNIQ(ctr, name)
#define REGISTER_OP_UNIQ(ctr, name) \
    static OpDefBuilderReceiver register_op##ctr = OpDefBuilderWrapper<SHOULD_REGISTOR_OP(name)>(name)
```
���Ƿ��֣�REGISTER_OP����һȦ�����վ�������OpDefBuilderWrapper�Բ������з�װ��Ȼ�������Ϊ�������ݸ�OpDefBuilderReceiver�Ĺ��캯��������������캯���У�����˶Բ�����ע�᣺
```
OpDefBuilderReceiver::OpDefBuilderReceiver(const OpDefBuilderWrapper<true>& wrapper) {
    OpRegistry::Global()->Register([wrapper](OpRegistrationData* op_reg_data) -> Status {
        return wrapper.builder().Finalize(op_reg_data);
        });
    }
}
```
��������������¸ղ����Ĺ��ӣ�OpDefBuilderWrapper<true>��������<true>���״���ʲô�أ�����֪����TFΪ����׼���˺ܶ�Ĳ���������Щʱ�����ǿ����ò��������еĲ���������Ҫ����һ���֣������������ȫ�����룬������ǵ�����ʱ������ܴ�ĸ�������ˣ�TF�����������һ��ͷ�ļ����ú�SHOULD_REGISTOR_OP����������Ҫ�����Ĳ��������磺
```
#define SHOULD_REGISTOR_OP(Add) true
#define SHOULD_REGISTOR_OP(Subtract) false
```
��ʾ����ϣ������Add��������ϣ������Subtract�������������ܹ�������Ҫ�����Լ���TF����ʱ���ˡ����Դ�����г������OpDefBuilderWrapper<true>��֮�⣬����һ��OpDefBuilderWrapper<false>�࣬�����Щ����ϵͳ�Ǳ���Ҫ�����ģ�����һЩ�ڲ�������TFΪ�����������һ���꣬��������SHOULD_REGISTOR_OP�ĺ궨�壬����Ȥ�Ķ��߿���ȥ����Դ���롣

## 5. op_gen_overrides
����TF�Ĳ�����չ��ԭ�еĲ�����ʱ�ᱻ������������������Ϊ�������е�ͼ���䣬TF�����OpGenOverrides�Ľṹ�����£�
```
message OpGenOverride {
    string name = 1;
    bool skip = 2;//ֱ�ӷ����������
    bool hide = 3;//��������
    string rename_to = 4;
    repeated string alias = 5;//����API������
    message AttrDefault {
        string name = 1;
        AttrValue value = 2;
    }
    repeated AttrDefault attr_default = 6;//�޸Ĳ���Ĭ��ֵ
    message Rename {
        string from = 1;
        string to = 2;
    }
    repeated Rename attr_rename = 7;
    repeated Rename input_rename = 8;
    repeated Rename output_rename = 9;
}
message OpGenOverrides {
    repeated OpGenOverride op = 1;
}
```
����������ṹ���ֵľ���ԭ�򣬻�û�������������������ټ��ϣ�
������滻��������OpGenOverrideMap�����ʵ�ֵģ�������һϵ�а���OpGenOverrides proto���ı��ļ���Ȼ��������������ÿ�����в����ĵ�����
```
class OpGenOverrideMap {
  public:
    Status LoadFile(Env* env, const string& filenames);
    const OpGenOverride* ApplyOverride(OpDef* op_def) const;
  private:
    std::unordered_map<string, std::unique_ptr<OpGenOverride>> map_;
};
```