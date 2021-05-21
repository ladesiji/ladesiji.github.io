---
layout: post
title: Pandas 学习总结 
subtitle: 采坑记录
categories: [leetcode]
---

最近几个月，写了几个小工具，都是用Pandas来处理工作中的EXCEL表格，将一些常用的，流程化的东西用工具来处理。
在这个过程中，对Pandas处理数据的一些操作慢慢有了一点点心得，赶紧记录一下，以备忘记了再来查询。


## 数据读取

数据读取是非常非常重要的步骤，Pandas为些提供了强力的支持，参数有四五十个，
用好了数据读取能省去大量的数据调整工作，非常值得仔细的研究。
Pandas 读取数据主要用到有两种 `read_csv` 和 `read_Excel`。 其中`csv`的稍微多一些，
两个里面参数有很多功能相同的，这里主要以`read_csv` 为例来介绍下我用过的。
参考链接[详解pandas的read_csv方法](https://www.cnblogs.com/traditional/p/12514914.html)

### 普通读取

read_csv中全部参数比较多，有很多其实用不太到，主要挑选我用过了几个详细说一下。

```python
# 全部参数
pandas.read_csv(filepath_or_buffer, sep=<object object>, delimiter=None, header='infer', names=None, index_col=None, usecols=None, squeeze=False, prefix=None, mangle_dupe_cols=True, dtype=None, engine=None, converters=None, true_values=None, false_values=None, skipinitialspace=False, skiprows=None, skipfooter=0, nrows=None, na_values=None, keep_default_na=True, na_filter=True, verbose=False, skip_blank_lines=True, parse_dates=False, infer_datetime_format=False, keep_date_col=False, date_parser=None, dayfirst=False, cache_dates=True, iterator=False, chunksize=None, compression='infer', thousands=None, decimal='.', lineterminator=None, quotechar='"', quoting=0, doublequote=True, escapechar=None, comment=None, encoding=None, dialect=None, error_bad_lines=True, warn_bad_lines=True, delim_whitespace=False, low_memory=True, memory_map=False, float_precision=None, storage_options=None)


```

1. `header`参数用于指定文件的第几行是表头，注意默认情况下，会跳过空行`skip_blank_lines=True`，所以文件头有空行的话，不算空行的行数。
2. `names`参数用列表格式指定文件的表头，跟header是配合使用的。使用方法如下：
    >
    > - csv文件有表头并且是第一行，那么names和header都无需指定;
    > - csv文件有表头、但表头不是第一行，可能从下面几行开始才是真正的表头和数据，这个时候指定header即可;
    > - csv文件没有表头，全部是纯数据，那么我们可以通过names手动生成表头;
    > - csv文件有表头、但是这个表头你不想用，这个时候同时指定names和header。先用header选出表头和数据，然后再用names将表头替换掉，就等价于将数据读取进来之后再对列名进行rename；
    >
3. `index_col` 指定索引列。
4. `usecols` 指定需要读取的列，适用于数据量比较大，只想取用到的列，可以输入列表`[1,2,5,7]`，也可以用于调整列的顺序，如`[5,2,1,7]`。
5. `dtype` 指定数据的格式，比如整数、浮点、字符串等。
6. `engine` 这个是指定读取引擎，默认是C的，可也以用`python`，有一些参数如'skipfooter' 只有Python引擎有。 C的更快一些，快4-5倍。
7. `parse_dates`指定哪些列的类型为时间，如果不是通用时间格式，可以用`date_parser` 传入函数来指定解析格式。
8. `converters` 对指定列进行数据变换，非常有用的一个参数，可传入多个函数可以直接在读取时把需要处理的列处理好。
9. `na_values` 指定需要设置为nan的值，非常有用。例如数据列中含有'NIL'，如果不设置，那这个列的dtype就是object，无法直接运算，如果设置`na_values=['NIL']`，那这一列数据dtype就能成为float类型，可以直接运算。
10. `low_memory` 跟上个参数有关，分块读取数据，如果数据中整数类型中含有'NIL'字符串，出现混合类型会报错，将`low_memory=False`能解决。
11. `compression` 这个参数让我们可以直接读取压缩文件，非常非常有用。
12. `encoding` 读取文件中的编码格式，一般用`gbk` 和`utf-8`

我用过主要的参数就是上面这些了，讲几个有代表性的例子

### 读取IO数据流

使用Requests模块从网页上下载了压缩包格式的CSV文件，直接用Pandas打开，可以省去很多不必要的步骤，非常Pythonic。

```python

import requests
import pandas as pd
from io import BytesIO

# 示例代码
r = session.post(url_download, data=download_info, stream=True)
df = pd.read_csv(BytesIO(r.content), compression='zip', encoding='gbk')

```

### 批量读取数据

- 如果想合并多个同样格式的文件，可以用列表循环+合并来解决

```python

import pandas as pd
import os
DIR = 'data/'
def read_csv(filename):
    # 读取tar.gz文件到pandas
    df = pd.read_csv(filename, compression='gzip', encoding='gbk', 
                    header=4, low_memory=False, na_values=['NIL'], 
                    parse_dates=['开始时间'],)

df_list = [read_csv(DIR + filename) for filename in os.listdir(DIR) if 'tar.gz' in filename]
df = pd.concat(df_list)

```

- 如果读取一个Excel中多个格式一样的sheet，然后合并，可以设置sheet_name为None，读取结果为字典，key为sheet_name，value为dataframe，再将结果集合并即可。

```python
# 读取所有sheet
df_dict = pd.read_excel(FileName, sheet_name=None)

df = pd.concat(df_list.values())
```

### 读取时处理文件

用Pandas读取时直接设置日期列、数据格式，并完成部分数据的处理任务。

```python
import pandas as pd
import re
df = pd.read_excel(filename, 
                    index_col=0,                         # 设置索引列为第0列
                    usecols=[0,2,7,8,9,17,18,19,31],     # 选取部分用到的数据列 
                    dtype={'投诉号码': object},           # 设置格式
                    parse_dates=['开始时间', '结束时间'],  # 设置 时间列
                    converters={'CGI  ': lambda x: re.sub(r'460-00-(?P<enodeb>\d+)-(?P<id>\d+)',   # 传入函数在读取时处理数据
                                                         r'\g<enodeb>', x)})

```

## 数据处理

Pandas数据处理功能齐全，内容丰富。所以这部分只罗列我个人用到过的，
毕竟这些才是辛辛苦苦学会的，如果有新的比较有用的知识点，会再添加。

### 数据选取

数据的选取和切片是重点内容，基础中的基础，必须熟练掌握。 主要的切片方法有３种，请视个人情况熟练掌握一种即可。

```python
 # 示例数据
import pandas as pd
import numpy as np
dates = pd.date_range('1/1/2021', periods=8)
df = pd.DataFrame(np.random.randn(8, 4),index=dates, columns=['A', 'B', 'C', 'D'])

```

1. `[]` 切片方法，最常用的数据选择法，跟python的列表切片用法基本一致，能够实现行选择、列选择、区块选择

```python

# 行选择
df[1:5] # 选取1-4行， 注意此处区间是左闭右开，不包括第5行数据

# 列选择
df['A'] # 选取A列
df[['A', 'B', 'C']]  # 多列选取时需要传入列表
df[['C', 'A', 'B']]  # 可调整顺序

# 区块选择
df[:7][['A','B']] # 选取前7行对应A和B列。 先写行和先写列都可以。

```

2. `loc` 方法，主要的数据标签来选取数据。 行和列都必需要数据标签。`loc`有固定的格式。`df.loc[index_lable, columns_lable]`。`index_lable`是必填，`columns_lable`是选填的。
    lable的格式有四种，行和列可自由组合：
    - 单个标签，如 `df.loc['2021-01-01', 'A']`
    - 连续的切片，全部行或全部列用`:`，如 `df.loc[:,'A':'C']` `df.loc[(]'2021-01-01':'2021-01-05']`
    - 自定义的行或列，传入标签列表`df.loc[['2021-01-01', '2021-01-03'], ['A', 'C']]`
    - 列表形式的布尔数组，这个在筛选的时候再介绍
    - 函数形式，不过函数返的值必须是上面四种之一

```python

# 行选择，必须用行标签来选择, 列标签可以省去不填，默认为全部列
df.loc['2021-01-01']  # 选择单行
df.loc['2021-01-01':'2021-01-05'] # 选择1-5行，注意此处区间为左闭右闭，包括第5行
df.loc[['2021-01-01','2021-01-03','2021-01-04']] # 选择不连续多行

# 列选择，行标签不能省，如果是全部行，用 ： 来代替
df.loc[:, 'A'] # 单列
df.loc[:, 'C':'D'] # 连续多列
df.loc[:, ['A', 'C', 'D']]   # 不连续的多列

# 块选择, 可用行列标签随意组合
df.loc['2021-01-01':'2021-01-05', 'C':'D'] 

```

3. `iloc`方法，用法同`loc`基本一致，只能用整数做索引，即行号和列号，唯一的区别是`iloc`的区间是左闭右开。

```python

# 行选择, 列标签可省略
df.iloc[5] # 单行
df.iloc[1:5] # 选择1-4行，不包括第5行。
df.iloc[[1,3,5]] # 选择不连续多行

# 列选择
df.iloc[:, 3]    # 单列
df.iloc[:, 2:4]  # 连续多列
df.iloc[:, [1,3]]  # 不连续多列

# 块选择
df.iloc[1:5, [1,3]] 

```

4. `at` 和 `iat` 方法，用于选取一个元素，用法同`loc`和`iloc`，一个是用标签，一个用整数。

```python

df.at['2021-01-05', 'A']
df.iat[1,2]

```

### 数据筛选

数据筛选其实是数据选择的一种形式：布尔数组选择。
比如说我要筛选A列大于0的数据，`df['A'] > 0` 返回一个列的布尔列表（长度为行的大小），
将这个结果应用到`[], loc, iloc` 的选取方法中，就可以实现筛选

```python

df['A'] > 0
# [False, False, True, False, False, False, True, False]

df.loc[df['A'] > 0, df.loc['2021-01-03'] > 0] # 行的筛选加上列的筛选

```

多条件筛选的实质还是生成一个布尔数组，只不过需要先通过布尔运算符来计算一下。
这里需要注意的是格式，`|` for `or`，`&` for `and`  `~`for `not`，
必须用括号对条件进行分组

```python

df.loc[(df['A'] > 0) & (df['B'] < 1), 'C']  # A列大于0且B列小于1 对应C 列的值

```

这里重点介绍一个非常常用的用法 `Series.isin()` , 这个方法能够快速生成一个布尔向量列表
如果有两个表格，取交叉数据时非常好用。

```Python

df[df.index.isin(['2021-01-07','2021-01-08'])]  # 筛选index在列表中的DF

df[df['小区'].isin(df_focus_cell['CellName'])]  # 筛选df中小区在df_focus_cell中相同的行。

```

### 添加列

从上面数据选取的用法来看，其实dataframe中列的操作类似于字典的方法，
列名为key, value为数组。在添加、删除、修改这三个操作上形式都差不多。

```python

# 删除列
del df['A'] 
df.drop(['B', 'C'], axis=1, inplace=True) 

# 添加列和修改列

df['E'] = 0  # 添加常数列
df['A'] = df['B'] + df['C'] 

```

### 调整顺序

有时候需要将df数据写入到文件，如果对数据列的顺序有要求的话，可以用以下方法调整

1. 在读取时，使用`usecols`参数，传入自定义的列表
2. 直接重新选择
3. 使用`set_index` 和 `reset_index` 方法，可以将部分列置于第一位

```python

df = pd.read_excel(filename, index_col=0, usecols=[0,7,8,2,9]) # 在读取时修改

df = df[['B','D','C','A']]  # 在选择时重新排序

INDEX_LIST = ['ECI','eNodeB标识','小区标识','网元名称','小区']
df.set_index(INDEX_LIST).reset_index()  # 将'ECI' 等索引列 设置成索引再取消

```

### 列转行

我之前有一个使用场景是将 日期、小区、数据 这三列数据，变成以小区为索引，日期为列的多维表，
类似于EXCEL数据透视表中行和列的变幻，在Python中涉及到多重列表的概念，这个部分我用的不多，
是通过`stack` 和 `unstack` 这两个方法实现的。

这是之前写的一个将日期由列转为行的函数

```python

def transform(df, index_nums:int):
    '''
        行转列处理
        index_nums为数据索引列个数
    '''

    # 将除数据列外的其他列设置为索引
    df.set_index(list(df.columns)[:INDEX_NUMS], inplace=True)
    
    # 行转列 将第0列的列索引转为行索引
    df = df.unstack(level=0) 

    # 为避免同索引的行合并单元格，将多索引重置为单索引。
    # 去除索引 
    df.reset_index(inplace=True)

    # 重设索引
    df.set_index(list(df.columns)[0][0], inplace=True)
    return df

```

### 汇总

汇总是一个核心的功能，我用的不多，详细的原理请参考[Groupby用法详解](https://zhuanlan.zhihu.com/p/101284491)

> groupby的过程就是将原有的DataFrame按照groupby的字段（这里是company），划分为若干个分组DataFrame，
> 被分为多少个组就有多少个分组DataFrame。所以说，在groupby之后的一系列操作（如agg、apply等），
> 均是基于子DataFrame的操作。理解了这点，也就基本摸清了Pandas中groupby操作的主要原理。

在实践中遇到一个场景，有30多列数据，其中只有个别列（假定有3列）是求'max','mean'，其他都是求和，
如果在汇总时为这3列传入一个30多个字段的字典，显然有点太low了。 最终的解决思路是：只给非求和的列自定义一个字典，
先按整体按求和汇总，再单独为3列非求和列进行汇总，最后用`update()`函数将非求和列数据更新到全部数据中即可

```python

def groupby_cell(df, func_dic = {}):
    """
        将数据按小区汇总
        func_dic 为非求和列的汇总方式，如平均、最大
    """
    index_list = ['网元名称','小区']
    grouped = df.groupby(index_list, as_index=False)
    # 先整体汇总求和
    result = grouped.aggregate(sum)
    # 计算特殊列
    if func_dic:
        special_col = grouped.aggregate(func_dic)
        # 将特殊列的计算结果更新到结果中
        result.update(special_col)
    return result

```

### 合并

pandas有多种合并方式，我个人用过的场景主要是将相同的dataframe合并在一起，所以只涉及到 `concat()` 函数。

```python

df = pd.concat(df_list)

```

### 列运算

数据处理过程中，非常常见的情景是需要根据现有的一列或几列数据，通过一些运算生成新的一列数据。
Pandas主要用apply方法来实现：

`DataFrame.apply(func, axis=0, raw=False, result_type=None, args=(), **kwds)`

具体的原理为apply将dataframe的行或列做为一个循环单元(item)，依次传入`func`中。
这里介绍两种我用过的形式。

1. 多列参运算，生成一列新数据

```python

df['new'] = df.apply(lambda x: x['A'] + x['B'] - x['C'], axis=1) # axis 为1 每传入1行，axis 为0 每次传入一列。

```

2. 多列参与运算，生成多列。

```python

def func_sum(row):
    """计算两列和"""
    new1 = row['A'] + row['B']
    new2 = row['B'] + row['C']
    new3 = row['C'] + row['D']
    
    return new1, new2, new3

df[['new1', 'new2', 'new3']] = df.apply(func_sum, axis=1, result_type='expand')

```

### 保存

将处理完成的结果保存到CSV或EXCEL中，才算最终完成。从我的经验来看，数据量大的时候，存CSV会比EXCEL快很多。
所以尽量存CSV。 如果有汉字的话，用'gbk'编码也能解决乱码问题。

```python

# 保存为CSV
df.to_csv('result.csv', index=False)

# 保存到EXCEL
writer = pd.ExcelWriter('result.xlsx')
data.to_excel(writer,'合并', index=False)
writer.save()

```

写入EXCEL时，默认是生成一个新的EXCEL。 如果我们想在已有的表格中，新写入一个sheet，需要使用openpyxl模块。

```python

from openpyxl import load_workbook

workbook = openpyxl.load_workbook('TEMPLATE.xlsx')
writer = pd.ExcelWriter('TEMPLATE.xlsx', engine='openpyxl')
writer.book = workbook
df.to_excel(writer, sheet_name='newsheet')
workbook.save('new.xlsx')

```

## 总结

综上是自己在使用Pandas过程中摸索出来的一点点东西，很多的方法、参数都是需要自己在实践中反复调试才明白。
以后如果有新的收获，会增补进去。
