# Epeius

English | [简体中文](./README-zh_CN.md) | [日本語](./README-ja_JP.md)

**WebSocket-to-TCP forwarding on Cloudflare Workers.**

Epeius is a lightweight network proxy that accepts Trojan-protocol requests over WebSocket and forwards traffic to TCP destinations using the Workers Sockets API. It brings protocol parsing, connection handling, and stream forwarding together in a single JavaScript Worker.

## How it works

```text
Trojan-compatible client
        |
   WebSocket over TLS
        |
Cloudflare Worker (Epeius)
        |
   Outbound TCP socket
        |
Destination service
```

1. The client opens a WebSocket connection to the Worker and sends its Trojan request.
2. The Worker checks the authentication field and reads the destination address and port.
3. It opens a TCP connection with `cloudflare:sockets` and forwards data in both directions.

TLS protects the client-to-Worker connection. The Worker opens a TCP socket to the destination; any encryption inside the forwarded traffic depends on the application protocol.

## Quick start

You need a Cloudflare account and a client that supports **Trojan over WebSocket with TLS**.

### 1. Prepare a password

Choose a strong password and calculate its SHA-224 digest locally. With Python 3:

```sh
python3 -c 'import getpass, hashlib; print(hashlib.sha224(getpass.getpass("Password: ").encode("utf-8")).hexdigest())'
```

Keep the original password for the client. The Worker uses the resulting **56-character lowercase hexadecimal digest**, not the original password. Treat both values as credentials.

### 2. Deploy the Worker

1. Create a Worker in the Cloudflare dashboard.
2. Add and deploy a Worker [secret](https://developers.cloudflare.com/workers/configuration/secrets/) named `SHA224PASS` containing the digest from step 1.
3. Open the code editor, replace the starter code with [src/worker.js](./src/worker.js), and deploy it.
4. Use the Worker's hostname, or optionally attach a [custom domain](https://developers.cloudflare.com/workers/configuration/routing/custom-domains/).

Configure your own `SHA224PASS` before using the proxy. Without it, the code falls back to a publicly known example credential bundled in the source.

### 3. Configure the client

| Setting | Value |
| --- | --- |
| Protocol | Trojan |
| Server | Your Worker's hostname, without `https://` |
| Port | `443` |
| Password | Your original password, **not** the SHA-224 digest |
| Transport | WebSocket |
| WebSocket path | `/` |
| TLS | Enabled; use the same hostname for SNI and the WebSocket Host header |

You can also visit `https://<your-worker-hostname>/link` for a client URI template. The template always contains the literal placeholder `ca110us`; replace it with your own password before importing it. It does not read or reveal the configured password and does not change when `SHA224PASS` changes.

If your password contains reserved URI characters, percent-encode the password before inserting it into the link, or configure the client manually using the settings above.

## Configuration

| Name | Purpose |
| --- | --- |
| `SHA224PASS` | SHA-224 digest of the client password. Set it as a Worker secret; it overrides `sha224Password` in the source. |
| `PROXYIP` | Optional fallback TCP hostname or IP address. Leave unset unless you have a suitable alternative destination. |

The fallback is attempted when the initial outbound read ends or fails without receiving any data. It uses `PROXYIP` with the original destination port, or retries the original address if `PROXYIP` is unset. This is a direct TCP connection, not an HTTP or SOCKS proxy handshake.

## Scope and limitations

- TCP `CONNECT` requests only; UDP is not implemented.
- The request parser accepts IPv4, IPv6, and domain-name destinations. Actual connectivity is subject to [Cloudflare's TCP socket restrictions](https://developers.cloudflare.com/workers/runtime-apis/tcp-sockets/#considerations).
- Clients must support the WebSocket transport. A client using only native Trojan over TCP/TLS cannot connect directly to this Worker.
- Authentication uses a single configured password digest. The project does not include user management, per-user quotas, or application-level rate limiting.

Use the project with network resources you are authorized to access and in accordance with the hosting provider's terms.

## Acknowledgements

This implementation references [zizifn/edgetunnel](https://github.com/zizifn/edgetunnel).

## License

See [LICENSE](./LICENSE) for the GNU GPL v2 license text.
