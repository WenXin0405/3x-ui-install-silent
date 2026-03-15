# 3x-ui 静默安装脚本

基于原版 `install.sh` 修改的静默安装版本，支持通过环境变量进行非交互式安装，适用于自动化部署场景。

## 主要特性

- **无交互安装**: 全程无需用户输入，通过环境变量预配置
- **灵活配置**: 支持自定义端口、用户名、密码、访问路径
- **SSL 自动化**: 支持域名证书、IP证书、自定义证书三种方式
- **网络优化**: 支持 BBR 加速、TCP Fast Open
- **版本指定**: 可指定安装特定版本
- **向后兼容**: 未配置的选项自动生成随机值

## 环境变量

### 基本配置

| 变量名 | 说明 | 默认值 |
|--------|------|--------|
| `XUI_PORT` | 面板监听端口 | 随机生成 (1024-62000) |
| `XUI_USERNAME` | 登录用户名 | 随机生成 (10位字母数字) |
| `XUI_PASSWORD` | 登录密码 | 随机生成 (10位字母数字) |
| `XUI_WEB_BASE_PATH` | 访问路径前缀 | 随机生成 (18位字母数字) |
| `XUI_VERSION` | 安装版本号 (如 `v2.3.5`) | 最新稳定版 |

### SSL 证书配置

| 变量名 | 说明 | 默认值 |
|--------|------|--------|
| `XUI_SSL_TYPE` | SSL 类型: `none` / `domain` / `ip` / `custom` | `none` |
| `XUI_SSL_DOMAIN` | 域名 (SSL类型为 `domain` 时必填) | - |
| `XUI_SSL_PORT` | ACME HTTP-01 验证端口 (仅申请证书时使用) | `80` |
| `XUI_SSL_IPV6` | IPv6 地址 (IP证书可选) | - |
| `XUI_SSL_CERT_PATH` | 证书文件路径 (custom 类型) | - |
| `XUI_SSL_KEY_PATH` | 私钥文件路径 (custom 类型) | - |
| `XUI_SSL_RELOAD_CMD` | 证书更新后重载命令 | `systemctl restart x-ui` |
| `XUI_SKIP_SSL` | 跳过 SSL 配置 (`true` / `false`) | `false` |

> **端口说明**：
> - `XUI_PORT` 是面板访问端口，与SSL证书无关，可以是任意端口
> - `XUI_SSL_PORT` 是Let's Encrypt验证端口，仅申请证书时需要，默认80

### 网络优化配置

| 变量名 | 说明 | 默认值 |
|--------|------|--------|
| `XUI_BBR` | 开启 BBR 拥塞控制算法 (`true` / `false`) | `false` |
| `XUI_TCP_FASTOPEN` | 开启 TCP Fast Open (`true` / `false`) | `false` |

> **BBR**: 需要 Linux 内核 4.9+，可显著提升网络传输速度
> **TCP Fast Open**: 需要 Linux 内核 3.7+，可减少 TCP 连接建立延迟

### 高级配置

| 变量名 | 说明 | 默认值 |
|--------|------|--------|
| `XUI_MAIN_FOLDER` | 安装目录 | `/usr/local/x-ui` |
| `XUI_SERVICE` | systemd 服务目录 | `/etc/systemd/system` |

## 使用示例

### 1. 最小化安装

自动生成所有配置，适合快速测试：

```bash
curl -Ls https://raw.githubusercontent.com/WenXin0405/3x-ui-install-silent/main/install-silent.sh | bash
```

### 2. 指定端口和凭据

```bash
curl -Ls https://raw.githubusercontent.com/WenXin0405/3x-ui-install-silent/main/install-silent.sh | \
  XUI_PORT=8443 XUI_USERNAME=admin XUI_PASSWORD=MySecurePass123 XUI_WEB_BASE_PATH=secretadmin bash
```

### 3. 使用域名 SSL 证书

Let's Encrypt 域名证书，有效期 90 天，自动续期：

```bash
curl -Ls https://raw.githubusercontent.com/WenXin0405/3x-ui-install-silent/main/install-silent.sh | \
  XUI_PORT=443 XUI_USERNAME=admin XUI_PASSWORD=Pass123 XUI_SSL_TYPE=domain XUI_SSL_DOMAIN=panel.example.com bash
```

### 4. 使用 IP 证书

Let's Encrypt IP 短期证书，有效期约 6 天，自动续期：

```bash
curl -Ls https://raw.githubusercontent.com/WenXin0405/3x-ui-install-silent/main/install-silent.sh | \
  XUI_PORT=443 XUI_USERNAME=admin XUI_PASSWORD=Pass123 XUI_SSL_TYPE=ip bash
```

### 5. 完整配置（推荐）

开启所有优化选项：

```bash
curl -Ls https://raw.githubusercontent.com/WenXin0405/3x-ui-install-silent/main/install-silent.sh | \
  XUI_PORT=8878 \
  XUI_USERNAME=admin \
  XUI_PASSWORD=SecurePass123 \
  XUI_WEB_BASE_PATH=secret \
  XUI_SSL_TYPE=ip \
  XUI_BBR=true \
  XUI_TCP_FASTOPEN=true \
  bash
```

### 6. 使用自定义证书

适用于已有证书的场景：

```bash
curl -Ls https://raw.githubusercontent.com/WenXin0405/3x-ui-install-silent/main/install-silent.sh | \
  XUI_SSL_TYPE=custom \
  XUI_SSL_CERT_PATH=/etc/ssl/certs/fullchain.pem \
  XUI_SSL_KEY_PATH=/etc/ssl/private/privkey.pem \
  XUI_SSL_DOMAIN=panel.example.com \
  bash
```

