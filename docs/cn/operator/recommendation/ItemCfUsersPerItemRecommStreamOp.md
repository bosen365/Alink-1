# ItemCF 为实时item推荐user list

## 功能介绍
用ItemCF模型 为实时item推荐user list。


## 参数说明

| 名称 | 中文名称 | 描述 | 类型 | 是否必须？ | 默认值 |
| --- | --- | --- | --- | --- | --- |
| itemCol | Item列列名 | Item列列名 | String | ✓ |  |
| k | 推荐TOP数量 | 推荐TOP数量 | Integer |  | 10 |
| excludeKnown | 排除已知的关联 | 推荐结果中是否排除训练数据中已知的关联 | Boolean |  | false |
| recommCol | 推荐结果列名 | 推荐结果列名 | String | ✓ |  |
| reservedCols | 算法保留列名 | 算法保留列 | String[] |  | null |
| numThreads | 组件多线程线程个数 | 组件多线程线程个数 | Integer |  | 1 |

## 脚本示例
### 脚本代码

```python
from pyalink.alink import *
import pandas as pd
import numpy as np

data = np.array([
    [1, 1, 0.6],
    [2, 2, 0.8],
    [2, 3, 0.6],
    [4, 1, 0.6],
    [4, 2, 0.3],
    [4, 3, 0.4],
])

df_data = pd.DataFrame({
    "user": data[:, 0],
    "item": data[:, 1],
    "rating": data[:, 2],
})
df_data["user"] = df_data["user"].astype('int')
df_data["item"] = df_data["item"].astype('int')

schema = 'user bigint, item bigint, rating double'
data = dataframeToOperator(df_data, schemaStr=schema, op_type='batch')
sdata = dataframeToOperator(df_data, schemaStr=schema, op_type='stream')

model = ItemCfTrainBatchOp()\
    .setUserCol("user")\
    .setItemCol("item")\
    .setRateCol("rating").linkFrom(data);

predictor = ItemCfUsersPerItemRecommStreamOp(model)\
    .setItemCol("item")\
    .setReservedCols(["item"])\
    .setK(1)\
    .setRecommCol("prediction_result");

predictor.linkFrom(sdata).print()
StreamOperator.execute()
```

### 脚本运行结果
```
        item	prediction_result
0	1	{"object":"[2]","score":"[0.21698238771519462]"}
1	2	{"object":"[2]","score":"[0.29215236292253793]"}
2	3	{"object":"[2]","score":"[0.38953648389671724]"}
3	1	{"object":"[2]","score":"[0.21698238771519462]"}
4	2	{"object":"[2]","score":"[0.29215236292253793]"}
5	3	{"object":"[2]","score":"[0.38953648389671724]"}
```
