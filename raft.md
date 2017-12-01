(raft展示)[http://thesecretlivesofdata.com/raft/]
* ### (raft详解)[https://zhuanlan.zhihu.com/p/27207160] 为了避免这种致命错误，需要对协议进行一个微调：

> 只允许主节点提交包含当前term的日志
就是说当前term之前的log是不允许被commit的， 那么以前term的log如何被commit呢？见下
针对上述情况就是：即使日志（2，2）已经被大多数节点（S1、S2、S3）确认了，但是它不能被Commit，因为它是来自之前term(2)的日志，直到S1在当前term（4）产生的日志（4， 4）被大多数Follower确认，S1方可Commit（4，4）这条日志，当然，根据Raft定义，**（4，4）之前的所有日志也会被Commit, 此时之前term的log也会被commit, 并且也不会再丢**。此时即使S1再下线，重新选主时S5不可能成为Leader，因为它没有包含大多数节点已经拥有的日志（4，4）。