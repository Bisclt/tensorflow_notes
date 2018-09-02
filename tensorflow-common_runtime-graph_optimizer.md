## Ŀ¼
1. ���ĸ���
2. graph_optimizer
3. function
4. optimization_registry

## 1. ���ĸ���
��ƪ��Ҫ��ͼ���Ż��������������ڹ���ԭʼͼ��ʱ��רע�ڴﵽĿ�ģ�������ȥ����ͼ��ִ��Ч�ʡ������ͼ����ƹ��̱���Ϊ�߼����Եı�д����ôͼ���Ż����̾��൱�ڣ����߼����Ա���Ϊ�������ԵĹ����У�Ϊ���ܹ����ٽ��еı����Ż������磬����ͬ�ĳ����۵�����Identity�ڵ�ȥ���ȵȡ�������Ҫ�������ۣ���ͼ�Ż���ص���ͺ�����

## 2. graph_optimizer
����ͼ�Ż�����Ҫ��һ��ͳһ����ڣ�����������ͼ�����Լ�ͼִ�еĻ������Լ��Ż������ã�������Ż����ͼ�������ھ���GraphOptimizer�����������������Ľṹ�ͽӿڣ�
```
class GraphOptimizer {
  public:
    GraphOptimizer(const OptimizerOptions& opts);
    void Optimize(FunctionLibraryRuntime* runtime, Env* env, Device* device, std::unique_ptr<Graph>* graph, const std::unordered_map<const Node*, std::vector<PartialTensorShape>>* shape_map);
  private:
    OptimizerOptions opts_;
};
```
��Ȼ�����е�Optimize�������������Ҫ��API������ͼ�Ż�����opts�е��Ż�����Ӧ�õ�graph�ϡ����ܻὫgraph�滻Ϊ����һ��ͼ����device������ͼ��Ҫ���е��豸����ʹ���Ż��㷨���Կ�������豸Ӧ�����ǵ��Ż�ѡ�shape_map����ǷǿյĻ�������ͼ�нڵ������ӳ��Ϊ���ֿ�֪�Ľڵ������״��������ĳЩͼ�Ż��лᱻӦ�ã����糣���۵��Ż���
����ͼ�Ż���������Ҫ�˽�ĸ�Ϊϸ��һЩ�����ԣ��ȿ�һ�������Ĺ��캯�������ʵ�ַ�ʽ��
```
GraphOptimizer::GraphOptimizer(const OptimizerOptions& opts) : opts_(opts) {
    if(opts_.opt_level()>=OptimizerOptions::L1){
        opts_.set_do_common_subexpression_elimination(true);
        opts_.set_do_constant_folding(true);
    }
}
```
ͨ��������������˽⵽���Ż��������м������ģ���������ڵ���1ʱ��ĳЩĬ�ϵ��Ż�������Ҫ�����������硰���������������͡������۵�������Щ���������ھ�����Ż�������Ҳ�ῴ�������������һ�º���API��Optimize�����ݣ�
```
void GraphOptimizer::Optimize(FunctionLibraryRuntime* runtime, Env* env, Device* device, std::unique_ptr<Graph>* graph, const std::unordered_map<const Node*, std::vector<PartialTensorShape>>* shape_map){
    Graph* g = graph->get();
    DumpGraph("Initial",g);//������ǰͼ�Ľṹ
    
    bool changed = true;
    const int kMaxRounds = 10;
    for(int rounds = 0; rounds < kMaxRounds; ++rounds){
        changed = false;
        if(RemoveListArrayConverter(g)){
            DumpGraph("RemoveListArrayConverter", g);
            changed = true;
        }
        if(opts_.do_function_inlining() && RemoveDeadNodes(g)){
            DumpGraph("RemoveDeadNodes", g);
            changed = true;
        }
        if(opts_.do_function_inlining() && RemoveIdentityNodes(g)){
            DumpGraph("RemoveIdentityNodes", g);
            changed = true;
        }
        if(opts_.do_constant_folding()){
            ConstantFoldingOptions cf_opts;
            cf_opts.shape_map = shape_map;
            bool was_mutated;
            ConstantFold(cf_opts, runtime, env, device, g, &was_mutated).IgnoreError();
            if(was_mutated){
                RemoveDeadNodes(g);
                DumpGraph("ConstFolding",g);
                changed = true;
            }
        }
        if(opts_.do_function_inlining() && FixupSourceAndSinkEdges(g)){
            DumpGraph("FixupSourceAndSinkEdges",g);
            changed = true;
        }
        if(opts_.do_common_subexpression_elimination() && OptimizeCSE(g,nullptr)){
            DumpGraph("ExpandInlineFunctions",g);
            changed = true;
        }
        if(!changed) break;
    }
    
    //����flib_def��Զ������ʧ��������ǿ��Է��ĵ�ʹ������������ͼ
    std::unique_ptr<Graph> copy(new Graph(g->flib_def()));
    CopyGraph(*g, copy.get());
    graph->swap(copy);
    
    DumpGraph("ReCopy", graph->get());
}
```
�ڶ�ͼ�����Ż�ʱ�����ǲ�����һ����͵ģ���Ϊ�Ż�֮����໥Ӱ�죬�������Ƕ�ͼ������A�Ż�������A�Ż���˵����ʱͼ�Ѿ������ŵ��ˣ���֮�������ֶ�ͼ������B�Ż�����ʱ����B�Ż���˵��ͼ�Ѿ������ŵ��ˣ�������A�Ż���˵��δ�ء����ͼ�Ż���һ��ѭ�������Ĺ��̣�TF��������ߵ��Ż���10�飬���ڴ����ͼ��˵��Ҳ���㹻�ˡ�
��ͼ�Ż��Ĺ����У����Ƿ����˺ܶ�֮ǰû�����ĺ�������Щ�����Ķ��嶼��function.h�ļ��У�Ϊ�˼������ͼ�Ż����̵���⣬���������˽�������ļ��еĺ�����

