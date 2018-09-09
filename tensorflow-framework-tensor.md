# Ŀ¼
1. ʲô��tensor
2. tensor�̳���ϵ
3. ��Eigen3��Ĺ�ϵ
4. ʲô��tensor_reference
5. tensor_shape
6. tensor_slice
7. �����ṹ
8. ��ϵͼ
9. �漰���ļ�
10. ������¼

# 1. ʲô��tensor
TFȫ�ƽ���TensorFlow���ɼ�tensor����Ҫ�ԡ�����������һ���Ը�ά���ݵķ�װ���ṩ�˷ḻ��API�������Դ����У����ǳ�����������������ʾ���ݣ��������ѧϰӦ���У��жԸ���ά���ݵ����󡣱����ڶ�ͼ����д���ʱ����ɫͼ����ʹ�����ά����Ϣ����������ɫͨ������ͨ������Ҫ�Բ�ɫͼ�������������������������ݱ�Ϊ��ά����һЩ����������£���������Ҫ����ά�ȵ����ݡ�������ÿ�ֶ�ά���ݶ���һ�ֽṹ����Ȼ������������㡣TF�������ǣ�Ϊ��ά���ݶ���ͳһ������Tensor��

����ά���ݵĸ����е����Ϊ���ô���ܶ�Tensor�ڲ������ݽṹ�и�ֱ�۵�ӡ�������ȿ�һ��Tensor���˽�����ݳ�Ա��
```
class Tensor {
    //...
  private:
    TensorShape shape_;
    TensorBuffer* buffer_;
}
```
�������ṹ��û�м���������û��ϵ��ֻ�����ǵ�����������״�͵ײ�����ָ��ͺ��ˡ�Tensor��Ϊһ�����������࣬��Ȼ�ṩ�˺ܶ�API�����糣��Ĺ��졢��������ֵ�����ơ���ֵ���Ի�ȡ�ȡ�����֮�⣬���ṩ������Ƚ�����Ľӿڣ����Ǿ���˵����
```
class Tensor {
  public:
    //...
    //��proto���ݵ��໥ת��
    bool FromProto(const TensorProto& other);
    void AsProtoField(TensorProto* proto);
    //Ϊ�ײ����ݴ�������ͼ
    template <typename T> typename TTypes<T>::Vec vec();
    template <typename T> typename TTypes<T>::Matrix matrix();
    template <typename T> typename TTypes<T, NDIMS>::Tensor tensor();
}
```
���е�һ�ཫTensor�����л���proto֮���໥ת�����������豸֮�䴫��Tensor���ڶ�����Ϊ��ǰ��Tensor�ĵײ������ṩ����һ����ͼ�������ص���˵һ����ͼ�ĸ��

�ع�Tensor������˽�����ݣ�`TensorBuffer* buffer_`��һ��ָ��ײ����ݵ�ָ�룬�������Ľṹ�������л���ϸ˵��������ζ�ţ�Tensor��������ʵ�ʵĵײ����ݣ���ʵ����ֻ�ǶԵײ����ݵ�һ����ͼ��ͬ��һ�ݵײ����ݣ������ṩ������ͼ���������һ������Ϊ12�����飬���������������������һ��1x12����������������������󣬿�����Ϊ��3x4����2x6�ľ��������������������������Ϊ��3x2x2��������ͨ�����ַ��������ǿ��Զ�ͬһ�ݵײ����ݽ��и��ã��������ظ������ڴ�ռ䣬������Ч�ʡ�
```mermaid
graph TB
    A("Tensor A, shape=[3,4]")-->D(�ײ�����TensorBuffer)
    B("Tensor B, shape=[2,6]")-->D(�ײ�����TensorBuffer)
    C("Tensor C, shape=[3,2,2]")-->D(�ײ�����TensorBuffer)
```
˳����һ�䣬numpy�жԶ�ά�����ʵ�֣�Ҳ��ͬ����ԭ��

ϸ�ĵĶ��߿��ܷ����ˣ��ڶԵײ����ݴ�������ͼʱ��������һ����ֵ���������`typename TTypes<T>::Vec`�����漰TF�е�Tensor��Eigen3��Ĺ�ϵ�����ǽ�����������ϸ˵����

# 2. tensor�̳���ϵ
���������ǿ�һ��TensorBuffer������ʲô���Ľṹ����ֻ��һ���̳������ü����������ӿڣ��������κ�ʵ�֣�
```
class TensorBuffer : public core::RefCounted {
    //...
}
```
��˻��ɣ�TensorBufferֻ��һ���ṩ�ӿڵĻ��࣬ʵ�������õ�ֻ���������ࡣ���ǿ������ļ̳нṹ��
```
class BufferBase : public TensorBuffer {
    //...
}
class Buffer : public BufferBase {
    //...
  private:
    T* data_;
    int64 elem_;
}
```
�ṹ�Ѿ��ǳ������ˣ�BufferBase��̳���TensorBuffer�������˰���һ���ڴ������ָ���⣬���Ի����еĲ���API������ʵ�֡���Buffer����ʵ�ʿ��õģ���������ָ��ʵ�����ݵ�ָ��data_�Լ�Ԫ������elem_��

