# Pandas常用操作
---
## 数据预处理
---
1. 导入、导出文件  
   * 读取CSV文件  

     ```python
     data = pd.read_csv('data/filename.csv'，index_col=0)
     # index_col=0 则读取的数据中不会出现'Unnamed: 0'
     ```

   * 如果是时间序列，则加上：  
     `data = pd.read_csv('data/filename.csv', parse_dates=['时间列名'])`

   * 导出CSV文件

     ```python
     data.to_csv(‘data/filename.csv’, index=False)
     # index=False 则导出的数据中不会出现'Unnamed: 0'
     ```

2. 查看数据前5行和后5行(默认)，head(10)，则为前10行  
    `data.head().append(data.tail())`

3. 查看数据集整体信息  
    `data.describe()`  
    `data.info()`

4. 查看数据缺失值情况  

    ```python
    data.isnull().sum()
    # 查询某值和缺失值位置
    df
        x   y   z
    a NaN   1   2
    b NaN   4   5
    c NaN   7   8
    d NaN  10  11
    
    np.where(df==5) # 查询某具体值
    Out: (array([1], dtype=int64), array([2], dtype=int64)) 返回一个行列的元组
    
    np.where(np.isnan(df))
    Out: (array([0, 1, 2, 3], dtype=int64), array([0, 0, 0, 0], dtype=int64))
    
    np.where(np.isnan(df))[0] # 选出tuple里面的第一个元素，就是行号
    Out: array([0, 1, 2, 3], dtype=int64)
    
    df.index[np.where(np.isnan(df))[0]] # df.index 是获取行名称，对应后面的[0]取行号
    Out: Index(['a', 'b', 'c', 'd'], dtype='object')
    
    df.columns[np.where(np.isnan(df))[1]]
    Out: Index(['x', 'x', 'x', 'x'], dtype='object') # df.columns 是获取列名称，对应后面的[1]取列号
    ```

    

    同时还可以查看是否存在无穷大值

    `np.isinf(data).sum().sum()`

5. 查看某特征唯一值

  `data['列名'].unique()`

6. 转化数字  
    参数：`errors: {‘ignore’, ‘raise’, ‘coerce’}, default ‘raise’`  
    raise返回异常；ignore则忽略无效值；coerce将无效值转为NaN.  
    `df.apply(pd.to_numeric, errors='ignore')`  
    该函数将被应用于整个DataFrame，可以转换为数字类型的列将被转换，而不能(例如，包含非数字字符串或日期)的列将被单独保留.

7. 计算各类别数量  
    `df['列名'].value_counts()`  

8. 通过字典对 df 进行替换
```python
fat_dict = {'Low Fat':0, 'Regular':1, 'LF':0, 'reg':1, 'low fat':0} 
df['列名'] = combi['列名'].replace(fat_content_dict)
```

8. 将某些字符串替换成 DataFrame 可处理的NaN格式

    `data = data.replace('Not Available', np.nan)`

9. 将某特征转换成其它数据类型

    `data[column] = data[column].astype(float)`

    还可以是 **int64，int32，str** 等

10. 更改某列特征名字

    `data = data.rename(columns={'旧名字': '新名字'})`

11. 对某列特征排序

     ```python
     data['列名'].sort_values()
     默认升序，降序则为
     data['列名'].sort_values(ascending=False)
     ```

12. 对DataFrame重置索引值

    ```python
    data.reset_index(drop=True)
    
    *True* 表示丢弃原先索引值，默认为False，保留  
    ```

13. 删除某一列

        data = data.drop(columns=['列名'])
    
14. 删除某一列的NaN值

         data = data.dropna(subset=['列名'])
         data = data.dropna(how='all', axis=1) # 删除全为nan的列
    
15. 合并数据集，此方法比较复杂，参数较多，还有

     ```python
     # 以左边的列名为参考，向左合并
     data = data.merge(data1, how='left', left_on='Id2', right_on='Id2')
     # outer 表示并集，默认是 inner，交集
     data = data.merge(data1, how='outer', on=列名)
     类似的还有**join, concat**细节后面再补充
     ```
    


16. 删除某列重复值

    ```python
    data = data.drop_duplicates(['列名'], keep='first')
    first 表示保留第一个重复值，还有 last
    ```
    
17. 选择

    ```python
    data.loc[ : ]
    按具体的索引值选择
    data.iloc[ : ]
    按顺序索引值选择
    ```

18. 选出数据集中数值类特征和字符类特征`

       ```python
       num_data = data.select_dtypes('number')
       obj_data = data.select_dtypes('object')
       ```

19. 缺失值填充

      ```python
      data = data.fillna(data.median())
       # 用每列的中值填充，还可以用，0，均值mean等
    
      ```

20. 更改列名

    ```python
    # 将第三列的列名改为'new name'
    df.rename(columns={ df.columns[2]: "new name" }, inplace=True)
    
    # 假如df一共有三列，你想把所有列名依次改为'col_1', 'col_2', 'col_3'
    df.columns = ['col_1', 'col_2', 'col_3']
    
    假设你的df里有一列叫做'col_1'，如果你想把这列的名字改成'col_a'，那么可以这样
    df = df.rename(columns={'col_1': 'col_a'})
    ```

21. 选择某列

      ```python
      # 互信息计算
      value = mutual_info_regression(data.iloc[:, i:i+1], data.iloc[:, j])[0]
      # 赋值
      data.iloc[i, j] = value
      ```

22. 按行索引值删除

      ```python
      # row_remove 是列表,按索引值删除，不是顺序
      X = X.drop(index=row_remove)
      ```

23. 数据操作符

      ```python
      | (or), & (and), ~ (not)
      例如：
      X[(X[col] < (Q1 - 15 * IQR)) | (X[col] > (Q3 + 15 * IQR)) # 取“或”
      # 输出满足条件的data的索引值，存在列表中
      X[(X[col] < (Q1 - 15 * IQR)) | (X[col] > (Q3 + 15 * IQR))].index.tolist()
      ```

24. 找出最大和最小值位置

      ```python
      # 找到最大值和最大值所对应的位置
      df.stack().max()
      Out: 89
      df.stack().idxmax()  # 最小值同理，idxmin()
      Out: (1, 2)
      # 如果只是某一列
      df['col'].idxmax() 
      Out: 1
      ```

25. 查看类别特征数量

    ```python
    data.groupby(by=['列名']).size().reset_index()
    # 还可以使用
    from collections import Counter
    print(sorted(Counter(data['列名']).items()))
    ```

26. 对数值特征进行分类

    ```python
    # cut，‘[-1, WI.mean(), WI.max()]’，像这样，第一个类是包含均值的，半开半闭，可以是要分的类别整数
    WI = data['WI']
    wi = pd.cut(WI, [-1, WI.mean(), WI.max()], labels=[0, 1])
    data['WI'] = wi
    
    
    ```

    

