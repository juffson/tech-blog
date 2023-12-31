## 基于 clickhouse 的 历史 K 线存储方案

### 美股正股数据（股票、基金、债券等）

#### 特点

美股相比于 A 股，标的数量大每日正股数量 13000 左右，行情的分类较细：全美市场、Basic 行情、OTC 行情、粉股等

#### 数据写入方式

在数据写入上，行情相关的数据有两种方式

- 实时更新
  实时更新：即分钟 K 产生后，即写入到数据库中进行持久化，但是考虑到实际的使用场景：
  - 当日分钟 K 访问频繁
  - 实时数据可能存在缺失导致计算错误，写入后需要进行数据清洗才能保证数据的质量
- 批量更新
  在数据清洗完成后，批量写入到数据库中进行持久化。参考前面对 clickhouse 和其他产品的比较和业务数据的使用场景，批量写入的方式非常合适。

#### 数据量估算

#### 上线后情况

2. 一天的市场数据量(分钟 K)

- 美股正股

  | 名称      | 引擎      | 估计行数 | 数据大小 | 索引大小 | 注释 |
  | --------- | --------- | -------- | -------- | -------- | ---- |
  | nus_kline | MergeTree | 5783374  | 64.78 MB | 0 B      |
  | otc_kline | MergeTree | 419725   | 2.77 MB  | 0 B      |
  | us_kline  | MergeTree | 5783980  | 60.67 MB | 0 B      |

### 美股期权

相比于股票市场，期权的特点是

- 标的数据巨大（每一个股票会对应多个期权）
- 存在明确的过期日，某些业务场景下，需要对过期的期权进行屏蔽。
