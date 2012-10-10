1 S2E�еĲ��(���ߣ��ܶ������䣺caoding8483@163.com)
=================================================

1.1           S2E�в����ͨ�Ż���
----------------------------------------

��S2E���캯���У������initPlugins()����ʼ���������������Ҫ���������������Ҫ�Ƕ�ȡ�����ļ��е�plugins�������õĲ�������Ҽ����Щ����Ƿ����������������initPlugins()������ʵ�����£�
   
void S2E::initPlugins()    
{    
        m_pluginsFactory = new PluginsFactory();
        
        //CorePlugin�ز�����
        m_corePlugin = dynamic_cast<CorePlugin*>(
        m_pluginsFactory->createPlugin(this, "CorePlugin"));
        assert(m_corePlugin);
        
        m_activePluginsList.push_back(m_corePlugin);
        m_activePluginsMap.insert(
        make_pair(m_corePlugin->getPluginInfo()->name, m_corePlugin));
        if(!m_corePlugin->getPluginInfo()->functionName.empty())
        m_activePluginsMap.insert(
        make_pair(m_corePlugin->getPluginInfo()->functionName, m_corePlugin));
        
        //��ȡ�����ļ��е�plugins��
        vector<string> pluginNames = getConfig()->getStringList("plugins");
        
        //��ÿ��������м���װ��
        /* Check and load plugins */
        foreach(const string& pluginName, pluginNames) {
                const PluginInfo* pluginInfo = m_pluginsFactory->getPluginInfo(pluginName);
                if(!pluginInfo) {
                        std::cerr << "ERROR: plugin '" << pluginName
                        << "' does not exists in this S2E installation" << std::endl;
                        exit(1);
                } else if(getPlugin(pluginInfo->name)) {
                        std::cerr << "ERROR: plugin '" << pluginInfo->name
                        << "' was already loaded "
                        << "(is it enabled multiple times ?)" << std::endl;
                        exit(1);
                } else if(!pluginInfo->functionName.empty() &&
                getPlugin(pluginInfo->functionName)) {
                        std::cerr << "ERROR: plugin '" << pluginInfo->name
                        << "' with function '" << pluginInfo->functionName
                        << "' can not be loaded because" << std::endl
                        <<  "    this function is already provided by '"
                        << getPlugin(pluginInfo->functionName)->getPluginInfo()->name
                        << "' plugin" << std::endl;
                        exit(1);
                } else {
                        Plugin* plugin = m_pluginsFactory->createPlugin(this, pluginName);
                        assert(plugin);
                        
                        m_activePluginsList.push_back(plugin);
                        m_activePluginsMap.insert(
                        make_pair(plugin->getPluginInfo()->name, plugin));
                        if(!plugin->getPluginInfo()->functionName.empty())
                        m_activePluginsMap.insert(
                        make_pair(plugin->getPluginInfo()->functionName, plugin));
                }
        }
        
        //������Ƿ�������
        /* Check dependencies */
        foreach(Plugin* p, m_activePluginsList) {
                foreach(const string& name, p->getPluginInfo()->dependencies) {
                        if(!getPlugin(name)) {
                                std::cerr << "ERROR: plugin '" << p->getPluginInfo()->name
                                << "' depends on plugin '" << name
                                << "' which is not enabled in config" << std::endl;
                                exit(1);
                        }
                }
        }
        
        //��ÿ��������г�ʼ��������ĳ�ʼ������initialize()���麯��
        /* Initialize plugins */
        foreach(Plugin* p, m_activePluginsList) {
                p->initialize();
        }
}

����ȱ�ٵ���CorePlugin������������в���Ļ��������ж�����һϵ���źţ������ڼ�⵽ָ��ʱ������Ƿ����źţ�emit������Ӧ�Ĳ����������ӣ�connect�����ҵ�����Ӧ�ĺ�����CorePlugin�ж�����ź���CorePlugin.h�У�

