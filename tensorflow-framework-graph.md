# Ŀ¼
1. ʲô��graph
2. ͼ������������
3. graph_transfer_info
4. ��ϵͼ
5. �漰���ļ�
6. ������¼

# 1. ʲô��graph
graph��TF������Ƶ����壬�����TF�����ִ�к�Java����ִ����ȣ����൱��Java���ֽ��롣����graph��ִ�й��̣�����������򵥽���һ�¡���graph������ɣ���������һЩ���Ż�֮�󣬻��ͼ���зָʵ���Ͼ���ִ��һ���ڵ����Ĺ��̣�Ȼ���ڸ��豸�Ϸֱ����ͼ��������ǰ���Ż��������ø��豸��ִ��������������������ͼ����node�½����ǽ�����node�������Դ�ͼ�ṹ�ģ�һ��node�ļ��Ͼ��ܸ�ԭһ��������graph�����ԣ�graph������ӵ��������ݲ����ࡣ�������ǿ���GraphDef�Ķ��壺
```
message GraphDef {
    repeated NodeDef node = 1;
    VersionDef versions = 4;
    int32 version = 3 [deprecated = true];
    FunctionDefLibrary library = 2;
};
```
���Կ���������ṹ����������һ�����������֮�⣬û�����������ˡ�

# 2. ͼ������������
TFΪ�˷��㹹��GraphDef���ṩ�˺ܶศ���������������������£�
```
//����һ���ɶ��ԺõĹ���GraphDef�������������Ƿ���һ���ɶ��Բ��proto�ı�
string SummarizeGraphDef(const GraphDef& graph_def);

//У��һ��GraphDef
Status ValidateExternalGraphDefSyntax(const GraphDef& graph_def);

//�ӽڵ�����node_offset��ʼ��ΪGraphDef�еĽڵ����Ĭ�ϲ���ֵ���ڵ��Ӧ��op��Ĭ�ϲ���ֵ��op_registry��
Status AddDefaultAttrsToGraphDef(GraphDef* graph_def, const OpRegistryInterface& op_registry, int node_offset);

//��GraphDef�г�ȥ��Щ����producer_op_registry�г��ֹ�������consumer_op_registry��δ���ֹ���Ĭ�ϲ���ֵ
Status RemoveNewDefaultAttrsFromGraphDef(GraphDef* graph_def, const OpRegistryInterface& consumer_op_registry, const OpRegistryInterface& producer_op_registry, std::set<std::pair<string,string>>* op_attr_removed);

//�ռ�ͼʹ�õ�op�����ַ������ϵ���ʽ����
void OpsUsedByGraph(const GraphDef& graph_def, std::set<string>* ops_used_in_graph);

//��graph_def�г��ֹ���ͬʱҲ��op_registry�г��ֹ���op����stripped_op_list
Status StrippedOpListForGraph(const GraphDef& graph_def, const OpRegistryInterface& op_registry, OpList* stripped_op_list);
```
���Ƿ��֣������кܶ��Ǹ�ͼ��op��Ĭ�ϲ���ֵ��صĺ�������Щ�������ֵı����������ģ�������һ���ֲ�ʽ�Ļ����£�master��Ҫworkderִ��һ����ͼ�������ͼ��master�����ģ�ͼ�в�����Ĭ��ֵʹ�õ���master���ڻ���������ʱ�����У�OpRegistry��ע��Ĳ�����������Ĭ��ֵ�����ؼ��ǣ�workder���ڻ���ʹ�õ�����ʱ��������master���ܲ�һ�������磬master������ʱ��TF��������������workderȴû�С�������֮��master��������ʱ��op����������֮ǰû��Ĭ��ֵ�����ڼ�����Ĭ��ֵ������֮ǰ��Ĭ��ֵ�ĳ������ڵ�Ĭ��ֵ����ʱ��Ϊ����������ͼ������ǰ�������ԣ���Ϊ�������ܹ���workder���������У���Ҫ��������ͼ���д���ɾ������master����ʱ��OpRegistry�г��ֵ�op����Ĭ��ֵ�����Ǿͳ�������������������

# 3. graph_transfer_info
Ϊ�������Ƕ���ļ���ͼ�ܹ��������豸������DSP�������У���Ҫ��ͼ�ṹ����һЩת����ĿǰTF��֧��ת����HEXAGON�����[HEXAGON��SDK](https://developer.qualcomm.com/software/hexagon-dsp-sdk/tools)��
```
message GraphTransferInfo {
    enum Destination {
        NOP = 0;
        HEXAGON = 1;
    }
    message NodeInput {
        int32 node_id = 1;
        int32 output_port = 2;
    }
    message NodeInfo {
        string name = 1;
        int32 node_id = 2;
        string type_name = 3;
        int32 soc_op_id = 4;
        int32 padding_id = 5;
        int32 input_count = 6;
        int32 output_count = 7;
    };
    message ConstNodeInfo {
        string name = 1;
        int32 node_id = 2;
        repeated int64 shape = 3;
        bytes data = 4;
        DataType dtype = 5;
    };
    message NodeInputInfo {
        int32 node_id = 1;
        repeated NodeInput node_input = 2;
    };
    message NodeOutputInfo {
        int32 node_id = 1;
        repeated int32 max_byte_size = 2;
    };
    message GraphInputNodeInfo {
        string name = 1;
        repeated int64 shape = 2;
        DataType dtype = 3;
    };
    message GraphOutputNodeInfo {
        string name = 1;
        repeated int64 shape = 2;
        DataType dtype = 3;
    };
    repeated NodeInfo node_info = 1;
    repeated ConstNodeInfo const_node_info = 2;
    repeated NodeInputInfo node_input_info = 3;
    repeated NodeOutputInfo node_output_info = 4;
    repeated GraphInputNodeInfo graph_input_node_info = 5;
    repeated GraphOutputNodeInfo graph_output_node_info = 6;
    Destination destination = 7;
};
```

# 4. ��ϵͼ

# 5. �漰���ļ�
- graph
- graph_def_util
- graph_transfer_info

# 6. ������¼
- v1.0 2018-08-28 �ĵ�����
- v2.0 2018-09-09 �ĵ��ع�

[github��ַ](https://github.com/tengkz/tensorflow_notes)