# Ansible Inventory 构建指南

Inventory（清单）是 Ansible 部署和配置的托管节点（或主机）的列表。通过创建库存文件，可以跟踪和管理你想要自动化的服务器和设备。

---

## 库存文件格式

Ansible 支持多种库存文件格式，最常用的是 INI 和 YAML 格式。
- 推荐使用 YAML 格式，因为结构更清晰，支持更复杂的配置，并且与 Ansible Playbook 风格一致。
- 组名应使用描述性名称，如 `webservers`、`dbservers` 等，便于理解和管理。

### 1. INI 格式示例

```ini
mail.example.com

[webservers]
foo.example.com
bar.example.com

[dbservers]
one.example.com
two.example.com
three.example.com
```

### 2. YAML 格式示例

```yaml
ungrouped:
  hosts:
    mail.example.com:
webservers:
  hosts:
    foo.example.com:
    bar.example.com:
dbservers:
  hosts:
    one.example.com:
    two.example.com:
    three.example.com:
```

---

### 分组说明

- 方括号中的名称代表组名（如 `[webservers]`）。
- 组名用于对主机进行分类管理。
- 组名命名规则需遵循有效的变量名规则。

---

### 默认分组

即使不定义任何分组，Ansible 也会自动创建两个默认组：

| 组名       | 包含内容         | 说明                       |
| ---------- | ---------------- | -------------------------- |
| all        | 所有主机         | 包含库存中的所有主机        |
| ungrouped  | 无其他分组的主机 | 仅属于 all 组的主机         |

- 每台主机至少属于 2 个组（all + ungrouped 或 all + 其他自定义组）。
- 如果主机属于任何自定义组，则不再属于 ungrouped 组。

### 分组嵌套：父/子组关系


### 基本概念

- **父组（嵌套组/组中组）**：可以包含其他组的组。
- **子组**：被父组包含的组。
- **优势**：通过管理子组来自动更新父组，减少维护成本。

- **在 INI 格式中**，使用 `:children` 后缀来定义父组。
- **在 YAML 格式中**，使用 `children:` 条目来定义父组。
---

### 创建方法  YAML 格式

```yaml
prod:
  children:    # 使用 children 条目
    atlanta_prod:
    denver_prod:

test:
  children:
    atlanta_test:
    denver_test:
```

---

### 嵌套组特性

- 子组成员自动成为父组成员。
- 组可以有多个父组和子组，但不能有循环依赖。
- 主机可以属于多个组，运行时只会有一个实例（Ansible 会合并数据）。

---

### 主机范围表示法

```数字范围yaml
webservers:
  hosts:
    www[01:50].example.com:
    www[01:50:2].example.com:
```

---

```字母范围ini
[databases]
db-[a:f].example.com  # db-a到db-f
```

- 前导零可选。
- 范围包含边界值。
- 步长可自定义。
```
---
```
### 使用多库存源

Ansible 支持同时指定多个库存源（静态文件、目录、动态库存脚本或任何库存插件支持的源），可以通过以下方式实现：

---

### 1. 命令行指定方式

```bash
ansible-playbook get_logs.yml -i staging -i production
```
> 通过 `-i` 参数多次指定不同的库存源。

---

### 2. 配置方式

- 设置 `ANSIBLE_INVENTORY` 环境变量：

  ```bash
  export ANSIBLE_INVENTORY=staging:production
  ```

- 在 `ansible.cfg` 中配置 `DEFAULT_HOST_LIST`：

  ```ini
  [defaults]
  inventory = staging:production
  ```

---

### 应用场景

当您需要**同时对通常分离的环境**（如预发布环境和生产环境）执行特定操作时，多库存源功能特别有用。
```
```markdown
## 目录式库存组织

您可以将多个库存源整合到单个目录中，这是比使用单一库存文件更可维护的方案。

### 优势特点

- 解决单个文件过长导致的维护困难问题
- 便于多团队协作（每个团队/项目维护自己的文件）
- 支持混合使用不同格式（YAML、INI等）和插件配置
- 可灵活组合使用单个文件或子集

