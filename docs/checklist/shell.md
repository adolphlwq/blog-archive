# Shell
- learn about `shell` basic infonation such as: `&`, `command` ...
- grep
- find
- nc
- dig
- nslookup
- awk
- sed
- command
- -n,-z,-x,-t ...
- $#,$@,$*,$?,$0,$1,$2
- `xargs`: docker ps -aq | xargs docker rmi
- jq
- apt-get install --only-upgrade <packagename>
- `sudo update-alternatives --config editor`
- `2>&1`

## centos
- 查看版本
- 关闭SELinux
```
#临时关闭
sudo /usr/sbin/setenforce 0
#永久关闭
vim /etc/sysconfig/selinux
#SELINUX=enforcing
#SELINUXTYPE=targeted
SELINUX=disabled
重启
```
- config rsyslog
- 关闭防火墙

## sed
- 打印指定行：echo indo|sed -n '3,4p'
- 替换：sed 's/a/b/g' 
- 替换特殊字符：sed 's/\[\b\g'
    - \t
    - \n
    - \[
    - \/
- sed -e 's/a/b/g' -e 's/[ ]/a/g'
- 替换多个空格： sed 's/[ ][ ]*/a/g'
- 删除空行：sed 's/^$//g' | sed '/^$/d'
- 替换换行符：sed ':a;N;ba;s/\n/dd/g'
- tr '\n' 'da'
- 第一行加一行：sed "1a add text" 
- http://blog.csdn.net/hello_hwc/article/details/40118129

## if条件判断
- 字符串判断
```
if [ $a = $b ]
if [ $a == $b ]
判断a是否为空（null） if [ -z $a ]
判断a是否bu为空（null） if [ -n $a ]
```
- 整数比较
```
-eq 等于,如:if [ "$a" -eq "$b" ]   
-ne 不等于,如:if [ "$a" -ne "$b" ]   
-gt 大于,如:if [ "$a" -gt "$b" ]   
-ge 大于等于,如:if [ "$a" -ge "$b" ]   
-lt 小于,如:if [ "$a" -lt "$b" ]   
-le 小于等于,如:if [ "$a" -le "$b" ]   
<   小于(需要双括号),如:(("$a" < "$b"))   
<=  小于等于(需要双括号),如:(("$a" <= "$b"))   
>   大于(需要双括号),如:(("$a" > "$b"))   
>=  大于等于(需要双括号),如:(("$a" >= "$b"))
```
- [[]]和[]的区别
```
[[ $a == z* ]]   # 如果$a以"z"开头(模式匹配)那么将为true   
[[ $a == "z*" ]] # 如果$a等于z*(字符匹配),那么结果为true   
  
[ $a == z* ]     # File globbing 和word splitting将会发生   
[ "$a" == "z*" ] # 如果$a等于z*(字符匹配),那么结果为true
```
- 文件和目录的比较
```
-e                          文件存在
-f                          被测文件是一个regular文件（正常文件，非目录或设备）
-s                          文件长度不为0
-d                          被测对象是目录
-b                          被测对象是块设备
-c                          被测对象是字符设备
-p                          被测对象是管道
-h                          被测文件是符号连接
-L                          被测文件是符号连接
-S(大写)                    被测文件是一个socket
-t                          关联到一个终端设备的文件描述符。用来检测脚本的stdin[-t0]或[-t1]是一个终端
-r                          文件具有读权限，针对运行脚本的用户
-w                          文件具有写权限，针对运行脚本的用户
-x                          文件具有执行权限，针对运行脚本的用户
-u                          set-user-id(suid)标志到文件，即普通用户可以使用的root权限文件，通过chmod +s file实现
-O                          运行脚本的用户是文件的所有者
-G                          文件的group-id和运行脚本的用户相同
-N                          从文件最后被阅读到现在，是否被修改
f1 -nt f2                   文件f1是否比f2新
f1 -ot f2                   文件f1是否比f2旧
f1 -ef f2                   文件f1和f2是否硬连接到同一个文件
```
[参考： shell脚本----if（数字条件，字符串条件，字符串为空）](http://blog.csdn.net/yf210yf/article/details/9207147)
## magic
- 进度条：http://297020555.blog.51cto.com/1396304/494315