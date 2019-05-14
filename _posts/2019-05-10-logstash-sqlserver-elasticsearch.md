# 通过Logstash由SQLServer向Elasticsearch同步数据

> 延用上篇ELK所需环境，新增logstash配置文件
>>* 需要数据库链接驱动 Microsoft JDBC driver 6.2 for SQL Server https://www.microsoft.com/zh-CN/download/details.aspx?id=55539
---
# 主要配置文件详解

#
```
input {
    jdbc {
     jdbc_driver_library => "D:\ELK_logs\logstash-6.3.2\bin\jdbcconfig\mssql-jdbc-6.2.2.jre8.jar"
            jdbc_driver_class => "com.microsoft.sqlserver.jdbc.SQLServerDriver"
            jdbc_connection_string => "jdbc:sqlserver://192.168.100.51:1433;DatabaseName=BTPreservation;"
            jdbc_user => "sa"
            jdbc_password => "Rl123456"
                        # schedule => 分 时 天 月 年  
                        # schedule => * 22  *  *  *     //will execute at 22:00 every day
            schedule => "* * * * *"
            jdbc_paging_enabled => true
            jdbc_page_size => 1000
            clean_run => false
            use_column_value => true
            #设置查询条件的字段
            tracking_column => FID
            record_last_run => true
            last_run_metadata_path => "D:\ELK_logs\logstash-6.3.2\bin\jdbcconfig\FID.txt"
            #设置列名小写
            lowercase_column_names => false
            statement_filepath => "D:\ELK_logs\logstash-6.3.2\bin\jdbcconfig\x_Loan_PreservationAdvanceList.sql"
            #索引的类型
            type => "advancelist"
    }
}

output {
    elasticsearch {
        hosts => ["192.168.100.50:9200"]
        index => "advancelist"
        document_id => "%{FID}"
    }
    stdout {
        #codec => json_lines
        #设置输出的格式
        codec => line {
            format => "FID: %{[FID]} FPersonName: %{[FPERSONNAME]} FAddTime: %{[FADDTIME]}"
        }
    }
}
```
> 注意：启动时因为是同台机器运行多个logstash实例，所以需要指定不同的数据存储目录 path.Data 