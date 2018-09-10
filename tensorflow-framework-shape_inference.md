# Ŀ¼
1. ʲô����״�ƶ�
2. InferenceContext
3. ��ϵͼ
4. �漰���ļ�
5. ������¼

# 1. ʲô����״�ƶ�
ǰ�����ǽ���op��ʱ���ᵽ�˲�����ע����OpRegistry�������ᵽ������ע���������һ���ṹOpRegistrationData������ṹ�г���OpDef֮�⣬��������һ��OpShapeInferenceFn�������������ʲô�õ��أ�
����֪����opֻ�Ƕ����˲�������������Ͳ���������û�ж�����������������״���ٸ����ӣ�MatMul�������������˷�����ֻ��һ������ı�ʾ��û�о���˵���������˷��������[2,3]x[3,4]=[2,4]������[100,200]x[200,300]=[100,300]��������ʵ��Ӧ���У��ڵõ�����֮ǰ���������ʵ��״���޷�Ԥ֪�ģ����ڵõ�����֮�����Ǳ����ܹ������������״���Լ���ǰop�����ã��ж�����ľ�����״�����ܸ��������Ӧ��С���ڴ�ռ䡣���ԣ�������ҪΪÿһ���������䱸һ����״�ƶϵĺ������������״�ƶϵ�������

# 2. InferenceContext
ǰ���ᵽ��OpShapeInferenceFn����������һ��������ϸ���壺
```
typedef std::function<Status(shape_inference::InferenceContext* c)> OpShapeInferenceFn;
```
�ɼ���OpShapeInferenceFn��һ������InferenceContext�����ĺ�����TFΪ����op����״�ƶϺ�����׼��������һ��ͳһ�Ľӿڡ����и���״�ƶ���ص����ݺ͹��ܺ�����������InferenceContext�������ڲ�������һ��ǰ�潲����OpKernelContext����ʵ���ǵĹ��ܺ���OpKernelContext����ΪOpKernel�ĺ���API Compute�����Ĳ��������м�����صĲ��������������������С�InferenceContextҲ��һ�������ǰ����и���״�ƶ���ص����ݺ͹��ܺ�����װ��һ��InferenceContext�����У�Ȼ���������󴫵ݸ�OpShapeInferenceFn���Ϳ���ʵ����״�ƶϡ��������ʵ�������ݲ��ֺ�ʵ���߼��Ľ��
�ھ��忴ShapeInference��֮ǰ��������Ҫ��һЩ�����ࣺ
```
class Dimension {
  private:
    //...
    const int64 value_;
};
class DimensionHandle {
  private:
    //...
    const Dimension* ptr_ = nullptr;
};
class Shape {
    //...
  private:
    const int32 rank_;
    const std::vector<DimensionHandle> dims_;
};
class ShapeHandle {
    //...
  private:
    const Shape* ptr = nullptr;
};
class DimensionOrConstant {
  public:
    //...
    DimensionHandle dim;
    int64 val;
};
class ShapeAndType {
    ShapeHandle shape;
    DataType dtype = DT_INVALID;
};
```
�⼸���඼�Ƚϼ򵥡��������õ�ʱ�ܹ��ϵþͺ��ˡ�
�������ǿ���InferenceContext����ࣺ
```
class InferenceContext {
  public:
    InferenceContext(int graph_def_version, const NodeDef* node_def, const OpDef& op_def, const std::vector<ShapeHandle>& input_shapes, const std::vector<const Tensor*>& input_tensors, const std::vector<ShapeHandle>& input_tensors_as_shapes, std::vector<std::unique_ptr<std::vector<ShapeAndType>>> input_handle_shapes_and_types);//���캯��
    Status Run(const std::function<Status(shape_inference::InferenceContext* c)>& fn);//����һ����thisΪ�����ĺ�����û���������еľ���OpShapeInferenceFn
    bool MergeInput(int idx, ShapeHandle shape);
    bool RelaxInput(int idx, ShapeHandle shape);
  private:
    ShapeManager shape_manager_;
    std::vector<ShapeHandle> inputs_;
    std::vector<const Tensor*> input_tensors_;
    std::vector<bool> requested_input_tensor_;
    std::vector<ShapeHandle> outputs_;
    std::vector<ShapeHandle> input_tensors_as_shapes_;
    std::vector<bool> requested_input_tensor_as_partial_shape_;
    std::vector<std::unique_ptr<std::vector<ShapeAndType>>> input_handle_shapes_and_types_;
    std::vector<std::unique_ptr<std::vector<ShapeAndType>>> output_handle_shapes_and_types_;
    const int graph_def_version_;
    const NodeDef& node_def_;
    NameRangeMap input_name_map_;
    NameRangeMap output_name_map_;
    Status construction_status_;
};
```
ǰ���Ѿ����ܹ������������ã�����Ϊ��������״�ƶϺ����Ĳ�����Ϊ��״�ƶ��ṩ�㹻�����ݺ͹��ܺ���֧�֣���ô�����ĳ�Ա�ͱȽ������ˣ�˽�����ݳ�ԱΪ��״�ƶ��ṩ����֧�֣�������API��Ϊ��״�ƶ��ṩ���õĹ��ܺ��������������ᵽ��MergeInput��RelaxOutput�����������ص�����������������Ĺ��ܣ�

