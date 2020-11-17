# utf8 bom编码处理

- decode字符传乱码： 原因是从Facebook下载的csv文件是utf8 bom编码的， 直接decode('utf-8')会出现'\ufeff'； 应该decode('utf-8-sig')
  - 原本以为是文件内容导致的问题，debug之后，发现文件开头有'\ufeff'； 不知道这个是什么东西，墨迹了一会儿之后，尝试Google了才有答案；豁然开朗了

```python
data_str = file.read().decode('utf-8-sig')
datas = data_str.split('\n')
reader = csv.DictReader(datas)
```

