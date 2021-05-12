# 自定义查询

当数据分析工作台现有分析模块无法满足查询需求时，可以选择使用自定义查询的方法进行数据查询。

自定义查询使用标准SQL语句进行查询，需要具备一定的技术基础。

## 自定义查询界面概览

![](https://raw.githubusercontent.com/liushuang1205/docs/dfedffd19e7a2c476283d11b3f29313256234ed4/配图/自定义查询/自定义查询界面.png)

**自定义查询界面**

## 标签释义

![](https://raw.githubusercontent.com/liushuang1205/docs/dfedffd19e7a2c476283d11b3f29313256234ed4/配图/自定义查询/自定义查询界面顶部标签.png)

**自定义查询界面顶部标签**

> 查询编辑器：自定义查询的基本命令编辑区
>
> 我的查询：用户最近运行的查询及最近保存的查询会展示在此处，后续可以直接复用之前的查询条件
>
> 保存的查询：用户已经保存的查询条件展示在此处，后续可以直接复用之前的查询条件
>
> 历史记录：用户之前查询的历史记录会保存在此处

## 使用方法

### 获取字段名

![](https://raw.githubusercontent.com/liushuang1205/docs/dfedffd19e7a2c476283d11b3f29313256234ed4/配图/自定义查询/表名与字段名.png)

**表名与字段名**

在自定义查询界面的左侧，展示当前项目的Event表与User表的表名与表内包含的所有字段名，点击表名即可展开该表内包含的所有字段名。

### 执行查询

![](https://raw.githubusercontent.com/liushuang1205/docs/dfedffd19e7a2c476283d11b3f29313256234ed4/配图/自定义查询/SQL语句查询.png)

**SQL语句查询**

在自定义查询界面的右上角为SQL语句编辑窗口，在窗口中编写标准SQL语句后点击查询，下方将实时展示查询得到的结果。

### 保存查询条件

![](https://raw.githubusercontent.com/liushuang1205/docs/dfedffd19e7a2c476283d11b3f29313256234ed4/配图/自定义查询/保存查询条件.png)

**保存查询条件**

在自定义查询编辑区，点击另存为，按业务需求设置名称与描述，可以将此查询条件保存至**我的查询**中，后续用户可通过“自定义查询”→“我的查询”→“最近保存的查询”中找到已经保存的查询直接执行。

### 执行新查询

![](https://raw.githubusercontent.com/liushuang1205/docs/dfedffd19e7a2c476283d11b3f29313256234ed4/配图/自定义查询/执行新查询.png)

**执行新查询**

用户在执行过一次查询之后，如果想要对其他条件进行查询，可以点击“新查询”按钮，此时查询条件编辑器清空，用户可以重新使用标准SQL语句对想要的条件进行查询。

## 查询样例

### 查询下午20点至24点用户的支付情况

```sql
SELECT date, COUNT(*) FROM events WHERE EXTRACT(HOUR FROM time) IN (20, 24) AND event = 'payment' GROUP BY 1
```

以上，我们使用SQL中的EXTRACT函数来获取小时信息。

### 查询2020-05-01至2020-05-10用户的下单情况

```sql
SELECT user_id, COUNT(*) AS c FROM events WHERE date BETWEEN '2020-05-01' AND '2020-05-10' AND event = 'payment' GROUP BY 1
```

以上，我们获取到了在2020-05-01至2020-05-10用户的下单情况。

## Impala函数与保留字段

[Impala函数参考](https://docs.cloudera.com/documentation/enterprise/latest/topics/impala_functions.html)

[Impala保留字段](https://impala.apache.org/docs/build3x/html/topics/impala_reserved_words.html)