### 目录结构示例

```
inventory/
  ├── openstack.yml          # 通过OpenStack插件获取云主机
  ├── dynamic-inventory.py   # 动态库存脚本添加额外主机
  ├── on-prem                # 静态主机和组定义
  └── parent-groups          # 静态主机和组定义
```

### 使用方式

```bash
ansible-playbook example.yml -i inventory
```

**注意：**
- 文件按字母顺序从上到下加载
- 默认会忽略某些目录和扩展名（可通过 `INVENTORY_IGNORE_PATTERNS` 和 `INVENTORY_IGNORE_EXTS` 配置修改）

### 库存加载顺序管理

- 按提供顺序加载库存源
- 最后根据需要添加 `all` 和 `ungrouped` 组
- 多次定义的变量遵循“最后定义者生效”原则

**注意事项：**
- 使用某些库存插件时，可能需要调整加载顺序以确保父/子组定义正确
- YAML 和 INI 格式的库存处理完每个源后会丢弃空组（没有关联主机的组）
- 变量冲突时，后加载的值会覆盖先加载的值

### 最佳实践建议

- **环境隔离**：为不同环境（开发/测试/生产）使用不同库存文件
- **混合部署**：结合静态和动态库存源管理混合基础设施
- **优先级控制**：通过文件名前缀（如 01-、02-）控制加载顺序
- **配置集中化**：在 `ansible.cfg` 中配置默认库存目录提高效率

**适用场景：**
- 管理大规模基础设施
- 混合云环境（云主机+物理机）
- 需要多团队协作维护库存的场景
- 要求不同环境严格隔离的场景

---

### 库存变量定义方式

### 直接定义变量

可以在 YAML 或 INI 格式的库存文件中直接为主机或组定义变量：

**YAML 格式示例**
```yaml
webservers:
  hosts:
    web1.example.com:
      http_port: 80
      max_connections: 200
    web2.example.com:
      http_port: 8080
      max_connections: 400
  vars:
    nginx_version: 1.18
```

### 使用 host_vars 和 group_vars 目录

更推荐的方式是使用独立的变量文件：

```
inventory/
├── group_vars/
│   └── webservers.yml
└── host_vars/
    └── web1.example.com.yml
```

---

### 变量继承与优先级

### 变量继承规则

继承顺序（从低到高）：

1. all 组（所有组的父组）
2. 父组
3. 子组
4. 主机变量

同级组合并：默认按字母顺序合并，后加载的组变量会覆盖先加载的。

### 优先级控制

使用 `ansible_group_priority` 调整组变量优先级（数值越大优先级越高）：

```yaml
production:
  vars:
    db_server: db.prod.example.com
    ansible_group_priority: 10
staging:
  vars:
    db_server: db.stage.example.com
    ansible_group_priority: 5
```

---

### 连接行为参数

| 参数                           | 说明                   | 示例                          |
| ------------------------------ | ---------------------- | ----------------------------- |
| ansible_connection             | 连接类型（ssh/local等）| ansible_connection=ssh        |
| ansible_host                   | 实际连接地址           | ansible_host=192.168.1.100    |
| ansible_port                   | SSH端口                | ansible_port=2222             |
| ansible_user                   | 登录用户               | ansible_user=deploy           |

### SSH专用参数

| 参数                        | 说明           |
| --------------------------- | -------------- |
| ansible_ssh_private_key_file| SSH私钥路径    |
| ansible_ssh_common_args     | 附加SSH参数    |

### 权限提升参数

| 参数                    | 说明               |
| ----------------------- | ------------------ |
| ansible_become          | 是否提权           |
| ansible_become_method   | 提权方式（sudo等） |
| ansible_become_user     | 提权目标用户       |

---

### 库存组织最佳实践

### 按环境分离库存

```
inventory/
├── production/
├── staging/
└── development/
```

### 混合组织结构

```ini
# 按功能分组
[webservers]
web[01:10].example.com

# 按位置分组
[us-east]
web[01:05].example.com

[us-west]
web[06:10].example.com
```

