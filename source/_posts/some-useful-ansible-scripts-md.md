---
title: 常见ansible的有用例子
date: 2018-08-09 22:30:56
tags:
    - ansible-playbook
    - yml
categories:
    - 编程
---

#### ansible里调用http服务,并根据返回决策

```shell
---
- hosts: "{{ group }}"
  remote_user: user
  serial: 1
  tasks:
    - name: invoke http service
      command: curl http://"{{ ansible_ssh_host }}":"{{ remotePort }}"/api
      register: response
      until: ("2000" in response.stdout) or ("404" in response.stdout)
      retries: 3
      delay: 5
      ignore_errors: no
    - name: wait 60s
      local_action:
          module: wait_for
          timeout: 60
    - name: copy file to remote server
      copy: src=../xxx/a dest=/xxx/
    - name: restart
      command: supervisorctl -c /xxx/supervisord.conf restart serviceA
    - name: wait 30s for server start
      local_action:
          module: wait_for
          timeout: 30
    - name: check service start
      command: curl http://"{{ ansible_ssh_host }}":"{{ remotePort }}"/health
      register: resp
      until: ("UP" in resp.stdout) or ("404" in resp.stdout)
      retries: 3
      delay: 5
      ignore_errors: no
```

<!--more-->



