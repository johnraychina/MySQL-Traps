* mysql数据量超过3千万性能猛降
* 删除历史partition后，性能猛增10倍：60多秒&gt;&gt;5秒不到
* 结论：其他partition分区的数据会影响本分区的查询

---

* 单条sql超时，由于sql语句先做order by在做limit，将sql语句内置（红色&gt;&gt;蓝色）
* 总体时间会变长，但是单条sql变短。
* 分页越到后面查询越慢（cursor从头开始移动距离变长）。
* 
* truncate table和drop partition会释放空间

* delete不会释放空间（可以看mysql的文件大小变化）

* 可以在delete后做重整优化，optimize table my\_table
```
SELECT AC_ORG as FRT_ACC_ORG, AC_TYP as FRT_AC_TYP, CAP_TYP as FRT_CAP_TYP, CCY as FRT_CCY, SUM( CASE WHEN UPD_DT > #{sumDt} THEN LAST_AC_BAL ELSE CUR_AC_BAL END ) AS FRT_AC_BAL FROM ACTTACBL GROUP BY AC_ORG, AC_TYP, CAP_TYP, CCY ORDER BY AC_ORG, AC_TYP, CAP_TYP, CCY

```

