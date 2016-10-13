* mysql数据量超过3千万性能猛降
* 删除历史partition后，性能猛增10倍：60多秒&gt;&gt;5秒不到
* 结论：其他partition分区的数据会影响本分区的查询

---

* 单条sql超时，由于sql语句先做order by在做limit，将sql语句内置（红色&gt;&gt;蓝色）
* 总体时间会变长，但是单条sql变短。
* 分页越到后面查询越慢（cursor从头开始移动距离变长）。

| SELECT AC\_ORG as FRT\_ACC\_ORG, AC\_TYP as FRT\_AC\_TYP, CAP\_TYP as FRT\_CAP\_TYP, CCY as FRT\_CCY,

SUM\( CASE WHEN UPD\_DT &gt; \#{sumDt} THEN LAST\_AC\_BAL ELSE CUR\_AC\_BAL END \) AS FRT\_AC\_BAL

FROM ACTTACBL

GROUP BY AC\_ORG, AC\_TYP, CAP\_TYP, CCY

ORDER BY AC\_ORG, AC\_TYP, CAP\_TYP, CCY |
| --- |
| SELECT AC\_ORG as FRT\_ACC\_ORG,

AC\_TYP as FRT\_AC\_TYP,

CAP\_TYP as FRT\_CAP\_TYP,

CCY as FRT\_CCY,

SUM\(AC\_BAL\) AS FRT\_AC\_BAL

FROM

\(

 SELECT AC\_ORG, AC\_TYP, CAP\_TYP, CCY,

 \( CASE WHEN UPD\_DT &gt; \#{sumDt} THEN LAST\_AC\_BAL ELSE CUR\_AC\_BAL END \) AS AC\_BAL

 FROM ACTTACBL

 ORDER BY ID

 LIMIT \#{offset}, \#{limit}

\) AS A

GROUP BY AC\_ORG, AC\_TYP, CAP\_TYP, CCY; |



` SELECT AC_ORG as FRT_ACC_ORG,````AC_TYP as FRT_AC_TYP,````CAP_TYP as FRT_CAP_TYP,````CCY as FRT_CCY,````SUM(AC_BAL) AS FRT_AC_BAL````FROM````(```` SELECT AC_ORG, AC_TYP, CAP_TYP, CCY,```` ( CASE WHEN UPD_DT > #{sumDt} THEN LAST_AC_BAL ELSE CUR_AC_BAL END ) AS AC_BAL```` FROM ACTTACBL```` ORDER BY ID```` LIMIT #{offset}, #{limit}````) AS A````GROUP BY AC_ORG, AC_TYP, CAP_TYP, CCY; `





* truncate table和drop partition会释放空间
* delete不会释放空间（可以看mysql的文件大小变化）
* 可以在delete后做重整优化，optimize table my\_table

