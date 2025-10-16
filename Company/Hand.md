# 汉得

20.补全这段SQL使其能够将SUM函数作为**窗口函数**使用，针对Product表进行统计得到右表

```sql
SELECT product_id,product_name,sale_price,
SUM(sale_price) OVER(ORDER BY product_id) AS current_sum 
FROM Product
```





问题：请统计每个创作者发布的内容总计浏览量和点赞量?

要求：输出 author_name 总计浏览量，总计浏览量；按照 author_name 顺序排序

```sql
SELECT 
    a.author_name,  -- 创作者姓名
    SUM(i.info_pv) AS 总计浏览量,  -- 总计浏览量（求和）
    SUM(i.praise_num) AS 总计点赞量  -- 总计点赞量（求和）
FROM 
    author_tb a
JOIN 
    put_action_tb p ON a.author_id = p.put_author  -- 连接创作者表和发布动作表
JOIN 
    info_tb i ON p.put_info_id = i.info_id  -- 连接发布动作表和内容信息表
GROUP BY 
    a.author_name  -- 按创作者姓名分组
ORDER BY 
    a.author_name;  -- 按创作者姓名排序
```



问题：计算各岗位平均投递转化率?

要求:1、输出job_name-岗位名称，job_type-岗位类型，crt-岗位平均投递转化率(以百分数形式输出并保留1位小数)
	 2、某岗位的平均投递转化率=总计投递简历量/总计岗位浏览量:

```java
SELECT 
    jd.job_name,  -- 岗位名称
    jd.job_type,  -- 岗位类型
    -- 计算转化率并格式化为百分数（保留1位小数）
    CONCAT(ROUND(SUM(ji.cv_nums) / SUM(ji.job_pv) * 100, 1), '%') AS job_crt
FROM 
    job_info ji
JOIN 
    job_det jd ON ji.job_id = jd.job_id  -- 连接两张表
GROUP BY 
    jd.job_name, jd.job_type;  -- 按岗位名称和类型分组
```

