# 利用gitbook+git+jenkins+nodejs搭建起一套多版本说明的文档网站

现有这么一场景，产品产出了大量的文档，这些文档要对外展示，且产品每个版本的文档内容不一样，如何实现这么复杂的需求呢。

首先拆解需求，然后再分步实现。

1).文档编写采用统一标准，都用Markdown格式的，最后可以用gitbook自动生成静态网页；

2).分配好各阶段（版本）相关责任人编写的文档，待文档编写完成后统一上传到git；

3).jenkins负责拉取正式版本（release开头）的文档，用gitbook编译后，同步到云端文档服务器

4).云端服务器制作一个入口页面，页面里面可以指引各个版本文档；http服务用nginx，指引到这个页面；

![1563504880709](images/1563504880709.png)

步骤有了，那就动手开始执行了：

前两步具体细节这里不再做描述；先用gitbook init初始化生成readme.md和summary.md后，完善summary目录后上传到仓库，其他同事按照summary归类以及分配文档任务编写即可。任务制定好了，一定按计划执行,这个是关键。

这里重点要说的是第三步，由于第三步比较复杂，这里会拆成多个小步骤来说明；至于第四步，用nginx增加一个静态网页入口，可以参考[ NGINX指向静态网页](../nginx/direct_static_web.md).

## jenkins自动获取有改动的正式版本，编译后同步到云端服务器

- 首先确认哪些版本能自动触发构建，若自动触发构建的是新分支(版本)，云端服务器没有此版本，如何在页面中创建此版本并把静态html文档同步过去。

  ### 1，把云端服务器目录通过ftp挂载到本地

  这一步也可以不做，直接通过rsync同步到云端就行了；但我这个环境是ftp服务站，并不是真正的服务器。

  ```bash
  
  # Create local mount path
  mkdir -p /mnt/myftp
  
  # Mount the destination ftp site using curlftpfs
  curlftpfs -o allow_other ftp://myusername:mypassword@ftp.mydomain.com /mnt/myftp
  
  ```

  

### 2，Jenkins匹配合适的分支自动构建

- 源码管理中，只拉取release开头分支（避免拉取到临时分支，个人分支）

![1563507835925](images/1563507835925.png)

- 采用合理的自动触发构建
  触发器选择“Build when a change is pushed to GitLab”  具体配置步骤，参考[jenkins的自由风格构建](../jenkins/freestyle_build_in_jenkins.md)

  ![1563508362278](images/1563508362278.png)

  这里触发构建版本，要用正则表达式筛选一下。

  ### 构建、同步以及用nodejs在入口页面中增加新版本模块

  

