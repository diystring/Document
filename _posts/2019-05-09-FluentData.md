## 快速上手如何使用FluentData
> 创建并且初始化一个IDbContext. 
```
public static IDbContext QueryDB()
{
    return new DbContext().ConnectionStringName("DBContext",DbProviderTypes.SqlServer);
}
```
>config中的连接字符串实例
```
<connectionStrings>
    <add name="DBContext"connectionString="server=192.168.1.100;uid=sa;pwd=sa;database=demoDB;" />
</connectionStrings>
```
>那么下面就可以在我们的数据业务层中根据自己的需求随心所欲的写sql了。 

>1.需要返回一个实体： 
```
Product product = QueryDB().Sql("select * from Product where ProductId = 1").QuerySingle<Product>()
```

>2.根据参数返回一个实体
```
Product product = QueryDB().Sql("select * from Product where id=@id")
                .Parameter("id", id)
                .QuerySingle<Product>()
```
>3.返回一个泛型。 
```
List<Product> product = QueryDB().Sql("select * from Product where id=@id")
                .Parameter("id", id)
                .Query<Product>()
```
>4.多表支持
```
using (var command = QueryDB().MultiResultSql())
{
    List<Category> categories = command.Sql(
            @"select * from Category;
            select * from Product;").Query<Category>();
    List<Product> products = command.Query<Product>();
}
```
>5.插入操作
```
var productId = QueryDB().Insert("Product")
                .Column("Name", "The Warren Buffet Way")
                .Column("CategoryId", 1)
                .ExecuteReturnLastId()
```
>6.直接执行sql
```
var productId = QueryDB().Sql("insert into Product(Name, CategoryId)
                    values('TT', 1);").ExecuteReturnLastId()
```
>7.修改操作
```
QueryDB().Update("Product")
        .Column("Name", "TT")
        .Column("CategoryId", 1)
        .Where("ProductId", 1)
        .Execute()
```
>8.删除操作 
```
QueryDB().Delete("Product").Where("ProductId", 1).Execute();
```
>9.朗姆达表达式
```
QueryDB().Delete<Product>("Product")
     .Where(x=>x.id,id)
     .Execute()
```
>10.事物的处理 
```
using (var context = QueryDB().UseTransaction)
{
    context.Sql("update Product set Name = @0 where ProductId = @1")
                .Parameters("TT", 1)
                .Execute();
    context.Sql("update Product set Name = @0 where ProductId = @1")
                .Parameters("TT", 2)
                .Execute();
    context.Commit();
}
```
##### 在事物的操作中记得context.Commit();方法的执行，楼主曾经在自己的一个项目中需要用到事物，却忘记了执行提交这个方法，最后在源码的汪 洋中探索许久 
>11.存储过程带分页
```
public static List<T> PageQuery<T>(string tableName,string tableFields, string sqlWhere,string order,intpageIndex, int pageSize, out int total)
{
   var store = QueryDB().StoredProcedure("Pro_Sys_Page")
                .ParameterOut("totalPage", DataTypes.Int16)
                .Parameter("tableName", tableName)
                .Parameter("tableFields", tableFields)
                .Parameter("sqlWhere", sqlWhere)
                .Parameter("orderFields", order)
                .Parameter("pageSize", pageSize)
                .Parameter("pageIndex", pageIndex);
    var result=store.Query<T>()
}
```
# 此致敬礼