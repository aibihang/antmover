# antmover
C端分布式存储系统
                                                       一种轻量级C端储存系统
1 概述
  
    本文意在设计一种数据存储在C端的服务系统，服务端提供轻量级的映射，协助点对点的数据通讯
   
2 设计
    2.1 数据存储
        按块存储，每一个块都有块id，全局唯一，块与实际存储地址的映射关系，data_map<blockid,urllist>;
    2.2 服务端
        提供一份完整性数据，及每一份数据N个副本,副本存储在C端,服务器提供写，同步至N个副本C端节点，确保数据不少于N个副本，保存并负责data_map<blockid,urllist> 映射关系更新;
    2.3 客户端
        存储block_data<blockid,data>，local_data_map<blockid,url>与服务端保持关联联系，定时更新位置信息;
    2.4 通信
        2.4.1 客户端请求数据
           C端确定blockid，先访问本地local_data_map，找不到blockid或者url无效，请求S端端提供url（服务端根据最新的）并告知访问S端无效的url，S端根据最近相邻原则返回url并处理无效的url，C端            获取数据并更新本地local_data_map;
        2.4.2 服务端更新数据
           S端发现C端上报的无效url或者失去心跳的url,重新选择一个近期活跃C端，并拷贝1个副本数据到C端，更新data_map;
        2.4.3 服务端新增数据
           C端新增数据，根据分布式全局算法生成一个blockid，发送S端请求确认，S端新增data_map<blockid,urllist>，C端本地存储block_data<blockid,data>,S端挑选N个副本活跃C端节点，将数据分            发到C端存储;
        2.4.4 服务端删除数据
           C端发起删除数据请求，发送S端请求确认，S端删除data_map<blockid,urllist>，C端本地存储block_data<blockid,data>,S端通知N个副本活跃C端节点执行删除操作;
    2.5 数据完整性保障
        2.5.1 副本节点选取算法
           挑选节点必须是活跃节点，确保数据有效存储，考虑到集群数据负载上线，动态调整单个节点数据保存上线，及时移除死亡节点，支持死亡节点恢复，N个节点必须均匀分布在整个节点系统中，确保平均通信成            本最低；
        2.5.2 副本存储算法
           同一个块数据至少存储在N个不同副本节点，数据首先按照block拆分，生成block，block大小合理估计4MB
        2.5.3 blockid生成算法
           全局唯一，逻辑时间钟生成，blockid={serviceid}_{clientid},clientid={clientdeviceid}_{unixtimestamp}_{incrid},serviceid={unixtimestamp}_{serverdeviceid}
3 实现

