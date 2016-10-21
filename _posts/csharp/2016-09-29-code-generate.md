---
layout: z
title: "T4 template生成实体和dapper为基础的数据访问方法"
category: "CSharp"
---

<b>[T4 template](http://msdn.microsoft.com/zh-cn/library/bb126445 "T4 template")</b>是一个文本生成引擎，可以生成cs文件。
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
从数据库系统视图里面查询出来的表、视图结构遍历生成实体时会涉及到一个数据库类型和
c sharp类型的映射，这里MySql和SQLServer的类型映射也有部分的类型不一样。

    public class DataType
    {
        private Hashtable _mySqlDataType;
        public Hashtable MySqlDataType
        {
            get
            {
                _mySqlDataType = new Hashtable();
                _mySqlDataType.Add("bit", "bool");
                _mySqlDataType.Add("tinyint", "sbyte");
                _mySqlDataType.Add("smallint", "short");
                _mySqlDataType.Add("mediumint", "int");
                _mySqlDataType.Add("int", "int");
                _mySqlDataType.Add("integer", "int");
                _mySqlDataType.Add("bigint", "long");
                _mySqlDataType.Add("real", "double");
                _mySqlDataType.Add("double", "double");
                _mySqlDataType.Add("float", "float");
                _mySqlDataType.Add("decimal", "decimal");
                _mySqlDataType.Add("numeric", "decimal");
                _mySqlDataType.Add("char", "string");
                _mySqlDataType.Add("varchar", "string");
                _mySqlDataType.Add("binary", "byte[]");
                _mySqlDataType.Add("varbinary", "byte[]");
                _mySqlDataType.Add("date", "DateTime");
                _mySqlDataType.Add("time", "TimeSpan");
                _mySqlDataType.Add("datetime", "DateTime");
                _mySqlDataType.Add("timestamp", "DateTime");
                _mySqlDataType.Add("year", "short");
                _mySqlDataType.Add("tinyblob", "byte[]");
                _mySqlDataType.Add("blob", "byte[]");
                _mySqlDataType.Add("longblob", "byte[]");
                _mySqlDataType.Add("mediumblob", "byte[]");
                _mySqlDataType.Add("tinytext", "string");
                _mySqlDataType.Add("text", "string");
                _mySqlDataType.Add("mediumtext", "string");
                return _mySqlDataType;
            }
        }

        private Hashtable _sqlDataType;

        public Hashtable SqlDataType
        {
            get
            {
                _sqlDataType = new Hashtable();
                #region
                _sqlDataType = new Hashtable();
                _sqlDataType.Add("bigint", "long");
                _sqlDataType.Add("binary", "byte[]");
                _sqlDataType.Add("bit", "bool");
                _sqlDataType.Add("char", "string");
                _sqlDataType.Add("date", "DateTime");
                _sqlDataType.Add("datetime", "DateTime");
                _sqlDataType.Add("datetime2", "DateTime");
                _sqlDataType.Add("datetimeoffset", "DateTimeOffset");
                _sqlDataType.Add("decimal", "decimal");
                _sqlDataType.Add("float", "double");
                _sqlDataType.Add("image", "byte[]");
                _sqlDataType.Add("int", "int");
                _sqlDataType.Add("money", "decimal");
                _sqlDataType.Add("nchar", "string");
                _sqlDataType.Add("ntext", "string");
                _sqlDataType.Add("numeric", "decimal");
                _sqlDataType.Add("nvarchar", "string");
                _sqlDataType.Add("real", "float");
                _sqlDataType.Add("smalldatetime", "DateTime");
                _sqlDataType.Add("smallint", "short");
                _sqlDataType.Add("smallmoney", "decimal");
                _sqlDataType.Add("text", "string");
                _sqlDataType.Add("time", "TimeSpan");
                _sqlDataType.Add("timespan", "byte[]");
                _sqlDataType.Add("tinyint", "byte");
                _sqlDataType.Add("uniqueidentifier", "Guid");
                _sqlDataType.Add("varbinary", "byte[]");
                _sqlDataType.Add("varchar", "string");
                _sqlDataType.Add("xml", "string");
                return _sqlDataType;
                #endregion
            }
        }
    }
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


详细代码看<b>[这里](https://github.com/lauriezc/dapper-demo)</b>
