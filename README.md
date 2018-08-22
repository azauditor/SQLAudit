# SQLAudit

These are queries to help assist with retrieving users inside of a SQL instance and the permissions that those users are assigned.  These queries are designed to only read data and will not modify or create data inside of the database.

## SQL Server Edition

This query is used to determine which edition of SQL Server is installed and what version is currently running.  This query was found at [Stackoverflow](https://stackoverflow.com/questions/18070177/how-to-get-current-instance-name-from-t-sql) and was designed by user with handle of "Nate S."

``` SQL
SELECT
    SERVERPROPERTY('ServerName') AS ServerName,
    SERVERPROPERTY('MachineName') AS MachineName,
    CASE
        WHEN  SERVERPROPERTY('InstanceName') IS NULL THEN ''
        ELSE SERVERPROPERTY('InstanceName')
    END AS InstanceName,
    '' as Port, --need to update to strip from Servername. Note: Assumes Registered Server is named with Port
    SUBSTRING ( (SELECT @@VERSION),1, CHARINDEX('-',(SELECT @@VERSION))-1 ) as ProductName,
    SERVERPROPERTY('ProductVersion') AS ProductVersion,
    SERVERPROPERTY('ProductLevel') AS ProductLevel,
    SERVERPROPERTY('ProductMajorVersion') AS ProductMajorVersion,
    SERVERPROPERTY('ProductMinorVersion') AS ProductMinorVersion,
    SERVERPROPERTY('ProductBuild') AS ProductBuild,
    SERVERPROPERTY('Edition') AS Edition,
    CASE SERVERPROPERTY('EngineEdition')
        WHEN 1 THEN 'PERSONAL'
        WHEN 2 THEN 'STANDARD'
        WHEN 3 THEN 'ENTERPRISE'
        WHEN 4 THEN 'EXPRESS'
        WHEN 5 THEN 'SQL DATABASE'
        WHEN 6 THEN 'SQL DATAWAREHOUSE'
    END AS EngineEdition,
    CASE SERVERPROPERTY('IsHadrEnabled')
        WHEN 0 THEN 'The Always On Availability Groups feature is disabled'
        WHEN 1 THEN 'The Always On Availability Groups feature is enabled'
        ELSE 'Not applicable'
    END AS HadrEnabled,
    CASE SERVERPROPERTY('HadrManagerStatus')
        WHEN 0 THEN 'Not started, pending communication'
        WHEN 1 THEN 'Started and running'
        WHEN 2 THEN 'Not started and failed'
        ELSE 'Not applicable'
    END AS HadrManagerStatus,
    CASE SERVERPROPERTY('IsSingleUser') WHEN 0 THEN 'No' ELSE 'Yes' END AS InSingleUserMode,
    CASE SERVERPROPERTY('IsClustered')
        WHEN 1 THEN 'Clustered'
        WHEN 0 THEN 'Not Clustered'
        ELSE 'Not applicable'
    END AS IsClustered,
    '' as ServerEnvironment,
    '' as ServerStatus,
    '' as Comments
```

## SQL Server Instance

The following query is used to determine what accounts have the ability to login to the SQL instance and the Server Roles that they are assigned to.

``` SQL
SELECT SP1.[name] AS 'Login', SP2.[name] AS 'ServerRole', SP1.*
FROM sys.server_principals AS SP1
  JOIN sys.server_role_members AS SRM
    ON SP1.principal_id = SRM.member_principal_id
  JOIN sys.server_principals AS SP2
    ON SRM.role_principal_id = SP2.principal_id
ORDER BY SP1.[name], SP2.[name]
```

## SQL Server SQL Login Password Settings

The following query is used to find accounts that can sign into the SQL Instance withouth being authenticated to AD and to display what password settings are being applied to those accounts. **Note** that you will have to enter the maximum number of days that your organization uses to expire passwords.

``` SQL
SELECT name AS 'SQL_User',
    LOGINPROPERTY(name, 'BadPasswordCount') AS 'BadPasswordCount',
    LOGINPROPERTY(name, 'DaysUntilExpiration') AS 'DaysUntilPasswordExpires',
    LOGINPROPERTY(name, 'HistoryLength') AS 'PasswordHistoryLength',
    LOGINPROPERTY(name, 'IsExpired') AS 'IsSQLLoginExpired',
    LOGINPROPERTY(name, 'IsLocked') AS 'IsSQLLoginLocked',
    LOGINPROPERTY(name, 'IsMustChange') AS 'IsPasswordMustChangeOnNextLogin',
    LOGINPROPERTY(name, 'LockoutTime') AS 'LockoutTime',
    LOGINPROPERTY(name, 'PasswordHashAlgorithm') AS 'PasswordHashAlgorithm',
    LOGINPROPERTY(name, 'PasswordLastSetTime') AS 'PasswordLastResetDT',
    '[INSERT NUMBER OF DAYS THAT PASSWORD ARE SUPPOSED TO EXPIRE]' - DATEDIFF(DAY, CONVERT(DATETIME, LOGINPROPERTY(name, 'PasswordLastSetTime')), GETDATE()) AS 'DaysUntilExpiration'
FROM sys.sql_logins
GROUP BY name
```

## SQL Server Database

The following query is used to determine what access has been granted to users for the 'Selected' database.

1. First right-click on the database that the financial system resides on and select 'New Query'.

    **Note: The first step must be followed otherwise the first database on the server will be selected.  Additionally, the user running the query must have view access to database that is being selected or the query will not run.**

2. Then run the following query:

``` SQL
SELECT DP1.name AS DatabaseRoleName,
    DB_NAME() AS DatabaseName,
    ISNULL(DP2.name, 'No members') AS DatabaseUserName
FROM sys.database_role_members AS DRM
    RIGHT OUTER JOIN sys.database_principals AS DP1
        ON DRM.role_principal_id = DP1.principal_id
    LEFT OUTER JOIN sys.database_principals AS DP2
        ON DRM.member_principal_id = DP2.principal_id
WHERE DP1.type = 'R'
ORDER BY DP1.name
```