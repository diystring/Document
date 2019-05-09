## 快速上手如何使用FluentData
> 创建并且初始化一个IDbContext. 
### 它是我们与数据库操作中的上下文，所有的有关数据操作都调用它下面的方法。初始化它的连接字符串web.config 
```
public static IDbContext QueryDB()
{
   return new DbContext().ConnectionStringName("testDBContext", DbProviderTypes.SqlServer);
}
```
>config中的连接字符串实例
```
<connectionStrings>
    <add name="testDBContext"connectionString="server=192.168.1.100;uid=sa;pwd=sa!;database=testDB;" />
</connectionStrings>
```
>那么下面就可以在我们的数据业务层中根据自己的需求随心所欲的写sql了。 
#
>1.需要返回一个实体： 

Product product = QueryDB().Sql(@"select * from Product where ProductId = 1").QuerySingle<Product>()

>2.根据参数返回一个实体？别急，尝尝那飘渺的链式操作吧 
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

 
>4.多表支持(这个楼主实际工作中倒是没有用到过) 
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
>6.当然我喜欢写我牛B的sql。 
```
var productId = QueryDB().Sql(@"insert into Product(Name, CategoryId)
                    values(\‘The Warren Buffet Way\‘, 1);").ExecuteReturnLastId()
```
>7.修改操作. 
```
QueryDB().Update("Product")
        .Column("Name", "The Warren Buffet Way")
        .Column("CategoryId", 1)
        .Where("ProductId", 1)
        .Execute()
```
    同上，也可以不用update（）方法，而直接写sql. 


>8.删除操作 
```
QueryDB().Delete("Product").Where("ProductId", 1).Execute();
```
>9.我想链式操作，我想写lambda表达式OK。 
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
                .Parameters("The Warren Buffet Way", 1)
                .Execute();
    context.Sql("update Product set Name = @0 where ProductId = @1")
                .Parameters("Bill Gates Bio", 2)
                .Execute();
    context.Commit();
}
```
##### 在事物的操作中记得context.Commit();方法的执行，楼主曾经在自己的一个项目中需要用到事物，却忘记了执行提交这个方法，最后在源码的汪 洋中探索许久 
>11.存储过程 
有关存储过程的使用，楼主在实际项目开发中，用上了存储过程。该存储过程的作用是分页，那么这里也贴出来分享一下 
```
public static List<T> getPage<T>(string tableName,string tableFields, string sqlWhere,string order,intpageIndex, int pageSize, out int total)
{
   var store = QueryDB().StoredProcedure("PF_Sys_PageControl")
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