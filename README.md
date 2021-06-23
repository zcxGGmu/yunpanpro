# cloud-disk
分布式网络云盘项目

### 项目环境
* Ubuntu 18.04
* Qt（windows）

### 服务端包含技术有：
* MySQL数据库、Redis数据库
* 分布式FastDFS集群搭建
* Nginx搭配FastCGI、Nginx搭配FastDFS
* 通过C API操作FastDFS、FastCGI、Redis、MySQL
* 以HTTP的接口接收客户端的数据

### 客户端包含技术有：
* Qt实现客户端
* 以HTTP的接口访问服务端