/** Signal that is emitted on begining and end of code generation
for each QEMU translation block.
��qemu��ÿһ��tb����Ϊtcg��ʱ�򣬶��ᷢ������ź�
*/
sigc::signal<void, ExecutionSignal*,    
S2EExecutionState*,   
TranslationBlock*,   
uint64_t /* block PC */>   
onTranslateBlockStart;   

/** Signal that is emitted upon end of translation block    
��һ��tb������ʱ�򣨷�����תָ���ʱ�򣬷�������ź�   
*/   
sigc::signal<void, ExecutionSignal*,    
S2EExecutionState*,   
TranslationBlock*,   
uint64_t /* ending instruction pc */,   
bool /* static target is valid */,   
uint64_t /* static target pc */>    
onTranslateBlockEnd;      


/** Signal that is emitted on code generation for each instruction */   
sigc::signal<void, ExecutionSignal*,   
S2EExecutionState*,   
TranslationBlock*,      
uint64_t /* instruction PC */>   
onTranslateInstructionStart, onTranslateInstructionEnd;   

/**   
*  Triggered *after* each instruction is translated to notify   
*  plugins of which registers are used by the instruction.   
*  Each bit of the mask corresponds to one of the registers of   
*  the architecture (e.g., R_EAX, R_ECX, etc).         
*/      
sigc::signal<void,   
ExecutionSignal*,   
S2EExecutionState* /* current state */,   
TranslationBlock*,   
uint64_t /* program counter of the instruction */,   
uint64_t /* registers read by the instruction */,   
uint64_t /* registers written by the instruction */,   
bool /* instruction accesses memory */>   
onTranslateRegisterAccessEnd;   
   
/** Signal that is emitted on code generation for each jump instruction */   
sigc::signal<void, ExecutionSignal*,   
S2EExecutionState*,   
TranslationBlock*,   
uint64_t /* instruction PC */,   
int /* jump_type */>   
onTranslateJumpStart;   
   
/** Signal that is emitted upon exception */   
sigc::signal<void, S2EExecutionState*,    
unsigned /* Exception Index */,   
uint64_t /* pc */>   
onException;   
   
/** Signal that is emitted when custom opcode is detected    
������s2e_opʱ����������ź�   
*/   
sigc::signal<void, S2EExecutionState*,    
uint64_t  /* arg */   
>   
onCustomInstruction;   
   
/** Signal that is emitted on each memory access */   
/* XXX: this signal is still not emmited for code */   
sigc::signal<void, S2EExecutionState*,   
klee::ref<klee::Expr> /* virtualAddress */,   
klee::ref<klee::Expr> /* hostAddress */,   
klee::ref<klee::Expr> /* value */,   
bool /* isWrite */, bool /* isIO */>   
onDataMemoryAccess;   
   
/** Signal that is emitted on each port access */   
sigc::signal<void, S2EExecutionState*,   
klee::ref<klee::Expr> /* port */,   
klee::ref<klee::Expr> /* value */,   
bool /* isWrite */>   
onPortAccess;   
   
sigc::signal<void> onTimer;   
   
/** Signal emitted when the state is forked */   
sigc::signal<void, S2EExecutionState* /* originalState */,   
const std::vector<S2EExecutionState*>& /* newStates */,   
const std::vector<klee::ref<klee::Expr> >& /* newConditions */>   
onStateFork;   
   
sigc::signal<void,   
S2EExecutionState*, /* currentState */   
S2EExecutionState*> /* nextState */   
onStateSwitch;   
   
/** Signal emitted when spawning a new S2E process */   
sigc::signal<void, bool /* prefork */,   
bool /* ischild */,   
unsigned /* parentProcId */> onProcessFork;   
   
/**   
* Signal emitted when a new S2E process was spawned and all   
* parent states were removed from the child and child states   
* removed from the parent.   
*/   
sigc::signal<void, bool /* isChild */> onProcessForkComplete;   
   
   
/** Signal that is emitted upon TLB miss */   
sigc::signal<void, S2EExecutionState*, uint64_t, bool> onTlbMiss;   
   