- 获取拉取下来的（release开头）分支名称后通过截取字段手段，保留下版本号；编译后同步过去，如果是新增的版本，创建这个版本目录并在页面中增加这个模块；

  #### shell 代码处理的功能

  execute shell 内容：

  ```
  echo "获取版本号"
  NEW_BRANCH=$(
  for branch in `git branch -r | grep -v HEAD`;do echo -e `git show --format="%ci %cr" $branch | head -n 1` \\t$branch; done | grep "release" | sort -r | head -1 | awk '{print $NF}' |awk -F/ '{print $2}'
  )
  
  echo "根据版本号 进行同步"
  if [[ "$NEW_BRANCH" =~ ^release ]];then  
  
    BRANCH_SHORT=${NEW_BRANCH#release-}
    echo  "编译文档，生成HTML"
    GITBOOK_PID=$(ps  -ef  | grep gitbook | grep -v 'grep' | awk '{print $2}')  
    if [[ -n $GITBOOK_PID ]];then
       kill $GITBOOK_PID
    fi  
      
    gitbook install
    install_status=$(ps  -ef  | grep "gitbook install" | grep -v  'grep' | wc -l)
    while [[ "$install_status" -ne 0 ]]
    do
       sleep 1s
    done
      
    BUILD_ID=dontKillMe nohup gitbook serve &>/dev/null &
    sleep 25s;  
    #  如果本地不需要gitbook server 只需编译出静态网页的话，"编译文档，生成HTML"这一块代码直接用`gitbook install` 和`gitbook build` 两行命令替换即可。
    echo  "同步到外网ftp服务器"
    if [[ ! -d /mnt/myftp/htdocs/doc/app/$BRANCH_SHORT ]];then
      mkdir  -pv   /mnt/myftp/htdocs/doc/app/$BRANCH_SHORT
      cp   /mnt/myftp/htdocs/doc/app/index.html   /home/backup-ftp/index_$(date  +%F-%H-%M-%S ).html # 备份引导页面
      cd /root/test-auto-html  
      node index.js 
      /bin/cp  -f  index.html  /mnt/myftp/htdocs/doc/app/index.html
      # 用nodejs 生成新的引导页面，新页面里面增加新版本模块
    fi
    cd  $WORKSPACE 
    rsync -avz  --temp-dir=/tmp \_book/*   /mnt/myftp/htdocs/doc/app/$BRANCH_SHORT/  
    # 把静态页面同步过去
    
  else
    echo "not  release version"
    exit 1 
    
  fi
  ```

  代码中有几处要重点说明的

  - `BUILD_ID=dontKillMe nohup gitbook serve &>/dev/null &` 之所以这么写是 jenkins任务执行完成后，会杀掉所有开启的进程，添加这个参数 是为了保留gitbook serve这个进程；

  - `rsync   --temp-dir=/tmp`   搞一个临时目录，是因为有的ftp同步对文件大小有限制，这么做，可以起到缓冲作用。
  - 如果不是通过ftp挂载到本地，可以通过ssh  user@server_ip 'test -d /your_path'  返回值是否为0，判断分支的存在（ssh前可以设为公钥登录）

  

#### nodejs 代码在引导页面增加新模块（可以求助前端开发的帮助）

html模板(body里面的代码片段)，命名为index.ejs

```html
 <section class="wrapper p-v-lg">
    <div class="row center-xs">
      <div class="cards col-md-9 center-xs text-center">

        <% for (var i = 0; i < versions.length; i++) { %>
          <div class="cards-item col-md-6">
              <div class="card">
                  <div class="card-content">
                      <div class="card-text">
                          <h3 class="m-t-0"><%=versions[i]%></h3>
                          <p>管理平台新增和优化流程分析，流程统计功能。xxxxxxxxxxxxx提供流程故障的快速排查能力。</p>
                      </div>
                      <div class="border-top p-t-xs m-t-sm">
                          <a href="http://www.yourdomain.com/doc/app/<%=versions[i]%>" class="btn bg-link">阅读文档</a>
                      </div>
                  </div>
              </div>
          </div>
        <% } %>

      </div>
    </div>
  </section>
```

node-ejs代码（命名为index.js，如果是其他名称.js也可以，但执行命令为 node  xx.js)：

```javascript
let fs = require("fs");
let ejs = require("ejs");
const path = require('path');
let p = path.resolve('E:/node/doc');
let tmpl = fs.readFileSync("./index.ejs", "utf8");
//console.log('tmpl: ', tmpl);
let data = { versions: [] };
fs.readdirSync(p).forEach(ele => {
  let info = fs.statSync(path.resolve(p, ele));
  if (info.isDirectory()) {
    data.versions.push(ele);
  }
});

let template = ejs.compile(tmpl, {});
let result = template(data);
fs.writeFileSync('./index.html', result);
//console.log(result);
```

最终效果就是本文第一幅插图；如果有新增版本，就会在引导页面增加一个版本引导模块。

index.js把index.ejs读取后，作为模板进行编辑和填充然后生成了一个新的html文件，替换之前的html页面

从引导页，点击任意模块中的按钮都可以进入对应版本文档展示界面：

![1563520849117](images/1563520849117.png)

像本人博客一样（都是gitbook 生成的）；但不限制于此，php jsp aspx 等等网页 都可以按照这个思路来实现，具体页面编码可以寻求前端开发的帮助。



