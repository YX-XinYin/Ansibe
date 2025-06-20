# 剧本（Playbook）简介

剧本是 Ansible 用于部署和配置清单中节点的自动化蓝图，采用 YAML 格式编写。

- **以提升的权限或以不同的用户身份执行任务**  
  可以通过 `become` 或 `remote_user` 参数指定任务以 sudo 或其他用户身份运行。

- **使用循环重复执行列表中项目的任务**  
  利用 `with_items` 或 `loop` 语法，实现对多个对象的批量操作。

- **委托剧本在不同的机器上执行任务**  
  通过 `delegate_to` 参数，将特定任务委托给其他主机执行。

- **运行条件任务并使用剧本测试评估条件**  
  使用 `when` 语句，根据变量或主机状态有条件地执行任务。

- **使用块对任务集进行分组**  
  利用 `block` 语法，将相关任务组织在一起，便于统一处理错误、权限提升等。

这些用法让 Ansible 剧本不仅能实现简单的自动化，还能灵活应对复杂的运维场景。



## Playbook 核心概念

Playbook 是 Ansible 的配置、部署和编排语言，采用 YAML 格式编写，具有以下核心特性：

- **声明式配置**：描述系统应达到的状态而非具体操作步骤
- **幂等性**：多次执行结果一致
- **可重复性**：支持版本控制，确保部署一致性

---

## 基础语法结构

```yaml
- name: 更新Web服务器  # 任务名称
  hosts: webservers    # 目标主机组
  remote_user: root    # 连接用户

  tasks:               # 任务列表
    - name: 安装Apache最新版
      ansible.builtin.yum:
        name: httpd
        state: latest
```

---

## 关键功能特性

### 1. 多机编排

```yaml
- name: 数据库部署阶段
  hosts: dbservers
  tasks:
    - name: 确保PostgreSQL运行
      ansible.builtin.service:
        name: postgresql
        state: started
```

### 2. 安全检查模式

```bash
ansible-playbook --check playbook.yml  # 仅模拟执行不实际修改
```

### 3. 高级语法特性

#### 安全字符串处理

```yaml
mypassword: !unsafe 234%234{435lkj{{lkjsdf  # 标记特殊字符不进行模板渲染
```

#### YAML 锚点与别名

```yaml
vars:
  app_defaults: &defaults
    port: 8080
    timeout: 30

  web_app:
    <<: *defaults
    name: web_service
```

---

## 数据操作能力

### 1. 复杂数据转换

```yaml
- name: 提取字典键值
  debug:
    msg: "{{ chains | map('extract', chains_config) | flatten }}"
```

### 2. 列表过滤

```yaml
unique_values: "{{ groups['all'] | map('extract', hostvars, 'var') | unique }}"
```

### 3. 条件合并

```yaml
final_list: "{{ (list_a + list_b) | unique }}"
```

---

## 验证与优化工具

### 1. ansible-lint

```bash
ansible-lint playbook.yml  # 检查最佳实践违规
```

### 2. 执行控制

```bash
ansible-playbook --limit webservers site.yml  # 限制执行范围
```

---

## 架构模式

### 1. 标准推送模式

```bash
ansible-playbook deploy.yml  # 中心节点主动推送配置
```

### 2. 拉取模式

```bash
ansible-pull -U https://repo/playbooks.git  # 节点主动拉取配置
```
```
