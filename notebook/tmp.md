#### 6 Kaggle影评数据分析实战

##### 1 了解数据

数据来自kaggle，共包括三个文件：

1. movies.dat
2. ratings.dat
3. users.dat

`movies.dat`包括三个字段：['Movie ID', 'Movie Title', 'Genre']

使用pandas导入此文件：

```python
import pandas as pd

movies = pd.read_csv('./data/movietweetings/movies.dat', delimiter='::', engine='python', header=None, names = ['Movie ID', 'Movie Title', 'Genre'])
```

导入后，显示前5行：

```python
   Movie ID                                        Movie Title  \
0         8      Edison Kinetoscopic Record of a Sneeze (1894)   
1        10                La sortie des usines Lumi猫re (1895)   
2        12                      The Arrival of a Train (1896)   
3        25  The Oxford and Cambridge University Boat Race ...   
4        91                         Le manoir du diable (1896)   
5       131                           Une nuit terrible (1896)   
6       417                      Le voyage dans la lune (1902)   
7       439                     The Great Train Robbery (1903)   
8       443        Hiawatha, the Messiah of the Ojibway (1903)   
9       628                    The Adventures of Dollie (1908)  
                                          Genre  
0                             Documentary|Short  
1                             Documentary|Short  
2                             Documentary|Short  
3                                           NaN  
4                                  Short|Horror  
5                           Short|Comedy|Horror  
6  Short|Action|Adventure|Comedy|Fantasy|Sci-Fi  
7                    Short|Action|Crime|Western  
8                                           NaN  
9                                  Action|Short  
```



次导入其他两个数据文件

`users.dat`:

```python
users = pd.read_csv('./data/movietweetings/users.dat', delimiter='::', engine='python', header=None, names = ['User ID', 'Twitter ID'])
print(users.head())
```

结果：

```python
   User ID  Twitter ID
0        1   397291295
1        2    40501255
2        3   417333257
3        4   138805259
4        5  2452094989
5        6   391774225
6        7    47317010
7        8    84541461
8        9  2445803544
9       10   995885060
```



`rating.data`:

```python
ratings = pd.read_csv('./data/movietweetings/ratings.dat', delimiter='::', engine='python', header=None, names = ['User ID', 'Movie ID', 'Rating', 'Rating Timestamp'])
print(ratings.head())
```

结果：

```python
   User ID  Movie ID  Rating  Rating Timestamp
0        1    111161      10        1373234211
1        1    117060       7        1373415231
2        1    120755       6        1373424360
3        1    317919       6        1373495763
4        1    454876      10        1373621125
5        1    790724       8        1374641320
6        1    882977       8        1372898763
7        1   1229238       9        1373506523
8        1   1288558       5        1373154354
9        1   1300854       8        1377165712
```

##### 2 read_csv使用说明

说明，本次导入`dat`文件使用`pandas.read_csv`函数。

第一个位置参数`./data/movietweetings/ratings.dat` 表示文件的相对路径

第二个关键字参数：`delimiter='::'`，表示文件分隔符使用`::`

后面几个关键字参数分别代表使用的引擎，文件没有表头，所以`header`为`None;`

导入后dataframe的列名使用`names`关键字设置，这个参数大家可以记住，比较有用。



Kaggle电影数据集第一节，我们使用数据处理利器 `pandas`， 函数`read_csv` 导入给定的三个数据文件。

```python
import pandas as pd

movies = pd.read_csv('./data/movietweetings/movies.dat', delimiter='::', engine='python', header=None, names = ['Movie ID', 'Movie Title', 'Genre'])
users = pd.read_csv('./data/movietweetings/users.dat', delimiter='::', engine='python', header=None, names = ['User ID', 'Twitter ID'])
ratings = pd.read_csv('./data/movietweetings/ratings.dat', delimiter='::', engine='python', header=None, names = ['User ID', 'Movie ID', 'Rating', 'Rating Timestamp'])
```
用到的`read_csv`，某些重要的参数，如何使用在上一节也有所提到。下面开始数据探索分析(EDA)

> 找出得分前10喜剧(comedy)



##### 3 处理组合值

表`movies`字段`Genre`表示电影的类型，可能有多个值，分隔符为`|`，取值也可能为`None`.

针对这类字段取值，可使用Pandas中Series提供的`str`做一步转化，**注意它是向量级的**，下一步，如Python原生的`str`类似，使用`contains`判断是否含有`comedy`字符串：

```python
mask = movies.Genre.str.contains('comedy',case=False,na=False)
```
注意使用的两个参数：`case`, `na`

case为 False，表示对大小写不敏感；
na Genre列某个单元格为`NaN`时，我们使用的充填值，此处填充为`False`

