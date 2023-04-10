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
- month-countAll
- month-countLy
- month-countAvgm
- month-countAvgy
- ranking-list
- region-count

#### 事故原因 events_reason
```sql
CREATE VIEW `events_reason` AS SELECT
Reason as 事故原因, 
COUNT(*) as 事故数量
FROM original_events
GROUP BY Reason
```

序号|字段名|标识符|类型及长度|有无空值
------ | ------ | ------ | ------ | ------
1|事故原因|Reason|date|无
2|事故数量|Counts|int|无

#### 月份历史总量统计 month-countAll
```sql
CREATE VIEW `month-countAll` AS 
select 
DATE_FORMAT(NewsT,'%m') as 月份,
count(*) as 历史总量
FROM original_events 
GROUP BY DATE_FORMAT(NewsT,'%m') 
ORDER BY DATE_FORMAT(NewsT,'%m')
```

序号|字段名|标识符|类型及长度|有无空值
------ | ------ | ------ | ------ | ------
1|月份|month|int|无
2|历史总量|AllCounts|int|无


#### 月份去年同期数量统计 month-countLy
```sql
CREATE VIEW `month-countLy` AS 
select 
DATE_FORMAT(NewsT,'%m') as 月份,
count(*) as 去年同期数量
FROM original_events 
GROUP BY DATE_FORMAT(NewsT,'%m') 
ORDER BY DATE_FORMAT(NewsT,'%m')
```

序号|字段名|标识符|类型及长度|有无空值
------ | ------ | ------ | ------ | ------
1|月份|month|int|无
2|数量|LyCounts|int|无


#### 月份平均数量统计 month-countAvgm
```sql
CREATE VIEW `month-countAvgm` AS 
SELECT 
DATE_FORMAT(NewsT,'%m') as 月份,
count(*)
/
(SELECT count(DATE_FORMAT(NewsT,'%Y'))
FROM original_events) 
as 平均数量
FROM original_events 
GROUP BY DATE_FORMAT(NewsT,'%m')
ORDER BY DATE_FORMAT(NewsT,'%m')
```

序号|字段名|标识符|类型及长度|有无空值
------ | ------ | ------ | ------ | ------
1|月份|month|int|无
2|平均数量|AvgmCounts|float(3)|无


#### 平均全年总数统计 month-countAvgy
```sql
CREATE VIEW `month-countAvgy` AS 
SELECT
count(*)
/
(
SELECT 
count(distinct DATE_FORMAT(NewsT,'%Y'))
FROM original_events
)
as 平均全年总数
FROM original_events
```

序号|字段名|标识符|类型及长度|有无空值
------ | ------ | ------ | ------ | ------
1|平均全年总数|Avgycounts|float(3)|无


#### 风险点排名 ranking-list
```sql
CREATE VIEW `ranking-list` AS 
SELECT
District as 区域,
count(*) as 数量
FROM original_events
GROUP BY District
```

序号|字段名|标识符|类型及长度|有无空值
------ | ------ | ------ | ------ | ------
1|区域|district|char(10)|无
2|数量|DistrictCounts|int|无


#### 地区风险点分布 region-count
```sql
CREATE VIEW `region-count` AS 
SELECT
Province as 省份,
count(*) as 数量
FROM original_events
GROUP BY Province
```

序号|字段名|标识符|类型及长度|有无空值
------ | ------ | ------ | ------ | ------
1|省份|province|char(10)|无
2|数量|RegionCounts|int|无