MergeInput�����ǽ���������idx����������shape�ϲ�������ĺϲ������ǣ�
- ���ShapeHandles��һ���ģ�����shape��δ֪�ģ���ô����ά�Ȳ��䡣�����������ά����δ֪�ģ���ô�����shape��
- ���������״������֪�ģ����Ǳ���ӵ����ͬ��rank��
- ��������һ��ά�ȣ������������״�����ά�ȶ���֪����ô���Ǳ�����ȣ�
- ���һ����״������ά���ϵ���Ϣ��������һ����״����ôӵ�и�����Ϣ����״�������ء�����һ���µ���״�������������أ�����µ���״�ۺ��������������״����Ϣ��
- ���磬�ϲ�[2,?]��[?,2]���õ�[2,2]��
- ���磬[2,2]���ܱ��ϲ���[1,2]

���˵MergeInput������������״�ǡ��������ģ���ô��RelaxInput��������������״���ǡ����š��ģ�������������״��ĸ�ģ��������Ĺ����ǣ�
- ���ShapeHandles��һ���ģ���ô��Ӧ��shape���ᱻ���أ�
- �����һ��ShapeHandle��δ֪�ģ���ôһ��δ֪��ShapeHandle���ᱻ���أ�
- ���������״��rank��֪������ͬ����ôһ��δ֪ShapeHandle���ᱻ���أ�
- ������һά�ȣ������һshape��δ֪�ģ���ô��Ӧ�����ά��Ҳ��δ֪�ģ�
- ������һά�ȣ��������shape��Ӧ��ά��λ�ö�����֪�ģ���������ͬ����ô��Ӧ�����ά��Ҳ��δ֪�ģ�
- �������shape��rank�Ͷ�Ӧά�ȴ�С��һ������ô�����״���ᱻ���أ�
- ���磬[2,?]��[?,2]��õ�[?,?]��
- ���磬[2,2]��[3,2]��õ�[?,2]��
- ���磬[2,2]��[1,2,3]��õ�?

# 3. ��ϵͼ
```mermaid
graph TB
    OpShapeInferenceFn-.ʹ�ò���.->InferenceContext
    OpKernel::Compute-.ʹ�ò���.->OpKernelContext
```

# 4. �漰���ļ�
- shape_inference

# 5. ������¼
- v1.0 2018-08-29 �ĵ�����
- v2.0 2018-09-10 �ĵ��ع�

[github��ַ](https://github.com/tengkz/tensorflow_notes)