### 7. 指定版本安装

```bash
curl -Ls https://raw.githubusercontent.com/WenXin0405/3x-ui-install-silent/main/install-silent.sh | \
  XUI_VERSION=v2.3.5 bash
```

### 8. 跳过 SSL 配置

```bash
curl -Ls https://raw.githubusercontent.com/WenXin0405/3x-ui-install-silent/main/install-silent.sh | \
  XUI_PORT=8080 XUI_SKIP_SSL=true bash
```

## Docker 集成示例

在 Dockerfile 中使用：

```dockerfile
FROM debian:bullseye

ENV XUI_PORT=443
ENV XUI_USERNAME=admin
ENV XUI_PASSWORD=admin123
ENV XUI_SSL_TYPE=none
ENV XUI_BBR=true
ENV XUI_TCP_FASTOPEN=true

COPY install-silent.sh /tmp/
RUN chmod +x /tmp/install-silent.sh && /tmp/install-silent.sh

EXPOSE 443
CMD ["x-ui", "start"]
```

## Ansible 集成示例

```yaml
- name: Install 3x-ui
  hosts: servers
  become: yes
  vars:
    xui_port: 8443
    xui_username: admin
    xui_password: "{{ vault_xui_password }}"
    xui_ssl_type: ip
    xui_bbr: true
    xui_tcp_fastopen: true
  tasks:
    - name: Download and run silent installer
      shell: |
        curl -Ls https://raw.githubusercontent.com/WenXin0405/3x-ui-install-silent/main/install-silent.sh | \
        XUI_PORT={{ xui_port }} \
        XUI_USERNAME={{ xui_username }} \
        XUI_PASSWORD={{ xui_password }} \
        XUI_SSL_TYPE={{ xui_ssl_type }} \
        XUI_BBR={{ xui_bbr }} \
        XUI_TCP_FASTOPEN={{ xui_tcp_fastopen }} \
        bash
```

## 安装后管理

安装完成后可使用以下命令管理面板：

| 命令 | 说明 |
|------|------|
| `x-ui` | 管理菜单 |
| `x-ui start` | 启动服务 |
| `x-ui stop` | 停止服务 |
| `x-ui restart` | 重启服务 |
| `x-ui status` | 查看状态 |
| `x-ui settings` | 查看设置 |
| `x-ui log` | 查看日志 |
| `x-ui update` | 更新版本 |
| `x-ui uninstall` | 卸载 |

## 证书管理

### 查看证书状态

```bash
# 查看已安装的证书列表
~/.acme.sh/acme.sh --list

# 查看证书详情
~/.acme.sh/acme.sh --info -d <域名或IP>

# 查看定时任务
crontab -l
```

### 手动续签证书

```bash
# 测试续签（dry-run）
~/.acme.sh/acme.sh --renew -d <域名或IP> --dry-run

# 强制续签
~/.acme.sh/acme.sh --renew -d <域名或IP> --force
```

### 证书文件位置

| 类型 | 路径 |
|------|------|
| 域名证书 | `/root/cert/<域名>/` |
| IP证书 | `/root/cert/ip/` |

## 网络优化验证

### 检查 BBR 状态

```bash
sysctl net.ipv4.tcp_congestion_control
# 输出: net.ipv4.tcp_congestion_control = bbr
```

### 检查 TCP Fast Open 状态

```bash
cat /proc/sys/net/ipv4/tcp_fastopen
# 输出: 3 (客户端+服务端已启用)
```

## 与原脚本对比

| 特性 | 原版 install.sh | install-silent.sh |
|------|----------------|-------------------|
| 安装方式 | 交互式 | 非交互式 |
| 配置方式 | 运行时输入 | 环境变量预设 |
| SSL 配置 | 必须交互选择 | 可预设或跳过 |
| BBR 加速 | 不支持 | 支持 |
| TCP Fast Open | 不支持 | 支持 |
| 凭据生成 | 可选择自定义或随机 | 未设置则自动随机 |
| 适用场景 | 手动安装 | 自动化部署、CI/CD |
| 输出信息 | 详细交互提示 | 简洁状态输出 |

## 注意事项

1. **端口可用性**: 确保 `XUI_PORT` 和 `XUI_SSL_PORT` 未被占用
2. **防火墙**: 安装前开放相应端口
3. **SSL 证书**: 
   - 域名证书需要域名已正确解析到服务器
   - IP 证书需要端口 80 可从公网访问
   - 自定义证书需确保证书文件存在且可读
4. **root 权限**: 必须以 root 用户运行
5. **内核版本**: BBR 需要 4.9+，TCP Fast Open 需要 3.7+

## 故障排除

### SSL 证书申请失败

```bash
# 检查端口是否开放
ss -tlnp | grep :80

# 检查防火墙
ufw status
# 或
firewall-cmd --list-ports
```

### 服务无法启动

```bash
# 查看服务状态
systemctl status x-ui

# 查看日志
x-ui log
# 或
journalctl -u x-ui -f
```

### 重置配置

```bash
# 重新运行安装脚本会重置配置
curl -Ls https://raw.githubusercontent.com/WenXin0405/3x-ui-install-silent/main/install-silent.sh | \
  XUI_PORT=新端口 XUI_USERNAME=新用户名 XUI_PASSWORD=新密码 bash
```

## 许可证

与原项目保持一致。
