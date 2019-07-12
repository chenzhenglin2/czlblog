新建一个自由风格的job

自由风格的job，最容易上手，一般项目初期或者小型团队和不是特别复杂工程，都可以选择这种风格job进行编译构建。

1，点击新建

![img](E:/youdao_cache/weixinobU7VjkHipGAuCKPROiKiXhe4-Uw/778155145f0041bcaf01d57187bbb79c/image4.png)

2，填写名称，选择构建什么类型项目

![img](E:/youdao_cache/weixinobU7VjkHipGAuCKPROiKiXhe4-Uw/78355caac0b9482aa726e32d027a693f/image5.png)

然后点击ok，进入job编辑界面

![img](E:/youdao_cache/weixinobU7VjkHipGAuCKPROiKiXhe4-Uw/849862fde2a248bf8e8c79970e68b1bf/image6.png)

**增加参数化 构建内容**



如果需要进行源码管理，根据实际情况选择源码管理工具，如svn

![img](E:/youdao_cache/weixinobU7VjkHipGAuCKPROiKiXhe4-Uw/44f87646c97d467b99efcc11e2aba931/image7.png)

如果是通过git管理源代码，选择git



如果多个仓库 可以现在scm

**设定正确的触发器；**

**Build after other projects are built：**

![img](E:/youdao_cache/weixinobU7VjkHipGAuCKPROiKiXhe4-Uw/bf5801dd94144188af43a38004e0f625/image8.png)

这种触发方式也非常实用，如果a程序是用来编译和打包的，b程序是用来更新的；b程序里面就可以用这种方式来触发，然后projects填写a就行，如果是pipeline项目填写到具体分支；



**Build periodically：**

![img](E:/youdao_cache/weixinobU7VjkHipGAuCKPROiKiXhe4-Uw/d70cbd5a1a4e40979a17762ec6b0533e/image9.png)



如上图选择build  periodically，然后按照规则进行设定，设定规则如下：

第一个参数代表的是分钟 minute，取值 0~59；

第二个参数代表的是小时 hour，取值 0~23；

第三个参数代表的是天 day，取值 1~31；

第四个参数代表的是月 month，取值 1~12；

最后一个参数代表的是星期 week，取值 0~7，0 和 7 都是表示星期天。

所以 0 * * * * 表示的就是每个小时的第 0 分钟执行一次构建。

例子如下： 

H 17 * * * 

17点

表示17点进行执行构建任务

H 8,12,15,16 * * 0-6 

一天的8点，12点，15点，16点， 每个星期的七天都是如此

如果设成间隔时间，可以参照下面poll scm中的  */5 格式

**Poll SCM：**

定时检查源码变更（根据SCM软件的版本号），如果有更新就checkout最新code下来，然后执行构建动作。里面为空就行了

**构建**

![img](E:/youdao_cache/weixinobU7VjkHipGAuCKPROiKiXhe4-Uw/69ca6fdce88e4d99bbf9805b57169172/image10.png)

如果构建选择执行shell脚本的话，需要注意的时，用sudo来执行脚本：sudo /home/apptrace/script/deploy_apptrace_frontend.sh

 点击“See the list of available environment variables”蓝色链接字体，可以查看到jenkins自带一些参数及用法示例；

![img](E:/youdao_cache/weixinobU7VjkHipGAuCKPROiKiXhe4-Uw/5804f9e5dfd8437b8e8cb84d7818fdbb/image11.png)



**Build when a change is pushed to GitLab：**

  如果想设定gitlab仓库上master分支一有提交或者合并就触发构建，我们就可以勾选Build when a change is pushed to GitLab，使用它来实现这个功能（前提是jenkins已经安装gitlab hook插件）；

![img](E:/youdao_cache/weixinobU7VjkHipGAuCKPROiKiXhe4-Uw/3fa37ea763654633931f3a4b7eab81e2/image13.png)

![img](E:/youdao_cache/weixinobU7VjkHipGAuCKPROiKiXhe4-Uw/6020f9257e2e4430b6165a57552293e0/image14.png)

如果勾选了Filter branches by name，就针对某些分支才能触发，如图圈中2，填写包含的分支，多个分支用，隔开；图中3填写排除的分支；

点击4下面的Generate，生成Secret token码；然后进入gitlab的webhooks界面；把上面图中圈中1的地方url填写到下面图中圈中1地方；生成Secret token copy到下图中4的地方；





![img](E:/youdao_cache/weixinobU7VjkHipGAuCKPROiKiXhe4-Uw/fc66375cbe414131847b430d6337cb69/image15.png)

![img](E:/youdao_cache/weixinobU7VjkHipGAuCKPROiKiXhe4-Uw/ca9393937e584a42a0faa0a2b34aaab3/image16.png)

并勾选上需要的触发条件，配置完成后，点击add webhook ,即可；我们可以通过点击右下角的test按钮进行测试；

![img](E:/youdao_cache/weixinobU7VjkHipGAuCKPROiKiXhe4-Uw/80d25fc297e44335a442905ad90c03d8/image17.png)

若点击test时，出现jenkins gitlab webhook 403 anonymous is missing the Job/Build permission提示，可通过“系统管理 -> 系统设置 -> 去掉 Enable authentication for ‘/project’ end-point” 来解决；

![img](E:/youdao_cache/weixinobU7VjkHipGAuCKPROiKiXhe4-Uw/c07821b07f7841cd8148f7a54b1de94f/image18.png)



查看构建时的日志输出：

![img](E:/youdao_cache/weixinobU7VjkHipGAuCKPROiKiXhe4-Uw/db521546590642a4a5340ca5775faf94/image19.png)

shell  脚本代码片段截图

![img](E:/youdao_cache/weixinobU7VjkHipGAuCKPROiKiXhe4-Uw/5337580e796c4d33b7dc0ebde09a81f4/image22.png)

如果有些路径，需要是变化，需要参数指定，有多种方法，如图${JOB_NAME} 属于系统参数可以直接拿来用的；$APPTRACE_HOME是节点机里面的参数，我们可以直接在节点里面配置

![img](E:/youdao_cache/weixinobU7VjkHipGAuCKPROiKiXhe4-Uw/3e9cf4efa2bc4f3db08bcf5c29f52780/image23.png)

这样写出来的脚本，更容易批量复制。