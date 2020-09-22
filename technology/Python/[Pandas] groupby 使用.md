## GroupBy 简介

## GroupBy 对象







## 分组操作

```
- For `DataFrame` objects, a string indicating a column to be used to group. Of course `df.groupby('A')` is just syntactic sugar for `df.groupby(df['A'])`, but it makes life simpler.
- A list or NumPy array of the same length as the selected axis.
- A dict or `Series`, providing a `label -> group name` mapping.
- A Python function, to be called on each of the axis labels.
- For `DataFrame` objects, a string indicating an index level to be used to group.
- A list of any of the above things.
```


- 按列的key分组
- 多层分组，group by 多列
- 按各列的数据类型分组
- 按自定义的key进行分组，可以新增一列用于分组，也可以直接指定原来的某列
- 通过自定义字典分组
- 通过函数分组
- 通过索引级别分组

