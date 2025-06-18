# Installing Ansible

Ansible 是一种无代理自动化工具，可安装在单个主机（称为控制节点）上。

大多数 Ansible 环境有三个主要组件：

![Ansible 架构图](images/ansible-arch.svg)

- **控制节点**（运行 Ansible 的计算机）：几乎可以使用任何安装了 Python 的类 UNIX 计算机。
- **托管节点**（Ansible 管理的机器）：托管节点不需要安装 Ansible，但需要 Python 来运行 Ansible 生成的 Python 代码。托管节点还需要一个用户帐户，该帐户可以通过 SSH 连接到具有交互式 POSIX shell 的节点。
- **Inventory**：受管节点列表。您可以在控制节点上创建一个清单，以向 Ansible 描述主机部署情况。

从控制节点，Ansible 可以使用 SSH、Powershell 远程处理和许多其他传输方式远程管理整个机器和其他设备（称为托管节点），所有这些都可以通过一个简单的命令行界面完成，无需数据库或守护进程。

---

## 社区版本 Ansible 的两种形态

- **ansible-core（极简）**：包含一组内置模块和插件语言和运行时包。
- **ansible（更大）**：“包含电池”的软件包，添加了社区精选的 Ansible Collections，用于自动化各种设备。

---

## 安装 Ansible

```bash
pip install ansible
```

在您的文件系统上创建一个项目文件夹：

```bash
mkdir ansible_quickstart && cd ansible_quickstart
```

您可以通过检查版本来测试 Ansible 是否正确安装：

```bash
ansible --version
```
此命令显示的版本是 ansible-core 已安装的相关包的版本。

要检查已安装的软件包的版本：

```bash
ansible-community --version
```

---

## argcomplete

`argcomplete` 是一个 Python 包，用于为命令行程序（如 Ansible）提供参数自动补全功能。

---

## 配置 Ansible

Ansible 中的某些设置可以通过配置文件（`ansible.cfg`）进行调整，您可以使用 `ansible-config` 命令行实用程序列出可用选项并检查当前值。

---

## 迁移指南

更新版本的迁移指南：[https://docs.ansible.com/ansible/latest/porting_guides/porting_guides.html](https://docs.ansible.com/ansible/latest/porting_guides/porting_guides.html)
```
