# Autonomous DB

## Shared Exadata

### Services / Concurrency / Parallelism

DB Type	Service	Concurrency	Parallelism	CPU/IO Share
ATP/ADW	LOW	300 x OCPU	no	1
ATP/ADW	MEDIUM	1.25 x OCPU	4 	2x
ATP/ADW	HIGH	3	All	4x
ATP	TP	300 x OCPU	No	8x
ATP	TPURGENT	300 x OCPU	Manual	12x

Update CPU/IO shares with cs_resource_manager.update_plan_directive.
Or through service console

Manage Runaway SQL Statements with Resource Management Rules


## Data Loading

- Object Storage
Username + auth token

- Object Storage Classic
Username + oracle cloud classic password

- Amazon S3
Username + AWS access key

- Azure Blob Storage
Username + Azure storage access key

Autonomous Data Warehouse provides a new PL/SQL package, DBMS_CLOUD

### Loading options:
Data Pump, SQL Developper, SQL*Loader, GoldenGate

Load data from OSS

```
BEGIN
 DBMS_CLOUD.COPY_DATA(
    table_name =>'CHANNELS',
    credential_name =>'DEF_CRED_NAME',
    file_uri_list =>'https://objectstorage.us-phoenix-1.oraclecloud.com/n/namespace-string/b/bucketname/o/channels.txt',
    format => json_object('delimiter' value ',')
 );
END;
/
```

DBMS_CLOUD compression format option to see the supported compression types.


DBMS_CLOUD.COPY_COLLECTION to load  line delimited JSON documents into a table

```
BEGIN  
 DBMS_CLOUD.COPY_COLLECTION(    
    collection_name => 'fruit',
    credential_name => 'DEF_CRED_NAME',
    file_uri_list   =>
      'https://objectstorage.us-ashburn-1.oraclecloud.com/n/namespace-string/b/fruit_bucket/o/myCollection.json',
    format          =>
      JSON_OBJECT('recorddelimiter' value '''\n''')  );
END;
/
```

For JSON arrays use:

```
BEGIN 
  DBMS_CLOUD.COPY_COLLECTION(    
    collection_name => 'fruit2',    
    credential_name => 'DEF_CRED_NAME',    
    file_uri_list => 'https://objectstorage.us-ashburn-1.oraclecloud.com/n/namespace-string/b/json/o/fruit_array.json',
    format => '{"recorddelimiter" : "0x''01''", "unpackarrays" : "TRUE", "maxdocsize" : "10240000"}'
  );
END;
/
```

### Troubleshoot load operations

dba_load_operations all loads
user_load_operations in user schema

LOGFILE_TABLE
BADFILE_TABLE show logs

DataPump
Options to use:
exclude=index,cluster,indextype,materialized_view,materialized_view_log,materialized_zonemap,db_link
data_options=group_partition_table_data 
parallel=n
schemas=schema_name
dumpfile=export%u.dmp

Set DBMS_CLOUD.CREATE CREDENTIAL
Then import from OSS

```
impdp admin/password@ADWC1_high \       
     directory=data_pump_dir \       
     credential=def_cred_name \      
     dumpfile= https://objectstorage.us-ashburn-1.oraclecloud.com/n/namespace-string/b/bucketname/o/export%u.dmp \
     parallel=16 \
     encryption_pwd_prompt=yes \
     partition_options=merge \ 
     transform=segment_attributes:n \
     transform=dwcs_cvt_iots:y transform=constraint_use_default_index:y \
     exclude=index,cluster,indextype,materialized_view,materialized_view_log,materialized_zonemap,db_link
```

For older datapump the credential must be the default credentials

```
ALTER DATABASE PROPERTY SET DEFAULT_CREDENTIAL = 'ADMIN.DEF_CRED_NAME'
```

To access datapump logs, put the object to OSS

```
BEGIN
  DBMS_CLOUD.PUT_OBJECT(
    credential_name => 'DEF_CRED_NAME',
    object_uri => 'https://objectstorage.us-ashburn-1.oraclecloud.com/n/namespace-string/b/bucketname/o/import.log',
    directory_name  => 'DATA_PUMP_DIR',
    file_name => 'import.log');
END;
/
```

## SQL Operation not permitted

