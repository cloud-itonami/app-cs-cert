# Operator quickstart — app-cs-cert

Nine files, 27 KB, no runtime: a design record, two sample certificates and a set of
SHACL shapes for a badge-issuing service. There is no `wrangler.jsonc`, no appview and
no test suite, so nothing here can be started — but the shapes and the data **can** be
checked against each other, and this document does that.

Steps marked ✅ were run on 2026-08-16.

---

## 1. ✅ The sample data satisfies its own shapes — checked, not assumed

`shacl/shapes.jsonld` holds four `sh:NodeShape`s carrying **34 property shapes** and
**87 constraint assertions** between them (the 34 `sh:path` entries name the properties;
the assertions are the `sh:datatype`, `sh:minCount` and friends attached to them). `content/certifications/seed.jsonld` holds two instances. Nothing in the
repository runs one against the other, so the first thing worth knowing is whether they
agree.

The shapes use a **closed, small vocabulary** — eight constraint kinds and nothing else:

```bash
python3 -c "
import json,io,collections
c=collections.Counter(k for n in json.load(io.open('shacl/shapes.jsonld'))['@graph']
                        for p in (n.get('sh:property') or []) for k in p)
print(dict(c))"
#   sh:path 34, sh:datatype 34, sh:minCount 33, sh:in 9,
#   sh:maxCount 4, sh:pattern 3, sh:minInclusive 2, sh:maxInclusive 2
```

No `sh:node`, no `sh:or`, no SPARQL constraints. That matters: a checker covering those
eight kinds is **complete for this file**, so a pass means something. A checker was
written for exactly those eight, refusing to report a pass if it ever meets a ninth,
and the result is:

```
constraint kinds in shapes: 8  implemented: 8  NOT implemented: none
instances in seed: 2   constraint evaluations: 20
  etzhayyim:CsCertificateShape   targets in seed: 1
  etzhayyim:CsAssessmentShape    targets in seed: 1
  etzhayyim:CsFindingShape       targets in seed: 0
  etzhayyim:CsBadgeShape         targets in seed: 0
violations: 0
```

**Proved load-bearing before being written down**, on a copy of the data: setting
`certLevel` to `cs-platinum` is caught by `sh:in`, rewriting `identifier` as `CERT-1`
is caught by the `^etzhayyim-cs-[a-z0-9]{8,}$` pattern, and an `overallScore` of 101 is
caught by `sh:maxInclusive`. Each yields exactly one violation; the unmodified data
yields none.

**Two of the four shapes have no instance to check.** `CsFindingShape` (8 property
shapes) and `CsBadgeShape` (6) target classes the seed never uses, so **14 of the 34
property shapes have never been evaluated against anything** — the 20 evaluations above
are all from the other two. They are not wrong — they are untested,
and a sample file is where that would be fixed.

One correction worth recording, because it would have been published as a defect in the
data: the first run reported one violation, `schema:actionStatus` missing on the
assessment. It is not missing. The seed writes it as the bare term `actionStatus`, and
the seed's `@context` is `["https://schema.org/", …]`, so a bare term there resolves
against schema.org — the same IRI the shape names as `schema:actionStatus`. The fault
was in the checker's expansion, not the data. **A validator that only half-expands
reports the data as broken.**

## 2. ⚠ A standard SHACL tool cannot load these shapes

`shapes.jsonld` declares its context as a remote URL:

```bash
python3 -c "import json,io;print(json.load(io.open('shacl/shapes.jsonld'))['@context'])"
#   ['https://resources.etzhayyim.com/ontology/cs-cert/context.jsonld', {'sh': …, 'xsd': …}]
```

That host does not exist:

```bash
python3 -c "
import socket
for h in ['resources.etzhayyim.com','etzhayyim.com']:
    try: print(h, socket.gethostbyname(h))
    except Exception as e: print(h, 'DOES NOT RESOLVE')"
#   resources.etzhayyim.com DOES NOT RESOLVE
#   etzhayyim.com 172.67.179.128
```

An IRI need not dereference — `etzhayyim:` expanding to
`https://resources.etzhayyim.com/ontology#` is fine as an identifier. **A `@context`
does need to**, because a JSON-LD processor must fetch it to expand the document. So
any off-the-shelf SHACL or JSON-LD tool pointed at `shapes.jsonld` fails at load, which
is the likeliest reason these shapes have never been run.

The content it needs is already in the repository. `shacl/context.jsonld` defines
`schema`, `prov`, `etzhayyim`, `xsd` and the term aliases — and **nothing references it
by path**:

```bash
grep -rl 'shacl/context' . --include='*.jsonld' --include='*.md' --include='*.edn'
#   (nothing)
```

The checker in §1 used the local file, which is what a tool would need to be handed.
Two ways out, both small: serve that hostname, or make the shapes' context a relative
reference to the file beside them.

## 3. What is here, precisely

```bash
git ls-files
#   CLAUDE.md  NOTICE  PROJECT.jsonld  README.edn  migration.edn
#   content/certifications/seed.jsonld
#   shacl/context.jsonld  shacl/shapes.jsonld
#   ux/260228-cs-cert-ux-design.jsonld
```

`README.edn` calls it `:kind :standalone-app-artifact` with
`:canonical-record-format :edn`, and `migration.edn` records the extraction from
`etzhayyim/root` at `691c245d` — 7 tracked files, 26,847 bytes, `:go-files-created 0`.
Seven plus the two records is the nine here.

`CLAUDE.md` describes the service: an SSL-seal-style badge for sites that have passed a
security assessment, three certification levels, and four public surfaces at
`certs.etzhayyim.com` — `/xrpc`, `/badge/{certId}.svg`, `/badge/{certId}.js`,
`/verify/{certId}`.

## 4. ⚠ Nothing serves `certs.etzhayyim.com`

```bash
grep -rl 'certs\.etzhayyim\.com' <root>/orgs/*/*/wrangler.jsonc \
                                <root>/orgs/*/*/appview/*/wrangler.jsonc
#   (nothing)
```

and the workspace's surface index has **zero rows** whose host contains `certs`. This
repository has no `wrangler.jsonc` of its own either. So all four documented surfaces
are design, not deployment — including `/verify/{certId}`, which is the one a
badge-holder's visitor would click. A badge that cannot be verified is worse than no
badge, so this is the gap to close before any certificate is issued to a third party.

Nothing here claims otherwise: `README.edn` says `:standalone-app-artifact`, and
`migration.edn` records zero Go files created. The record is honest; the operator just
needs to know that "documented surface" and "served surface" are different things.

## 5. What the maturity instrument sees ✅

```
· orgs/cloud-itonami/app-cs-cert  own=0.049  axis-docs=0bp → +2500bp
```

The instrument reads `README.md` for its README component and this repository's is
`README.edn` by declaration; `axis-substrate` reads 0 because there is no `src/`, and
that is **correct here** — JSON-LD data and SHACL shapes are not substrate, and this
repository does not claim to hold code. As with the other data repositories in this
fleet, the low score is the honest one (ADR-2608052000).
