# Other Operations {#concept_in2_nbd_5db .concept}

## ALIAS command {#section_kcp_t2f_vdb .section}

The ALIAS command is used to read different resources \(data\) using a fixed resource name in [MapReduce](intl.en-US/User Guide/MapReduce/Summary/MapReduce.md) or [UDF](intl.en-US/User Guide/SQL/UDF/UDF Summary.md)  without modifying the code.

**The command format is as follows:**

```
ALIAS <alias>=<real>;
```

**Action:**

Create alias for a resource.

**For example:**

```
ADD TABLE src_part PARTITION (ds='20121208') AS res_20121208;
ADD TABLE src_part PARTITION (ds='20121209') AS res_20121209;
ALIAS resName=res_20121208;
jar -resources resName -libjars work.jar -classpath ./work.jar com.company.MainClass args ... ;//job 1
ALIAS resName=res_20121209;
jar -resources resName -libjars work.jar -classpath ./work.jar com.company.MainClass args ... ;//job 2
```

In the preceding example resource alias **resName** refers to different resource tables in two jobs. Different data can be read without modifying the code.

## Set {#section_pqs_bff_vdb .section}

**The command format is as follows:**

```
set [<KEY>=<VALUE>]
```

**Action:**

You can use the set command to set MaxCompute or a user-defined system variables that affects the MaxCompute operation.

**Currently, the system variables supported in MaxCompute are as follow:**

```
--Set commands supported by MaxCompute SQL and Mapreduce (new version)
set odps.sql.allow.fullscan=  --Set whether to allow a full table scan on a partitioned table. True means allow, and false means not allow.
set odps.stage.mapper.mem=   --Set the memory size of each map worker. Unit is M and default value is 1024M.
set odps.stage.reducer.mem= --Set the memory size for each reduce worker in the unit of M. The default value is 1,024M.
set odps.stage.joiner.mem=   --Set the memory size of each join worker. Unit is M and default value is 1024M.
set odps.stage.mem =    --Set the memory size of all workers in MaxCompute specified job. The priority is lower than preceding three set key. Unit is M and no default value.
set odps.stage.mapper.split.size=   -- Modify the input data quantity of each map worker; that is the size of input file burst. Thus control the worker number of each map stage. Unit is M and the default value is 256M.
set odps.stage.reducer.num=    --Modify the worker number of each reduce stage and no default value.
set odps.stage.joiner.num=    --Modify the worker number of each join stage and no default value.
set odps.stage.num=    --Modify the worker concurrency of all stages in MaxCompute specified job. The priority is lower than preceding three set key and no default value.
set odps.sql.type.system.odps2=    --The default value is false. You must set true when there are new data types such as TINYINT, SMALLINT, INT, FLOAT, VARCHAR, TIMESTAMP, and BINARY in SQL statement.
```

## Show Flags {#section_ft4_jff_vdb .section}

**The command format is as follows:**

```
show flags;  --Display the parameters set by the Set command.
```

**Action:**

Running the Use Project command can clear the configurations set by the Set command.

## SetProject {#section_pyt_mff_vdb .section}

**The command format is as follows:**

```
setproject [<KEY>=<VALUE>];
```

**Action:**

-   You can use setproject command to set project attributes.
-   If the value of <KEY\>=<VALUE\> is not specified, the current project attribute configuration is displayed. The command format is as follows:

    ```
    setproject;  --Display the parameters set by the setproject command.
    ```


For example, the following example sets the method that allows full table scanning.

```
setproject odps.sql.allow.fullscan = true;
```

**The detailed description of project attributes is shown as follows:**

|Property name|Cunfigured permissions|Attribute description|Value range|
|:------------|:---------------------|:--------------------|:----------|
|odps.sql.allow.fullscan|ProjectOwner|item whether to allow full table scan|True \(permitted\) /false \(prohibited\)|
|odps.table.drop.ignorenonexistent|All users|Whether to report an error when deleting a table that does not exist. When the value is true, no error is reported.|True \(no error reported\)/false|
|odps.security.ip.whitelist|ProjectOwner|Specify an IP whitelist to access the project.|IP list separated by commas \(,\)|
|odps.table.lifecycle|ProjectOwner|optional: when creating a table, the lifecycle substatement is optional. If you do not set the lifecycle, the table will be permanently valid. mandatory: the lifecycle substatement is mandatory. inherit: if you do not set the lifecycle, odps.table.lifecycle.value will be the lifecycle of this table.|optional /mandatory/inherit|
|odps.table.lifecycle.value|ProjectOwner|Default lifecycle.|1~37231\(default value\)|
|odps.instance.remain.days|ProjectOwner|How long the instance information is retained.|3~30|
|READ\_TABLE\_MAX\_ROW|ProjectOwner|The number of data entries returned by running the Select statement in the client.|1~10000|

**Take odps.security.ip.whitelist as an example**

MaxCompute supports IP whitelist of the project level.

**Note:** 

-   With IP whitelist configured, only the IP \(console IP or IP of exit where SDK is located\) in the whitelist can access this project.
-   After setting the IP white list, you need to wait five minutes before it takes effect.
-   If in case of incorrect operation you have shield yourself, open a ticket to contact Alibaba Cloud technical support team.

You can type three formats for the IP list in the whitelist.

-   IP address: For example, 101.132.236.134.
-   Subnet mask: For example, 100.116.0.0/16.
-   Network segment: For example, 101.132.236.134-101.132.236.144.

These three formats can appear in the same command and must be separated by commas \(,\).

For example the command line tool set the IP white list of methods.

```
setproject odps.security.ip.whitelist=101.132.236.134,100.116.0.0/16,101.132.236.134-101.132.236.144;
```

If there is no IP address in whitelist, it means whitelist function is disabled.

```
setproject odps.security.ip.whitelist=;
```

## Cost SQL {#section_xm2_sgf_vdb .section}

**The command format is as follows:**

```
cost sql <SQL Sentence>;
```

**Action**:

Estimate an SQL measurement message, including the size of the input data, the number of UDFs, and the SQL complexity level.

**Note:** This information cannot be used as an actual charging standard, can be used only for reference.

**For example:**

```
odps@ $odps_project >cost sql select distinct project_name, user_name from meta.m_security_users distribute by project_name sort by project_name;  
ID = 20150715113033121gmsbjxl1
Input:65727592 Bytes
UDF:0
Complexity:1.0
```

