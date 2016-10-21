---
layout: z
title: "T4 template生成实体和dapper为基础的数据访问方法"
category: "CSharp"
---

[T4 template](http://msdn.microsoft.com/zh-cn/library/bb126445 "T4 template")是一个文本生成引擎，可以生成cs文件。
用文本生成引擎生成代码，可以节省部分编码时间，缺点是不够灵活。

##### 数据库中的数据结构
数据库中的表、视图、存储过程、函数等结构能通过查询知道，这次写的demo中只生成了表和视图相关的实体和方法。要生成数据访问方法首先要从数据库中查询出表和视图数据结构，MySql中关于数据库数据结构的数据都在information_schema数据库里面，通过查询里面的COLUMNS视图可以查询到数据库所有表和视图的字段定义（名称，类型，长度，是否自增等）与TABLES视图联合查询即可查询出每个表的所有字段的定义，用于生成相应的数据映射实体。
SQLServer每个数据库里面都有相对应的table和column视图，也是联合查询，只是SQLServer里面的有些字段的约束不是放在columns视图内，例如主键约束。
MySql示例

    SELECT 
    c.`TABLE_SCHEMA`,c.`TABLE_NAME`,c.`COLUMN_NAME`,c.`ORDINAL_POSITION`,c.`COLUMN_DEFAULT`,c.`IS_NULLABLE`,c.`DATA_TYPE`,c.`CHARACTER_MAXIMUM_LENGTH`,c.`CHARACTER_OCTET_LENGTH`,c.`NUMERIC_PRECISION`,c.`NUMERIC_SCALE`,c.`DATETIME_PRECISION`,c.`CHARACTER_SET_NAME`,c.`COLUMN_KEY`,c.`EXTRA`,t.TABLE_TYPE 
    from 
    information_schema.`COLUMNS` c 
    LEFT JOIN
    information_schema.`TABLES` t 
    ON 
    t.TABLE_NAME=c.TABLE_NAME  
    WHERE c.`TABLE_SCHEMA`='{0}'



##### 代码的生成
demo里面主要生成的代码是实体和sql命令以及调用dapper库执行sql返回结果的代码。
这里数据表和视图有区别，数据表有增删查改，视图只能查询，而且数据表需要有自增或者主键才能执行update和delete，这里采用的是EntityFramework的db first里面不生成没有主键的数据表对应的代码的机制。
如分页方法：

    public static Pager GetPageList(int pageIndex, int pageSize, string where="1=1", string orderField="id DESC")
    {
            string sql=string.Format("SELECT TOP {0} * FROM [dbo].[all_t] WHERE {1} AND id NOT IN (SELECT TOP {2} id FROM [dbo].[all_t] WHERE {1}) ORDER BY {3};", pageSize, where, (pageIndex-1)*pageSize, orderField);
            string countSql=string.Format("SELECT COUNT(0) FROM [dbo].[all_t] WHERE {0};", where);
            using(var con = new SqlConnection(_connectionString))
            {
                try {
                    var count = con.ExecuteScalar<int>(countSql);
                    var list = con.Query<all_t>(sql).AsList<all_t>();
                    return new Pager()
                    {
                        PageIndex = pageIndex,
                        PageSize = pageSize,
                        Total = count,
                        Data = list
                    };
                }
                catch(Exception e)
                {
                    if(con.State != ConnectionState.Closed)
                    {
                        con.Close();
                    }
                    return null;
                }
            }
        }
    } 


详细代码看[这里](https://github.com/lauriezc/dapper-demo)
