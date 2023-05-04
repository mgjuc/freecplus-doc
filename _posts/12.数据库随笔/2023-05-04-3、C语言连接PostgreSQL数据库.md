对很多C/C++程序员来说，操作PostgreSQL数据库是一个技术难点，本文讲述采用freecplus开源框架操作PostgreSQL数据库，freecplus框架是C语言技术网作者二十年的技术积累，功能强大，简单易用。

# 一、源代码说明

freecplus是一个Linux系统下的C/C++开源框架，源代码请前往C语言技术网(www.freecplus.net)下载。

本文介绍的是freecplus框架中采用connection和sqlstatement类操作PostgreSQL数据库。

类的声明文件是freecplus/db/postgresql/\_postgresql.h。

类的定义文件是freecplus/db/postgresql/\_postgresql.cpp。

示例程序位于freecplus/db/postgresql目录中。

编译规则文件是freecplus/db/postgresql/makefile。

# 二、概述

本文不会介绍PostgreSQL数据库、SQL语言和C/C++的基础知识，您应该是一个职业的C/C++程序员，在阅读本文之前，您已经掌握了PostgreSQL数据库和SQL语言的基础知识。

freecplus框架把PostgreSQL提供的库函数封装成了connection和sqlstatement类，采用封装后的类操作PostgreSQL数据库，代码简洁优雅，性能卓越。

接下来我先列出connection和sqlstatement类的声明，然后通过流程图和示例程序介绍它位的用法。

# 三、connection类

PostgreSQL数据库连接connection类的声明（程序员不必关心的私有成员和数据结构未列出）：

> // PostgreSQL数据库连接池类。
>
> class connection
>
> {
>
> public:
>
> int m_state; // 与数据库的连接状态，0-未连接，1-已连接。
>
> CDA_DEF m_cda; // 数据库操作的结果或最后一次执行SQL语句的结果。
>
> char m_sql\[10241\]; // SQL语句的文本，最长不能超过10240字节。
>
> connection(); // 构造函数。
>
> \~connection(); // 析构函数。
>
> // 登录数据库。
>
> // connstr：数据库的登录参数，格式：\"host= user= password= dbname=
> port=\",
>
> // 例如：\"host=172.16.0.15 user=qxidc password=qxidcpwd
> dbname=qxidcdb port=5432\"
>
> //
> username-登录的用户名，password-登录的密码，dbname-缺省数据库，port-mysql服务的端口。
>
> //
> charset：数据库的字符集，如\"gbk\"，必须与数据库保持一致，否则会出现中文乱码的情况。
>
> // autocommitopt：是否启用自动提交，0-不启用，1-启用，缺省是不启用。
>
> //
> 返回值：0-成功，其它失败，失败的代码在m_cda.rc中，失败的描述在m_cda.message中。
>
> int connecttodb(char \*connstr,char \*charset,unsigned int
> autocommitopt=0);
>
> // 提交事务。
>
> // 返回值：0-成功，其它失败，程序员一般不必关心返回值。
>
> int commit();
>
> // 回滚事务。
>
> // 返回值：0-成功，其它失败，程序员一般不必关心返回值。
>
> int rollback();
>
> // 断开与数据库的连接。
>
> // 注意，断开与数据库的连接时，全部未提交的事务自动回滚。
>
> // 返回值：0-成功，其它失败，程序员一般不必关心返回值。
>
> int disconnect();
>
> // 执行SQL语句。
>
> //
> 如果SQL语句不需要绑定输入和输出变量（无绑定变量、非查询语句），可以直接用此方法执行。
>
> // 参数说明：这是一个可变参数，用法与printf函数相同。
>
> //
> 返回值：0-成功，其它失败，失败的代码在m_cda.rc中，失败的描述在m_cda.message中，
>
> //
> 如果成功的执行了非查询语句，在m_cda.rpc中保存了本次执行SQL影响记录的行数。
>
> // 程序员必须检查execute方法的返回值。
>
> //
> 在connection类中提供了execute方法，是为了方便程序员，在该方法中，也是用sqlstatement类来完成功能。
>
> int execute(const char \*fmt,\...);
>
> };

# 四、sqlstatement类

PostgreSQL数据库的SQL语句操作sqlstatement类的声明（程序员不必关心的私有成员和数据结构未列出）：

