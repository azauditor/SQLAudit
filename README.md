# SQLAudit

These are queries to help assist with retrieving users inside of a SQL instance and the permissions that those users are assigned.  These queries are designed to only read data and will not modify or create data inside of the database.

## SQL Server Instance

The following query is used to determine what accounts have the ability to login to the SQL instance and the Server Roles that they are assigned to.

``` SQL
SELECT SP1.[name] AS 'Login', SP2.[name] AS 'ServerRole', SP1.* 
FROM sys.server_principals AS SP1 
  JOIN sys.server_role_members AS SRM 
    ON SP1.principal_id = SRM.member_principal_id 
  JOIN sys.server_principals AS SP2 
    ON SRM.role_principal_id = SP2.principal_id 
ORDER BY SP1.[name], SP2.[name]; 
```

## SQL Server Database

When we collect data we only run read-only queries, however currently we have not determined a way to run a read-only query to get Database users and permissions.  Until we develop a query that does not create tables on the server please provide the following information:

1. Screenshots of Users that have access to the financial system database
2. For each user with access provide the following:
    * Screenshot of user's Membership
    * Screenshot of user's Securables
    * Screenshot of user's Extended Properties