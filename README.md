# DotNetCore.CAP.DM
DotNetCore.CAP.DM 是 CAP的扩展库！
# DM

DM 是一个开源的关系型数据库，你可以使用DM 来作为 CAP 消息的持久化。

## 配置
本次提交是基于DotNetCore.CAP 6.2.0 版本开发

要使用 DM 存储，你需要从 NuGet 安装以下扩展包：

```shell
Install-Package DotNetCore.CAP.DM
```

然后，你可以在 `Startup.cs` 的 `ConfigureServices` 方法中添加基于内存的配置项。

```csharp

public void ConfigureServices(IServiceCollection services)
{
    // ...

    services.AddCap(x =>
    {
        x.UseDM(opt=>{

        });
        // x.UseXXX ...
    });
}

```

### 配置项

NAME | DESCRIPTION | TYPE | DEFAULT
:---|:---|---|:---
TableNamePrefix | Cap表名前缀 | string | cap 
ConnectionString | 数据库连接字符串 | string | null

### 自定义表名称

你可以通过重写 `IStorageInitializer` 接口获取表名称的方法来做到这一点

示例代码：

```C#

public class DMTableInitializer : DMStorageInitializer
{
    public override string GetPublishedTableName()
    {
        //你的 发送消息表 名称
    }

    public override string GetReceivedTableName()
    {
        //你的 接收消息表 名称
    }
}
```
然后将你的实现注册到容器中

```
services.AddSingleton<IStorageInitializer,DMInitializer>();
```

## 使用事务发布消息

### ADO.NET 

```csharp

private readonly ICapPublisher _capBus;

using (var connection = new DMConnection(AppDbContext.ConnectionString))
{
    using (var transaction = connection.BeginTransaction(_capBus, autoCommit: false))
    {
        //your business code
        connection.Execute("insert into test(name) values('test')", 
            transaction: (IDbTransaction)transaction.DbTransaction);
        
        _capBus.Publish("sample.rabbitmq.dm", DateTime.Now);

        transaction.Commit();
    }
}
```

### EntityFramework 

```csharp

private readonly ICapPublisher _capBus;

using (var trans = dbContext.Database.BeginTransaction(_capBus, autoCommit: false))
{
    dbContext.Persons.Add(new Person() { Name = "ef.transaction" });
    
    _capBus.Publish("sample.rabbitmq.dm", DateTime.Now);

    dbContext.SaveChanges();
    trans.Commit();
}

```
