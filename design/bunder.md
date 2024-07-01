# Bundler核心组件

* BundlerServer：捆绑服务，提供相关UserOperation和Bundler相关rpc服务，如eth_sendUserOperation、eth_estimateUserOperationGas、eth_getUserOperationReceipt、debug_bundler_dumpMempool等；

* ReputationManager：地址声誉管理器，包括白名单，黑名单管理，及地址限流；地址有三种状态（OK, THROTTLED：触发限流, BANNED：黑名单）

* MempoolManager：UserOperation内存池管理器；
1. 当前有UserOperation添加到内存池时，更新用户声誉opsSeen计数器；

* DepositManager：paymaster质押管理，确保质押金，足够支付内存中待支付的gas总和；
* BundleManager：捆绑UserOperation，提交（handleOps）到EntryPoint；
* ExecutionManager：UserOperation执行器，提供UserOperation的发送服务；
1. sendUserOperation：校验UserOperation，添加到内存池，并尝试捆绑；

* EventsManager：处理账户创建、链上执行UserOperation，聚合签名起变更事件SignatureAggregatorForUserOperations；
1. 如果监听到链上执行UserOperation，则从内存池管理器MempoolManager中移除UserOperation；并更新用户声誉opsIncluded计数器；
2. 监听到创建用户，则添加地址到声誉管理器：

## 核心流程
1. 启动捆绑服务：初始化MempoolManager，BundleManager， ExecutionManager（开启executionManager自动捆绑UserOperation），ReputationManager（黑白名单），
开启EventsManager监听链上事件服务，启动捆绑服务BundlerServer；
2. 用户通过RPC提交UserOperation，BundlerServer委托给ExecutionManager， ExecutionManager会验证UserOperation，没问题，提交到内存池MempoolManager（同时更新用户声誉），
并尝试捆绑UserOperation；
3. 内存池中的UserOperation数量超过maxMempoolSize，则委托BundleManager， 提交（handleOps）到EntryPoint，触发UserOperation链上执行事件，如果监听到事件，则更新用户声誉；