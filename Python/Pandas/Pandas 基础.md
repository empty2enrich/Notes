# Pandas 基础

* 读取数据
* 操作数据

## 读取数据

```
import pandas as pd
# 根据不同文件选择不同 read_ 方法，如：read_csv
data = pd.read_*(path)
```

## 操作数据

```
import pandas as pd
# 根据不同文件选择不同 read_ 方法，如：read_csv
data = pd.read_*(path)

# 根据行列下标截取
data_1 = data.iloc[start_row:end_row, start_col, end_col]
# 可以根据列名指定列，可以根据没列值进行行筛选（详见：https://pandas.pydata.org/docs/getting_started/intro_tutorials/03_subset_data.html）
# 访问某个属性使用 data["attribute_name"]
```