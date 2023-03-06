# Project

根据百度新闻引擎检索得到各城市的地面塌陷事故，进行统计探索分析和空间冷热点分析后所做的可视化效果

![image](https://github.com/ShirleyLoveGIS/UrbanResilence/blob/master/img/thumb.jpg)

## 总体设计

### 总体架构

后台：django + mysql

### 数据库设计

塌陷新闻events的表结构如下：

序号|字段名|标识符|类型及长度|有无空值|主键|索引序号
------ | ------ | ------ | ------ | ------| ------ | ------
1|新闻ID|ID|int|无|Y|1
2|时间|Date|date|无| |
3|省|Province|char(10)|无| |
4|市|City|char(10)|无| |
5|县|District|char(10)|有| |
6|伤亡人数|Casualty|int|无| |
7|事故原因|Reason|char(20)|无| |
8|链接|Link|date|无| |
9|地址|Adress|char(50)|有| |

其中，事故原因的分类包括：
- 建设工程 （地铁施工、管线施工……）
- 自身结构隐患
- 环境因素 （暴雨……）
- 管理缺陷
- 原因不明

如[样例新闻](https://baijiahao.baidu.com/s?id=1758430517323467726&wfr=spider&for=pc)可提取为：

字段  |  值
------ | ------
ID | 1
Date | 2023-02-21
Province | 黑龙江
City | 哈尔滨
District | 道里区
Casualty | 0
Reason | 建设工程
Linke | https://baijiahao.baidu.com/s?id=1758430517323467726&wfr=spider&for=pc
Adress | 道里区友谊路段

### 统计分析结果
对表格的统计分析则通过视图的方式，视图的结构包括：
- events_reason
- month-count
- ranking-list
- region-count

#### 事故原因 events_reason
```sql
CREATE VIEW `events_reason` AS SELECT
Reason, COUNT(*)
FROM events
GROUP BY Reason
```

序号|字段名|标识符|类型及长度|有无空值
------ | ------ | ------ | ------ | ------
1|事故原因|Reason|date|无
1|事故数量|Counts|int|无