The following SQL statements are not available in Autonomous Data Warehouse:
* ADMINISTER KEY MANAGEMENT
* ALTER TABLESPACE
* CREATE DATABASE LINKUse DBMS_CLOUD_ADMIN.CREATE_DATABASE_LINK to create database links in Autonomous Data Warehouse. See Create Database Links from Autonomous Database to Oracle Databases for more information.
* CREATE TABLESPACE
* DROP TABLESPACE


## Manual backup to OSS

```
ALTER DATABASE PROPERTY SET default_bucket='https://swiftobjectstorage.us-phoenix-1.oraclecloud.com/v1/namespace-string';
```
The format of the bucket name is backup_databasename

SET DEFINE OFF. (To avoid parsing & as special char)

```
BEGIN
  DBMS_CLOUD.CREATE_CREDENTIAL(
    credential_name => 'DEF_CRED_NAME',
    username => 'adwc_user@example.com', 
    password => 'password'
);
END;
/
ALTER DATABASE PROPERTY SET DEFAULT_CREDENTIAL = 'ADMIN.DEF_CRED_NAME';
```

Backup retention: 60 days


## Application Continuity

https://docs.oracle.com/en/cloud/paas/autonomous-data-warehouse-cloud/user/application-continuity.html#GUID-8874CB1D-0B20-461F-91D2-24E2EE4148A3

```
BEGIN
	DBMS_CLOUD_ADMIN.DISABLE_APP_CONT(
		service_name => 'nvt21_adb1_high.adwc.oraclecloud.com'
	);
END;
/
```


## Connectivity

SQL*Net, JDBC, ODBC

TLS/wallet


## SLQ Developer

* Username: Enter the database username. You can either use the default administrator database account (ADMIN) provided as part of the service or create a new schema, and use it.
* Password: Enter the password for the database user.
* Service: Enter the database TNS name. The client credentials file includes a tnsnames.ora file that provides database TNS names with corresponding services.

Enable proxy in sqlnet.ora
Define proxy host / port in tnsnames.ora


## Analytics Cloud
For connectivity with private endpoint, need a Data Gateway

## Machine Learning

Not supported with private endpoints.


## Users

```
CREATE USER new_user IDENTIFIED BY password;
GRANT CREATE SESSION TO new_user;
```

Password lifetime 360 days

```
DBMS_CLOUD_ADMIN.GRANT_TABLESPACE_QUOTA(
       username            IN VARCHAR2,
       tablespace_quota    IN VARCHAR2);
```

### Giving access to SQL Developer

enable schema access to SQL Developer Web

```
BEGIN
   ORDS_ADMIN.ENABLE_SCHEMA(
     p_enabled => TRUE,
     p_schema => 'schema-name',
     p_url_mapping_type => 'BASE_PATH',
     p_url_mapping_pattern => 'schema-alias',
     p_auto_rest_auth => TRUE
   );
   COMMIT;
END;
/
```

### Get URL

```
https://dbname.adb.us-ashburn-1.example.com/ords/<schema-alias>/_sdw/?nav=worksheet
```

Self-securing

enable and disable encryption at rest
Data Masking and Database Vault
Data Safe (secure user configs) Not supported with private endpoints


## Database Vault

```
DBMS_CLOUD_MACADM.CONFIGURE_DATABASE_VAULT('adb_dbv_owner', 'adb_dbv_acctmgr');
```
```
EXEC DBMS_CLOUD_MACADM.ENABLE_DATABASE_VAULT;
```
Restart DB

Check status
```
SELECT * FROM DBA_DV_STATUS;
```

Example enable APEX 
```
EXEC DBMS_CLOUD_MACADM.ENABLE_USERMGMT_DATABASE_VAULT('APEX');
```

## Querying

### Query external data

```
BEGIN
   DBMS_CLOUD.CREATE_EXTERNAL_TABLE(
    table_name =>'CHANNELS_EXT',
    credential_name =>'DEF_CRED_NAME',
    file_uri_list =>'https://objectstorage.us-phoenix-1.oraclecloud.com/n/namespace-string/b/bucketname/o/channels.txt',
    format => json_object('delimiter' value ','),
    column_list => 'CHANNEL_ID NUMBER, CHANNEL_DESC VARCHAR2(20), CHANNEL_CLASS VARCHAR2(20)' );
END;
/
```

For parquet, avrò, orc, use a format of the type:
format =>  '{"type":"parquet", "schema": "first"}'

### Query external partitioned data

```
DBMS_CLOUD.CREATE_EXTERNAL_PART_TABLE
```

Validate External Data

