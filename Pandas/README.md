# Pandas常用操作
---
## 数据预处理
---
1. 导入文件  
   * 读取CSV文件  
     `data = pd.read_csv('data/filename.csv')`  
     如果是时间序列，则加上：  
     `data = pd.read_csv('data/filename.csv', parse_dates=['时间列名'])`

2. 查看数据前5行和后5行(默认)，head(10)，则为前10行  
`data.head().append(data.tail())`
3. 查看数据集整体信息  
`data.describe()`  
`data.info()`
3. 查看数据缺失值情况  
`data.isnull().sum()`
4. 转化数字  
`data['列名'] = pd.to_numeric(data['列名'], errors='coerce')`  
参数：`errors: {‘ignore’, ‘raise’, ‘coerce’}, default ‘raise’`  
raise返回异常；ignore则忽略无效值；coerce将无效值转为NaN.  
`df.apply(pd.to_numeric, errors='ignore')`  
该函数将被应用于整个DataFrame，可以转换为数字类型的列将被转换，而不能(例如，包含非数字字符串或日期)的列将被单独保留.
5. 啦啦啦

`combi['Item_Fat_Content'].value_counts()`


Item_Fat_Content似乎只包含两个类别，即“低脂”和“常规”，未涉及到“冗余”类别，所以我们把它转换成二进制变量。
```
fat_content_dict = {'Low Fat':0, 'Regular':1, 'LF':0, 'reg':1, 
                    'low fat':0} 
combi['Item_Fat_Content'] = combi['Item_Fat_Content'].replace(   
                            fat_content_dict, regex=True)
```