## 3. function
function.h�ļ��У�û���ඨ�壬ȫ������Ӳ�����ĺ������壬�ɻ�������
```
//kernel������������FunctionLibraryRuntime��NodeDef������kernel
typedef std::function<Status(FunctionLibraryRuntime*, const NodeDef&, std::unique_ptr<OpKernel>*)> CustomKernelCreator;
void RegisterDefaultCustomKernelCreator(CusteomKernelCreator cb);//kernel��������ע����

//����һ��FunctionLibraryRuntime������ʵ����lib_def�еĺ���������device�����У����custom_kernel_creator�Ƿǿյģ����ᱻ���ص�runtime��������kernel
std::unique_ptr<FunctionLibraryRuntime> NewFunctionLibraryRuntime(const DeviceMgr* device_mgr, Env* env, Device* device, int graph_def_version, const FunctionLibraryDefinition* lib_def, const OptimizerOptions& optimizer_options, CusteomKernelCreator custom_kernel_creator);

//��֮ǰ�ĺ������ƣ�ֻ�������ص�runtimeֱ������RegisterDefaultCustomKernelCreatorע���ȫ��custom_kernel_creator�������µ�kernel
std::unique_ptr<FunctionLibraryRuntime> NewFunctionLibraryRuntime(const DeviceMgr* device_mgr, Env* env, Device* device, int graph_def_version, const FunctionLibraryDefinition* lib_def, const OptimizerOptions& optimizer_options);

//�����������
struct FunctionBody {
    FunctionDef fdef;
    Graph* graph = nullptr;
    DataTypeVector arg_types;
    DataTypeVector ret_types;
    gtl::InlinedVector<Node*, 4> arg_nodes;
    gtl::InlinedVector<Node*, 4> ret_nodes;
    
    FuntionBody(){}
    FunctionBody(const FunctionDef& f, DataTypeSlice arg_types, DataTypeSlice ret_types, Graph* g);
    ~FunctionBody();
};

//ɾ�����½ڵ㣬��һ����״̬�ģ��ڶ����޲����ģ�������������޹��׵�
bool RemoveDeadNodes(Graph* g);

//Ѱ�����µ�ģʽ��src-(in)->node-(out)->dst�����node��identity�ڵ㣬in��Ψһ���������ݱߣ�out��Ψһ��������ݱߣ���ʹ��src->dst��д����ģʽ
bool RemoveIdentityNodes(Graph* g);

//��ͼ�е�_ListToArray��_ArrayToListת��ΪIdentity�ڵ�
bool RemoveListArrayConverter(Graph* g);

//����ͼ�е�ÿ���ڵ㣬���libָ������ڵ���һ���������ã���ô������������塣�������һ���ڵ㱻�����ˣ�����true��
bool ExpandInlineFunctions(FunctionLibraryRuntime* lib, Graph* graph);

//��graph�е����ݵ�������־�ļ��������־�����㹻�ߵĻ�
void DumpGraph(StringPiece label, const Graph* g);

//Ӧ��ͼ��д���Ż����������������ڵ��Ƴ���
void OptimizeGraph(FunctionLibraryRuntime* lib, std::unique_ptr<Graph>* g);

//��һ��������ͼת��ΪGraphDef
void ToGraphDef(const Graph* g, GraphDef* gdef, bool pretty = false);

//����һ����ֵ�������������ĵ�������
FunctionBody* SymbolicGradient(const FunctionBody& f);

//��һ��FunctionDefʾ����Ϊһ��graph������fbodyָ��ӵ��FunctionDef��FunctionBody
Status FunctionDefToBodyHelper(const FunctionDef& fdef, const AttrSlice& attrs, const FunctionLibraryDefinition* const lib_def, const std::function<Status(const string&, const OpDef**)>& get_func_sig, FunctionBody** fbody);
```
���ڻع�ͷ����GraphOptimizer���е�Optimize��������������Array��List�໥ת���ڵ��ΪIdentity�ڵ㣬Ȼ��ɾ�������ڵ㣬ɾ��Identity�ڵ㣬���г����۵����޸���������ߣ����й���������������������˶�ͼ���Ż���

