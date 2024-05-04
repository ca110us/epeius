# Epeius
English | [简体中文](./README-zh_CN.md)

Deploy Trojan using a Serverless approach

## Quick start
- Create a new Worker in Cloudflare Workers dashboard. 
- Paste code from [worker.js](./src/worker.js) into the worker code editor. 
- Replace `sha224Password` with your own password. You can generate [here](https://www.atatus.com/tools/sha224-to-hash).
- Binding a custom domain to the worker.
- Visit `https://[YOUR_DOMAIN]/link` and replace `ca110us` with your plain password.

## Not supported
- UDP 🙅

## Disclaimer
This project is for study/research purposes only. Users are responsible for legal compliance and ethical conduct. The author disclaims all liability for misuse.

## Reference
[zizifn/edgetunnel](https://github.com/zizifn/edgetunnel)