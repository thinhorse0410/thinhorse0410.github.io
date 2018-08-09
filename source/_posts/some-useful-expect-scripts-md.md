---
title: 一些工作中有用的expect脚本
date: 2018-08-09 22:30:47
tags:
    - linux
    - expect
cagegories:
    - 编程
---

#### expect的作用

- 自动化的完成一些需要交互的任务
    - ssh,scp时,需要回答yes,需要输入密码验证(未设置互信的前提下)
    - 此时我们可以使用一些如JcSH(用Java实现的ssh库)工具库. 
    - 若只用脚本如何实现? 这就需要expect这个工具了.

### 实用例子

#### ssh登录机器并执行shell命令

##### demo.exp

```shell
#!/usr/bin/expect
set timeout 300
set username [lindex $argv 0]
set host [lindex $argv 1]
set password [lindex $argv 2]
set target [lindex $argv 3]
spawn ssh $username@$host "mkdir -p /tmp/kafka-log-gather/$target"
 expect {
 "(yes/no)?"
 {
  send "yes\n"
  expect "*assword:" { send "$password\n"}
 }
 "*assword:"
 {
 send "$password\n"
 }
 }
expect "100%"
exit 0
```
##### 运行

expect demo.exp username host password target

<!--more-->
