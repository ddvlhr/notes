# dotnet

> ## publish

```powershell
dotnet publish -p:PublishProfile={PublishProfiles中的配置文件的名称}
```

> ## ef migration

```powershell
// 生成迁移纪录
dotnet ef migrations add migration-name --project migration-project-absolute-url
// 更新到数据库
dotnet ef database update
```

> ## EF Core

自定义迁移历史纪录表

```c#
options => options.UseMysql(options.UseMySql(connectionString, ServerVersion.AutoDetect(connectionString),
            x =>
            {
              	// 自定义 migrations 文件夹所在项目
                x.MigrationsAssembly("Infrastructure");
              	// 自定义迁移历史纪录表表名
                x.MigrationsHistoryTable("__efmigrationshistory");
            })
```

