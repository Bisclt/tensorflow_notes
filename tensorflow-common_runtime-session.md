## Ŀ¼
1. ���ĸ���
2. session
3. session_factory

## 1. ���ĸ���
session������Ϊ��һ��ִ�д��������ڿͻ��˹�������ͼ���ṩ���룬Ȼ��Ѽ���ͼ����sessionȥִ�С���ˣ�sessionӦ�þ߱�һ����ִ�й��ܡ�����TF���ṩ��session�Ĺ����࣬session_factory�����ڲ���session��

## 2. session
sessionû���ṩͷ�ļ�������ֱ����session.cc�ļ����ṩ��ʵ�֣�������ȥ��ʵ�֣����£�
```
Status Session::Run(const RunOptions& run_options, const std::vector<std::pair<string, Tensor>>& inputs, const std::vector<string>& output_tensor_names, const std::vector<string>& target_node_names, std::vector<Tensor>* outputs, RunMetadata* run_metadata);
Status Session::PRunSetup(const std::vector<string>& input_names, const std::vector<string>& output_names, const std::vector<string>& target_nodes, string* handle);
Status Session::PRun(const string& handle, const std::vector<std::pair<string, Tensor>>& inputs, const std::vector<string>& output_names, std::vector<Tensor>* outputs);
Session* NewSession(const SessionOptions& options);
Status NewSession(const SessionOptions& options, Session** out_session);
Status Reset(const SessionOptions& options, const std::vector<string>& containers);
```
���Կ�����Session�����ṩRun�ӿ�֮�⣬���ṩ�˲���ִ�еĽӿڣ�PRunSetup��PRun��������������ָ�����ǲ�������������ͼ�����Ǹ�����ͼ�е�ĳ�����ڵ���Ϊ���룬ĳ�����ڵ���Ϊ��������в��ֵ�ͼ�����⣬Session�����Ը����ṩ��SessionOptions�����µĻỰ��

## 3. session_factory
SessionFactory�ṩ�˹�����Ĺ��ܣ���API���£�
```
class SessionFactory {
  public:
    virtual Session* NewSession(const SessionOptions& options) = 0;
    virtual bool AcceptsOptions(const SessionOptions& options) = 0;
    virtual Status Reset(const SessionOptions& options, const std::vector<string>& containers);
    static void Register(const string& runtime_type, SessionFactory* factory);
    static Status GetFactory(const SessionOptions& options, SessionFactory** out_factory);
}
```
- �������е�Reset������������������ֹ�͹ر��������е�session���Ͽ����е���Դ�����ǵ����ӡ�Reset�����ܹ�ʹ��Щ���й����������г���ĻỰ��ֹ�͹رգ������ͷ���֮��ص���Դ��Reset����ȴ��ɻỰ��ļ����������������һ����������ֹ�Ľ��̡�Ȼ���������Reset֮��������һ���µĻỰ����ô����µĻỰ��������ԾɻỰ�Ĳ������뿪��
- ����ɻỰ��ĳЩ��Դû�б�����containers����ô�ɻỰ��Ȼ������ĳЩ�ط�Ӱ������Ự�ļ��㣬��������Ӱ�����Ԥ�⣬��ˣ�Ϊ�˰�ȫ������Reset��containers�����а�������ȫ��������
- ���containers�����ǿյģ���ôĬ�ϵ�container���ᱻʹ�ã����containers�Ƿǿյģ���ôĬ��������Ҫ����ʽ�ķ��롣
- ֧����Դ�����ĻỰ����Ҫ��д���������