# Summary
* [关于博客说明](README.md)

* bat脚本
  * [bat脚本中巧用sed grep awk命令](bat/bat-use-sedawk.md)

* linux
  * ["-"在tar命令中的巧用](linux/tar-deal.md)
  * [利用iptables或防火墙指定ip访问服务器某个端口](linux/limit_ip.md)
  * [如何设置centos7.6和Ubuntu19.4的ip4](linux/set_ip.md)
  * [Linux中磁盘扩展与缩减](linux/extend_disk.md)
  * [二进制安装包的制作](linux/how_to_made_bin.md)
  * [heredocument的巧用](linux/use_heredoc.md)

* docker
  * [docker 安装与优化](docker/docker-install.md)
  * [harbor的安装和注意事项](docker/harbor-install.md)
  * [harbor仓库镜像同步](docker/harbor-sync.md)
  * [如何制作出合适的镜像](docker/dockerfile-rule.md)
  * [逐个和批量导出导入docker镜像](docker/save_load_images.md)

* rancher和kubernetes
  *  [kubectl常用命令汇总（以及curl注入）](k8s/kubectl-user-instruction.md)
  *  [docker run image -args对应yaml语法/rancher UI操作方式](k8s/docker-run-and-k8s-command.md)
  *  [在线安装高可用的rancher](k8s/rancher_online_installation.md)
  *  [离线helm安装高可用rancher](k8s/rancher_offline_installation.md)
  *  [rancher中快速部署应用](k8s/deploy_app_in_rancher.md)
  *  [利用rancher商店搭建es和beat](k8s/use_appstore_deploy_es_in_rancher.md)
  *  [rancher平台不可用下如何使用kubectl命令](k8s/how_to_use_kubectl_noserver.md)

* Jenkins
  * [Jenkins调用docker编译程序](jenkins/jenkins-slave-for-docker.md)
  * [利用docker 镜像，快速搭建Jenkins环境](jenkins/install-jenkins.md)
  * [jenkins自由风格构建job](jenkins/freestyle_build_in_jenkins.md)
  * [jenkins多种pipeline构建job](jenkins/variety_pipeline_build.md)
  * [jenkins config and job的备份](jenkins/thinBackup_jenkins.md)

* 持续集成项目实战
  * [利用Jenkins+gitbook+git+node-ejs搭建一套实时更新多版本文档网站](other/devops_practices_gitbook_web.md)
  * [采用jenkins+harbor+kubernetes自动部署最新程序](other/devops_practices_k8s.md)

* shell
  * [shell 脚本中获取脚本所在路径](shell/get_dir_in_shell.md)
  * [sed不常见用法](shell/sed_use_hard.md)
  * [getopt与getopts用法](shell/getopt_and_getopts_use.md)
  * [选择项用法](shell/ps3_use.md)
  * [字符串截取](shell/trim_string.md)

* jmeter

  * [利用jmeter模拟手机接口测试](jmeter/use_jmeter_test_app.md)

* 证书申请与制作

  * [私有证书制作](ca/make_key.md)

* NGINX
  * [如何利用nginx指向本地静态网页](nginx/direct_static_web.md)
  * [如何利用nginx实现负载均衡和反向代理](nginx/load_balance.md)
  * [nginx其他知识](nginx/nginx_other.md)

* git
  * [git日常使用命令](git/git-use.md)
  * [gitlab搭建和备份](git/install_and_bak_gitlab.md)

* 自动化测试技术
  * 手机自动化
    * [monkey测试](autotech/monkey_android.md)
    * [adb常用命令](autotech/adb_cmd.md)
    * [uiautomator 自动化测试](autotech/uiautomator_introduce.md)

* 数据库
  * mongo
    * [mongodb安装](data/install_mongodb.md)
    * [mongo分片式集群](data/use_mongo3.6_deploy_shard_cluster.md)
  * MySQL
    * [linux中安装MySQL5.6/5.7](data/install_mysql.md)

* 虚拟化技术
  * vcenter
    * [利用模板快速部署centos7服务器](vm/use_tem_deploy_centos7.md)

* Java开发与其中间件安装配置
  * [websphere8.5安装和使用](middleware/install_websphere8.5.md)
  * [tomcat、weblogic、was等组件jvm调整](middleware/update_jvm_value.md)
  * java基础知识大串联
    * [关于数组位置调整和元素出现次数统计代码](middleware/java-code.md)
    * [for循环实践：生成一万内的质数](middleware/prime_number_product.md)
    * [比较两个数组是否相同](middleware/compara-arry.md)
    * [for和while异同比较](middleware/for-while.md)
    * [利用单项链表学习递归](middleware/java-recursion.md)
    * [装饰者模式扩充方法](middleware/java-decortar.md)
    * [枚举和switch结合使用](middleware/java-enmu-switch.md)
    * [接口与lamda表达式](middleware/java-interface-lamda.md)
    * [利用购物车系统学习HashMap与equals方法重写](middleware/java-shopping-map.md)
    * [死锁案例](middleware/java-dead-lock.md)
    * [多线程轮流打印1-100](middleware/java-multithread.md)
    * [反射机制](middleware/java-reflect.md)

* 大数据处理
  * [druid安装与优化](data/install_druid.md)
  * kafka+zookeeper
    * [kafka常用命令](data/kafka_cmd.md)
    * [zookeeper和kafka参数调整](data/update_jvm_zk.md)
  * redis
    * [redis多实例](data/cluster_redis.md)

* 正确使用工具事半功倍
  * 搜索引擎的使用
  * gitbook 的巧用
  * 有道云笔记
  * GitHub搭建个人网站