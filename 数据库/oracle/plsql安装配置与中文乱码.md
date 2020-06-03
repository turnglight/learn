
# 安装配置
+ 安装oracle11的Pl/sql developer11
+ 安装oracle11的client
    + 下载地址:<a href="http://www.oracle.com/technetwork/database/features/instant-client/index-097480.html">http://www.oracle.com/technetwork/database/features/instant-client/index-097480.html</a>

+ 配置环境变量
    ~~~
    ORACLE_HOME = C:\instantclient_11_2(instantclient的安装路径)
    TNS_ADMIN = C:\instantclient_11_2
    NLS_LANG = SIMPLIFIED CHINESE_CHINA.ZHS16GBK
    ~~~

+ 新建tnsnames.ora文件
    ~~~
     192.168.0.111 =
     (DESCRIPTION =
        (ADDRESS_LIST =
            (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.0.111)(PORT =1521))
        )
        (CONNECT_DATA =
            (SERVICE_NAME = ORCL)
        )
     )
    ~~~

+ 配置plsql developer
    + tools -> preferences -> connection
        + oracle home填入instantclient的安装路径
        + ocl library输入oci.dll的文件路径（也是在instantclient的安装路径下面）

# 客户端中文乱码

+ 获取服务器的编码
    > select userenv('language') from dual;

+ 获取客户端的编码
    > select * from V$NLS_PARAMETERS

> 如果客户端与服务器的编码不一致，就会出现乱码
+ 设置环境变量NLS_LANG,将NLS_LANG=(服务器的编码)
+ 重启PLSQL


# 客户端日期格式设置
+ 添加环境变量
    + nls_date_format=YYYY-MM-DD HH24:MI:SS
    + nls_timestamp_format=YYYY-MM-DD HH24:MI:SS