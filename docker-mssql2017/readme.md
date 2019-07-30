## Docker

```bash

docker run -v "C:\DropboxFolderPath":/dropbox \
           --name sql2017 \
           -e 'ACCEPT_EULA=Y' \
           -e 'SA_PASSWORD=<NewStrong!Password>' \
           -p 49152:1433 \
           -d mcr.microsoft.com/mssql/server:2017-latest-ubuntu

docker exec -it sql2017 /opt/mssql-tools/bin/sqlcmd  \
            -S localhost -U SA -P "<NewStrong!Password>"
1> SELECT @@VERSION
2> GO
------------------------------------------------------------------------
Microsoft SQL Server 2017 (RTM-CU15-GDR) (KB4505225) - 14.0.3192.2 (X64)
        Copyright (C) 2017 Microsoft Corporation
        Express Edition (64-bit) on Linux (Ubuntu 16.04.6 LTS)
```

## Data Studio

![](https://i.imgur.com/h8FA7hX.png)

```sql
USE [master]
GO

CREATE DATABASE [AdventureWorksLT2012] ON
(FILENAME=N'/dropbox/AdventureWorksLT2012_Data.mdf')
FOR ATTACH;

CREATE DATABASE [Northwind] ON
(FILENAME=N'/dropbox/Northwind.mdf')
FOR ATTACH;

GO
```

![](https://i.imgur.com/z19XxHk.png)

## Ref

- https://tomssl.com/2018/01/15/running-microsoft-sql-server-on-a-linux-container-in-docker/
- https://blogs.msdn.microsoft.com/orrinedenfield/2017/10/sql-server-on-linux-on-docker-quick-and-easy/
- https://docs.microsoft.com/en-us/sql/linux/tutorial-restore-backup-in-sql-server-container?view=sql-server-2017
- https://dbafromthecold.com/2019/03/21/using-docker-named-volumes-to-persist-databases-in-sql-server/
- https://pythonkim.tistory.com/m/55
- https://superuser.com/questions/1344407/docker-on-wsl-wont-bind-mount-home
