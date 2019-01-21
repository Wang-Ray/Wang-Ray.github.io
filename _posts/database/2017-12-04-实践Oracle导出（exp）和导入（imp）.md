---
layout: post
title: "实践Oracle导出（exp）和导入（imp）"
date: 2017-12-04 14:05:00 +0800
categories: 数据库
tags: oracle tablespace
---

Oracle提供了命令行工具导出（exp）和导入（imp），可以仅导出或导入数据结构（包含表、视图、sequence、索引等数据库对象）或数据结构和数据一起，可以按用户导入导入，也可以仅导出导入特定表。命令的详细参数可以通过exp help=y或imp help=y查看。

需要切换到数据库安装用户下执行，为了避免乱码，最好检查和设置console的编码跟数据库编码保持一致，比如设置console编码为gbk，也可以在用户的环境变量配置好，避免每次都设置：

```shell
$ export NLS_LANG=AMERICAN_AMERICA.ZHS16GBK
```

按用户导出举例（导出REPORT_DW用户的数据结构和数据）：

```shell
$ exp  REPORT_DW/REPORT_DW owner=REPORT_DW rows=y file=report_dw.dmp
```

按表导出（test和test1表）

```shell
$ exp  REPORT_DW/REPORT_DW tables=test,test1 rows=y file=report_dw.dmp
```

向导模式导出

```shell
$  exp ITSDB_UAT/ITSDB_UAT_2014@itsdb
```

如果只导出数据结构不导出数据

```shell
$ exp  REPORT_DW/REPORT_DW owner=REPORT_DW rows=n file=report_dw.dmp
```

注：owner：导出哪个用户下的内容，tables：导出那些表，rows=y：是否导出数据，file：导出落地文件

导入举例（导入之前从REPORT_DW用户导出的内容到REPORT_DW_INT用户下）

```shell
$ imp REPORT_DW_INT/REPORT_DW_INT_2014 fromuser=REPORT_DW touser=REPORT_DW_INT file=report_dw.dmp
```

注：file：之前exp的文件，用来导入；fromuser：之前exp的owner；touser：导入到那个用户下。
如果存在相同数据结构会报冲突错，建议目标用户比较干净。

### exp help

```shell
$ exp help=y

Export: Release 11.2.0.3.0 - Production on Thu Nov 13 15:07:08 2014

Copyright (c) 1982, 2011, Oracle and/or its affiliates.  All rights reserved.



You can let Export prompt you for parameters by entering the EXP
command followed by your username/password:

     Example: EXP SCOTT/TIGER

Or, you can control how Export runs by entering the EXP command followed
by various arguments. To specify parameters, you use keywords:

     Format:  EXP KEYWORD=value or KEYWORD=(value1,value2,...,valueN)
     Example: EXP SCOTT/TIGER GRANTS=Y TABLES=(EMP,DEPT,MGR)
               or TABLES=(T1:P1,T1:P2), if T1 is partitioned table

USERID must be the first parameter on the command line.

Keyword    Description (Default)      Keyword      Description (Default)
--------------------------------------------------------------------------
USERID     username/password          FULL         export entire file (N)
BUFFER     size of data buffer        OWNER        list of owner usernames
FILE       output files (EXPDAT.DMP)  TABLES       list of table names
COMPRESS   import into one extent (Y) RECORDLENGTH length of IO record
GRANTS     export grants (Y)          INCTYPE      incremental export type
INDEXES    export indexes (Y)         RECORD       track incr. export (Y)
DIRECT     direct path (N)            TRIGGERS     export triggers (Y)
LOG        log file of screen output  STATISTICS   analyze objects (ESTIMATE)
ROWS       export data rows (Y)       PARFILE      parameter filename
CONSISTENT cross-table consistency(N) CONSTRAINTS  export constraints (Y)

OBJECT_CONSISTENT    transaction set to read only during object export (N)
FEEDBACK             display progress every x rows (0)
FILESIZE             maximum size of each dump file
FLASHBACK_SCN        SCN used to set session snapshot back to
FLASHBACK_TIME       time used to get the SCN closest to the specified time
QUERY                select clause used to export a subset of a table
RESUMABLE            suspend when a space related error is encountered(N)
RESUMABLE_NAME       text string used to identify resumable statement
RESUMABLE_TIMEOUT    wait time for RESUMABLE
TTS_FULL_CHECK       perform full or partial dependency check for TTS
VOLSIZE              number of bytes to write to each tape volume
TABLESPACES          list of tablespaces to export
TRANSPORT_TABLESPACE export transportable tablespace metadata (N)
TEMPLATE             template name which invokes iAS mode export

Export terminated successfully without warnings.
```

### imp help

```shell
imp help=y

Import: Release 11.2.0.3.0 - Production on Thu Nov 13 15:20:14 2014

Copyright (c) 1982, 2011, Oracle and/or its affiliates.  All rights reserved.



You can let Import prompt you for parameters by entering the IMP
command followed by your username/password:

     Example: IMP SCOTT/TIGER

Or, you can control how Import runs by entering the IMP command followed
by various arguments. To specify parameters, you use keywords:

     Format:  IMP KEYWORD=value or KEYWORD=(value1,value2,...,valueN)
     Example: IMP SCOTT/TIGER IGNORE=Y TABLES=(EMP,DEPT) FULL=N
               or TABLES=(T1:P1,T1:P2), if T1 is partitioned table

USERID must be the first parameter on the command line.

Keyword  Description (Default)       Keyword      Description (Default)
--------------------------------------------------------------------------
USERID   username/password           FULL         import entire file (N)
BUFFER   size of data buffer         FROMUSER     list of owner usernames
FILE     input files (EXPDAT.DMP)    TOUSER       list of usernames
SHOW     just list file contents (N) TABLES       list of table names
IGNORE   ignore create errors (N)    RECORDLENGTH length of IO record
GRANTS   import grants (Y)           INCTYPE      incremental import type
INDEXES  import indexes (Y)          COMMIT       commit array insert (N)
ROWS     import data rows (Y)        PARFILE      parameter filename
LOG      log file of screen output   CONSTRAINTS  import constraints (Y)
DESTROY                overwrite tablespace data file (N)
INDEXFILE              write table/index info to specified file
SKIP_UNUSABLE_INDEXES  skip maintenance of unusable indexes (N)
FEEDBACK               display progress every x rows(0)
TOID_NOVALIDATE        skip validation of specified type ids
FILESIZE               maximum size of each dump file
STATISTICS             import precomputed statistics (always)
RESUMABLE              suspend when a space related error is encountered(N)
RESUMABLE_NAME         text string used to identify resumable statement
RESUMABLE_TIMEOUT      wait time for RESUMABLE
COMPILE                compile procedures, packages, and functions (Y)
STREAMS_CONFIGURATION  import streams general metadata (Y)
STREAMS_INSTANTIATION  import streams instantiation metadata (N)
DATA_ONLY              import only data (N)
VOLSIZE                number of bytes in file on each volume of a file on tape

The following keywords only apply to transportable tablespaces
TRANSPORT_TABLESPACE import transportable tablespace metadata (N)
TABLESPACES tablespaces to be transported into database
DATAFILES datafiles to be transported into database
TTS_OWNERS users that own data in the transportable tablespace set

Import terminated successfully without warnings.
```

## Q&A

### IMP-00032: SQL statement exceeded buffer length

设置buffer=1000000即可