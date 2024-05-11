# Epeius
[English](./README.md) | 简体中文 | [日本語](./README-ja_JP.md)

以 Serverless 的方式部署 Trojan

## 快速上手
- 在 Cloudflare Workers 仪表盘中创建一个新的 Worker
- 将 [worker.js](./src/worker.js) 文件中的代码粘贴到 Worker 代码编辑器中
- 用你自己的密码替换 `sha224Password`，你可以在 [这里](https://www.atatus.com/tools/sha224-to-hash) 生成密码。或者，你可以之后在 Cloudflare Workers 设置中增加 `SHA224PASS` 环境变量
- 将自定义域名绑定到 Worker
- 访问 `https://[你的域名]/link` 并用你的明文密码替换 `ca110us`

## 未支持事项
- UDP 🙅(Cloudflare workers runtime 当前不支持 UDP)

## 免责声明 
该项目仅供学习/研究目的，用户对法律合规和道德行为负责，作者对任何滥用行为概不负责。

## 参考
[zizifn/edgetunnel](https://github.com/zizifn/edgetunnel)
