# Checklist por camada — o que checar e como provar

Para cada camada: rode os checks que se aplicam ao stack, anote evidência (arquivo:linha / comando / probe), dê nota 0-10.

## 1. Frontend
- Páginas-chave renderizam em PROD (curl/browser, não localhost).
- Meta/OG/SEO (`layout`/head), título e description reais.
- Resíduos de direção antiga (copy off-brand), elementos `disabled`/mortos, dados mocados (regra no-mocks: 0,00 > valor fake).
- Mobile: layout quebra? (viewport, overflow)
- **Se o produto vende:** landing = vendedor. Tabela promessa/prova social/oferta explícita/CTA/preço/objeções — hoje × falta.

## 2. APIs & Backend Logic
- Listar rotas (`find app/api -name route.ts` / equivalente do framework).
- Cada rota: auth-gated? validação de input (zod/manual)? erro devolve status certo (401/402/422)?
- Probe real: rota protegida sem sessão → espera 401/307.
- Idempotência onde há dinheiro/crédito (chave única, retry seguro).
- Webhooks: verificação de assinatura OU reconfirmação server-side; nunca confiar no payload.
- Timeout/retry em chamadas a terceiros.

## 3. Database & Storage
- Migrations versionadas? (dir de migrations, as-code)
- Constraints que protegem dinheiro: unique em ext_event_id, append-only (trigger), FKs.
- Índices pras queries quentes (ok pular em volume baixo — anotar).
- Storage: o que guarda, cresce sem limite? lixo acumulando (runs de QA)?

## 4. Auth & Permissions
- Gate de sessão (middleware) provado em prod (307/401).
- RLS/authz em TODAS as tabelas — inclusive as criadas depois do setup inicial (grep `enable row level security` em cada migration).
- Deny-all deliberado documentado (tabela só de service-role).
- Grant/trial farmável? (email descartável, sem confirmação, sem cap por IP) — estimar custo do abuso.

## 5. Hosting & Deployment
- Deploy automático? **Provar**: comparar commit de prod vs HEAD do main (gotcha real: push sem auto-deploy = prod velha silenciosa). **Gotcha pior — manual deploy de checkout stale:** `vercel --prod` (ou equivalente) rodado de pasta local DESATUALIZADA sobrescreve o deploy bom com código semanas atrás. **Detecção:** na Vercel, ver a origem do deploy (`Source`: `vercel-cli` = manual vs `github` = CI). Se veio de CLI, validar que o commit no ar é HEAD do branch correto.
- Env vars de prod completas (listar via CLI da plataforma, comparar com o que o código lê: `grep -r "process.env"`).
- Domínio próprio, staging/preview.

## 6. Cloud & Compute
- TODO componente fora do serverless (worker, cron, sidecar): roda ONDE? PC de dono = 🔴 automático.
- Health check existe e responde AGORA (curl).
- Auto-restart (pm2/systemd/Docker restart policy)? Sobrevive a reboot?

## 7. CI/CD & Version Control
- `.github/workflows` (ou equivalente) existe? Roda testes+typecheck+build em push/PR?
- Testes existem e passam AGORA (rodar).
- Branch protection, convenção de commits.

## 8. Security & RLS
- Segredos: no repo? (git log -p por padrões `sk-`, `key=`...) vazaram por canal externo? rotação pendente?
- Headers de segurança (CSP etc.) — anotar, raramente bloqueador.
- Dado sensível (cartão, PII): onde trafega, onde persiste, onde loga. PCI: cartão só em memória de request.
- Service-role/admin keys só server-side (grep no client bundle).

## 9. Rate Limiting
- Limite por IP/user nas rotas públicas e caras? (criar cobrança, disparar job, chamar IA)
- Limite econômico (créditos/quota) conta, mas não segura martelo de rota.
- Fila: justiça por tenant (1 user pode monopolizar o worker?).
- Circuit breaker em webhook (N payloads inválidos → corta).

## 10. Caching & CDN
- CDN default da plataforma ok pra estático.
- Recomputação cara repetida? (mesmo input re-processa tudo + re-paga IA) → cache por chave+janela.
- Prompt caching se usa LLM.

## 11. Load Balancing & Scaling
- Web escala pela plataforma; achar o GARGALO real (quase sempre o worker/job).
- Quantos consumidores? Concorrência configurada? Caminho pra N workers desenhado?
- Fila aguenta (Postgres/Redis) — o consumidor é o limite?

## 12. Error Tracking & Logs
- Logs estruturados? Vão pra onde — arquivo local = 🔴 (morre com a máquina).
- Alerting: se o worker morrer AGORA, alguém fica sabendo? (teste mental obrigatório)
- Erro de negócio persiste em dado consultável (não só log)?
- Sentry/uptime check/status page — mínimo viável: uptime no /health + Sentry free.

## 13. Availability & Recovery
- Backup: existe? AGENDADO ou manual? testado restore?
- Job preso: reconciler/retry re-enfileira sozinho?
- Reboot da máquina do worker: volta sozinho?
- Runbook de recuperação escrito?

---

## Calibração de nota
- 10 = provado em prod, resiliente, monitorado.
- 8 = roda em prod com evidência, gaps menores.
- 6 = roda mas com resíduo/risco conhecido.
- 4 = existe código real, não deployado/agendado/provado.
- 2 = decisão tomada, nada construído.
- 0 = nem decisão.