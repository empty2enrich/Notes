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
# 可以根据列名指定列，可以根据每列值进行行筛选（详见：https://pandas.pydata.org/docs/getting_started/intro_tutorials/03_subset_data.html）
data[data["attr"].isin([2, 3])]
data[(data["attr"] == 2) | (data["attr"] == 3)]
data[data["attr"].notna()] # 筛选非空

# 筛选 Attribute_1 > 35, Attribute_2 的值。
data.loc[data["Attribute_1"] > 35, "Attribute_2"]

# 访问某个属性使用 
data["attribute_name"]

# 访问多列，示例
data[["Attr_1", "Attr_2"]]

# 删除某列
data.drop(columns=["Attribute_name"])

# 修改列名, columns 长度应该与 data 列数一致，若columns 长度小于 data 列数，
# 测试值都变为 NaN,可能还有其他情况，暂时未测试
data.reindex(columns=['names'])
```