### 使用动态库存

```bash
ansible-playbook -i ec2.py deploy.yml
```

- 使用动态库存来跟踪不断启动和停止的服务器和设备的云服务。
## Ansible 动态库存管理指南

### 动态库存概述

当主机资源随业务需求动态变化（如云环境自动扩缩容）时，静态库存文件无法满足需求。Ansible 通过以下两种方式支持动态库存：

- **库存插件（推荐）**：利用 Ansible 核心代码的最新功能。
- **库存脚本**：保持向后兼容性。

---

### Cobbler 库存脚本示例

配置步骤

1. 将脚本保存为 `/etc/ansible/cobbler.py` 并添加执行权限。
2. 创建配置文件 `/etc/ansible/cobbler.ini`：

    ```ini
    [cobbler]
    host = http://127.0.0.1/cobbler_api  # Cobbler服务器地址
    cache_path = /tmp                    # 缓存目录
    cache_max_age = 900                  # 缓存有效期(秒)
    ```

使用场景：这两个命令是针对 Cobbler（一个Linux系统安装和配置管理服务器）的操作指令，用于定义系统安装配置模板（Profile）和具体主机（System）的配置。

```bash
cobbler profile add --name=webserver --distro=CentOS6-x86_64
cobbler system edit --name=foo --dns-name="foo.example.com" --mgmt-classes="atlanta"
```

功能特性

- 自动继承 Cobbler 中定义的组（如 webserver/atlanta）
- 支持 ksmeta 数据作为 Ansible 变量
- 变量优先级：外部库存变量 > Playbook 变量

---

### 库存目录与多源整合

目录结构规则

```
inventory/
├── dynamic/           # 动态库存脚本
│   └── cloud.py
├── static/            # 静态库存文件
│   ├── production
│   └── staging
├── group_vars/        # 组变量
└── host_vars/         # 主机变量
```

文件处理规则

- **可执行文件**：作为动态库存源处理
- **普通文件**：作为静态库存源处理
- **忽略文件**：默认跳过 ~、.bak、.retry 等后缀文件（可通过 ansible.cfg 配置）

---

### 静态组与动态组的混合使用

在静态清单文件中定义组的组时，子组也必须在静态清单文件中定义，否则 Ansible 将返回错误。如果您要定义一个包含动态子组的静态组，请在静态清单文件中将动态组定义为空。

---

### 最佳实践对比

| 特性         | 库存插件           | 库存脚本             |
| ------------ | ------------------ | -------------------- |
| 执行效率     | 高（原生集成）     | 中（外部进程调用）   |
| 维护成本     | 低（标准API）      | 高（需维护脚本）     |
| 功能扩展     | 支持最新功能       | 有限兼容             |
| 调试难度     | 容易（内置日志）   | 较难（需单独调试）   |
| 适用场景     | 云平台/现代基础设施| 传统系统/特殊CMDB集成|

---


## 模式基础用法
- 使用模式来自动化库存的特定子集。
  
模式（Patterns）在临时命令（ad-hoc）和剧本（playbook）中广泛使用，用于指定目标主机或组。

---

### 临时命令中的模式

```bash
ansible <模式> -m <模块名> -a "<模块选项>"
```

**示例：**

```bash
ansible webservers -m service -a "name=httpd state=restarted"
```

---

### Playbook 中的模式

```yaml
- name: <play名称>
  hosts: <模式>
```

**示例：**

```yaml
- name: 重启Web服务器
  hosts: webservers
```

---

### 常用模式对照表

| 描述         | 模式                       | 目标范围                |
| ------------ | -------------------------- | ----------------------- |
| 所有主机     | all (或 *)                 | 全部主机                |
| 单个主机     | host1                      | 指定主机                |
| 多个主机     | host1:host2 (或 host1,host2)| 多台主机                |
| 单个组       | webservers                 | 组内所有主机            |
| 多个组       | webservers:dbservers       | 两组所有主机            |
| 排除组       | webservers:!atlanta        | webservers组排除atlanta组|
| 组交集       | webservers:&staging        | 同时在webservers和staging组的主机 |

