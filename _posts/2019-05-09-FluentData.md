# 无需废话，直接上码
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
> List 类型参数
>> 请注意，不要在（…）语法中留下任何空格.
```
List<int> ids = new List<int>() { 1, 2, 3, 4 };

dynamic products = Context.Sql(@"select * from Product
			where ProductId in(@0)", ids).QueryMany<dynamic>();
```
> like operator:
```
string cens = "%abc%";
Context.Sql("select * from Product where ProductName like @0",cens);
```
> Mapping Automapping - 1:1 match between the database and the .NET object:
```
List<Product> products = Context.Sql(@"select *
			from Product")
			.QueryMany<Product>();
```
> Automap to a custom collection:
```
ProductionCollection products = Context.Sql("select * from Product").QueryMany<Product, ProductionCollection>();
```
> Automapping - Mismatch between the database and the .NET object, use the alias keyword in SQL: Weakly typed:
```
List<Product> products = Context.Sql(@"select p.*,
			c.CategoryId as Category_CategoryId,
			c.Name as Category_Name
			from Product p
			inner join Category c on p.CategoryId = c.CategoryId")
				.QueryMany<Product>();
```
> Here the p.* which is ProductId and Name would be automapped to the properties Product.Name and Product.ProductId, and Category_CategoryId and Category_Name would be automapped to Product.Category.CategoryId and Product.Category.Name. Custom mapping using dynamic:
```
List<Product> products = Context.Sql(@"select * from Product")
			.QueryMany<Product>(Custom_mapper_using_dynamic);
public void Custom_mapper_using_dynamic(Product product, dynamic row)
{
	product.ProductId = row.ProductId;
	product.Name = row.Name;
}
```
> Custom mapping using a datareader:
```
List<Product> products = Context.Sql(@"select * from Product")
			.QueryMany<Product>(Custom_mapper_using_datareader);
public void Custom_mapper_using_datareader(Product product, IDataReader row)
{
	product.ProductId = row.GetInt32("ProductId");
	product.Name = row.GetString("Name");
}
```
> Or if you have a complex entity type where you need to control how it is created then the QueryComplexMany/QueryComplexSingle can be used: var products = new List<Product>();
```
Context.Sql("select * from Product").QueryComplexMany<Product>(products, MapComplexProduct);
private void MapComplexProduct(IList<Product> products, IDataReader reader)
{
	var product = new Product();
	product.ProductId = reader.GetInt32("ProductId");
	product.Name = reader.GetString("Name");
	products.Add(product);
}
```
> Multiple result sets FluentData supports multiple resultsets. This allows you to do multiple queries in a single database call. When this feature is used it's important to wrap the code inside a using statement as shown below in order to make sure that the database connection is closed.
```
using (var command = Context.MultiResultSql)
{
	List<Category> categories = command.Sql(
			@"select * from Category;
			select * from Product;").QueryMany<Category>();

	List<Product> products = command.QueryMany<Product>();
}
```
> The first time the Query method is called it does a single query against the database. The second time the Query is called, FluentData already knows that it's running in a multiple result set mode, so it reuses the data retrieved from the first query. Select data and Paging A select builder exists to make selecting data and paging easy:
```
List<Product> products = Context.Select<Product>("p.*, c.Name as Category_Name")
			       .From(@"Product p 
					inner join Category c on c.CategoryId = p.CategoryId")
			       .Where("p.ProductId > 0 and p.Name is not null")
			       .OrderBy("p.Name")
			       .Paging(1, 10).QueryMany();
```
> By calling Paging(1, 10) then the first 10 products will be returned. Insert data Using SQL:
```
int productId = Context.Sql(@"insert into Product(Name, CategoryId)
			values(@0, @1);")
			.Parameters("The Warren Buffet Way", 1)
			.ExecuteReturnLastId<int>();
```
> Using a builder:
```
int productId = Context.Insert("Product")
			.Column("Name", "The Warren Buffet Way")
			.Column("CategoryId", 1)
			.ExecuteReturnLastId<int>();
```
> Using a builder with automapping:
```
Product product = new Product();
product.Name = "The Warren Buffet Way";
product.CategoryId = 1;
product.ProductId = Context.Insert<Product>("Product", product)
			.AutoMap(x => x.ProductId)
			.ExecuteReturnLastId<int>();
```
> We send in ProductId to the AutoMap method to get AutoMap to ignore and not map the ProductId since this property is an identity field where the value is generated in the database. Update data Using SQL:
```
int rowsAffected = Context.Sql(@"update Product set Name = @0
			where ProductId = @1")
			.Parameters("The Warren Buffet Way", 1)
			.Execute();
```
> Using a builder:
```
int rowsAffected = Context.Update("Product")
			.Column("Name", "The Warren Buffet Way")
			.Where("ProductId", 1)
			.Execute();
```
> Using a builder with automapping:
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
> We send in ProductId to the AutoMap method to get AutoMap to ignore and not map the ProductId since this is the identity field that should not get updated. IgnoreIfAutoMapFails When read from database,If some data columns not mappinged with entity class,by default ,will throw exception. if you want ignore the exception, or the property not used for map data table,then you can use the IgnoreIfAutoMapFails(true),this will ignore the exception when read mapping error. context.IgnoreIfAutoMapFails(true); Insert and update - common Fill method
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
> Delete data Using SQL:
```
int rowsAffected = Context.Sql(@"delete from Product
			where ProductId = 1")
			.Execute();
```
> Using a builder:
```
int rowsAffected = Context.Delete("Product")
			.Where("ProductId", 1)
			.Execute();
```
> Stored procedure Using SQL:
```
var rowsAffected = Context.Sql("ProductUpdate")
			.CommandType(DbCommandTypes.StoredProcedure)
			.Parameter("ProductId", 1)
			.Parameter("Name", "The Warren Buffet Way")
			.Execute();
```
> Using a builder:
```
var rowsAffected = Context.StoredProcedure("ProductUpdate")
			.Parameter("Name", "The Warren Buffet Way")
			.Parameter("ProductId", 1).Execute();
```
> Using a builder with automapping:
```
var product = Context.Sql("select * from Product where ProductId = 1")
			.QuerySingle<Product>();

product.Name = "The Warren Buffet Way";

var rowsAffected = Context.StoredProcedure<Product>("ProductUpdate", product)
			.AutoMap(x => x.CategoryId).Execute();
```
> Using a builder with automapping and expressions:
```
var product = Context.Sql("select * from Product where ProductId = 1")
			.QuerySingle<Product>();
product.Name = "The Warren Buffet Way";

var rowsAffected = Context.StoredProcedure<Product>("ProductUpdate", product)
			.Parameter(x => x.ProductId)
			.Parameter(x => x.Name).Execute();
```
> Transactions FluentData supports transactions. When you use transactions its important to wrap the code inside a using statement to make sure that the database connection is closed. By default, if any exception occur or if Commit is not called then Rollback will automatically be called.
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
> Entity factory The entity factory is responsible for creating object instances during automapping. If you have some complex business objects that require special actions during creation, you can create your own custom entity factory:
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