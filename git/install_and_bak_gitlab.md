# gitlab 在线安装与备份

## gitlab 安装方法说明

- 在线安装文档：https://mirror.tuna.tsinghua.edu.cn/help/gitlab-ce/

  ### 编辑yum gitlab源 

  ```bash
  vim  /etc/yum.repos.d/gitlab-ce.repo
  [gitlab-ce]
  name=Gitlab CE Repository
  baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/
  gpgcheck=0
  enabled=1
  ```

  

注：镜像地址：https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/ 这个地址根据具体情况来，centos7 就填写el7

- repo源生效

```
sudo yum makecache
sudo yum install gitlab-ce
```



### 安装指定版本

```bash
sudo yum install gitlab-ce-8.11.5  -y
```

完成后， 放开 80 8080端口，或者指定的端口。

#### 修改默认配置

安装完成后 要先执行`gitlab-ctl reconfigure`
再来修改配置文件
`vim /etc/gitlab/gitlab.rb`

- 修改备份文件路径

  ```
  gitlab_rails['backup_path'] = '/mnt/backups'
  ```

  

- 设置只保存最近7天的备份

  ```
  gitlab_rails['backup_keep_time'] = 604800 
  ```

  

- 修改gitlab 存储路径

  ```
  git_data_dirs({"default" => "/home/git-data"})
  ```

  

- 修改访问url

  ```
  external_url 'http://172.18.35.66'  或者使用域名
  ```

  

- 再次执行gitlab-ctl reconfigure，使配置生效

  ## 恢复与备份

### 恢复gitlab

```bash
gitlab-rake gitlab:backup:restore BACKUP=备份文件编号
```



### 备份gitlab

```bash
gitlab-rake gitlab:backup:create CRON=1
```



### 定时任务脚本

```bash
#!/bin/bash 
/usr/bin/gitlab-rake gitlab:backup:create CRON=1
sleep   10s 
rsync -e "ssh -p 22" -avz /home/gitlab-backups  远端服务器IP:/gitlab-backups --delete
```

注：不使用 `-e "ssh -p 22"`这个 参数也可以同步，但本人远程服务器只有22端口是放开的

- 定时任务 （crontab -e后编辑)

```
0 2 * * 2-6 /path/xxx.sh &>/dev/null 
```

