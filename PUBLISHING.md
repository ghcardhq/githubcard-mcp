# Publishing to the MCP registry

`server.json` registers the server as `com.githubcard/mcp` in the
[official MCP registry](https://registry.modelcontextprotocol.io). Registry
names under `com.githubcard/` are proved by a DNS record on `githubcard.com`.
The npm package `ghcard` is checked separately, through its
`"mcpName": "com.githubcard/mcp"` field.

## Before the first publish

1. **Publish the npm package first.** The registry checks that `ghcard@<version>`
   exists on npm and carries the matching `mcpName`. See
   [ghcardhq/ghcard](https://github.com/ghcardhq/ghcard) for how to release it.
2. **Deploy githubcard.com** with the `/mcp` endpoint, so the remote URL answers.
3. **Make this repo public.** `server.json` links to it.

## One-time DNS setup

Generate an Ed25519 key pair. Keep the private key out of this repo, for
example in a password manager:

```bash
openssl genpkey -algorithm Ed25519 -out githubcard-mcp-key.pem
```

Print the public key for the DNS record:

```bash
echo "v=MCPv1; k=ed25519; p=$(openssl pkey -in githubcard-mcp-key.pem -pubout -outform DER | tail -c 32 | base64)"
```

Add that string as a **TXT** record on the apex `githubcard.com` (in Cloudflare
DNS: name `@`, type TXT). Check it resolves:

```bash
dig +short TXT githubcard.com
```

## Publish

Install the publisher (macOS: `brew install mcp-publisher`; other systems:
[releases](https://github.com/modelcontextprotocol/registry/releases)), then from
this repo:

```bash
mcp-publisher login dns --domain githubcard.com --private-key "$(openssl pkey -in githubcard-mcp-key.pem -noout -text | grep -A3 'priv:' | tail -n +2 | tr -d ' :\n')"
mcp-publisher publish
```

Check the listing:

```bash
curl -s "https://registry.modelcontextprotocol.io/v0/servers?search=com.githubcard/mcp"
```

## New versions

When `ghcard` is released at a new version, update both `version` fields in
`server.json` (the top-level one and `packages[0].version`), commit, and run
`mcp-publisher publish` again. The registry rejects a version it already has.