���⻹Ҫ˵��һ�㣬Buffer���������ڴ�֮�⣬���ܵ���Ŀ����Ĺ����������������ʼ��Buffer�����ݣ�TFΪ������˺ܶศ����ͺ���������Ͳ�һһ׸���ˡ�

Tensor�ļ̳���ϵͼ���£�
```mermaid
graph TB
    A(core::RefCounted)-->|����|B(TensorBuffer)
    B(TensorBuffer)-->|����|C(BufferBase)
    C(BufferBase)-->|����|D(Buffer)
    C(BufferBase)-.����.->E(Allocator* alloc_)
    D(Buffer)-.����.->F(T* data_)
    D(Buffer)-.����.->G(int64 elem_)
    H(Tensor)-.����.->B(TensorBuffer)
```

# 3. ��Eigen3��Ĺ�ϵ
�ղ��ᵽ�ˣ���ΪTensor�������ṩ��ͬ��ͼ��ʱ�򣬷�����һ����ֵ�����`TTypes<T>::Vec`����������ΪTF�е�Tensor��Eigen3���е�Tensor��������ϵ��������tensor_types.h�ļ��У��ҵ����������͵Ķ��壺
```
struct TTypes {
    typedef Eigen::TensorMap<Eigen::Tensor<T,NDIMS,Eigen::RowMajor,IndexType>,Eigen::Aligned> Tensor;
    typedef Eigen::TensorMap<Eigen::Tensor<T,1,Eigen::RowMajor,IndexType>,Eigen::Aligned> Vec;
    //...
}
```
ԭ������Eigen3����Tensor��ʹ��������������ֶ��屻������TTypes�ṹ���У����Բ������ⲿTF�Զ����Tensor��ɳ�ͻ��

���»ص�Tensor�Ķ��壬���Ƿ��֣�ԭ���ڶ�Tensor�ײ������ṩ������ͼ��ʱ�򣬷��ص��Ѿ�����Tensor�ṹ������TTypes::TensorMap�����Ƿ���ζ�ţ�TF�ж����Tensorֻ�Ƕ�Eigen::Tensor��һ�ַ�װ�أ�����׷����Դ���ҵ�vec������ʵ�֣�
```
template <typename T>
typename TTypes<T>::Vec vec() {
    return tensor<T,1>();
}

template <typename T, size_t NDIMS>
typename TTypes<T, NDIMS>::Tensor Tensor::tensor() {
    CheckTypeAndIsAligned(DataTypeToEnum<T>::v());
    return typename TTypes<T, NDIMS>::Tensor(base<T>(), shape().AsEigenDSizes<NDIMS>());
}
```
������Ԥ�����ȫһ�����ڶ�vec�����ĵ����У�������tensor��������������������ã����ǽ�TF�ж����Tensorת��ΪTTypes::Tensor�������߾���Eigen::TensorMap��Ҳ����˵��tensor���صı�������һ��Eigen::TensorMap�����⣬����֪��base()��shape()�����������ֱ𷵻���TensorBufferָ���TensorShape�����ʵ���Ͼ���ʹ��TF��Tensor�洢�����ݣ���Ϊ��Eigen::TensorMap�Ĺ��캯���Ĳ�����

����˵��TF�е�Tensorʵ�����Ƕ�Eigen::TensorMap��һ�ָ߼���װ�������Ǽ򵥵���˽�����ݳ�Ա�������ߣ����ǰ����˹����������Ҫ�����ݣ�����Ҫ���ߵ�ʱ�򣬹��첢���ء����ַ�ʽ��ʹ��TF�е�Tensor��������Eigen��Ч���������㷽����Ҳ��ΪTensor����һЩAPI��

# 4. ʲô��tensor_reference
Tensor��Ķ�����˰���ָ��ײ����ݵ�ָ���⣬�������˶�������״�����͵�������ͨ��TensorShape����������ǲ���������Щ��ֱ��ʹ��Tensor�����ӹ��������ƶ��ĸ��������TF�Ƴ���tensor_reference����࣬����������һ��ָ��TensorBuffer��ָ�룬����ÿ����һ��TensorReference���󣬾ͻ�����һ����Եײ�TensorBuffer�����ü�����������TensorReference��˵������Ψһ�����ľ���������֮��Unref�������������ڴ�й©��
```
class TensorReference {
  public:
    //...
  private:
    TensorBuffer* buf_;
}
```

