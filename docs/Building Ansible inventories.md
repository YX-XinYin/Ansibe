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

## 分组嵌套：父/子组关系


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
## 使用多库存源

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

## 库存变量定义方式

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

## 变量继承与优先级

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

## 连接行为参数

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

## 库存组织最佳实践

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







## 进阶用法

- 使用动态库存来跟踪不断启动和停止的服务器和设备的云服务。
- 使用模式来自动化库存的特定子集。
- 扩展和改进 Ansible 用于你的库存的连接方法。
