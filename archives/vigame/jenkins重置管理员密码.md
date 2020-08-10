1、进入/Users/xxx/.jenkins/users目录，找到config.xml。
2、找到config.xml文件中passwordHash标签，修改为如下
#jbcrypt:$2a$10$DdaWzN64JgUtLdvxWIflcuQu2fgrrMSAMabF5TSrGK5nXitqK9ZMS  （该字符串默认密码为6个1）
3、重启Jenkins：jenkins restart
