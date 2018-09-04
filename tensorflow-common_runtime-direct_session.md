# Ŀ¼
1. ���ĸ���
2. direct_session
    1. direct_session.h
    2. direct_session.cc 

# 1. ���ĸ���
����֮ǰ���µĶ���Ӧ�û��ǵã�session��һ��ִ�д������ǰѼ���ͼ�����뽻��session������������ִ������ִ�м�����������TF�������ṩ��һ����򵥵�ִ����direction_session�����յ�ǰ����⣬���Ǿ���direction_session��ʵ��Ӧ���Ƿǳ��򵥶�ֱ�ӵģ��Ͼ�ִ�����ĸ��ӽṹ������executor��ƪ�Ѿ������ˡ���ʵ���ϣ�������ѵ����ڣ���ʱ������ֻ��ϣ���Լ���ͼ��ĳЩ�ڵ�Ϊ���룬ĳЩ�ڵ�Ϊ�������ִ��ͼ�е�һС���ּ��㣬������Ҫִ������ͼ������һ�����棬���ֶ�ͼ����ִ�е�������ͬһ��ͼ�Ͽ���ͬʱ���ڶ����Ϊ��Ӧ�����������direct_session���������˺ܶศ�����ݡ�

# 2. direct_session
## 2.1 direct_session.h
DirectSession���ṩ�˷ḻ�����ݺͽӿڣ�����Ϊ�˱���࣬������ȥ�˲��ֺ������βΣ�
```
class DirectSession : public Session {
  public:
    DirectionSession(const SessionOptions& options, const Device* device_mgr, DirectSessionFactory* factory);
    
    Status Create(const GraphDef& graph) override;
    Status Extend(const GraphDef& graph) override;
    Status Run(...) override;//����ͼ
    
    Status PRunSetup(...);//��������ͼ׼��
    Status PRun(...);//��������ͼ
    
    Status Reset(const std::vector<string>& containers);//���device_mgr�е�containers�����containers������ǿյģ���ô���Ĭ������
    
    Status ListDevice(...) override;
    Status Close() overrides;
    Status LocalDeviceManager(const DeviceMgr** output) overrides;
    
    void ExportCostModels(...);

  private:
    Status MaybeInitializeExecutionState(...);//����graph֮�����ִ����״̬û�г�ʼ�������ʼ��������ִ����״̬
    
    Status GetOrCreateExecutors(...);//����һ�������������������һ��������ִ���������м������Ƿ���ں��ʵ�ִ���������û�У�����һ��
    
    Status CreateGraphs(...);//����graph_def_���豸���Լ������������������ͼ����Щ�´�����ͼ����һ�������ĺ�����flib_def
    
    Status ExtendLocked(const GraphDef& graph);//Extend���ڲ�ִ����
    
    Status ResourceHandleToInputTensor(...);
    
    Status SendPRunInputs(...);//������������ṩ��ִ����������������ִ��
    
    Status RecvPRunOutputs(...);//��ִ�����л�ȡ��������������ȴ�ֱ����������������
    
    Status CheckFetch(...);//������������ܷ���ݸ���������������
    
    Status WaitForNotification(...);
    Status CheckNotClosed();
    
    const SessionOptions options_;
    
    //�豸��صĽṹ
    const std::unique_ptr<const DeviceMgr> device_mgr_;
    std::vector<Device*> devices_;
    DeviceSet device_set_;
    
    string session_handle_;
    bool graph_created_ GUARDED_BY(graph_def_lock_) = false;
    mutex graph_def_lock_;
    GraphDef graph_def_ GUARDED_BY(graph_def_lock_);
    
    std::vector<std::pair<thread::ThreadPool*, bool>> thread_pools_;//������ִ��op���̳߳أ���һ������ֵ����־���Ƿ�ӵ������̳߳�
    
    Status init_error_;
    
    bool sync_on_finish_ = true;//���Ϊ�棬�����߳�ֱ���豸�Ѿ������ĳ�������ڵ����ж����еĲ���
    void SchedClosure(thread::ThreadPool* pool, std::function<void()> c);//���̳߳��е���c
    
    mutex executor_lock_;//����ִ����
    
    std::unordered_map<string, std::shared_ptr<ExecutorsAndkeys>> executor_ GUARDED_BY(executor_lock_);//��ǩ��ӳ�䵽����ִ������ǩ�������˲���ִ��ͼ������������������������Ψһȷ��һ������ִ��ͼ
    
    std::unordered_map<string, std::shared_ptr<RunState>> partial_runs_ GUARDED_BY(executor_lock_);//��ǩ��������ִ��״̬��ÿһ������ִ�ж�����һ��ר�ű�����״̬�Ľṹ
    
    SessionState session_state_;//���������е�ǰ�ڻỰ�����ڴ�������
    
    DirectSessionFactory* const factory_;
    CancellationManager* cancellation_manager_;
    
    std::unordered_map<string, string> stateful_placements_ GUARDED_BY(graph_def_lock_);//������״̬�Ľڵ㣨����params��queue��������ڵ����Ƶ��ڵ������豸��ӳ�䣬һ����Щ�ڵ㱻��������ĳ���豸�ϣ��ǲ��������ƶ���
    
    std::unique_ptr<SimpleGraphExecutionState> execution_state_ GUARDED_BY(graph_def_lock_);//��������ͼʱʹ��
    
    std::unique_ptr<FunctionLibraryDefinition> flib_def_;//���κε���д���Ż�֮ǰ�ĺ����⣬�ر��ǣ�CreateGraphs�������޸ĺ�����
    
    mutex closed_lock_;
    bool closed_ GUARDED_BY(closed_lock_) = false;//����Ự�Ѿ����رգ���Ϊtrue
    
    //Ϊ����Ự����Ψһ������
    std::atomic<int64> edge_name_counter_ = {0};
    std::atomic<int64> handle_name_counter_ = {0};
    
    static std::atomic_int_fast64_t step_id_counter_;//Ϊ���еĻỰ����Ψһ��step id
    
    const int64 operation_timeout_in_ms_ = 0;//ȫ�ֶ����������ĳ�ʱ��ֵ
    
    CostModelManager cost_model_manager_;//Ϊ��ǰ�Ự��ִ�е�ͼ�������е���ʧģ��
}
```
�ɼ���DirectSession����ĺܶ����ݶ���Ϊ����ִ��׼���ġ����ڼ���ͼ����һ������Ĺ滮�����ǿ���ͨ��Ϊͬһ��ͼѡȡ��ͬ��������������ִ�в�ͬ�ļ��㡣����ͬ�ļ�����Ҫ��ͬ��ִ������Ҳ��Ҫ��ͬ�Ĵ洢�ṹ�������������ĵ�ǰ״̬��Ϊ�ˣ�TFר�Ÿ����˼����ṹ�壬������������һ�¶Բ�ͬ����ִ�����ķ�װ��
```
//Ϊÿһ��partition׼����ִ�����ͺ�������ʱ��
struct PerPartionExecutorAndLib {
    Graph* graph = nullptr;
    std::unique_ptr<FunctionLibraryRuntime> flib;
    std::unique_ptr<Executor> executor;
};

//Ϊÿһ�μ����ṩ�����ݽṹ
struct ExecutorsAndKeys {
    std::atomic_int_fast64_t step_count;
    std::unique_ptr<Graph> graph;
    NameNodeMap name_to_node;
    std::unique_ptr<FunctionLibraryDefinition> flib_def;
    std::vector<PerPartitionExecutorsAndLib> items;
    std::unordered_map<string, size_t> input_name_to_index;
    std::unordered_map<string, string> input_name_to_rendezvous_key;
    std::unordered_map<string, size_t> output_name_to_index;
    std::unordered_map<string, string> output_name_to_rendezvous_key;
    
    DataTypeVector input_types;
    DataTypeVector output_types;
};
```
����һ�ż���ͼ��˵�����ǵ�ÿһ�μ����ִ�У�����������ͼ�ļ��㻹�ǲ���ͼ�ļ��㣬���п����ǿ��豸�ģ���˶���Ҫ�����ڵ���ã���ͼ�Ľڵ�ָ��ͬ���豸�ϣ�ÿһ���豸�Ϸ�����һ��ͼ��partition��ÿ��partition�ж�Ӧ������ʱ�������ִ������������ÿһ�ּ�����˵��������Ҫһ��vector�Ѳ�ͬpartition����Ϣ�洢������
���⣬�ղ��ᵽ���ǻ���ҪΪÿһ�μ����ṩ���浱ǰ״̬�Ľṹ�����������һ�£�
```
//����ÿһ��partition�ڵ�ִ�У��Ự������һ��RunState
struct RunState {
    mutex mu_;
    Status status GUARDED_BY(mu_);
    IntraProcessRendezvous* rendez = nullptr;
    std::unique_ptr<StepStatsCollector> collector;
    Notification executors_done;
    std::unordered_map<string, bool> pending_inputs;//����Ѿ��ṩ�����룬��Ϊtrue
    std::unordered_map<string, bool> pending_outputs;//����Ѿ�������������Ϊtrue
    TensorStore tensor_store;
    ScopedStepContainer step-container;
    //...
};

struct RunStateArgs {
    RunStateArgs(const DebugOption& options) : debug_options(options) {}
    bool is_partial_run = false;
    string handle;
    std::unique_ptr<Graph> graph;
    const DebugOptions& debug_options;
};
```
���У�RunStateΪÿһ��partition��ִ���ṩ��״̬����Ĺ��ܣ���RunStateArgs��Ϊǰ���ṩ�����ڵ��ԵĲ��������á�

