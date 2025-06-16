# 两台服务器角色

- 跳板机：10.169.3.234
- 测试机：10.67.16.25

---

## 社区文档

- [社区文档](https://docs.ansible.com/ansible/latest/index.html)
- [初步了解各个模块的功能和角色](https://docs.ansible.com/ansible/latest/getting_started/index.html)
- [完成基础环节的配置](https://docs.ansible.com/ansible/latest/getting_started_ee/index.html)

---

## 1. 创建专用学习目录

```bash
mkdir ~/ansible_yinxin
cd ~/ansible_yinxin

```markdown
# 两台服务器角色

- 跳板机：10.169.3.234
- 测试机：10.67.16.25

---

## 社区文档

- [社区文档](https://docs.ansible.com/ansible/latest/index.html)
- [初步了解各个模块的功能和角色](https://docs.ansible.com/ansible/latest/getting_started/index.html)
- [完成基础环节的配置](https://docs.ansible.com/ansible/latest/getting_started_ee/index.html)

---

## 1. 创建专用学习目录

```bash
mkdir ~/ansible_yinxin
cd ~/ansible_yinxin
```

**你可以在自己的项目目录（如 `/home/toc/SSE/ansible_yinxin/`）下创建和维护自己的 `ansible.cfg` 文件。不会影响其他用户，也不会被其他路径下的配置覆盖。只需保证你在运行 ansible 命令时位于你的项目目录即可。**  
只要你在运行 Ansible 命令时的当前目录下有 `ansible.cfg`，Ansible 会优先读取你当前目录下的配置文件，而不是系统或其他路径下的配置文件。

**Ansible 配置文件的优先级如下（从高到低）：**  
1. 命令行指定的 `-c` 参数  
2. 当前目录下的 `ansible.cfg`  
3. 用户家目录下的 `~/.ansible.cfg`  
4. 系统目录下的 `/etc/ansible/ansible.cfg`  

---

## 2. 创建自定义 ansible.cfg

在你的学习目录中创建一个新的 ansible.cfg 文件：

```bash
cat > ansible.cfg << 'EOF'
[defaults]
# 禁用主机密钥检查(仅测试环境使用)
host_key_checking = False
# 设置库存文件路径
inventory = ./hosts
# 设置默认远程用户
remote_user = root
# 启用回调插件显示输出
stdout_callback = yaml
# 禁用"changed"任务的结果缓存
retry_files_enabled = False

[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False
EOF
```

查看内容：

```bash
cat ansible.cfg
```

内容如下：

```ini
[defaults]
# 禁用主机密钥检查(仅测试环境使用)
host_key_checking = False
# 设置库存文件路径
inventory = ./hosts
# 设置默认远程用户
remote_user = root
# 启用回调插件显示输出
stdout_callback = yaml
# 禁用"changed"任务的结果缓存
retry_files_enabled = False

[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False
```

---

## 3. 创建测试库存文件

创建 hosts 文件定义测试节点：

```bash
cat > hosts << 'EOF'
[local]
localhost ansible_connection=local

[test]
# 这里可以添加你的测试服务器
# test-server1 ansible_host=10.67.16.25
EOF
```

查看内容：

```bash
cat hosts
```

内容如下：

```ini
[local]
localhost ansible_connection=local

[test]
# 这里可以添加你的测试服务器
test-server1 ansible_host=10.67.16.25
```

---

## 4. 验证配置

运行以下命令验证配置：

```bash
ansible --version

ansible all -m ping
```

示例输出：

```
localhost | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
test-server1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
```

---

## 5. 基础命令尝试

列出所有主机：

```bash
ansible all --list-hosts
```

输出示例：

```
  hosts (2):
    localhost
    test-server1
```

执行命令：

```bash
ansible localhost -m command -a "uptime"
```

输出示例：

```
localhost | CHANGED | rc=0 >>
 11:59:35 up 770 days, 32 min, 10 users,  load average: 0.11, 0.19, 0.22
```

---

## 6. 编写并运行 Playbook

创建 playbook 文件：

```bash
echo -e "---\n- hosts: localhost\n  tasks:\n    - name: Test message\n      debug:\n        msg: Hello Ansible\!" > test.yml
```

运行 playbook：

```bash
ansible-playbook test.yml
```

输出示例：

```
PLAY [localhost] **************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************
ok: [localhost]

TASK [Test message] ***********************************************************************************************************
ok: [localhost] => 
  msg: Hello Ansible!

PLAY RECAP ********************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

---

将 playbook 的 `hosts` 字段从 `localhost` 改为 `all`，这样所有主机都会执行该任务：

```yaml
---
- hosts: all
  tasks:
    - name: Test message
      debug:
        msg: HelloAnsible！
```

再次运行：

```bash
ansible-playbook test.yml
```

输出示例：

```
PLAY [all] ********************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************
ok: [test-server1]
ok: [localhost]

TASK [Test message] ***********************************************************************************************************
ok: [localhost] => 
  msg: Hello Ansible!
ok: [test-server1] => 
  msg: Hello Ansible!

PLAY RECAP ********************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
test-server1               : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
```