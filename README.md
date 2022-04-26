## 项目维护地址
https://github.com/222momo/cassandra-fdw/
## 项目原址
https://github.com/rankactive/cassandra-fdw
### tips：作为rankactive / cassandra-fdw 项目的补丁，该项目已年久失修，无法联系作者提交补丁，因此，在本项目中做一些更新
## 1.修改项目支持python3语法
cassandra_provider.py
```
from cStringIO import StringIO
改为
from io import StringIO

rowid_values.append(unicode(line[idcolumn]))
改为
rowid_values.append(str(line[idcolumn]))
```
time_utils.py
```
from cStringIO import StringIO
改为
from io import StringIO
```
types_mapper.py
```
cassandra_types.cql_text: lambda: obj if obj is unicode else obj.encode('utf8'),
cassandra_types.cql_blob: lambda: unicode(obj),
cassandra_types.cql_ascii: lambda: unicode(obj),
改为
cassandra_types.cql_text: lambda: obj if obj is str else str(obj),
cassandra_types.cql_blob: lambda: str(obj),
cassandra_types.cql_ascii: lambda: str(obj),
```
## 2.原有bug，无法处理原始数据类型具有多层list、frozen的情况
如
frozen<tuple<text, frozen<tuple<float, float>>>> 查询失败 
list<frozen<tuple<text, frozen<tuple<float, float>>>>> 查询失败
types_mapper.py
```
simple_type = {
    'uuid': cassandra_types.cql_uuid,
    'bigint': cassandra_types.cql_bigint,
    'boolean': cassandra_types.cql_boolean,
    'decimal': cassandra_types.cql_decimal,
    'double': cassandra_types.cql_double,
    'float': cassandra_types.cql_float,
    'int': cassandra_types.cql_int,
    'timestamp': cassandra_types.cql_timestamp,
    'timeuuid': cassandra_types.cql_timeuuid,
    'text': cassandra_types.cql_text,
    'inet': cassandra_types.cql_inet,
    'counter': cassandra_types.cql_counter,
    'varint': cassandra_types.cql_varint,
    'blob': cassandra_types.cql_blob,
    'ascii': cassandra_types.cql_ascii,
    'tinyint': cassandra_types.cql_tinyint,
    'smallint': cassandra_types.cql_smallint,
    'time': cassandra_types.cql_time,
    'date': cassandra_types.cql_date
}[validator]
return CqlType(simple_type, [])
改为
dict = {
'uuid': cassandra_types.cql_uuid,
    'bigint': cassandra_types.cql_bigint,
    'boolean': cassandra_types.cql_boolean,
    'decimal': cassandra_types.cql_decimal,
    'double': cassandra_types.cql_double,
    'float': cassandra_types.cql_float,
    'int': cassandra_types.cql_int,
    'timestamp': cassandra_types.cql_timestamp,
    'timeuuid': cassandra_types.cql_timeuuid,
    'text': cassandra_types.cql_text,
    'inet': cassandra_types.cql_inet,
    'counter': cassandra_types.cql_counter,
    'varint': cassandra_types.cql_varint,
    'blob': cassandra_types.cql_blob,
    'ascii': cassandra_types.cql_ascii,
    'tinyint': cassandra_types.cql_tinyint,
    'smallint': cassandra_types.cql_smallint,
    'time': cassandra_types.cql_time,
    'date': cassandra_types.cql_date
}
if(validator in dict):
    simple_type = dict[validator]
else:
    simple_type = cassandra_types.cql_text
return CqlType(simple_type, [])
```
## 3.增加固定的依赖地址
__init__.py 
```
import sys
import struct
import platform
sys.path.append("/opt/target/cassandra-fdw/cassandra-fdw")  # 写清楚依赖地址
```
# cassandra-fdw
PostgreSQL Foreign Data Wrapper for Cassandra
## Requirements
* PostgreSQL 9.3+
* PostgreSQL development packages (postgresql-server-dev-9.x)
* Cassandra 2.1+, 2.2+, 3+

## Features
* Foreign schema import
* Full CQL types support
* CQL query optimizations

## How to install
#### install additional packages
```bash
sudo apt-get install git build-essential python-dev python-setuptools pgxnclient
```
#### install Multicorn
```bash
sudo pgxn install multicorn
```
#### install Cassandra driver and modules
```bash
sudo easy_install pip
sudo pip install cassandra-driver
sudo pip install pytz
```
#### clone repository
```bash
git clone https://github.com/rankactive/cassandra-fdw.git
```
#### install FDW
```bash
cd cassandra-fdw
python setup.py install
```

## Usage
```SQL
-- Create test database
CREATE DATABASE fdw_test;
```
Switch to database fdw_test
```SQL
-- Create extension for database
CREATE EXTENSION multicorn;
```
```SQL
-- Create server
CREATE SERVER fdw_server FOREIGN DATA WRAPPER multicorn
OPTIONS (
  wrapper 'cassandra-fdw.CassandraFDW',
  hosts '10.10.10.1,10.10.10.2',
  port '9042',
  username 'cassandra user', -- optional
  password 'cassandra password' -- optional
);
```
```SQL
-- Create foreign table
CREATE FOREIGN TABLE fdw_table
(
  col1 text,
  col2 int,
  col3 bigint
) SERVER fdw_server OPTIONS (keyspace 'cassandra keyspace', columnfamily 'cassandra columnfamily');
```

Use it!
```SQL
SELECT * FROM fdw_table WHERE col1 = 'some text';
```

```SQL
INSERT INTO fdw_table VALUES ('text', 123, 1234);
```

 By default, the concurrency level of modifications is 4. It means that batch modifications will be sent in 4 threads to Cassandra. To change it use:
```SQL
ALTER SERVER fdw_srv OPTIONS (modify_concurency 'your integer value');
```
Or if it has been set before, use:
```SQL
ALTER SERVER fdw_srv OPTIONS (SET modify_concurency 'your integer value');
```

If you want to use updates and deletes, you must create column named "\_\_rowid\_\_" with type "TEXT"

Import foreign schema example:
```SQL
CREATE SCHEMA fdw_test;
IMPORT FOREIGN SCHEMA cassandra_keyspace FROM SERVER fdw_server INTO fdw_test;
```

## Types mapping

| CQL type | PostgreSQL type |
| --- | --- |
| bigint | bigint |
| blob | bytea |
| boolean | boolean |
| counter | bigint |
| date | date |
| decimal | decimal |
| double | float8 |
| float | float4 |
| inet | inet |
| int | int |
| list\<type\> | type[] |
| map\<type, type\> | json |
| set\<type\> | type[] |
| smallint | smallint |
| text | text |
| time | timetz |
| timestamp | timestamptz |
| timeuuid | uuid |
| tinyint | smallint |
| tuple\<type,type,...\> | json |
| uuid | uuid |
| varint | int |