# 5. tensor_shape
TensorShape��Ȼ��������������״��ص���Ϣ������ʵ������ˣ����������˶������������͵�������TensorShape��صĺ�����̳���ϵ���£�
```mermaid
graph LR
    I(TensorShapeRep)-->|����|J(TensorShapeBase)
    J(TensorShapeBase)-->|����|K(TensorShape)
    J(TensorShapeBase)-->|����|L(PartialTensorShape)
```
��������һ�£���ײ��TensorShapeRep��˽�����ݳ�Ա��
```
class TensorShapeRep {
    //...
  private:
    union {
        uint8 buf[16];
        Rep64* unused_aligner;//����ǿ��u_��ָ������⣬û���κ�����
    } u_;
    int64 num_elements_;
}
```
buf������������˼������ǰ12��Ԫ�������洢��״����ȻTensor�����֧�ֵ�256ά������������õĲ�����3ά��Ϊ��Ч�ʣ�TF�ṩ������������12���ֽڵķ�ʽ�����£�
```
struct Rep16 {
    uint16 dims_[6];//���ɱ�ʾ6ά��������ÿһά�ĳ��Ȳ�����2^16-1
};
struct Rep32 {
    uint32 dims_[3];//���ɱ�ʾ3ά��������ÿһά�ĳ��Ȳ�����2^32-1
};
struct Rep64 {
    gtl::InlinedVector<int64, 4>* dims_;//֧������ά�ȵ�����
};
```
ʣ�µ�4���ֽ�Ҳ�����˷ѣ��ڵ�14-16���ֽ��У��ֱ�洢�������е��������ͱ�š�������ά����Ŀ������ά�ȵı�ʾ���ͣ�Rep16, Rep32, Rep64������������ά�ȵ���Ŀ����һ���ֽڴ洢�ģ�������֧��256ά����ϧ����Ŀǰ��û�з��ֵ�13���ֽڵ����ã��з��ֵĶ��߻�ӭ��֪�ҡ�

TensorShapeBase�ಢû����Ӷ�������ݳ�Ա����ֻ�������һЩ���������޸�����ά�ȵ�API�ӿڡ���TensorShape��Ҳֻ�������һЩ����״���м��ͱȽϵĽӿڣ�û���������ݳ�Ա��

�����������PartialTensorShape�࣬�ڹ���һ����������״ʱ���������ĳЩά�����ǻ���֪�������ά��ֵ�����԰����ά����Ϊδ֪����˾ͻ��õ�PartialTensorShape�࣬�������Ҳ������һЩδ֪ά�Ȳ�����API������Ͳ������ˡ�

# 6. tensor_slice
TensorSlice���ʾһ���������������������ݽṹ�ǳ��򵥣�
```
class TensorSlice {
    //...
  private:
    gtl::InlinedVector<int64,4> starts_;
    gtl::InlinedVector<int64,4> lengths_;
}
```
�ֱ���ÿһ��ά�������Ŀ�ʼλ�ú��������ȣ��ɴ�����Ҳ֪����TF��Tensorֻ֧��������������֧�ּ��������
����TensorSlice��;�㷺��������г�ʼ���ķ���Ҳ���ֶ�����������
- ����������
- �ӵ���ά�ȴ�����������ȫ����ʱ��
- ��һ�����������鴴��
- ��һ��TensorSliceProto����
- ��һ���ַ��������д���

# 7. �����ṹ
Ϊ�˷������������֮��ص����ݽṹ�������л���TF����˺ܶ�protos�����������Լ򵥣���ֻ˵�������ǵ���;������Ȥ�Ķ��߿���ȥ��Դ���롣
```
message TensorDescription;//�����������������������͡���״���ڴ������Ϣ
message TensorProto;//�������������ͣ��汾��ԭʼ���ݵ�
message VariantTensorDataProto;//��DT_VARIANT���͵����л���ʾ
message TensorShapeProto;//������״
message TensorSliceProto;//��������
```

# 8. ��ϵͼ
```mermaid
graph TB
    A(core::RefCounted)-->|����|B(TensorBuffer)
    B(TensorBuffer)-->|����|C(BufferBase)
    C(BufferBase)-->|����|D(Buffer)
    C(BufferBase)-.����.->E(Allocator* alloc_)
    D(Buffer)-.����.->F(T* data_)
    D(Buffer)-.����.->G(int64 elem_)
    H(Tensor)-.����.->B(TensorBuffer)
    I(TensorShapeRep)-->|����|J(TensorShapeBase)
    J(TensorShapeBase)-->|����|K(TensorShape)
    J(TensorShapeBase)-->|����|L(PartialTensorShape)
    H(Tensor)-.�����ṹ.->M(TensorSlice)
    H(Tensor)-.��״��������������.->K(TensorShape)
    H(Tensor)-.ת��Ϊ.->N(TTypes::Tensor)
    N(TTypes::Tensor)-.ת��Ϊ.->O(Eigen::TensorMap)
```

# 9. �漰���ļ�
- tensor
- tensor_reference
- tensor_types
- tensor_shape
- tensor_slice
- tensor_description

# 10. ������¼
- v1.0 2018-08-26 �ĵ�����
- v2.0 2018-09-09 �ĵ��ع�

[github��ַ](https://github.com/tengkz/tensorflow_notes)