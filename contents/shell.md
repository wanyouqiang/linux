shell 编程
=================

### 第一行
```
	#!/bin/bash
```

### 变量
* 定义变量：name="youqiang",age=28,addr="nanjing"
* 销毁变量：unset name
* 输出变量：echo $name
* 特殊变量：
    * $0 :shell脚本文件名
    * $1 :第一个参数
    * $* :所有字符串参数
* 数组：arr=([name]=youqiang [addr]=shanghai)

### 运算符
* 算术运算符：+,-,*,/,%
```
    a=10
    b=8
    c=`expr a + b`
    let c=a+b
    echo "$c"

    x=8.2
    y=3
    echo "${x} + ${y}" | bc

    res=`echo "${x} + ${y}" | bc`

    # 指定精度
    echo "scale=3;$a / $b" | bc
    # 进制转换
    echo "obase=2;$b" | bc


```
* 关系运算符
    * -eq [ $a -eq $b ] 是否相等
    * -ne [ $a -ne $b ] 是否不相等
    * -gt [ $a -gt $b ] 是否大于
    * -lt [ $a -lt $b ] 是否小于
* 逻辑运算符
    * ! 逻辑非
    * -a 逻辑与
    * -o 逻辑或

### 控制结构
* 选择
```
    if [ $a -eq $b ]
    then
        echo "a is equal to b"
    fi

    if [ $a -gt $b ]
    then
        echo "$a is greater than b"
    else
        echo "$a is less than b"
    fi

```

* 循环
```
    a=10
    while [ $a -gt 0]
    do
        echo "$a"
    done

```

### 自定义函数
```
    add () {
        res=`expr $1 + $2`
        echo "$1 + $2 = $res"
    }

    add 1 2
```