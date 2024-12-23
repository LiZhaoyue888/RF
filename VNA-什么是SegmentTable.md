Segment Sweep：分段配置,整体扫描
现有配置方式：一次配置，重复扫描
VNA_ConfigConditionDT(ch,freqArray[],freqCnt,ifbwArray[],bandpowerArray[])
VNA_ConfigPoints(ch,freqCnt)
分段扫描：不均等频点合并扫描，并且覆写Channel频点数组
考虑影响：出来的曲线不均等
目前Segment类只包含：state/start/stop/points/ifbw/power
问：段扫描硬件支持，每个频点可以不一样，频点可以不连续
问：算法支持分段校准吗？支持！不用分段校准。
考虑分段扫描是否开启均匀显示(频点平铺)？可以，并且很有必要！
1、校准不需要动
2、显示不需要动
3、扫描不用动
## 要动哪些？
1、加扫描方式选择
2、加段Segment类
3、加入Segment编辑表格
4、加入Segment编辑对话框
