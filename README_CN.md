# [.NET 7/8 �����ϵ��û����뿴����](https://github.com/yangzhongke/Zack.EFCore.Batch/blob/main/README_CN_NET7.md) 

# Zack.EFCore.Batch
ʹ�����������, Entity Framework Core �û�����ʹ��LINQ���ɾ�����߸��¶������ݿ��¼������ִֻ��һ��SQL��䲢�Ҳ���Ҫ���Ȱ�ʵ�������ص��ڴ��С� 
���������֧�� Entity Framework Core 5/6��  

## ����˵��:  
 ##### ��һ��

����.NET 5�û���
```
SQLServer: Install-Package Zack.EFCore.Batch.MSSQL
MySQL: Install-Package Zack.EFCore.Batch.MySQL.Pomelo
Postgresql: Install-Package Zack.EFCore.Batch.Npgsql
Sqlite: Install-Package Zack.EFCore.Batch.Sqlite
Oracle:Install-Package Zack.EFCore.Batch.Oracle
Dm(����): Install-Package ZackEFCore.Batch.Dm
In Memory(�ڴ����ݿ�)��Install-Package Zack.EFCore.Batch.InMemory
``` 
����.NET 6�û�:
```
SQLServer: Install-Package Zack.EFCore.Batch.MSSQL_NET6
MySQL: Install-Package Zack.EFCore.Batch.MySQL.Pomelo_NET6
Postgresql: Install-Package Zack.EFCore.Batch.Npgsql_NET6
Sqlite: Install-Package Zack.EFCore.Batch.Sqlite_NET6
Oracle: Install-Package Zack.EFCore.Batch.Oracle_NET6
In Memory(�ڴ����ݿ�)��Install-Package Zack.EFCore.Batch.InMemory_NET6
```
MySQL֧��Pomelo.EntityFrameworkCore.MySql���EF Core Provider����֧��MySQL�ٷ�EF Core Provider��


 ##### �ڶ���:
���ݲ�ͬ�����ݿ⣬��ֱ�����´�����ӵ����DbContext���OnConfiguring�����У�
```csharp
optionsBuilder.UseBatchEF_MSSQL();// MSSQL Server �û������
optionsBuilder.UseBatchEF_Npgsql();//Postgresql �û������
optionsBuilder.UseBatchEF_MySQLPomelo();//MySQL �û������
optionsBuilder.UseBatchEF_Sqlite();//Sqlite �û������
optionsBuilder.UseBatchEF_Oracle();//Oracle �û������
optionsBuilder.UseBatchEF_DM();//DM(����) �û������
optionsBuilder.UseBatchEF_InMemory();//In Memory(�ڴ����ݿ�) �û������
```

 ##### ������:
ʹ��DbContext����չ����DeleteRangeAsync()��ɾ��һ������.
DeleteRangeAsync()�Ĳ������ǹ���������lambda���ʽ��
���Ӵ���:
```csharp
await ctx.DeleteRangeAsync<Book>(b => b.Price > n || b.AuthorName == "zack yang"); 
```

����Ĵ��뽫�������ݿ���ִ������SQL��䣺
```SQL
Delete FROM [T_Books] WHERE ([Price] > @__p_0) OR ([AuthorName] = @__s_1)
```

DeleteRange()������DeleteRangeAsync()��ͬ�������汾��

ʹ��DbContext����չ����BatchUpdate()������һ��BatchUpdateBuilder����
BatchUpdateBuilder���������ĸ�������
* Set()�������ڸ�һ�����Ը�ֵ�������ĵ�һ�����������Ե�lambda���ʽ,�ڶ���������ֵ��lambda���ʽ��
* Where() �ǹ�������
* ExecuteAsync()ʹ������ִ��BatchUpdateBuilder���첽����,Execute()��ExecuteAsync()��ͬ�������汾��

 ���Ӵ���:
```csharp
await ctx.BatchUpdate<Book>()
    .Set(b => b.Price, b => b.Price + 3)
    .Set(b => b.Title, b => s)
    .Set(b=>b.AuthorName,b=>b.Title.Substring(3,2)+b.AuthorName.ToUpper())
    .Set(b => b.PubTime, DateTime.Now)
    .Where(b => b.Id > n || b.AuthorName.StartsWith("Zack"))
    .ExecuteAsync();
```

