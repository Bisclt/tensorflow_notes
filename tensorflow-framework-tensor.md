## Ŀ¼
1. ���ĸ���
2. tensor
3. tensor_reference
4. tensor_shape
5. tensor_slice
5. protos

## 1. ���ĸ���
TF�ĺ������ݽṹTensor��ʾһ��������������eigen3�⣬���ṩ�˷ḻ��API��Ϊ�˷������������ĵײ����ݣ������TensorInference�ࡣTensorShape���ڱ�ʾ��������״���������͵���Ϣ��TensorSlice���ڱ�ʾ������������

## 2. tensor
TFȫ�ƽ���TensorFlow���ɼ�tensor����Ҫ�ԡ�TF�е�tensor����eigen3�⣬�ǶԶ�ά���ݵ�һ����װ��Tensor����������ݳ�Ա�ǳ��򵥣�
```
class Tensor {
    //...
  private:
    TensorShape shape_;
    TensorBuffer* buffer_;
}
```
����˼�壬һ������������״��һ����ָ��ײ����ݵ�ָ�롣Tensor��Ϊһ���������ݽṹ����Ȼ�ṩ�˺ܶ�API�ӿڣ����糣��Ĺ��졢��������ֵ�����ơ���ֵ���Ի�ȡ�ȡ�����֮�⣬���ṩ������Ƚ�����Ľӿڣ����Ǿ���˵����
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
���е�һ�ཫTensor�����л���proto֮���໥ת�������豸���໥����Tensorʱ����Ҫ�Ƚ������л����ڶ�����Ϊ��ǰ��Tensor�ĵײ������ṩ����һ����ͼ�������ص���˵һ����ͼ�ĸ��
�ع�Tensor������˽�����ݣ�TensorBuffer* buffer_��һ��ָ��ײ����ݵ�ָ�룬�������Ľṹ�������л���ϸ˵����Ҳ����˵��Tensor��������ʵ�ʵĵײ����ݣ���ʵ����ֻ�ǶԵײ����ݵ�һ����ͼ��ͬ��һ�ݵײ����ݣ������ṩ������ͼ���������һ������Ϊ12�����飬���������������������һ��1x12����������������������󣬿�����Ϊ��3x4����2x6�ľ��������������������������Ϊ��3x2x2��������ͨ�����ַ��������ǿ��Զ�ͬһ�ݵײ����ݽ��и��ã��������ظ������ڴ�ռ䣬������Ч�ʡ�numpy�жԶ�ά�����ʵ�֣�Ҳ��ͬ���ĵ���
���������ǿ�һ��TensorBuffer������ʲô���Ľṹ���ҵ����Ķ��壬������ֻ��һ���̳������ü����������ӿڣ��������κ�ʵ�֣�
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

## 3. tensor_reference
Tensor��Ķ�����˰���ָ��ײ����ݵ�ָ���⣬�������˶�������״�����͵�������������ǲ���������Щ��ֱ��ʹ��Tensor�����ӹ��������ƶ��ĸ��������TF�Ƴ���tensor_reference����࣬����������һ��ָ��TensorBuffer��ָ�룬����ÿ����һ��TensorReference���󣬾ͻ�����һ����Եײ�TensorBuffer�����ü�����������TensorReference��˵������Ψһ�����ľ���������֮��Unref�������������ڴ�й©��

## 4. tensor_shape
TensorShape��صĺ�����̳���ϵ���£�
```
graph LR
TensorShape-->TensorShapeBase
TensorShapeBase-->TensorShapeRep
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
TensorShapeBase�ಢû����Ӷ�������ݳ�Ա����ֻ�������һЩ���������޸�����ά�ȵ�API�ӿڡ�
�����������PartialTensorShape�࣬�ڹ���һ����������״ʱ���������ĳЩά�����ǻ���֪�������ά��ֵ�����԰����ά����Ϊδ֪����˾ͻ��õ�PartialTensorShape�࣬�������Ҳ������һЩδ֪ά�Ȳ�����API������Ͳ������ˡ�

## 5. tensor_slice
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

## 6. protos
Ϊ�˷������������֮��ص����ݽṹ�������л���TF����˺ܶ�protos�����������Լ򵥣���ֻ˵�������ǵ���;������Ȥ�Ķ��߿���ȥ��Դ���롣
```
message TensorDescription;//�����������������������͡���״���ڴ������Ϣ
message TensorProto;//�������������ͣ��汾��ԭʼ���ݵ�
message VariantTensorDataProto;//��DT_VARIANT���͵����л���ʾ
message TensorShapeProto;//������״
message TensorSliceProto;//��������
```