> **注意**：主机列表分隔符推荐使用逗号(,)，特别是在处理IP范围和IPv6地址时。

---

### 组合模式示例

```bash
webservers:dbservers:&staging:!phoenix
```
表示：

- 包含 webservers 和 dbservers 组
- 同时必须在 staging 组
- 排除 phoenix 组的所有主机

---

### 通配符模式

支持对 FQDN 或 IP 地址使用通配符：

```bash
192.0.*         # 匹配192.0.x.x所有IP
*.example.com   # 匹配该域名下所有主机
*.com           # 匹配所有.com域名主机
```

可与组名组合使用：

```bash
one*.com:dbservers
```

---

### 模式限制条件

- **依赖库存定义**：未在库存中定义的主机/组无法通过模式匹配
- **别名使用**：必须使用库存中定义的别名而非IP地址

```yaml
atlanta:
  hosts:
    host1:  # 必须使用host1而非127.0.0.2
      ansible_host: 127.0.0.2
```

---

### 模式处理顺序

处理优先级从高到低：

1. `:` 和 `,`（并集）
2. `&`（交集）
3. `!`（排除）

**示例等价关系：**

```
a:b:&c:!d:!e == &c:a:!d:b:!e == !d:a:!e:&c:b
```

**最终效果：**

- 主机属于 a 或 b 组
- 且属于 c 组
- 且不属于 d 组和 e 组

---

### 高级模式选项

#### 变量模式

```bash
webservers:!{{ excluded }}:&{{ required }}
```

#### 组位置选择

给定组：

```ini
[webservers]
cobweb
webbing
weber
```

单主机选择：

```bash
webservers[0]    # cobweb (索引从0开始)
webservers[-1]   # weber (倒数第一个)
```

范围选择：

```bash
webservers[0:2]  # cobweb,webbing,weber
webservers[1:]   # webbing,weber
webservers[:2]   # cobweb,webbing
```

#### 正则表达式

以 ~ 开头使用正则匹配：

```bash
~(web|db).*\.example\.com
```

---

### 临时命令限制参数

```bash
# 限制单个主机
ansible all -m <module> -a "<options>" --limit "host1"

# 限制多个主机
ansible all -m <module> -a "<options>" --limit "host1,host2"

# 排除主机(必须用单引号)
ansible all -m <module> -a "<options>" --limit 'all:!host1'

# 限制组
ansible all -m <module> -a "<options>" --limit 'group1'
```

---

### Playbook 限制参数

```bash
# 限制运行主机(即使未在库存定义)
ansible-playbook site.yml -i "127.0.0.2,"

# 通过库存限制
ansible-playbook site.yml --limit datacenter2

# 从文件读取主机列表
ansible-playbook site.yml --limit @retry_hosts.txt

# 使用重试文件(RETRY_FILES_ENABLED=True时自动生成)
ansible-playbook site.yml --limit @site.retry
```
```markdown
### 模式与 Ansible 库存系统的互补关系

| 特性       | 库存(Inventory)           | 模式(Patterns)           |
| ---------- | ------------------------ | ------------------------ |
| 功能定位   | 定义主机和组的静态关系   | 动态选择主机和组         |
| 使用场景   | 基础设施的长期组织结构   | 临时性的操作目标选择     |
| 变更成本   | 高（需要修改文件）       | 低（命令行参数）         |
| 灵活性     | 固定结构                 | 实时组合                 |

---

实际工程案例
某电商平台使用模式实现的自动化运维：

 黑五大促扩容

```bash
# 扩容所有非GPU类型的web服务器
ansible 'webservers:!gpu' -m amazon.aws.ec2_instance \
  -a "instance_type=c5.2xlarge count=1"
```

 安全补丁分批次更新

```bash
# 先更新测试环境
ansible 'webservers:&staging' -m yum -a "name=openssl state=latest"