## 4. optimization_registry
optimization_registry.h�ļ��У�������һЩά��һ��ȫ�ֵ�ͼ�Ż�����ע��������Ҫ���࣬�ڻỰ��ʼ��һ��ͼʱ����ʹ�����ȫ���Ż�����ע��������ͼ�����Ż���
��������������һ���࣬GraphOptimizationPassOptions������˼�壬��������ͼ�Ż���������Ҫ�Ĳ�������Щ�㹻��Ϊһ���ֵ�ļ�ֵ������ͨ����ʹ��һ���ֵ������ָ���ͼ�Ż���������״̬��
```
struct GraphOptimizationPassOptions {
    string session_handle;
    const SessionOptions* session_options = nullptr;
    const CostModel* cost_model = nullptr;
    FunctionLibraryDefinition* flib_def = nullptr;
    const DeviceSet* device_set = nullptr;
    //����Ż�������ͼ�ָ�֮ǰ��ʹ�ã���ô���Ż��Ķ���������graph�������ͼ�ָ�֮��ʹ�ã���ô���graph��null
    std::unique_ptr<Graph>* graph = nullptr;
    //����ͼ�ָ����Ż�����ʱʹ��
    std::unordered_map<string, std::unique_ptr<Graph>* partition_graphs = nullptr;
};
```
ͼ�Ż�������������ͼ�ָ�֮ǰ����֮����У����Է�Ϊ���࣬������ʹ����GraphOptimizationPassOptions����һ���ӿڡ�
��������GraphOptimizationPass�࣬���е�ͼ�Ż������࣬�������������࣬���ĽṹҲ�ǳ��򵥡�
```
class GraphOptimizationPass {
  public:
    virtual ~GraphOptimizationPass() {}
    virtual Status Run(const GraphOptimizationPassOption& options) = 0;
};
```
������ӵ���˶���ͼ�Ż��������㷨֮����Ҫ����Щ����ͳһ�������TF�����һ�ֶ�ͼ�Ż������㷨����ͳһע��͹�����ࣺ
```
//����ļ�ֵΪphase��ͼ�Ż������㷨�ǰ���phase������˳��ִ�еģ���һ��phase�ڲ���ִ��˳����δ�����
typedef std::map<int, std::vector<std::unique_ptr<GraphOptimizationPass>>> GraphOptimizationPasses;

class OptimizationPassRegistry {
  public:
    enum Grouping {
        PRE_PLACEMENT,//��cost model��ֵ֮���ڽڵ�����㷨֮ǰ
        POST_PLACEMENT,//�ڽڵ�����㷨֮��
        POST_REWRITE_FOR_EXEC,//������feed/fetch�ڵ������д֮��
        POST_PARTITIONING,//��ͼ�ָ�֮��
    };
    void Register(Grouping grouping, int phase, std::unique_ptr<GraphOptimizationPass> pass);//ע��ͼ�Ż������㷨
    Status RunGrouping(Grouping grouping, const GraphOptimizationPassOptions& options);//����һ��groupping�����е�ͼ�Ż������㷨������phase����������
    static OptimizationPassRegistry* Global();//����һ��ȫ�ֵ�ͼ�Ż�����ע����
  private:
    std::map<Grouping, GraphOptimizationPasses> groups_;
};
```
�ܽ�һ�£�groups��һ��˫���ӳ�䣬�ȴ�Groupingӳ�䵽ͼ�Ż������㷨�飬����㷨�鱾��Ҳ�Ǹ�ӳ�䣬��phaseӳ�䵽������ͼ�Ż������㷨�����£�
```
graph LR
Grouping-->GraphOptimizationPasses
phase-->GraphOptimizationPass
```
���TFΪ�ղŵ�ע�����ṩ��һ��ȫ�ֵ���ڣ�
```
class OptimizationPassRegistration {
  public:
    OptimizationPassRegistration(OptimizationPassRegistry::Grouping grouping, int phase, std::unique_ptr<GraphOptimizationPass> pass){
        OptimizationPassRegistry::Global->Register(grouping,phase,std::move(pass));
    }
};
```