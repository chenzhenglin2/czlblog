## 逐个导出镜像

```
#!/bin/bash
IMAGES_LIST=($(docker  images   | sed  '1d' | awk  '{print $1}'))
IMAGES_NM_LIST=($(docker  images   | sed  '1d' | awk  '{print $1"-"$2}'| awk -F/ '{print $NF}'))
IMAGES_NUM=${#IMAGES_LIST[*]}
for((i=0;i<$IMAGES_NUM;i++))
do
  docker save "${IMAGES_LIST[$i]}"  -o "${IMAGES_NM_LIST[$i]}".tar.gz 
done
```
#注意 这个慎用，如果一个镜像有多个版本，容易出现问题，采用下面的批量导入


## 批量导入到一个压缩包
```
#!/bin/bash
IMAGES_LIST=($(docker  images   | sed  '1d' | awk  '{print $1":"$2}'))
docker save ${IMAGES_LIST[*]}  -o  all-images.tar.gz

```
## 把一个压缩包所有镜像导出并上传到仓库

```
./rancher-load-images.sh   -l  rancher-images.txt   -i  rancher-images.tar.gz   -r  reg.czl.com/library  
#仓库地址包含具体的项目
```

rancher-load-images.sh 脚本下载地址：https://github.com/rancher/rancher/releases
选择需要版本，点击assets 展开后就有这个脚本下载

## 逐个导入镜像

```
cd $DIR/images_file
 
for image_name in $(ls ./)
 
do
 
  docker load < ${image_name}
 
done 

```

