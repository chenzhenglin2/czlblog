# 界面选择项用法



```
PS3="Enter option: "
select option in "Install All" "Install Elasticsearch" "Install Mongo" "Install Nginx" "Install Kafka" "Install Redis" \
"Install Druid" "Reboot System" "Exit Program"
do
  case $option in
  "Exit Program")
        break ;;
  "Install All")
        install_all ;;
  "Install Elasticsearch")
        install_es ;;
  "Install Mongo")
        install_mongodb ;;
  "Install Nginx")
        install_nginx_only ;;
  "Install Redis")
        install_redis_only ;;
  "Install Kafka")
        install_kafka_only ;;
  "Install Druid")
        install_druid ;;
  "Reboot System")
        reboot_system ;;
   *)

 echo "Sorry. Wrong selection"

   esac
done
```



最终形成界面多选项，选择某一项的序号，就可以执行此选项脚本代码