返回的`mask`是一维的`Series`，结构与 movies.Genre相同，取值为True 或 False.

观察结果：

```python
0    False
1    False
2    False
3    False
4    False
5     True
6     True
7    False
8    False
9    False
Name: Genre, dtype: bool
```




##### 4 访问某列

得到掩码mask后，pandas非常方便地能提取出目标记录：

```python
comedy = movies[mask]
comdey_ids = comedy['Movie ID']
```

以上，在pandas中被最频率使用，不再解释。看结果`comedy_ids.head()`：

```python
5      131
6      417
15    2354
18    3863
19    4099
20    4100
21    4101
22    4210
23    4395
25    4518
Name: Movie ID, dtype: int64
```



1-4介绍`数据读入`，`处理组合值`，`索引数据`等, pandas中使用较多的函数，基于Kaggle真实电影影评数据集，最后得到所有`喜剧 ID`：
```python
5      131
6      417
15    2354
18    3863
19    4099
20    4100
21    4101
22    4210
23    4395
25    4518
Name: Movie ID, dtype: int64
```

下面继续数据探索之旅~

##### 5 连接两个表

拿到所有喜剧的ID后，要想找出其中平均得分最高的前10喜剧，需要关联另一张表：`ratings`:

再回顾下ratings表结构：

```python
   User ID  Movie ID  Rating  Rating Timestamp
0        1    111161      10        1373234211
1        1    117060       7        1373415231
2        1    120755       6        1373424360
3        1    317919       6        1373495763
4        1    454876      10        1373621125
5        1    790724       8        1374641320
6        1    882977       8        1372898763
7        1   1229238       9        1373506523
8        1   1288558       5        1373154354
9        1   1300854       8        1377165712
```


pandas 中使用`join`关联两张表，连接字段是`Movie ID`，如果顺其自然这么使用`join`：

```python
combine = ratings.join(comedy, on='Movie ID', rsuffix='2')
```
左右滑动，查看完整代码

大家可验证这种写法，仔细一看，会发现结果非常诡异。

究其原因，这是pandas join函数使用的一个算是坑点，它在官档中介绍，连接右表时，此处右表是`comedy`，它的`index`要求是连接字段，也就是 `Movie ID`. 

左表的index不要求，但是要在参数 `on`中给定。

**以上是要注意的一点**

修改为：

```python
combine = ratings.join(comedy.set_index('Movie ID'), on='Movie ID')
print(combine.head(10))
```

以上是OK的写法

观察结果：

```python
   User ID  Movie ID  Rating  Rating Timestamp Movie Title Genre
0        1    111161      10        1373234211         NaN   NaN
1        1    117060       7        1373415231         NaN   NaN
2        1    120755       6        1373424360         NaN   NaN
3        1    317919       6        1373495763         NaN   NaN
4        1    454876      10        1373621125         NaN   NaN
5        1    790724       8        1374641320         NaN   NaN
6        1    882977       8        1372898763         NaN   NaN
7        1   1229238       9        1373506523         NaN   NaN
8        1   1288558       5        1373154354         NaN   NaN
9        1   1300854       8        1377165712         NaN   NaN
```

Genre列为`NaN`表明，这不是喜剧。需要筛选出此列不为`NaN` 的记录。

##### 6 按列筛选

pandas最方便的地方，就是向量化运算，尽可能减少了for循环的嵌套。

按列筛选这种常见需求，自然可以轻松应对。

为了照顾初次接触 pandas 的朋友，分两步去写：

```python
mask = pd.notnull(combine['Genre'])
```

结果是一列只含`True 或 False`的值

```python
result = combine[mask]
print(result.head())
```

结果中，Genre字段中至少含有一个Comedy字符串，表明验证了我们以上操作是OK的。

```python
    User ID  Movie ID  Rating  Rating Timestamp             Movie Title  \
12        1   1588173       9        1372821281      Warm Bodies (2013)   
13        1   1711425       3        1372604878        21 & Over (2013)   
14        1   2024432       8        1372703553   Identity Thief (2013)   
17        1   2101441       1        1372633473  Spring Breakers (2012)   
28        2   1431045       7        1457733508         Deadpool (2016)   

                             Genre  
12           Comedy|Horror|Romance  
13                          Comedy  
14    Adventure|Comedy|Crime|Drama  
17              Comedy|Crime|Drama  
28  Action|Adventure|Comedy|Sci-Fi  

```

工作紧张，现在每天只能写一点。我尽量写的详细点，一步一步来吧，希望能帮助到想入门Python数据分析的朋友。

明天继续朝着目标，前进一点点~