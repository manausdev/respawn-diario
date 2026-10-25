# ADR-0001: Correções de segurança — atualização do tar (6.2.1 → 7.5.22)

- **Status**: Aceito
- **Data**: 2026-10-25
- **Relacionado**: Issue #1 (revisão de segurança diária)

## Contexto

A revisão de segurança diária automatizada identificou 1 vulnerabilidade
CRITICAL conhecida no repositório: `tar@6.2.1`, dependência transitiva de
`cacache@15.3.0`, `node-gyp@8.4.1` e `sqlite3@5.1.7` (resolvida no
`package-lock.json`).

O pacote `tar` 6.2.1 está afetado por 7 CVEs conhecidas:

| CVE | GHSA | Tipo | Corrigida em |
|-----|------|------|--------------|
| CVE-2026-23745 | GHSA-8qq5-rm4j-mr97 | Path traversal via linkpath | tar 7.5.3 |
| CVE-2026-26960 | — | Hardlink com escape de destino | tar 7.5.8 |
| CVE-2026-31802 | — | Path traversal via symlink drive-relativo (Windows) | tar 7.5.11 |
| CVE-2026-59873 | GHSA-23hp-3jrh-7fpw | DoS via gzip bomb (CVSS 9.2 — disparou a revisão) | tar 7.5.19 |
| CVE-2026-59875 | GHSA-gvwx-54wh-qm9j | DoS via bytes NUL em headers PAX | tar 7.5.17 |
| CVE-2026-59871 | GHSA-w8wr-v893-vjvp | TypeError via conversão de tipo em PAX | tar 7.5.18 |
| CVE-2026-73566 | GHSA-r292-9mhp-454m | Stack overflow via filesFilter | tar 7.5.21 |

## Decisão

1. **Atualizar `tar` para 7.5.22** — versão mais recente, corrige todas
   as 7 CVEs listadas.
2. **Usar override do npm** (`"overrides": { "tar": "^7.5.22" }` no
   `package.json`) — `tar` é dependência transitiva (não declarada
   diretamente); o override força a resolução para a versão corrigida
   sem alterar os declarantes (`cacache`, `node-gyp`, `sqlite3`).
3. **Atualizar `package-lock.json` cirurgicamente** — substituição da
   entrada `node_modules/tar` e das entradas aninhadas correspondentes
   (`@isaacs/fs-minipass@4.0.1`, `chownr@3.0.0`, `minipass@7.1.3`,
   `minizlib@3.1.0`, `yallist@5.0.0`), preservando todas as demais
   entradas. Hashes de integridade verificados no registry npm.
4. **Não alterar os declarantes** — `cacache@15.3.0`, `node-gyp@8.4.1` e
   `sqlite3@5.1.7` permanecem nas versões atuais.

## Consequências

**Positivas:**

- Todas as 7 CVEs CRITICAL/HIGH em `tar` ficam corrigidas.
- O override documenta explicitamente a versão mínima segura, evitando
  regressão para versão vulnerável em futuros `npm install`.
- Nenhuma outra entrada do lockfile é afetada; nenhuma entrada
  top-level fica órfã (`chownr@2.0.0`, `fs-minipass@2.1.0`,
  `minipass@3.3.6`, `minizlib@2.1.2`, `yallist@4.0.0` seguem usadas
  por outros pacotes).

**Negativas/Riscos:**

- `tar` 7.x exige Node >= 18 (a série 6.x exigia >= 10): ambientes com
  Node antigo não conseguirão instalar as dependências.
- Mudança de major (6 → 7) pode introduzir incompatibilidades de API
  para `cacache`/`node-gyp` em cenários não cobertos pelos testes
  atuais (risco avaliado como baixo; uso conservador da API de tar).
- O override aplica-se a toda a árvore de dependências; se algum pacote
  passar a exigir estritamente `tar` 6.x, haverá conflito de resolução
  (hoje não há — verificado: nenhum peer/dependência restringe `tar`).
