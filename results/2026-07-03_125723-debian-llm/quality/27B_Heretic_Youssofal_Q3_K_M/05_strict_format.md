# 27B_Heretic_Youssofal_Q3_K_M — 05_strict_format

## Prompt

Extract the structured data from the text below and output ONLY a JSON array — no prose, no markdown fences, no trailing commentary. Each object must have exactly these keys in this order: "name" (string), "port" (integer or null), "tls" (boolean), "tags" (array of strings, sorted ascending). If a port is not stated, use null. If TLS is not mentioned, use false.

Text:
- auth-service listens on 8443 over TLS, tagged: security, core
- cache is on port 6379, tags: perf
- worker has no exposed port, tagged worker, async, and it uses TLS
- gateway on 443 TLS, no tags

Your entire response must be valid JSON parseable by a strict parser on the first line.

## Response

[
  {
    "name": "auth-service",
    "port": 8443,
    "tls": true,
    "tags": [
      "core",
      "security"
    ]
  },
  {
    "name": "cache",
    "port": 6379,
    "tls": false,
    "tags": [
      "perf"
    ]
  },
  {
    "name": "worker",
    "port": null,
    "tls": true,
    "tags": [
      "async",
      "worker"
    ]
  },
  {
    "name": "gateway",
    "port": 443,
    "tls": true,
    "tags": []
  }
]