> // 操作SQL语句类。
>
> class sqlstatement
>
> {
>
> public:
>
> int m_state; // 与数据库连接池的绑定状态，0-未绑定，1-已绑定。
>
> char m_sql\[10241\]; // SQL语句的文本，最长不能超过10240字节。
>
> CDA_DEF m_cda; // 执行SQL语句的结果。
>
> sqlstatement(); // 构造函数。
>
> sqlstatement(connection \*conn); // 构造函数，同时绑定数据库连接池。
>
> \~sqlstatement(); // 析构函数。
>
> // 绑定数据库连接池。
>
> // conn：数据库连接池connection对象的地址。
>
> //
> 返回值：0-成功，其它失败，只要conn参数是有效的，并且数据库的游标资源足够，connect方法不会返回失败。
>
> // 程序员一般不必关心connect方法的返回值。
>
> //
> 注意，每个sqlstatement只需要绑定一次，在绑定新的connection前，必须先调用disconnect方法。
>
> int connect(connection \*conn);
>
> // 取消与数据库连接池的绑定。
>
> // 返回值：0-成功，其它失败，程序员一般不必关心返回值。
>
> int disconnect();
>
> // 准备SQL语句。
>
> // 参数说明：这是一个可变参数，用法与printf函数相同。
>
> // 返回值：0-成功，其它失败，程序员一般不必关心返回值。
>
> // 注意：如果SQL语句没有改变，只需要prepare一次就可以了。
>
> int prepare(const char \*fmt,\...);
>
> // 绑定输入变量的地址。
>
> //
> position：字段的顺序，从1开始，必须与prepare方法中的SQL的序号一一对应。
>
> //
> value：输入变量的地址，如果是字符串，内存大小应该是表对应的字段长度加1。
>
> //
> len：如果输入变量的数据类型是字符串，用len指定它的最大长度，建议采用表对应的字段长度。
>
> // 返回值：0-成功，其它失败，程序员一般不必关心返回值。
>
> //
> 注意：1）如果SQL语句没有改变，只需要bindin一次就可以了，2）绑定输入变量的总数不能超过256个。
>
> int bindin(unsigned int position,int \*value);
>
> int bindin(unsigned int position,long \*value);
>
> int bindin(unsigned int position,unsigned int \*value);
>
> int bindin(unsigned int position,unsigned long \*value);
>
> int bindin(unsigned int position,float \*value);
>
> int bindin(unsigned int position,double \*value);
>
> int bindin(unsigned int position,char \*value,unsigned int len);
>
> // 绑定输出变量的地址。
>
> // position：字段的顺序，从1开始，与SQL的结果集一一对应。
>
> //
> value：输出变量的地址，如果是字符串，内存大小应该是表对应的字段长度加1。
>
> //
> len：如果输出变量的数据类型是字符串，用len指定它的最大长度，建议采用表对应的字段长度。
>
> // 返回值：0-成功，其它失败，程序员一般不必关心返回值。
>
> //
> 注意：1）如果SQL语句没有改变，只需要bindout一次就可以了，2）绑定输出变量的总数不能超过256个。
>
> int bindout(unsigned int position,int \*value);
>
> int bindout(unsigned int position,long \*value);
>
> int bindout(unsigned int position,unsigned int \*value);
>
> int bindout(unsigned int position,unsigned long \*value);
>
> int bindout(unsigned int position,float \*value);
>
> int bindout(unsigned int position,double \*value);
>
> int bindout(unsigned int position,char \*value,unsigned int len);
>
> // 执行SQL语句。
>
> //
> 返回值：0-成功，其它失败，失败的代码在m_cda.rc中，失败的描述在m_cda.message中。
>
> //
> 如果成功的执行了非查询语句，在m_cda.rpc中保存了本次执行SQL影响记录的行数。
>
> // 程序员必须检查execute方法的返回值。
>
> int execute();
>
> // 执行SQL语句。
>
> //
> 如果SQL语句不需要绑定输入和输出变量（无绑定变量、非查询语句），可以直接用此方法执行。
>
> // 参数说明：这是一个可变参数，用法与printf函数相同。
>
> //
> 返回值：0-成功，其它失败，失败的代码在m_cda.rc中，失败的描述在m_cda.message中，
>
> //
> 如果成功的执行了非查询语句，在m_cda.rpc中保存了本次执行SQL影响记录的行数。
>
> // 程序员必须检查execute方法的返回值。
>
> int execute(const char \*fmt,\...);
>
> // 从结果集中获取一条记录。
>
> //
> 如果执行的SQL语句是查询语句，调用execute方法后，会产生一个结果集（存放在数据库的缓冲区中）。
>
> // next方法从结果集中获取一条记录，把字段的值放入已绑定的输出变量中。
>
> //
> 返回值：0-成功，1403-结果集已无记录，其它-失败，失败的代码在m_cda.rc中，失败的描述在m_cda.message中。
>
> //
> 返回失败的原因主要有两个：1）与数据库的连接已断开；2）绑定输出变量的内存太小。
>
> // 每执行一次next方法，m_cda.rpc的值加1。
>
> // 程序员必须检查next方法的返回值。
>
> int next();
>
> };

