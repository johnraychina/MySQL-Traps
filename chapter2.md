**最佳实践：**

limit分页要加order by， 注意order by中要带唯一键值，否则可能不同页的数据会重复。
order by和where共用索引，会比不共用快很多。

**MySQL有时候会对带limit且不带having的查询进行优化：**

* 如果用limit只查询少数几行，有时候会走索引，但通常会优先做全表扫描。
* 如果用limit+order by，MYSQL找到对应条数的排序结果集后结束查询，不会对整个where结果集排序。如果order by走了索引就会很快，否则用filesort，会先把where大部分或全部结果集查出来排序，然后取对应条数。

**因为优化，会产生一个现象：order by带limit与不带limit会返回不同的顺序：**

* **limit+distinct优化：如果MySQL或再找到对应条数记录后马上停止查询。**
* **limit+group by优化：group by如果走索引，直接对索引做分组计算，找到对应条数就停止计算。**
* limit 0直接返回空集。主要用于检测表是否存在，获取表结构信息。
* temporary table临时表优化：用limit计算需要的空间。

如果order by中的列有多行相同的值，则这些相同值的行之间的排序是任意的\(nondeterministic \)。

limit会影响执行计划，所以order by的查询，带limit和不带limit返回的结果的排序可能不同。

如果你要求order by加不加limit都是一样的排序，你可以加上其他列使得排序是确定的\(deterministic\).

优化器会处理这样的sql：

```
SELECT ... FROM single_table ... ORDER BY non_index_column [DESC] LIMIT [M,]N;
```

根据mysql的内存排序缓冲区\[sort\_buffer\_size\]大小确定如何优化：

1、如果缓冲区足够大，则把查询结果放到队列里面排序。

2、否则将选定的\[M\]+N行数据存入merge file（临时存放中间结果），对文件排序后，返回M开始的N行数据。

两种方式的代价不同：

1、内存排序比较消耗CPU

2、文件排序比较消耗I\/O

针对不同的N行和行的大小，MySQL会自己权衡使用哪一种方式优化。

