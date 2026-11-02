# Epeius

[English](./README.md) | 简体中文 | [日本語](./README-ja_JP.md)

**运行在 Cloudflare Workers 上的 WebSocket 到 TCP 转发工具。**

Epeius 是一个轻量网络代理，通过 WebSocket 接收 Trojan 协议请求，再使用 Workers Sockets API 将流量转发到 TCP 目标。协议解析、连接处理和双向数据转发均在一个 JavaScript Worker 中完成。

## 工作原理

```text
兼容 Trojan 的客户端
        |
   WebSocket over TLS
        |
Cloudflare Worker（Epeius）
        |
   出站 TCP 连接
        |
     目标服务
```

1. 客户端与 Worker 建立 WebSocket 连接，并发送 Trojan 请求。
2. Worker 校验认证字段，读取目标地址和端口。
3. Worker 通过 `cloudflare:sockets` 建立 TCP 连接，并在两端之间双向转发数据。

TLS 保护客户端到 Worker 的连接。Worker 到目标服务之间使用 TCP 连接；转发内容本身是否加密，取决于所承载的应用协议。

## 快速开始

需要一个 Cloudflare 账号，以及支持 **Trojan + WebSocket + TLS** 的客户端。

### 1. 准备密码

选择一个强密码，并在本地计算其 SHA-224 摘要。使用 Python 3 可以执行：

```sh
python3 -c 'import getpass, hashlib; print(hashlib.sha224(getpass.getpass("Password: ").encode("utf-8")).hexdigest())'
```

客户端使用原始密码，Worker 使用计算得到的 **56 位小写十六进制摘要**，不要将原始密码填入 Worker 的摘要配置。原始密码和摘要都应妥善保管。

### 2. 部署 Worker

1. 在 Cloudflare 控制台创建 Worker。
2. 添加名为 `SHA224PASS` 的 Worker [Secret](https://developers.cloudflare.com/workers/configuration/secrets/)，填入步骤 1 中生成的摘要，并部署该配置。
3. 打开代码编辑器，用 [src/worker.js](./src/worker.js) 替换初始代码，然后部署。
4. 使用 Worker 的域名，也可以按需绑定[自定义域名](https://developers.cloudflare.com/workers/configuration/routing/custom-domains/)。

使用代理前，请设置自己的 `SHA224PASS`。未设置时，代码会回退到源码中公开的示例凭据。

### 3. 配置客户端

| 配置项 | 值 |
| --- | --- |
| 协议 | Trojan |
| 服务器地址 | Worker 的域名，不含 `https://` |
| 端口 | `443` |
| 密码 | 原始密码，**不是** SHA-224 摘要 |
| 传输方式 | WebSocket |
| WebSocket 路径 | `/` |
| TLS | 开启；SNI 和 WebSocket Host 均使用同一个 Worker 域名 |

也可以访问 `https://<your-worker-hostname>/link` 获取客户端 URI 模板。模板始终使用固定占位符 `ca110us`，导入前需要将其替换为自己的密码。该接口不会读取或显示实际配置的密码，也不会随 `SHA224PASS` 的变化而更新。

如果密码包含 URI 保留字符，请对密码部分进行百分号编码后再填入链接，或直接按照上方表格手动配置客户端。

## 配置说明

| 名称 | 用途 |
| --- | --- |
| `SHA224PASS` | 客户端密码的 SHA-224 摘要。建议通过 Worker Secret 设置，它会覆盖源码中的 `sha224Password`。 |
| `PROXYIP` | 可选的备用 TCP 目标域名或 IP。没有合适的备用目标时，保持未设置即可。 |

当首次出站连接的读取结束或失败，且始终没有收到数据时，代码会尝试连接 `PROXYIP`，并沿用原目标端口；未设置 `PROXYIP` 时则重试原目标地址。这是直接 TCP 连接，不会执行 HTTP 或 SOCKS 代理握手。

## 支持范围与限制

- 仅支持 TCP `CONNECT` 请求，尚未实现 UDP。
- 请求解析支持 IPv4、IPv6 和域名；实际可连接的目标受 [Cloudflare TCP Socket 限制](https://developers.cloudflare.com/workers/runtime-apis/tcp-sockets/#considerations)约束。
- 客户端必须支持 WebSocket 传输。仅使用原生 Trojan TCP/TLS 传输的客户端不能直接连接这个 Worker。
- 认证使用单个配置的密码摘要，不包含用户管理、按用户配额或应用层限流功能。

请在有权访问的网络资源范围内使用，并遵守托管服务提供方的使用条款。

## 致谢

本实现参考了 [zizifn/edgetunnel](https://github.com/zizifn/edgetunnel)。

## 许可证

GNU GPL v2 许可证文本见 [LICENSE](./LICENSE)。