```
BEGIN
  DBMS_CLOUD.VALIDATE_EXTERNAL_TABLE (
    table_name => 'CHANNELS_EXT' );
END;
/
```

ADB-D

Licences


## Maintenance

Container DB maintenance
You cannot skip scheduled maintenance more than twice, consecutively.

* Release Update (RU): Autonomous Database installs only the most current release update.
* Release Update Revision (RUR): Autonomous Database installs the release update plus additional fixes.

## Performance

Autonomous Data Warehouse uses Hybrid Columnar Compression for all tables by default.
Use NOCOMPRESS; option to disable compression for tables doing lots of DELETE / UPDATE

To compress an not compressed table you can use

```
ALTER TABLE sales MOVE COLUMN STORE COMPRESS FOR QUERY HIGH;
```


ADW ignores optimizer hints by default
Can be activated by 

```
ALTER SESSION
   SET OPTIMIZER_IGNORE_HINTS=FALSE;
```

Enable parallelization hints:

```
ALTER SESSION 
   SET OPTIMIZER_IGNORE_PARALLEL_HINTS=FALSE;
```

Auto indexing disabled by default

```
EXEC DBMS_AUTO_INDEX.CONFIGURE('AUTO_INDEX_MODE','IMPLEMENT');
EXEC DBMS_AUTO_INDEX.CONFIGURE('AUTO_INDEX_MODE','OFF');
```

### CPU Utilization

CPU utilization (%)
This chart shows the historical CPU utilization of the service:
* Auto Scaling Disabled: This chart shows hourly data. A data point shows the average CPU utilization for that hour. For example, a data point at 10:00 shows the average CPU utilization for 9:00-10:00.The utilization percentage is reported with respect to the number of CPUs the database is allowed to use which is two times the number of OCPUs. For example, if the database has four (4) OCPUs, the percentage in this graph is based on 8 CPUs.
* Auto Scaling Enabled: For databases with auto scaling enabled the utilization percentage is reported with respect to the maximum number of CPUs the database is allowed to use, which is six times the number of OCPUs. For example, if the database has four OCPUs with auto scaling enabled the percentage in this graph is based on 24 CPUs.


## Logging

Logs are retained for 30 days

```
DBMS_WORKLOAD_REPOSITORY.MODIFY_SNAPSHOT_SETTINGS
```
to change time


## Backup / Restore

When you restore, the Oracle Machine Learning workspaces, projects, and notebooks are not restored.

### Manual backup 

Retention 60 days like other backups
During backup DB is operational but no lifecycle is permitted

Set default bucket
```
ALTER DATABASE PROPERTY SET default_bucket='https://swiftobjectstorage.us-phoenix-1.oraclecloud.com/v1/namespace-string';
```

Create credentials
Set default credentials

### Cloning

Use cloning to upgrade DB version
When creating a clone from a backup, you must select a backup that is at least 2 hours old, or the clone operation will fail.

If you define a network Access Control List (ACL) on the source database, the currently set network ACL is cloned to the new database. If a database is cloned from a backup, the current source database's ACL is applied (not the ACL that was valid of the time of the backup).

For a Metadata Clone, the APEX Apps and the OML Projects and Notebooks are copied to the clone. For a Metadata Clone, the underlying database data of the APEX App or OML Notebook is not cloned.

You can only clone an Autonomous Data Warehouse instance to the same tenancy and the same region as the source database.

* During the provisioning for either a Full Clone or a Metadata Clone, any resource management rule changed by the user in the source database is carried over to the cloned database.

During the provisioning for either a Full Clone or a Metadata Clone, the optimizer statistics are copied from the source database to the cloned database.

### Files / directory 

```
SELECT * FROM DBMS_CLOUD.LIST_FILES('STAGING');

BEGIN
   DBMS_CLOUD.GET_OBJECT(
   credential_name => 'DEF_CRED_NAME',
   object_uri => 'https://objectstorage.usphoenix-1.oraclecloud.com/n/namespace-string/b/bucketname/o/cwallet.sso',
   directory_name => 'STAGE');
END;
/
```

## Audit
Each Autonomous Database instance runs an automated purge job once a day to remove all audit records older than fourteen (14) days.


## Permissions

Aggregate Resource-Type
autonomous-database-family
Individual Resource-Types:
autonomous-databases
autonomous-backups
autonomous-container-databases
autonomous-exadata-infrastructures


autonomous-data-warehouses
autonomous-data-warehouse-backups

