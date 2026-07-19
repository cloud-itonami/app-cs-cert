# etzhayyim-project-cs-cert

## Overview

etzhayyim CS Certified — サイバーセキュリティ診断済みサイトに認証ロゴ (バッジ) を発行・表示するサービス。
SSL trust seal のように、セキュリティ診断完了サイトが信頼性を証明するバッジを埋め込める。

- **公開ドメイン**: `certs.etzhayyim.com`
- **API**: `certs.etzhayyim.com/xrpc` (XRPC-Web)
- **バッジ配信**: `certs.etzhayyim.com/badge/{certId}.svg` / `.js`
- **検証ページ**: `certs.etzhayyim.com/verify/{certId}`

## Certification Levels

| Level | ID | Badge Color | Validity | Checks |
|-------|-----|-------------|----------|--------|
| Basic | `cs-basic` | `#4CAF50` (Green) | 90 days | OWASP Top 10, SSL/TLS, Security headers, DNS |
| Standard | `cs-standard` | `#2196F3` (Blue) | 180 days | Basic + vuln scan, API security, auth flow |
| Advanced | `cs-advanced` | `#9C27B0` (Purple) | 365 days | Standard + pentest, code review, infra audit |

## Directory Structure

```
etzhayyim-project-cs-cert/
├── PROJECT.jsonld              # Project metadata
├── CLAUDE.md                   # This file
├── content/certifications/     # Certificate and assessment data (JSON-LD)
├── shacl/                      # SHACL validation shapes
│   ├── context.jsonld          # JSON-LD context
│   └── shapes.jsonld           # CsCertificate, CsAssessment, CsFinding, CsBadge shapes
├── ux/                         # UX design specifications
└── wasm/
    └── etzhayyim-wasm-cs-cert-badge-k7m3p9x2/
        ├── src/app.ts             # TS Native
        ├── kotodama.jsonld           # kotodama config
        ├── go.mod              # Go module
        ├── wit/world.wit       # WIT interfaces
        └── k8s/                # App manifests
```

## Entity Types

- `etzhayyim:CsCertificate` — 発行済み認証 (ID, domain, level, status, score, expiry)
- `etzhayyim:CsAssessment` — セキュリティ診断レコード (checks, findings, score)
- `etzhayyim:CsFinding` — 個別検出事項 (severity, category, status)
- `etzhayyim:CsBadge` — バッジメタデータ (format, color, embed URL)

## Key Rules

- **XRPC**: component 間通信は XRPC (HTTP/2)
- **App namespace**: `kotodama-runtime`
- **Performer framework**: `70-tools/performer` を使用
- **Static assets**: App 内の static delivery (`certs.etzhayyim.com`) で配信
- **API-Only component**: wasm component は API + バッジ SVG のみ処理
