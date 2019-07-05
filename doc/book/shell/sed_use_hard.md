## 1



    sed -i '/ADMIN URL/s/localhost/172.17.0.1/;
            /DATA URL/s/localhost/172.17.0.1/;
            s/localhost:8082/172.17.0.1:8082/;      
             N;/16379/s/localhost/172.17.0.1/;P;D;' $CONFIG_DIR/fusion_config/production.js     

   


sed 可以一次性执行多行代码，每行代码用分号隔开；上面代码块，前两行表示定位后进行替换操作，第三行代码匹配后就替换；第四行代码是模式空间指针到16379后（模式空间有两行），把模式空间里面的localhost替换掉。大写PD 表示打印模式空间第一行和删除第一行；


## 2
sed -i  '/^\s*$/d'  test.txt  删除空白行
sed -i 's/\s*//' test.txt  表示删除每行开头的空格 
sed -i 's/\s*//g' test.txt  表示删除每行所有的空格 
sed -i 's/\s*//;s/\s*$//g' test.txt  删除行首和行尾的空格 也可以拆开写
把删除行尾的空格单独为：sed  -i 's/\s*$//g'
正则表达式\s表示任意空白字符 不管是tab 还是空格
sed 模式匹配的g参数：
g是起到一个全局的作用，这个范围是每一行，也就是说是一行为单位，作为一个全局。+g :匹配每一行有行首到行尾的所有字符
所以慎用参数g
## 3
指定行修改，如第4行末尾追加一行，内容为test
sed -i 'N;4atest' test.txt   
#第4行行首追加一行，内容为test
sed -i 'N;4itest' test.txt 

