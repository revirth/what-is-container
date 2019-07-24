## Docker

```bash
docker run --name sql2017 -e 'ACCEPT_EULA=Y' -e 'SA_PASSWORD=<NewStrong!Passw0rd>' -e 'MSSQL_PID=Express' -p 1433:1433 -d mcr.microsoft.com/mssql/server:2017-latest-ubuntu

docker exec -it sql2017 /opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P "<NewStrong!Passw0rd>"

docker cp Northwind.mdf sql2017:/var/opt/mssql/data/

docker cp AdventureWorksLT2012_Data.mdf sql2017:/var/opt/mssql/data/
```

## Data Studio

![](https://i.imgur.com/X2Gjcr9.png)

```sql
USE [master]
GO

CREATE DATABASE [AdventureWorksLT2012] ON
(FILENAME=N'/var/opt/mssql/data/AdventureWorksLT2012_Data.mdf')
FOR ATTACH;

CREATE DATABASE [Northwind] ON
(FILENAME=N'/var/opt/mssql/data/Northwind.mdf')
FOR ATTACH;

GO
```

![](https://i.imgur.com/z19XxHk.png)

## Ref

- https://tomssl.com/2018/01/15/running-microsoft-sql-server-on-a-linux-container-in-docker/
- https://blogs.msdn.microsoft.com/orrinedenfield/2017/10/sql-server-on-linux-on-docker-quick-and-easy/
- https://docs.microsoft.com/en-us/sql/linux/tutorial-restore-backup-in-sql-server-container?view=sql-server-2017
