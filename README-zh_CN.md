# Epeius
[English](./README.md) | 简体中文

以 Serverless 的方式部署 Trojan

## 快速上手
- 在 Cloudflare Workers 仪表盘中创建一个新的 Worker
- 增加变量 PASSWORD ,并设置成你自己的明文密码所对应的hash，你可以在 [这里](https://www.atatus.com/tools/sha224-to-hash) 生成密码
- 将 [worker.js](./src/worker.js) 文件中的代码粘贴到 Worker 代码编辑器中
- 将自定义域名绑定到 Worker
- 访问 `https://[你的域名]/link` 并用你的明文密码替换 `ca110us`

## 配置 (example for clash)
增加节点配置
```yaml
- name: cf-trojan
  type: trojan
  server: trojan.cf.xxx.xxx
  port: 443
  password: xxxxxxxx
  udp: false
  tls: true
  network: ws
```

## 未支持事项
- UDP 🙅(Cloudflare workers runtime 当前不支持 UDP)

## 免责声明 
该项目仅供学习/研究目的，用户对法律合规和道德行为负责，作者对任何滥用行为概不负责。

## 参考
[zizifn/edgetunnel](https://github.com/zizifn/edgetunnel)
