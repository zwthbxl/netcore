# Zack.EFCore.Batch
.NET 7��ʼ��EF Core�Ѿ������˶�����ɾ�����������µ�֧�֣���˱���Ŀ��������.NET7���Ժ�İ汾��֧�����������ܣ�[����������](https://learn.microsoft.com/zh-cn/ef/core/what-is-new/ef-core-7.0/whatsnew?WT.mc_id=DT-MVP-5004444#executeupdate-and-executedelete-bulk-updates)�������Ǳ���Ŀ��Ȼ��.NET 7���Ժ�İ汾��֧�����ݵ��������롣

ʹ�����������, Entity Framework Core �û����Կ��������������ݡ�
���������֧�� Entity Framework Core 7/8�����ϰ汾�� 

## Ϊʲô����������ܣ�

Entity Framework Core�п���ͨ��AddRange()�����������������ݣ�����AddRange()��ӵ�������Ȼ�Ǳ�����ִ��Insert��������뵽���ݿ��еģ�ִ��Ч�ʱȽϵ͡�����֪�������ǿ���ͨ��SqlBulkCopy�����ٵز�����������ݵ�SQLServer���ݿ⣬��ΪSqlBulkCopy�ǰѶ������ݴ��һ�����ݰ����͵�SQLServer�ģ����Բ���Ч�ʷǳ��ߡ�MySQL��PostgreSQL��Oracle��Ҳ�����Ƶ�֧�֡�

��Ȼ��ֱ��ʹ��SqlBulkCopy���������ݲ�����Ҫ����Ա��������䵽DataTable��������Ҫ�����е�ӳ��Ȳ���������Ҫ����ValueConverter�����⣬�������Ƚ��鷳������Ҷ���Щ���ܷ�װ���Ӷ���EF Core�Ŀ������ܹ��������������ģ�͵ķ�ʽ���������ݡ�

## ���ܶԱ�

����SQLServer���ݿ�����һ�²���10�������ݵĲ��ԣ���AddRange�����ʱԼ21�룬�����������Դ��Ŀ���в����ʱֻ��Լ5�롣

## ����˵��:  

### ��װNuget����

.NET 7

```
SQLServer: Install-Package Zack.EFCore.Batch.MSSQL_NET7
MySQL: Install-Package Zack.EFCore.Batch.MySQL.Pomelo_NET7
Postgresql: Install-Package Zack.EFCore.Batch.Npgsql_NET7
Oracle: Install-Package Zack.EFCore.Batch.Oracle_NET7 
```

.NET 8

```
SQLServer: Install-Package Zack.EFCore.Batch.MSSQL_NET8
MySQL: Install-Package Zack.EFCore.Batch.MySQL.Pomelo_NET8
Postgresql: Install-Package Zack.EFCore.Batch.Npgsql_NET8
```

### ������������
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