# 再更新生产环境的非核心服务
ansible 'webservers:&production:!core' -m yum -a "name=openssl state=latest"
```

 跨云服务管理

```bash
# 同时操作AWS和阿里云上的数据库服务
ansible 'aws_dbs:alibaba_dbs' -m mysql_query -a "query='SHOW SLAVE STATUS'"
```


## Ansible 连接方法与配置详情
- 扩展和改进 Ansible 用于你的库存的连接方法。
### 连接方法概述

本节介绍如何扩展和优化 Ansible 用于连接目标主机的各种方法。

---
 ControlPersist 与 paramiko

- **默认连接方式**：Ansible 使用原生 OpenSSH（支持 ControlPersist 性能优化特性、Kerberos 认证和 `~/.ssh/config` 中的跳板机配置）。
- **兼容性回退**：当控制节点使用不支持 ControlPersist 的旧版 OpenSSH 时，自动切换至 Python 实现的 paramiko 库。

---

### 设置远程用户

**配置方式：**

- **Playbook 中指定：**

  ```yaml
  - name: 更新Web服务器
    hosts: webservers
    remote_user: admin  # 指定连接用户
  ```

- **主机变量配置：**

  ```ini
  other1.example.com ansible_connection=ssh ansible_user=myuser
  ```

- **组变量配置：**

  ```yaml
  cloud:
    vars:
      ansible_user: admin  # 组内统一连接用户
  ```

---

### SSH 密钥配置

**认证方式选择：**

- 推荐方案：SSH 密钥认证（默认方式）
- 备选方案：密码认证（需配合 `--ask-pass` 参数）
- 提权密码：使用 `--ask-become-pass` 提供 sudo 密码

**密钥管理实践：**

```bash
# 启动 ssh-agent 管理会话
ssh-agent bash

# 添加默认 RSA 密钥
ssh-add ~/.ssh/id_rsa

# 添加 PEM 格式密钥
ssh-add ~/.ssh/keypair.pem

# 替代方案：通过库存文件指定密钥
ansible_ssh_private_key_file=/path/to/key.pem
```

---

### 本地执行配置

- **临时执行方式：**

  ```bash
  ansible localhost -m ping -e 'ansible_python_interpreter="/usr/bin/env python"'
  ```

- **库存文件配置：**

  ```ini
  localhost ansible_connection=local ansible_python_interpreter="/usr/bin/env python"
  ```

---

### 主机密钥检查管理

**安全与便利权衡：**

- 默认启用：防止中间人攻击（MITM）

**禁用方式：**

- 在配置文件中：

  ```ini
  [defaults]
  host_key_checking = False
  ```

- 或通过环境变量：

  ```bash
  export ANSIBLE_HOST_KEY_CHECKING=False
  ```

> paramiko 模式的主机密钥检查性能较低，建议改用原生 SSH 连接。

---

### 其他连接方式

Ansible 支持多种非 SSH 连接插件：

- **本地管理**：`ansible_connection=local`
- **容器管理**：支持 chroot/LXC/jail 容器
- **反向拉取模式**：`ansible-pull` 通过定期从中央仓库拉取配置实现逆向管理

---

### 连接方法对比表

| 连接类型      | 适用场景             | 性能表现 | 安全等级 | 典型配置参数                  |
| ------------- | -------------------- | -------- | -------- | ----------------------------- |
| OpenSSH       | 标准 Linux 服务器管理 | ★★★★★    | ★★★★★    | ansible_ssh_private_key_file  |
| paramiko      | 旧系统兼容环境       | ★★★☆☆    | ★★★★☆    | ansible_password              |
| local         | 控制节点自身管理     | ★★★★★    | -        | ansible_connection=local      |
| docker        | Docker 容器管理      | ★★★★☆    | ★★★☆☆    | ansible_connection=docker     |
| ansible-pull  | 大规模节点自治部署   | ★★★☆☆    | ★★★★☆    | cron 定时任务触发             |

---

### 安全最佳实践

- **最小权限原则**：为不同业务主机配置专用运维账号
- **密钥轮换机制**：定期更新 SSH 密钥对
- **敏感信息保护**：密码类参数应使用 Ansible Vault 加密
- **审计日志**：记录所有关键连接操作

```