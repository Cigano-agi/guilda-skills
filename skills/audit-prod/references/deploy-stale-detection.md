# Deploy Stale Check — manual deploy de checkout desatualizado

> Descoberto em 27/07/2026 durante auditoria Baitmap. Prod rodava código de 11/07 porque alguém rodou `vercel --prod` de dentro de uma pasta local parada em 53b22c0, enquanto origin/main estava em 9609086 (23/07). 34 commits perdidos. 7 dias de regressão silenciosa.

## O problema

`vercel --prod` sobe a **pasta atual**, não o branch git. Se você está num checkout stale (sem `git pull` recente), o que vai pro ar é o build da pasta — que pode estar semanas atrás do remoto. O deploy da CLI sobrescreve o melhor deploy que estava no ar, e ninguém percebe porque:

- As rotas que sumiram (Pix, crons, pixel) não têm heartbeat próprio
- O alarme morava no código apagado (reconcile + monitor-load faziam ntfy)
- Sistema ficou mudo sobre a própria mudez

## Detecção

1. **Comparar commit do deploy com HEAD do branch correto** — `vercel list` mostra o commit de cada deploy. Se o deploy veio de CLI (`Source: vercel-cli`), risco alto.
2. **Validar data do deploy vs data do último commit no branch** — deploy de 25/07 com commit de 11/07 é sinal vermelho.
3. **Sondar rotas que mudaram entre o commit stale e o HEAD** — se o main tem Pix store webhook, e `/api/pay/pix/store` não existe em prod, confirma.

## Recuperação

1. `git pull --ff-only` no checkout local (zero conflito se main é ancestral)
2. `vercel --prod` do repo sincronizado
3. Pós-prova obrigatória:
   - 5+ rotas que estavam 404 viram 401
   - Pixel no HTML (se foi adicionado)
   - Crons registrados (se voltaram no vercel.json)
4. Verificar que migrations já aplicadas ao banco casam com o schema do código novo (não re-rodar)

## Prevenção

- Conectar GitHub ↔ Vercel pra push no main virar deploy automático — elimina deploy manual como única fonte de deploy
- Se deploy manual for inevitável: checklist de pré-deploy com `git status` + `git log -1 --oneline` + `git fetch && git rev-list --count HEAD..origin/main` pra ver quantos commits você está atrás