����Ĵ��뽫�������ݿ���ִ������SQL��䣺
```SQL
Update [T_Books] SET [Price] = [Price] + 3.0E0, [Title] = @__s_1, [AuthorName] = COALESCE(SUBSTRING([Title], 3 + 1, 2), N'') + COALESCE(UPPER([AuthorName]), N''), [PubTime] = GETDATE()
WHERE ([Id] > @__p_0) OR ([AuthorName] IS NOT NULL AND ([AuthorName] LIKE N'Zack%'))
```


## Take(), Skip()
Take() and Skip() ������������DeleteRangeAsync �� BatchUpdateӰ���������
```CSharp
await ctx.Comments.Where(c => c.Article.Id == id).OrderBy(c => c.Message)
	.Skip(3).DeleteRangeAsync<Comment>(ctx);
await ctx.Comments.Where(c => c.Article.Id == id).Skip(3).Take(10).DeleteRangeAsync<Comment>(ctx);
await ctx.Comments.Where(c => c.Article.Id == id).Take(10).DeleteRangeAsync<Comment>(ctx);

await ctx.BatchUpdate<Comment>().Set(c => c.Message, c => c.Message + "abc")
	.Where(c => c.Article.Id == id)
	.Skip(3)
	.ExecuteAsync();

await ctx.BatchUpdate<Comment>()
	.Set(c => c.Message, c => "abc")
	.Where(c => c.Article.Id == id)
	.ExecuteAsync();
	
await ctx.BatchUpdate<Comment>()
	.Set("Message","abc")
	.Where(c => c.Article.Id == id)
	.ExecuteAsync();

await ctx.BatchUpdate<Comment>().Set(c => c.Message, c => c.Message + "abc")
	.Where(c => c.Article.Id == id)
	.Skip(3)
	.Take(10)
	.ExecuteAsync();
await ctx.BatchUpdate<Comment>().Set(c => c.Message, c => c.Message + "abc")
   .Where(c => c.Article.Id == id)
   .Take(10)
   .ExecuteAsync();
```

## BulkInsert��������

Ŀǰ�����������ݲ�֧��SQLite��
```
List<Book> books = new List<Book>();
for (int i = 0; i < 100; i++)
{
	books.Add(new Book { AuthorName = "abc" + i, Price = new Random().NextDouble(), PubTime = DateTime.Now, Title = Guid.NewGuid().ToString() });
}
using (TestDbContext ctx = new TestDbContext())
{
	ctx.BulkInsert(books);
}
```
�� mysql��, ���ʹ��BulkInsert�����ڷ������˺Ϳͻ��˶�����local_infile����mysql server������������"local_infile=ON"��Ȼ���������ַ�������� "AllowLoadLocalInfile=true"��

## ����˵��

���������ʹ��EF Coreʵ�ֵ�lambda���ʽ��SQL���ķ��룬���Լ�������EF Core֧�ֵ�lambda���ʽд������֧�֡�

�������ݿ��Ѿ������ԣ����Ա�Zack.EFCore.Batch֧��: MS SQLServer(Microsoft.EntityFrameworkCore.SqlServer), MySQL(Pomelo.EntityFrameworkCore.MySql), PostgreSQL(Npgsql.EntityFrameworkCore.PostgreSQL), Oracle(Oracle.EntityFrameworkCore)��

��������˵��ֻҪһ�����ݿ��ж�Ӧ��EF Core 5/6��Provider����ôZack.EFCore.Batch�Ϳ���֧��������ݿ⡣�����ʹ�õ����ݿ�Ŀǰ���ڱ�֧�ֵķ�Χ�ڣ����ύIssue����һ�������һ���������ڿ�����ɡ�


[���������Ŀ������棨Bվ��](https://www.bilibili.com/read/cv8545714)  

[���������Ŀ������棨����ͷ����](https://www.toutiao.com/i6899423396355293708/)  
