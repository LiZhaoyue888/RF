# 什么是分段扫描，如何引入分段扫描，引入后对现有系统有何影响，具体要做哪些工作？

Segment Sweep：分段配置频点，合并下发配置，整体进行扫描
现有配置方式：一次配置，重复扫描
```C++
VNA_ConfigConditionDT(ch,freqArray[],freqCnt,ifbwArray[],bandpowerArray[])
VNA_ConfigPoints(ch,freqCnt)
```
分段扫描:不均等频点合并扫描，并且覆写Channel频点数组  
考虑影响:出来的曲线不均等

目前Segment类只包含:state/start/stop/points/ifbw/power  
问：段扫描硬件支持，每个频点可以不一样，频点可以不连续  
问：算法支持分段校准吗？支持！不用分段校准  
考虑分段扫描是否开启均匀显示(频点平铺)？可以，并且很有必要！
1. 校准不需要动
3. 显示不需要动
4. 扫描不用动

## 要动哪些？

1. 加扫描方式选择
2. 加段Segment类
3. 加入Segment编辑表格
4. 加入Segment编辑对话框

## 什么是Trigger

trigger就是触发。不同厂家的触发方案不一样，当前设备的触发是：全局Channel监听同一个触发。  
引入触发功能的影响点：UI和measure流程,Trigger的修改从UI界面发出，经过事件循环到达measure线程，然后调用函数进行测量。  
相应的，引入Trigger类型是必要的。   
``c++
  enum class 
  {
    SOFT,
    IMM,
    EXT_Raise,
    EXT_Falling,
    EXT_DualEdge
  }mTriggerType;
} 
``
有几点需要注意  
- 除非切换Trigger，否则不能手动放弃Trigger，不提供手动切换Trigger的按钮
- 每次测量所有Channel之前配置一下Trigger，但是要先TriggerTerminal放弃上次听Trigger
- 每次测量之前都要先配置一下Trigger?不用吧！测量只管测量，除非Trigger方式是continue
- 
