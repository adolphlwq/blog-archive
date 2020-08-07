# Vim
- 撤销操作:u
- 查看特殊字符
    - :%!cat -A
    - :set invlist
- 行首添加字符：`:%s/pattern/add/`,`:2,4s/pattern/add/`
- 替换： `:s/origin/dest/option`
- 加密文件：
    - `:X`后输入密码
    - `:set key=`取消密码

## 复制
- `yy`...复制光标所在行
- `6,8 co 12`...复制6-8行到12行下面
- `nyy`...复制光标所在行以下n行
- `dd`...剪切光标所在行
- `ndd`...剪切光标所在行以下n行
- `p`...所在行下黏贴
- `P`...所在行上黏贴

## 多行缩进/反缩进
- 第10行至第100行缩进
```
:10,100>
```

- 第20行至第80行反缩进
```
:20,80<
```

- :start,end `num of tabs` >|<

## 搜索
基础： `/things to search`
逆向搜索：`?things to search`
n：向下匹配
N：向上匹配
ggn：匹配到地一个
GN：匹配到最后一个

## 分屏
- vsp 垂直分
- sp 水平分

## 折叠
- zR 展开所有折叠
- zn 全部展开
- zN 全部折叠

## 删除
1. 删除空行
```shell
:g/^$/d
```
2. 删除以`#`开头的行
```shell
:g/^#/d
```
3. 删除5-10行
```shell
：5,10d
```
4. 删除不包含指定字符的行
```shell
:g!/pattern/d
# 或
:v/pattern/d
```