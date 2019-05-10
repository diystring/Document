# 最全Fluentdata详解， 无需废话，直接上码
> 创建并且初始化一个IDbContext. 二选一
```
public IDbContext Context()
{
	return new DbContext().ConnectionStringName("MyDatabase",
			new SqlServerProvider());
}
public IDbContext Context()
{
	return new DbContext().ConnectionString(
	"Server=MyServerAddress;Database=MyDatabase;Trusted_Connection=True;", new SqlServerProvider());
}
```

> 返回 dynamic类型的List 集合:
```
List<dynamic> products = Context.Sql("select * from Product").QueryMany<dynamic>();
```
> 返回强类型的List 集合
```
List<Product> products = Context.Sql("select * from Product").QueryMany<Product>();
```
> 返回一个自定义类型的集合类型:
```
ProductionCollection products = Context.Sql("select * from Product").QueryMany<Product, ProductionCollection>();
```
> 返回单个动态对象数据表
```
dynamic product = Context.Sql(@"select * from Product
				where ProductId = 1").QuerySingle<dynamic>();
```
> 返回一个强类型对象
```
Product product = Context.Sql(@"select * from Product
			where ProductId = 1").QuerySingle<Product>();
```
> 返回一个DataTable对象，QueryMany< DataTable > and QuerySingle<DataTable> 都可以返回DataTable, 只不过QuerMany返回的是 List< DataTable >
```
DataTable products = Context.Sql("select * from Product").QuerySingle<DataTable>();
```
> 返回一个 int 类型
```
int numberOfProducts = Context.Sql(@"select count(*)
			from Product").QuerySingle<int>();
```
> 返回一个List< int >
```
List<int> productIds = Context.Sql(@"select ProductId
				from Product").QueryMany<int>();
```
> 索引传参SQL
```
dynamic products = Context.Sql(@"select * from Product
			where ProductId = @0 or ProductId = @1", 1, 2).QueryMany<dynamic>();
or:
dynamic products = Context.Sql(@"select * from Product
			where ProductId = @0 or ProductId = @1")
			.Parameters(1, 2).QueryMany<dynamic>();
```
> 参数名传参
```
dynamic products = Context.Sql(@"select * from Product
			where ProductId = @ProductId1 or ProductId = @ProductId2")
			.Parameter("ProductId1", 1)
			.Parameter("ProductId2", 2)
			.QueryMany<dynamic>();
```
> 输出参数
```
var command = Context.Sql(@"select @ProductName = Name from Product
			where ProductId=1")
			.ParameterOut("ProductName", DataTypes.String, 100);
    command.Execute();
string productName = command.ParameterValue<string>("ProductName");
```
> List 类型参数,请注意，不要在（…）语法中留下任何空格
```
List<int> ids = new List<int>() { 1, 2, 3, 4 };
dynamic products = Context.Sql(@"select * from Product where ProductId in(@0)", ids).QueryMany<dynamic>();
```
> like 模糊查询:
```
string cens = "%abc%";
Context.Sql("select * from Product where ProductName like @0",cens);
```
> 实体集合自动映射
```
List<Product> products = Context.Sql(@"select *
			from Product")
			.QueryMany<Product>();
```
> 自定义映射对象
```
ProductionCollection products = Context.Sql("select * from Product").QueryMany<Product, ProductionCollection>();
```
> 也可以将链表查询结果集映射到自定义对象集合
```
List<Product> products = Context.Sql(@"select p.*,
			c.CategoryId as Category_CategoryId,
			c.Name as Category_Name
			from Product p
			inner join Category c on p.CategoryId = c.CategoryId")
				.QueryMany<Product>();
```
> 使用动态的自定义映射
```
List<Product> products = Context.Sql(@"select * from Product")
			.QueryMany<Product>(Custom_mapper_using_dynamic);
public void Custom_mapper_using_dynamic(Product product, dynamic row)
{
	product.ProductId = row.ProductId;
	product.Name = row.Name;
}
```
> 基于 datareader 的动态的自定义映射
```
List<Product> products = Context.Sql(@"select * from Product")
			.QueryMany<Product>(Custom_mapper_using_datareader);
public void Custom_mapper_using_datareader(Product product, IDataReader row)
{
	product.ProductId = row.GetInt32("ProductId");
	product.Name = row.GetString("Name");
}
```
>如果你有一个复杂的实体类型需要控制它的创建方式，那么可以使用 QueryComplexMany/QueryComplexSingle 
```
var products = new List<Product>();
Context.Sql("select * from Product").QueryComplexMany<Product>(products, MapComplexProduct);
private void MapComplexProduct(IList<Product> products, IDataReader reader)
{
	var product = new Product();
	product.ProductId = reader.GetInt32("ProductId");
	product.Name = reader.GetString("Name");
	products.Add(product);
}
```
>支持多个查询结果表映射成一个实体对象集，且单次链接中执行多次查询
```
using (var command = Context.MultiResultSql)
{
	List<Category> categories = command.Sql(
			@"select * from Category;
			  select * from Product;").QueryMany<Category>();
	List<Product> products = command.QueryMany<Product>();
}
```
> 链表查询并支持分页
```
List<Product> products = Context.Select<Product>("p.*, c.Name as Category_Name")
			       .From(@"Product p 
					inner join Category c on c.CategoryId = p.CategoryId")
			       .Where("p.ProductId > 0 and p.Name is not null")
			       .OrderBy("p.Name")
			       .Paging(1, 10).QueryMany();
```
> 插入数据，返回自增ID
```
int productId = Context.Sql(@"insert into Product(Name, CategoryId)
			values(@0, @1);")
			.Parameters("The Warren Buffet Way", 1)
			.ExecuteReturnLastId<int>();
int productId = Context.Insert("Product")
			.Column("Name", "The Warren Buffet Way")
			.Column("CategoryId", 1)
			.ExecuteReturnLastId<int>();
```
> 使用自动应用的生成器
```
Product product = new Product();
product.Name = "The Warren Buffet Way";
product.CategoryId = 1;
product.ProductId = Context.Insert<Product>("Product", product)
			.AutoMap(x => x.ProductId)
			.ExecuteReturnLastId<int>();
```
> 更新操作，返回受影响行数
```
int rowsAffected = Context.Sql(@"update Product set Name = @0
			where ProductId = @1")
			.Parameters("The Warren Buffet Way", 1)
			.Execute();
int rowsAffected = Context.Update("Product")
			.Column("Name", "The Warren Buffet Way")
			.Where("ProductId", 1)
			.Execute();
```
> 使用自定义映射来更新实体
```
Product product = Context.Sql(@"select * from Product
			where ProductId = 1")
			.QuerySingle<Product>();
product.Name = "The Warren Buffet Way";
int rowsAffected = Context.Update<Product>("Product", product)
			.AutoMap(x => x.ProductId)
			.Where(x => x.ProductId)
			.Execute();
```
> 自定义插入或更新列操作
```
var product = new Product();
product.Name = "The Warren Buffet Way";
product.CategoryId = 1;
var insertBuilder = Context.Insert<Product>("Product", product).Fill(FillBuilder);
var updateBuilder = Context.Update<Product>("Product", product).Fill(FillBuilder);
public void FillBuilder(IInsertUpdateBuilder<Product> builder)
{
	builder.Column(x => x.Name);
	builder.Column(x => x.CategoryId);
}
```
> 删除操作
```
int rowsAffected = Context.Sql(@"delete from Product
			where ProductId = 1")
			.Execute();
int rowsAffected = Context.Delete("Product")
			.Where("ProductId", 1)
			.Execute();
```
> 执行存储过程
```
var rowsAffected = Context.Sql("ProductUpdate")
			.CommandType(DbCommandTypes.StoredProcedure)
			.Parameter("ProductId", 1)
			.Parameter("Name", "The Warren Buffet Way")
			.Execute();
var rowsAffected = Context.StoredProcedure("ProductUpdate")
			.Parameter("Name", "The Warren Buffet Way")
			.Parameter("ProductId", 1).Execute();
```
> 骚操作1
```
var product = Context.Sql("select * from Product where ProductId = 1")
			.QuerySingle<Product>();
product.Name = "The Warren Buffet Way";
var rowsAffected = Context.StoredProcedure<Product>("ProductUpdate", product)
			.AutoMap(x => x.CategoryId).Execute();
```
> 骚操作2
```
var product = Context.Sql("select * from Product where ProductId = 1")
			.QuerySingle<Product>();
product.Name = "The Warren Buffet Way";
var rowsAffected = Context.StoredProcedure<Product>("ProductUpdate", product)
			.Parameter(x => x.ProductId)
			.Parameter(x => x.Name).Execute();
```
> 事务使用
```
using (var context = Context.UseTransaction(true))
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
> 在查询一个实体对象集时如果需要再创建实体时就进行一些特殊的操作可以自定义实体工厂来满足你的需求
```
List<Product> products = Context.EntityFactory(new CustomEntityFactory())
			.Sql("select * from Product")
			.QueryMany<Product>();
public class CustomEntityFactory : IEntityFactory
{
	public virtual object Resolve(Type type)
	{
		return Activator.CreateInstance(type);
	}
}
```

> 留记