/** Signal that is emitted upon page fault */   
sigc::signal<void, S2EExecutionState*, uint64_t, bool> onPageFault;   
   
/** Signal emitted when QEMU is ready to accept registration of new devices */   
sigc::signal<void> onDeviceRegistration;   
   
/** Signal emitted when QEMU is ready to activate registered devices */   
sigc::signal<void, struct PCIBus*> onDeviceActivation;   
   
/**   
* The current execution privilege level was changed (e.g., kernel-mode=>user-mode)   
* previous and current are privilege levels. The meaning of the value may   
* depend on the architecture.   
�ڴ����жϻ���ִ�з���ָ��ʱ����Ȩ�������仯ʱ��������ź�   
*/   
sigc::signal<void,   
S2EExecutionState* /* current state */,   
unsigned /* previous level */,   
unsigned /* current level */>   
onPrivilegeChange;   
   
/**   
* The current page directory was changed.   
* This may occur, e.g., when the OS swaps address spaces.   
* The addresses correspond to physical addresses.   
��ҳĿ¼�����仯ʱ����������ź�   
*/   
sigc::signal<void,   
S2EExecutionState* /* current state */,   
uint64_t /* previous page directory base */,   
uint64_t /* current page directory base */>   
onPageDirectoryChange;   
   
/**   
* S2E completed initialization and is about to enter   
* the main execution loop for the first time.   
*/   
sigc::signal<void,   
S2EExecutionState* /* current state */>   
onInitializationComplete;   
����guest binaryת��Ϊtcg irʱ������s2e_op��0x13f��ʱ��������´���   
case 0x13f: /* s2e_op */   
{   
        #ifdef CONFIG_S2E   
        uint64_t arg = ldq_code(s->pc);   
        s2e_tcg_emit_custom_instruction(g_s2e, arg);   
        #else   
        /* Simply skip the S2E opcodes when building vanilla qemu */   
        ldq_code(s->pc);   
        #endif   
        s->pc+=8;   
        break;   
           
}   
���е�����s2e_tcg_emit_custom_instruction(g_s2e, arg)��    
void s2e_tcg_emit_custom_instruction(S2E*, uint64_t arg)   
{   
        TCGv_ptr t0 = tcg_temp_new_i64();   
        tcg_gen_movi_i64(t0, arg);   
           
        TCGArg args[1];   
        args[0] = GET_TCGV_I64(t0);   
        tcg_gen_helperN((void*) s2e_tcg_custom_instruction_handler,   
        0, 2, TCG_CALL_DUMMY_ARG, 1, args);   
           
        tcg_temp_free_i64(t0);   
}   
   
void s2e_tcg_custom_instruction_handler(uint64_t arg)   
{   
        assert(!g_s2e->getCorePlugin()->onCustomInstruction.empty());   
           
        try {   
                g_s2e->getCorePlugin()->onCustomInstruction.emit(g_s2e_state, arg);   
        } catch(s2e::CpuExitException&) {   
                s2e_longjmp(env->jmp_env, 1);   
        }   
}   
   
����CorePlugin�ᷢ��onCustomInstruction�źţ������ÿ�����������Ӧ�ĳ�ʼ�������н���connect�����������Ӧ�Ĺ��ܡ�   
   
�����ÿһ���������initialize()�������г�ʼ���������initialize()��һ���麯������ÿһ������ж����Լ��ĳ�ʼ����������BaseInstruction�У���ʼ���������¶��壺   
   
void BaseInstructions::initialize()   
{   
        s2e()->getCorePlugin()->onCustomInstruction.connect(   
        sigc::mem_fun(*this, &BaseInstructions::onCustomInstruction));   
           
}   
   
�����BaseInstruction���յ�Guest��������Ϣʱ���ͻ����onCustomInstruction����ÿһ�����������Ӧ�Ĵ���