# 五、程序流程

freecplus框架把对PostgreSQL数据库操作的SQL语句分为两种：有结果集的SQL语句和无结果集的SQL语句。

如果SQL语句被执行后，有结果集的产生，称为有结果集的SQL，即数据查询语言DQL，以select关键字，各种简单查询，连接查询等都属于DQL。

如果SQL语句被执行后，没有结果集的产生，称为无结果集的SQL，包括数据定义语言DDL（主要是create、drop和alter）和数据操纵语言DML（insert、update和insert）。

也可以这么说，查询的SQL语句会产生结果集，其它的SQL语句不会产生结果集。

## 1、无结果集SQL的程序的流程

> ![](/images/97/media/image1.emf)

这是一个完程的流程，在实际开发中，如果是执行简单的SQL语句，第6步和第7步可能不需要，如果SQL语句只执行一次，第7步和第8步之间的循环也不需要。

## 2、有结果集SQL的程序的流程

> ![](/images/97/media/image2.emf)

这是一个完程的流程，在实际开发中，如果是执行简单的查询语句，第6步、第7步和第8步可能不需要，如果结果集中最多只有一条记录，第10步和第11步之间的循环也不需要。

# 六、示例程序

## 1、创建超女信息表

**示例（createtable.cpp）**

> /\*
>
> \*
> 程序名：createtable.cpp，此程序演示freecplus框架操作PostgreSQL数据库（创建表）。
>
> \* 作者：C语言技术网(www.freecplus.net) 日期：20190525
>
> \*/
>
> #include \"\_postgresql.h\" // freecplus框架操作PostgreSQL的头文件。
>
> int main(int argc,char \*argv\[\])
>
> {
>
> connection conn; // 数据库连接池。
>
> // 登录数据库，返回值：0-成功，其它-失败。
>
> // 失败代码在conn.m_cda.rc中，失败描述在conn.m_cda.message中。
>
> if (conn.connecttodb(\"host=172.16.0.15 user=postgres password=pwdidc
> dbname=postgres port=5432\",\"gbk\")!=0)
>
> {
>
> printf(\"connect database failed.\\n%s\\n\",conn.m_cda.message);
> return -1;
>
> }
>
> sqlstatement stmt(&conn); // 操作SQL语句的对象。
>
> // 准备创建表的SQL语句。
>
> //
> 超女表girls，超女编号id，超女姓名name，体重weight，报名时间btime，超女说明memo，超女图片pic。
>
> stmt.prepare(\"\\
>
> create table girls(id int,\\
>
> name varchar(30),\\
>
> weight numeric(8,2),\\
>
> btime timestamp,\\
>
> memo text,\\
>
> pic bytea,\\
>
> primary key (id))\");
>
> // prepare方法不需要判断返回值。
>
> // 执行SQL语句，一定要判断返回值，0-成功，其它-失败。
>
> // 失败代码在stmt.m_cda.rc中，失败描述在stmt.m_cda.message中。
>
> if (stmt.execute() != 0)
>
> {
>
> printf(\"stmt.execute()
> failed.\\n%s\\n%s\\n\",stmt.m_sql,stmt.m_cda.message); return -1;
>
> }
>
> //
> 注意，在postgresql数据库中，创建表也要提交事务，和Oracle、MySQL数据库不同。
>
> conn.commit();
>
> printf(\"create table girls ok.\\n\");
>
> }

**运行效果**

![](/images/97/media/image3.png){width="7.263888888888889in"
height="0.8333333333333334in"}

## 2、向超女表中插入5条记录

**示例（inserttable.cpp）**

> /\*
>
> \*
> 程序名：inserttable.cpp，此程序演示freecplus框架操作PostgreSQL数据库（向表中插入5条记录）。
>
> \* 作者：C语言技术网(www.freecplus.net) 日期：20190525
>
> \*/
>
> #include \"\_postgresql.h\" // freecplus框架操作PostgreSQL的头文件。
>
> // 定义用于超女信息的结构，与表中的字段对应。
>
> struct st_girls
>
> {
>
> long id; // 超女编号，用long数据类型对应Oracle无小数的number(10)。
>
> char name\[11\]; // 超女姓名，用char\[31\]对应Oracle的varchar2(30)。
>
> double weight; //
> 超女体重，用double数据类型对应Oracle有小数的number(8,2)。
>
> char btime\[20\]; //
> 报名时间，用char对应Oracle的date，格式：\'yyyy-mm-dd hh24:mi:ssi\'。
>
> } stgirls;
>
> int main(int argc,char \*argv\[\])
>
> {
>
> connection conn; // 数据库连接类
>
> // 登录数据库，返回值：0-成功，其它-失败。
>
> // 失败代码在conn.m_cda.rc中，失败描述在conn.m_cda.message中。
>
> if (conn.connecttodb(\"host=172.16.0.15 user=postgres password=pwdidc
> dbname=postgres port=5432\",\"gbk\")!=0)
>
> {
>
> printf(\"connect database failed.\\n%s\\n\",conn.m_cda.message);
> return -1;
>
> }
>
> sqlstatement stmt(&conn); // 操作SQL语句的对象。
>
> // 准备插入表的SQL语句。
>
> stmt.prepare(\"\\
>
> insert into girls(id,name,weight,btime) \\
>
> values(:1,:2,:3,to_date(:4,\'yyyy-mm-dd hh24:mi:ss\'))\");
>
> // prepare方法不需要判断返回值。
>
> // 为SQL语句绑定输入变量的地址，bindin方法不需要判断返回值。
>
> stmt.bindin(1,&stgirls.id);
>
> stmt.bindin(2, stgirls.name,10);
>
> stmt.bindin(3,&stgirls.weight);
>
> stmt.bindin(4, stgirls.btime,19);
>
> // 模拟超女数据，向表中插入5条测试信息。
>
> for (int ii=1;ii\<=5;ii++)
>
> {
>
> memset(&stgirls,0,sizeof(stgirls)); // 结构体变量初始化。
>
> // 为结构体变量的成员赋值。
>
> stgirls.id=ii; // 超女编号。
>
> sprintf(stgirls.name,\"超女%02d\",ii); // 超女姓名。
>
> stgirls.weight=ii\*2.11; // 超女体重。
>
> strcpy(stgirls.btime,\"2018-03-01 12:25:31\"); // 报名时间。
>
> // 执行SQL语句，一定要判断返回值，0-成功，其它-失败。
>
> // 失败代码在stmt.m_cda.rc中，失败描述在stmt.m_cda.message中。
>
> if (stmt.execute() != 0)
>
> {
>
> printf(\"stmt.execute()
> failed.\\n%s\\n%s\\n\",stmt.m_sql,stmt.m_cda.message); return -1;
>
> }
>
> printf(\"成功插入了%ld条记录。\\n\",stmt.m_cda.rpc); //
> stmt.m_cda.rpc是本次执行SQL影响的记录数。
>
> }
>
> printf(\"insert table girls ok.\\n\");
>
> conn.commit(); // 提交数据库事务。
>
> }

**运行效果**

![](/images/97/media/image4.png){width="7.263888888888889in"
height="1.4791666666666667in"}

## 3、更新超女表中的记录

**示例（updatetable.cpp）**

> /\*
>
> \*
> 程序名：updatetable.cpp，此程序演示freecplus框架操作PostgreSQL数据库（修改表中的记录）。
>
> \* 作者：C语言技术网(www.freecplus.net) 日期：20190525
>
> \*/
>
> #include \"\_postgresql.h\" // freecplus框架操作PostgreSQL的头文件。
>
> int main(int argc,char \*argv\[\])
>
> {
>
> connection conn; // 数据库连接类
>
> // 登录数据库，返回值：0-成功，其它-失败。
>
> // 失败代码在conn.m_cda.rc中，失败描述在conn.m_cda.message中。
>
> if (conn.connecttodb(\"host=172.16.0.15 user=postgres password=pwdidc
> dbname=postgres port=5432\",\"gbk\")!=0)
>
> {
>
> printf(\"connect database failed.\\n%s\\n\",conn.m_cda.message);
> return -1;
>
> }
>
> sqlstatement stmt(&conn); // 操作SQL语句的对象。
>
> char strbtime\[20\]; // 用于存放超女的报名时间。
>
> memset(strbtime,0,sizeof(strbtime));
>
> strcpy(strbtime,\"2019-12-20 09:45:30\");
>
> // 准备更新数据的SQL语句，不需要判断返回值。
>
> stmt.prepare(\"\\
>
> update girls set btime=to_date(:1,\'yyyy-mm-dd hh24:mi:ss\') where
> id\>=2 and id\<=4\");
>
> // prepare方法不需要判断返回值。
>
> // 为SQL语句绑定输入变量的地址，bindin方法不需要判断返回值。
>
> stmt.bindin(1,strbtime,19);
>
> //
> 如果不采用绑定输入变量的方法，把strbtime的值直接写在SQL语句中也是可以的，如下：
>
> /\*
>
> stmt.prepare(\"\\
>
> update girls set btime=to_date(\'%s\',\'yyyy-mm-dd hh24:mi:ss\') where
> id\>=2 and id\<=4\",strbtime);
>
> \*/
>
> // 执行SQL语句，一定要判断返回值，0-成功，其它-失败。
>
> // 失败代码在stmt.m_cda.rc中，失败描述在stmt.m_cda.message中。
>
> if (stmt.execute() != 0)
>
> {
>
> printf(\"stmt.execute()
> failed.\\n%s\\n%s\\n\",stmt.m_sql,stmt.m_cda.message); return -1;
>
> }
>
> //
> 请注意，stmt.m_cda.rpc变量非常重要，它保存了SQL被执行后影响的记录数。
>
> printf(\"本次更新了girls表%ld条记录。\\n\",stmt.m_cda.rpc);
>
> // 提交事务
>
> conn.commit();
>
> }

**运行效果**

![](/images/97/media/image5.png){width="7.25in"
height="0.9166666666666666in"}

## 4、查询超女表中的记录

**示例（selecttable.cpp）**

> /\*
>
> \*
> 程序名：selecttable.cpp，此程序演示freecplus框架操作PostgreSQL数据库（查询表中的记录）。
>
> \* 作者：C语言技术网(www.freecplus.net) 日期：20190525
>
> \*/
>
> #include \"\_postgresql.h\" // freecplus框架操作PostgreSQL的头文件。
>
> // 定义用于超女信息的结构，与表中的字段对应。
>
> struct st_girls
>
> {
>
> long id; // 超女编号，用long数据类型对应Oracle无小数的number(10)。
>
> char name\[31\]; // 超女姓名，用char\[31\]对应Oracle的varchar2(30)。
>
> double weight; //
> 超女体重，用double数据类型对应Oracle有小数的number(8,2)。
>
> char btime\[20\]; //
> 报名时间，用char对应Oracle的date，格式：\'yyyy-mm-dd hh24:mi:ss\'。
>
> } stgirls;
>
> int main(int argc,char \*argv\[\])
>
> {
>
> connection conn; // 数据库连接类
>
> // 登录数据库，返回值：0-成功，其它-失败。
>
> // 失败代码在conn.m_cda.rc中，失败描述在conn.m_cda.message中。
>
> if (conn.connecttodb(\"host=172.16.0.15 user=postgres password=pwdidc
> dbname=postgres port=5432\",\"gbk\")!=0)
>
> {
>
> printf(\"connect database failed.\\n%s\\n\",conn.m_cda.message);
> return -1;
>
> }
>
> sqlstatement stmt(&conn); // 操作SQL语句的对象。
>
> int iminid,imaxid; // 查询条件最小和最大的id。
>
> // 准备查询表的SQL语句。
>
> stmt.prepare(\"\\
>
> select id,name,weight,to_char(btime,\'yyyy-mm-dd hh24:mi:ss\') from
> girls where id\>=:1 and id\<=:2\");
>
> // prepare方法不需要判断返回值。
>
> // 为SQL语句绑定输入变量的地址，bindin方法不需要判断返回值。
>
> stmt.bindin(1,&iminid);
>
> stmt.bindin(2,&imaxid);
>
> // 为SQL语句绑定输出变量的地址，bindout方法不需要判断返回值。
>
> stmt.bindout(1,&stgirls.id);
>
> stmt.bindout(2, stgirls.name,30);
>
> stmt.bindout(3,&stgirls.weight);
>
> stmt.bindout(4, stgirls.btime,19);
>
> iminid=2; // 指定待查询记录的最小id的值。
>
> imaxid=4; // 指定待查询记录的最大id的值。
>
> // 执行SQL语句，一定要判断返回值，0-成功，其它-失败。
>
> // 失败代码在stmt.m_cda.rc中，失败描述在stmt.m_cda.message中。
>
> if (stmt.execute() != 0)
>
> {
>
> printf(\"stmt.execute()
> failed.\\n%s\\n%s\\n\",stmt.m_sql,stmt.m_cda.message); return -1;
>
> }
>
> //
> 本程序执行的是查询语句，执行stmt.execute()后，将会在数据库的缓冲区中产生一个结果集。
>
> while (1)
>
> {
>
> memset(&stgirls,0,sizeof(stgirls)); // 先把结构体变量初始化。
>
> //
> 从结果集中获取一条记录，一定要判断返回值，0-成功，1403-无记录，其它-失败。
>
> // 在实际开发中，除了0和1403，其它的情况极少出现。
>
> if (stmt.next() !=0) break;
>
> // 把获取到的记录的值打印出来。
>
> printf(\"id=%ld,name=%s,weight=%.02f,btime=%s\\n\",stgirls.id,stgirls.name,stgirls.weight,stgirls.btime);
>
> }
>
> //
> 请注意，stmt.m_cda.rpc变量非常重要，它保存了SQL被执行后影响的记录数。
>
> printf(\"本次查询了girls表%ld条记录。\\n\",stmt.m_cda.rpc);
>
> }

**运行效果**

![](/images/97/media/image6.png){width="7.263888888888889in"
height="1.3402777777777777in"}

## 5、查询超女表中的记录数

**示例（counttable.cpp）**

> /\*
>
> \*
> 程序名：counttable.cpp，此程序演示freecplus框架操作PostgreSQL数据库（查询表中的记录数）。
>
> \* 作者：C语言技术网(www.freecplus.net) 日期：20190525
>
> \*/
>
> #include \"\_postgresql.h\" // freecplus框架操作PostgreSQL的头文件。
>
> int main(int argc,char \*argv\[\])
>
> {
>
> connection conn; // 数据库连接类
>
> // 登录数据库，返回值：0-成功，其它-失败。
>
> // 失败代码在conn.m_cda.rc中，失败描述在conn.m_cda.message中。
>
> if (conn.connecttodb(\"host=172.16.0.15 user=postgres password=pwdidc
> dbname=postgres port=5432\",\"gbk\")!=0)
>
> {
>
> printf(\"connect database failed.\\n%s\\n\",conn.m_cda.message);
> return -1;
>
> }
>
> sqlstatement stmt(&conn); // 操作SQL语句的对象。
>
> int icount=0; // 用于存放查询结果的记录数。
>
> //
> 准备查询表的SQL语句，把查询条件直接写在SQL语句中，没有采用绑定输入变量的方法。
>
> stmt.prepare(\"select count(\*) from girls where id\>=2 and id\<=4\");
>
> // prepare方法不需要判断返回值。
>
> // 为SQL语句绑定输出变量的地址，bindout方法不需要判断返回值。
>
> stmt.bindout(1,&icount);
>
> // 执行SQL语句，一定要判断返回值，0-成功，其它-失败。
>
> // 失败代码在stmt.m_cda.rc中，失败描述在stmt.m_cda.message中。
>
> if (stmt.execute() != 0)
>
> {
>
> printf(\"stmt.execute()
> failed.\\n%s\\n%s\\n\",stmt.m_sql,stmt.m_cda.message); return -1;
>
> }
>
> //
> 本程序执行的是查询语句，执行stmt.execute()后，将会在数据库的缓冲区中产生一个结果集。
>
> //
> 但是，在本程序中，结果集永远只有一条记录，调用stmt.next()一次就行，不需要循环。
>
> stmt.next();
>
> printf(\"girls表中符合条件的记录数是%d。\\n\",icount);
>
> }

**运行效果**

![](/images/97/media/image7.png){width="7.268055555555556in"
height="0.9284722222222223in"}

## 7、删除超女表中的记录

**示例（deletetable.cpp）**

> /\*
>
> \*
> 程序名：deletetable.cpp，此程序演示freecplus框架操作PostgreSQL数据库（删除表中的记录）。
>
> \* 作者：C语言技术网(www.freecplus.net) 日期：20190525
>
> \*/
>
> #include \"\_postgresql.h\" // freecplus框架操作PostgreSQL的头文件。
>
> int main(int argc,char \*argv\[\])
>
> {
>
> connection conn; // 数据库连接类
>
> // 登录数据库，返回值：0-成功，其它-失败。
>
> // 失败代码在conn.m_cda.rc中，失败描述在conn.m_cda.message中。
>
> if (conn.connecttodb(\"host=172.16.0.15 user=postgres password=pwdidc
> dbname=postgres port=5432\",\"gbk\")!=0)
>
> {
>
> printf(\"connect database failed.\\n%s\\n\",conn.m_cda.message);
> return -1;
>
> }
>
> sqlstatement stmt(&conn); // 操作SQL语句的对象。
>
> // 执行SQL语句，一定要判断返回值，0-成功，其它-失败。
>
> // 失败代码在stmt.m_cda.rc中，失败描述在stmt.m_cda.message中。
>
> //
> 如果不需要绑定输入和输出变量，用stmt.execute()方法直接执行SQL语句，不需要stmt.prepare()。
>
> if (stmt.execute(\"delete from girls where id\>=2 and id\<=4\") != 0)
>
> {
>
> printf(\"stmt.execute()
> failed.\\n%s\\n%s\\n\",stmt.m_sql,stmt.m_cda.message); return -1;
>
> }
>
> //
> 请注意，stmt.m_cda.rpc变量非常重要，它保存了SQL被执行后影响的记录数。
>
> printf(\"本次从girls表中删除了%ld条记录。\\n\",stmt.m_cda.rpc);
>
> // 提交事务
>
> conn.commit();
>
> }

**运行效果**

![](/images/97/media/image8.png){width="7.256944444444445in"
height="0.8402777777777778in"}

## 8、与Oracle的兼容性处理

在封装sqlstatement类的时候，为了与Oracle兼容，做了以下方面的处理：

1）在PostgreSQL中，绑定输入和输出变量采用的是\"\$\"，Oracle采用的是\":n\"（n表示变量的序号），在sqlstatement的prepare方法中，把\":n\"替换成了\"\$\"。

2）在PostgreSQL中，把字符串输换为日期时间的函数是to_timestamp，Oracle是to_date，在sqlstatement的prepare方法中，把to_date替换成to_timestamp。

4）PostgreSQL的sqlstatement类绑定输入或输出变量的最大数量缺省是256，在\"\_mysql.h\"头文件中定义了MAXPARAMS宏，您可以根据实际需求修改它。

> //
> 执行SQL语句前绑定输入或输出变量个数的最大值，256是很大的了，可以根据实际情况调整。
>
> #define MAXPARAMS 256

5）sqlstatement类绑定输入或输出变量时，如果是字符串，最大长度缺省是2000，在\"\_postgresql.h\"头文件中定义了MAXFIELDLENGTH宏，您可以根据实际需求修改它。

> //
> 如果绑定输入或输出变量是字符串，指定字符串的最大长度，不包括字符串的结束符。
>
> #define MAXFIELDLENGTH 2000

## 9、text和bytea字段的操作

PostgreSQL提供的库函数支持对text和bytea字段的操作，本人的技术水平有限，找不到这方面的资料和示例程序，所以还没有封装对text和bytea字段的操作，希望各位能提供技术帮助，通过C语言技术网与我联系，我们共同完善freecplus框架，非常感谢。

# 七、应用经验

本文提供的示例程序看上去简单，实则很精妙，希望大家多多思考，慢慢体会。

为了让大家完全掌握connection和sqlstatement类的用法，我将录制freecplus框架的专题视频，请大家多关注C语言技术网（www.freecplus.net）发布的内容。

# 八、版权声明

C语言技术网原创文章，转载请说明文章的来源、作者和原文的链接。

来源：C语言技术网（[www.freecplus.net](http://www.freecplus.net)）

作者：码农有道