## 2.2 direct_session.cc
��Դ�ļ��������DirectSessionFactory�Ķ��壬���ṩ�˶���DirectSession�������ɺ͹���Ĺ��ܣ���Ҫժ¼���£�
```
class DirectSessionFactory : public SessionFactory {
  public:
    Session* NewSession(const SessionOptions& options) override;
    Status Reset(...) override;
    void Deregister(const DirectSession* session);
  private:
    mutex session_lock_;
    std::vector<DirectSession*> session_ GUARDED_BY(sessions_lock_);//���ڴ洢���ɵ�DirectSession
};
```
���⣬���ṩ��һ������ֱ�ӹ���ע����ࣺ
```
class DirectSessionRegistrar {
  public:
    DirectSessionRegistrar() {
        SessionFactory::Register("DIRECT_SESSION", new DirectSessionFactory());
    }
};
static DirectSessionRegistrar registrar;
```
���棬���ǻᰴ��˳���DirectSession����Ҫ�ĺ��������в�⣬���ڲ��ֺ���ϸ�ڱȽ϶࣬���˺��Ĵ���֮�⣬���ǽ��������ܽ��ͣ�
```
DirectSession::DirectSession(const SessionOptions& options, const DeviceMgr* device_mgr, DirectSessionFactory* const factory){
    //����options׼���̳߳�
    //����device_mgr׼��device_��device_set_��ÿ���豸��op_segment()
}

Status DirectSession::Run(...){
    //��ȡ���ڵ�ǰ�Ự�ı������е����������
    //���������������������Ƿ��Ѿ������ֳɵ�ִ����
    //����һ������֡��call frame��������Ự��ִ����֮�䴫����������
    //����һ������ʱ״̬�Ľṹ��RunState��
    //��ʼ����ִ�У����Ĵ�������
    for(const auto& item : executors_and_keys->items){
        item.executor->RunAsync(args, barrier->Get());
    }
    //��ȡ���
    //���汾������������ϣ��������������
    //������������ʧģ�ͣ�cost model��
    //���RunOptions����������ã�����ָ���ͼ
}

Status DirectSession::GetOrCreateExecutors(...){
    //���ٲ���·��
    //������·��������������������ʹ����ͬ�����������ϻ�õ���ͬ��ǩ��
    //���δ�ҵ����򴴽����ִ����������
    //����ִ��ͼ�����Ĵ�������
    CreateGraphs(options, &graphs, &ek->flib_def, run_state_args, &ek->input_types, &ek->output_types));
    //Ϊ����ͼ׼������ʱ����
}

Status DirectSession::CreateGraphs(...){
    //ǰ��Ԥ����
    //ͼ�ָ��㷨�����Ĵ�������
    Partition(popts, &client_graph->graph, &partitions);
    //���ָ�������Ч��
    //ͼ�Ż����������Ĵ�������
    OptimizationPassRegistry::Global()->RunGrouping(OptimizationPassRegistry::POST_PARTITIONING, optimization_options);
    //�����豸��д��ӵ�е���ͼ
}
```
�ɼ��������ִ�й�������Run�����ڲ�������executor->RunAsync������ʵ�ֵģ��ھ���ִ��֮ǰ�����ǻ���Ҫͨ��GetOrCreateExecutors�������ִ����������������ڲ�������ͨ��CreateGraphs������ԭͼ�����˷ָ������ͼ�Ż������㷨��ͼ�������Ż���