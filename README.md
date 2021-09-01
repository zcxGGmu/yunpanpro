
**项目的简单介绍和部分流程解析，想进一步了解的可以移步CSDN文章[云盘项目分析与介绍](https://blog.csdn.net/weixin_46084744/article/details/119983856)**

# 项目整体架构

* <font color=red>**Qt 作为云盘的客户端，**</font>支持文件的上传、下载、删除、共享等；
* 后端使用<font color=red>**Nginx作为代理服务器，**</font>将Qt客户端发送过来的动态请求转发给FastCGI程序进行处理；
* <font color=red>**FastCGI**</font>对转发的请求进行解析处理；
* <font color=red>**FastDFS**</font>中`storage`节点保存着客户端的文件；
* <font color=red>**MySQL**</font>保存着文件信息；



<br/>
<br/>
<br/>

# MySQL 表设计
## 用户信息表 (user_info)
>此表用来存储每个用户的信息，当注册新用户的时候就会向这个表中插入一条数据。

| 字段名 | 意义  | 类型	| 是否允许为空值  | 默认值 | 备注  |
|--|--|--|--|--|--|
| id | 序号 | bigint | 否  | 无 | 自动递增，主键 |
| user_name | 用户名称 | varchar | 否  | ' ' |  |
| nick_name | 用户昵称  | varcbar | 否  | ' ' |  |
| phone | 手机号码 | varchar | 否  | ' ' |  |
| email | 邮箱 | varchar | 否  | ' ' |  |
| password | 密码 | varchar | 是  | ' ' | 保存md5后的加密值 |
| create_time | 用户创建时间 | TIMESTAMP | 否  | CURRENT_TIMES_TAMP |  |


<br/>

## 文件信息表（file_info）
> * 此表格用来存储FastDFS中所有文件信息；
> * 其中：
> 	* `file_id`：文件的路径;
> 	* `url`：可以通过Nginx用哪个url访问到文件;
> 	* `count`：被任何用户下载，对应文件的count数量就会增加；

| 字段名 | 意义  | 类型	| 是否允许为空值  | 默认值 | 备注  |
|--|--|--|--|--|--|
| id | 序号 | bigint | 否  | 无 | 自动递增，主键 |
| md5 | 文件md5 | varchar | 否  | ' ' |  |
| file_id | 文件id | varchar | 否  | ' ' | 格式例如：/group1/M00/00/00/xxx.jpg |
| url | 文件url | varchar | 否  | ' ' | 格式例如：192.168.1.2:80/group1/M00/00/00/xxx.jpg |
| size | 文件大小| bigint | 否  | 0 | 以字节为单位 |
| type | 文件类型 | varchar | 是  | ' ' | png、zip、mp4等 |
| count | 文件引用计数 | int | 否 | 0 | 默认为1，每增加一个用户拥有此文件，对应计数器加1 |

<br/>

## 用户文件列表（user_file_list）
>与`file_info`表类似，但不同的是此表每一行的文件都会指出其拥有者是谁。
> * `shared_status`：表示此用户的这个文件是否被共享出来；
> * `pv`：文件被下载一次，这个字段就会加1；

| 字段名 | 意义  | 类型	| 是否允许为空值  | 默认值 | 备注  |
|--|--|--|--|--|--|
| id | 序号 | int | 否  | 无 | 自动递增，主键 |
| user | 文件所属用户 | varchar | 否  | 无 |  |
| md5 | 文件md5 | varchar | 否  | 无 |  |
| file_name | 文件名字 | varchar | 否  | ' ' |  |
| shared_status | 共享状态 | int | 否  | 0 | 0表示没有共享;1表示共享 |
| pv | 文件下载量 | int | 否  | 0 | 默认值为0;每下载一次,这个数量就加1 |
| create_time | 文件创建时间 | timestamp | 否  | CURRENT_TIMESTAMP |  |


<br/>

## 用户文件数量表（user_file_count）
>此表用来统计每个用户拥有的文件数量；
>当客户端第一次登陆的时候，其就会先发送一个HTTP请求来查询自己拥有多少个文件，然后再显示出来。
>`注意`：因为我们有一个共享文件的功能，所以我们的服务端也有一个“共享用户”角色，xxx_share_xxx_file_xxx_list_xxx_count_xxx表示共享用户。

 | 字段名 | 意义  | 类型	| 是否允许为空值  | 默认值 | 备注  |
|--|--|--|--|--|--|
| id | 序号 | int | 否  | 无 | 自动递增，主键 |
| user | 文件所属用户 | varchar | 否  | 无 |  |
| count | 此用户的文件数量 | int | 否  | 0 |  |


<br/>

## 共享文件列表（share_file_list）
>因为客户端可以共享文件，所以每当共享自己的文件之后，就会把自己共享的文件信息添加到这个表中。

| 字段名 | 意义  | 类型	| 是否允许为空值  | 默认值 | 备注  |
|--|--|--|--|--|--|
| id | 序号 | int | 否  | 无 | 自动递增，主键 |
| user | 文件所属用户 | varchar | 否  | 无 |  |
| md5 | 文件md5 | varchar |  否 | 无 |  |
| pv | 文件下载量 | int | 否  | 无 | 默认值为1；每下载一次，这个数就加1 |
| file_name | 文件名字 | varchar | 否  | 无 |  |
| create_time | 文件共享时间 | timestamp | 否  | CURRENT_TIMESTAMP |  |


<br/>
<br/>
<br/>

# FastDFS 






<br/>
<br/>
<br/>

# Nginx访问FastCGI、FastDFS






<br/>
<br/>
<br/>

# 各功能流程解析
## 注册
<br/>

客户端向Nginx发送一个HTTP请求（URL以reg结尾），Nginx接收到这个请求将请求传递给FastCGI，FastCGI通过自己一些内置的环境变量获取到URL和报文主体进行解析，然后与数据库交互得到结果，将结果（以json格式回复）返回给客户端，客户端根据状态码和主体内容进行处理。

![在这里插入图片描述](https://img-blog.csdnimg.cn/e978e3a762464d89bbf72d5c5909fe22.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAaGVsbG9oZWxs5Li2,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

<br/>

### 客户端流程
<font color=MediumPurple>***基本流程***</font>

1. 首先需要手动设置一下服务器的IP和端口；  
2. 用户点击注册按钮后，会在`login.cpp`文件中响应`on_register_btn_clicked()`函数，从控件中取出用户输入的数据;
3. 进行数据校验，判断用户输入的各项内容格式是否正确，使用到了`QRegExp`类；
4. 调用setRegisterJson()函数，将获取到的用户输入内容转换为JSON格式（因为这些内容要作为HTTP请求报文的主体发送出去）；
5. 主体转换成JSON格式后，下面开始构造URL和请求报文，以`post`方式将报文发送给服务端；
6. 然后客户端这边等待服务端的响应消息并解析，因为服务端给自己返回的内容也是JSON格式的，因此需要我们解析JSON中`code`字符串对应的值，再进行后续处理；
7. `002`则显示“注册成功，请登录”，`003`显示“该用户已存在”，`004`显示“注册失败“。


<br/>

### 服务端流程
1. 当Nginx接收到的URL中含有`/reg`时，就会将请求转发给127.0.0.1:10001这个FastCGI程序进行处理；
```shell
location /reg {
	fastcgi_pass 127.0.0.1:100001;
	include fastcgi.conf;
}
```
2. 查看服务端脚本`fcgi.sh`可以看到，`127.0.0.1:100001`对应的FastCGI程序名为register;
3. FastCGI程序通过`FCGI_Accept()`接口阻塞等待用户连接，当用户连接到来时唤醒此进程，然后通过内置的`CONTENT_LENGTH`环境变量判断是否内容，没有则报错有则继续处理；
4. 由于客户端发送过来的HTTP请求报文的主体部分是`JSON`格式的，所以需要调用JSON的CAPI，将JSON信息解析出来，获得用户注册信息；
5. 连接MySQL，查看该用户是否在MySQL中存在，存在的话就直接返回否则就将这条用户数据插入到`user_info`表中；
6. 根据返回值来构造不同的JSON串，最后返回给Nginx，Nginx再发送给客户端；

`注：为了保证传输过程中密码不被窃取，客户端登录的时候会把用户密码经过MD5加密，然后再传递给服务端。`

<br/>
<br/>

## 登录
登陆的时候会发送三个 `URL` 请求报文给服务端，分别为：

* `/login URL`：用来判断这个用户的账号和密码是否正确（服务端从MySQL中判断）；
* `/myfiles?cmd=count URL`：登录成功后，客户端再发送一个请求获得自己的文件数量；
* `/myfiles?cmd=normal`： 获取该登录用户对应的文件信息；

<font color=#0000FF >**登录业务流程如下：**</font>

1. <font color=red>**第一次发送登录URL：**</font>客户端通过发送一个HTTP请求给Nginx（URL以login为结尾），Nginx接收到这个请求后转发给FastCGI，FastCGI通过自己内置的环境变量获取到URL同时对报文主体进行解析，提取数据（用户名和密码）然后连接MySQL验证用户名和密码，验证成功后生成一个`token`字符串，然后将状态码和`token`打包发送给Nginx；
2. <font color=red>**登录成功后获取用户文件数量和文件信息：**</font>登录成功之后先调用`/myfiles?cmd=count`并携带`user`和`token`信息来获取当前登录用户在服务端拥有文件的数量，后端进行token验证和MySQL查询后将结果返回，客户端解析响应报文后发现文件数等于零就结束； <font color=#0000FF >**如果文件数量大于零，**</font>调用`myfiles?cmd=normal`来获取文件信息，后端查询并返回的文件结果集如下所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/cff66900940841af9eb6a98f1ffb5d58.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAaGVsbG9oZWxs5Li2,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)
<br/>

## 客户端流程
这里讲一下**登录成功后，客户端的后续处理方式：**







<br/>

## 服务端流程








<br/>
<br/>

## 上传文件、删除文件





<br/>
<br/>

## 按下载量升序、降序






<br/>
<br/>

## 共享文件列表






<br/>
<br/>

## 
