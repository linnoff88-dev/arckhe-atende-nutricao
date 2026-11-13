# atende-nutricao

**ðŸ‡§ðŸ‡· [PortuguÃªs](README.md) Â· ðŸ‡ºðŸ‡¸ [English](README.en.md) Â· ðŸ‡ªðŸ‡¸ [EspaÃ±ol](README.es.md)**

[![Atende NutriÃ§Ã£o](guia/assets/banner.jpg)](https://inematds.github.io/atende-nutricao/guia/)

## ðŸ“– Guia de uso

Landing e guia passo a passo (PT/EN/ES): **https://inematds.github.io/atende-nutricao/guia/** Â· [English](https://inematds.github.io/atende-nutricao/guia/en/) Â· [EspaÃ±ol](https://inematds.github.io/atende-nutricao/guia/es/)

ðŸŽ¬ VÃ­deo: https://www.youtube.com/watch?v=qwkQLYkiiXs

Sistema de atendimento ao paciente para **um consultÃ³rio de nutriÃ§Ã£o**, em Python 3 com biblioteca padrÃ£o e SQLite: agenda de consultas (primeira consulta e retornos, presencial ou online com link informado pela equipe), confirmaÃ§Ã£o automÃ¡tica, lista de espera, retorno programado, **plano alimentar** registrado pelo nutricionista e consultado pelo paciente na conversa, **diÃ¡rio** (Ã¡gua, refeiÃ§Ãµes feitas/puladas, peso, foto de refeiÃ§Ã£o por metadados) com lembretes opt-in e metas, FAQ, fila humana no Telegram, sinais de alerta, campanhas com trava mÃ­nima de publicidade, LGPD de dado de saÃºde, cadastros, backup e exportaÃ§Ã£o mensal para o Painel de RecuperaÃ§Ã£o do **Raio-X de Margem** (pacote `clinica`). Base: o `atende-clinica`, adaptado.

**Estado (19/10/2026): v0.8.0 implementada.** Backend, chat web (`/`) e Ã¡rea da equipe (`/equipe`), Evolution, Telegram, Docker e instruÃ§Ãµes de VPS; 99/99 testes passam, `verificar-limites.sh` â†’ LIMITES OK e `verificar-independente.py` â†’ INDEPENDENTE OK (inclui `docker build` e healthcheck). Deploy na VPS Ã© feito pelo responsÃ¡vel. Estado detalhado em `longrun/2026-10-05-nutri-v1/state.md`.

## O que o sistema faz (v1)

- **Agenda:** grade por profissional, primeira consulta/retorno/bioimpedÃ¢ncia, presencial ou **online** (link informado pela equipe entra na confirmaÃ§Ã£o e no chat), sem horÃ¡rio duplo (pedidos concorrentes: um 201, os outros 409), bloqueios, remarcaÃ§Ã£o, presenÃ§a/falta.
- **Conversa** (chat web e WhatsApp pela Evolution): agendar pelo chat, confirmar com `1`/`2`, lista de espera, FAQ, `meu plano` / `o que posso comer no lanche da tarde` (responde sÃ³ o que estÃ¡ no plano do paciente), registrar `2 copos`, `pulei o almoÃ§o`, `peso 82,5` (resposta neutra, sem julgamento), `link` da consulta online, atendente humano, `PARAR`, `parar lembretes`.
- **Limites do bot:** nunca prescreve dieta, suplemento ou cÃ¡lculo; dÃºvida fora do plano/FAQ vai para o humano; sinais de alerta (fÃ­sicos â†’ 192; compulsÃ£o/purgaÃ§Ã£o/ideaÃ§Ã£o â†’ 188) abrem fila com prioridade alta.
- **Plano alimentar:** refeiÃ§Ãµes por horÃ¡rio com opÃ§Ãµes e observaÃ§Ãµes, versÃµes, envio ao paciente pela equipe; sem cÃ¡lculo nutricional no v1.
- **DiÃ¡rio e metas:** sÃ©rie de peso com variaÃ§Ã£o, copos de Ã¡gua com meta, adesÃ£o Ã s refeiÃ§Ãµes, foto como registro (sÃ³ metadados), lembretes de Ã¡gua e das refeiÃ§Ãµes do plano em horÃ¡rios escolhidos, com janela de tolerÃ¢ncia.
- **Equipe:** pÃ¡gina `/equipe` com token (agenda, pacientes, plano, diÃ¡rio/adesÃ£o, sÃ©rie de peso, FAQ, fila humana); Telegram com `/agenda`, `/alertas`, `/fila`, `/responder`, `/encerrar` e avisos de agendamento/cancelamento.
- **LGPD e Raio-X:** exportar e apagar o dado de saÃºde do paciente; campanha sÃ³ com consentimento e trava mÃ­nima (âš  CFN a confirmar pelo dono); `GET /api/raiox?mes=` e CLI `raiox --acompanhamento` com os ids do pacote `clinica`; backup/restauraÃ§Ã£o; Docker.

O que fica de fora do v1 estÃ¡ na seÃ§Ã£o 14 da especificaÃ§Ã£o.

## Como rodar no seu ambiente

### PrÃ©-requisitos

- Linux/macOS com `git` e `python3` â‰¥ 3.10. A aplicaÃ§Ã£o nÃ£o tem dependÃªncias externas.
- `pytest` sÃ³ para rodar os testes (`pip install pytest`; Ã© ferramenta de desenvolvimento).
- `docker` (com Compose) para a verificaÃ§Ã£o independente e o deploy na VPS.
- Evolution e bot do Telegram sÃ£o opcionais: sem as variÃ¡veis, o WhatsApp usa caixa de saÃ­da simulada e nada Ã© enviado ao Telegram.

### 1. Conferir os testes

```sh
git clone https://github.com/inematds/atende-nutricao && cd atende-nutricao
python3 -m pytest -q tests/ | tail -n 1     # 99 passed
./atende --help                             # serve, raiox, backup, restaurar
```

### 2. Executar

```sh
mkdir -p dados && cp exemplos/nutricao.json dados/nutricao.json   # troque TROQUE-ESTE-TOKEN
./atende serve --porta 8080 --dados dados
```

O chat do paciente fica em `/`, a pÃ¡gina da equipe em `/equipe` (header `X-Token`). `ATENDE_AGORA` fixa o relÃ³gio sÃ³ para testes. Antes de usar com pacientes, leia `docs/DECISOES-ABERTAS.md`: os itens marcados com âš  (trava de campanha, palavras de alerta, regras do peso, regras do CFN) pedem a confirmaÃ§Ã£o do dono. O deploy (compose, proxy HTTPS, webhooks da Evolution e do Telegram, backup diÃ¡rio) estÃ¡ na seÃ§Ã£o [Deploy na VPS](#deploy-na-vps).

### 3. VerificaÃ§Ãµes

```sh
bash longrun/2026-10-05-nutri-v1/verificar-limites.sh            # LIMITES OK no fechamento; hoje acusa sÃ³ guia/ e README.en/es (docs posteriores)
python3 longrun/2026-10-05-nutri-v1/verificar-independente.py   # INDEPENDENTE OK
```

`verificar-limites.sh` confere o hash do contrato, o nÂº de testes, `hash-congelado.txt` igual ao gravado pelo commit de congelamento, sÃ³ arquivos do escopo, sem dependÃªncia externa, sem URL externa, sem chave, sem download de mÃ­dia, sem `0.0.0.0` no cÃ³digo. `verificar-independente.py` procura valores do fixture copiados no cÃ³digo, sobe um consultÃ³rio nunca visto (passo de 20 min, consulta de 40 min, retorno em 15 dias, sem atendimento online, plano com refeiÃ§Ãµes de outros nomes, outro relÃ³gio) e confere regras que sÃ³ passam se a lÃ³gica for geral â€” agenda, chat, plano, peso, Ã¡gua, refeiÃ§Ã£o, lembrete, retorno, trava de campanha, concorrÃªncia, Raio-X, LGPD â€”; depois faz `docker build` e testa o healthcheck do container.

## Como foi feito (e como refazer ou adaptar)

O sistema foi construÃ­do pelo mÃ©todo de execuÃ§Ã£o longa (`~/projetos/execucao-longa`): primeiro o contrato (`docs/ESPECIFICACAO.md`), as decisÃµes propostas e os 99 testes caixa-preta; depois o congelamento por hash (`longrun/2026-10-05-nutri-v1/congelar.sh`); entÃ£o um agente pela **assinatura** (Codex ou Claude Code, nenhuma API) trabalhou em ciclos atÃ© `99 passed`, `LIMITES OK` e `INDEPENDENTE OK`. Com o contrato congelado, testes e especificaÃ§Ã£o ficaram protegidos: o agente nÃ£o podia alterÃ¡-los; teste em conflito com a spec ia para `failures.md` (portÃ£o humano).

Para adaptar a outro consultÃ³rio ou mudar uma regra marcada com âš , o mÃ©todo continua no repositÃ³rio: edite `docs/ESPECIFICACAO.md` e os testes, congele de novo e rode um dos caminhos abaixo.

```sh
bash longrun/2026-10-05-nutri-v1/congelar.sh                                  # grava hash-congelado.txt e faz 1 commit
~/projetos/execucao-longa/tools/loop-longrun.sh longrun/2026-10-05-nutri-v1   # loop headless (Codex gpt-6-astra)
```

- O loop usa `longrun/2026-10-05-nutri-v1/loop.env` (20 ciclos Ã— 30 min, 8 GB, parada apÃ³s 3 ciclos sem avanÃ§o, `CODEX_ARGS` liberando socket local no sandbox); faz `flock`, `timeout`, `MemoryMax`, reverte ciclo que mexer em arquivo protegido, commita checkpoint por ciclo e para sozinho quando o teste final passa.
- `/goal` no Codex TUI: `codex -c sandbox_workspace_write.network_access=true` e cole `longrun/2026-10-05-nutri-v1/prompt-goal-codex.md`.
- `/goal` no Claude Code: sessÃ£o nova na pasta do projeto, seguindo `longrun/2026-10-05-nutri-v1/prompt-goal-claude.md`.
- Acompanhar: `longrun/2026-10-05-nutri-v1/loop.log`, `state.md` (Funciona / Falta / Como retomar) e `progress.md`.

## Arquivos

- `docs/ESPECIFICACAO.md` â€” contrato (21 seÃ§Ãµes, tabela de rotas, regras com fronteiras, fora do v1).
- `docs/DECISOES-ABERTAS.md` â€” decisÃµes com proposta padrÃ£o; perguntas para o dono (inclusive o âš  do CFN).
- `docs/MAPA-RAIO-X.md` â€” cada vazamento do pacote `clinica` â†’ peÃ§a do v1 ou motivo de ficar fora.
- `docs/VALIDACAO.md` â€” validaÃ§Ã£o adversarial do plano (05/10/2026): checklist teste Ã— spec, contas conferidas, saÃ­da dos checks de trava/harness.
- `tests/` â€” 99 testes caixa-preta (`conftest.py` sobe o CLI, relÃ³gio fixo, servidor falso de Evolution/Telegram). Fixture em `tests/fixtures/nutricao.json`; cÃ³pia em `exemplos/`.
- `longrun/2026-10-05-nutri-v1/` â€” goal (nÃ­vel 3+), plan, state, canal, loop.env, prompts, `congelar.sh`, `verificar-limites.sh`, `verificar-independente.py`.
- `FALHAS.md` â€” uma linha por falha (data, o que quebrou, menor correÃ§Ã£o, prompt | infra).

## Deploy na VPS

Execute estes passos na VPS com Docker e Compose instalados. A publicaÃ§Ã£o e a configuraÃ§Ã£o das integraÃ§Ãµes sÃ£o feitas pelo responsÃ¡vel pelo consultÃ³rio.

### Preparar e subir

Na pasta do projeto:

```sh
umask 077
cp .env.exemplo .env
mkdir -p dados backups
# Apenas na primeira instalaÃ§Ã£o (nÃ£o sobrescreva uma configuraÃ§Ã£o existente):
test -f dados/nutricao.json || cp exemplos/nutricao.json dados/nutricao.json
chmod 700 dados backups
chmod 600 .env dados/nutricao.json
```

Edite `dados/nutricao.json`: substitua `TROQUE-ESTE-TOKEN` por um segredo prÃ³prio e ajuste o consultÃ³rio, profissionais, serviÃ§os e horÃ¡rios. Profissionais, serviÃ§os e FAQ semeiam o banco somente na primeira execuÃ§Ã£o; depois sÃ£o alterados pela API. Deixe `ATENDE_AGORA` ausente em produÃ§Ã£o.

```sh
docker compose up -d --build
docker compose ps
docker compose logs --tail=30 atende
```

O volume `./dados:/dados` mantÃ©m o banco e a configuraÃ§Ã£o entre reinÃ­cios; `./backups:/backups` recebe os backups. A imagem tambÃ©m cria `/dados` quando iniciada sem volume. O healthcheck consulta `/api/saude` com Python, sem instalar dependÃªncias.

### Proxy HTTPS

Configure no proxy reverso da VPS um domÃ­nio com certificado TLS e encaminhamento para `127.0.0.1:8080`. A porta publicada pelo Compose fica somente em loopback. Encaminhe os caminhos `/`, `/equipe`, `/api/` e `/webhook/`, preservando os headers `X-Token` e `X-Telegram-Bot-Api-Secret-Token`. A configuraÃ§Ã£o do proxy depende do domÃ­nio e do servidor jÃ¡ usados na VPS.

### Evolution e Telegram

Sem as variÃ¡veis preenchidas, o WhatsApp usa a caixa de saÃ­da simulada e nenhum aviso Ã© enviado ao Telegram. Configure no `.env` somente os serviÃ§os que for usar:

- Evolution: `EVOLUTION_URL` deve ser o endereÃ§o acessÃ­vel **de dentro do container**, junto de `EVOLUTION_API_KEY`, `EVOLUTION_INSTANCIA` e `WEBHOOK_SEGREDO`. No painel da Evolution, configure o webhook do domÃ­nio HTTPS no caminho `/webhook/evolution/<segredo>`, com evento `MESSAGES_UPSERT`. O segredo do caminho deve coincidir com `WEBHOOK_SEGREDO`. Fotos sÃ£o registradas apenas por metadados.
- Telegram: preencha `TELEGRAM_API_URL`, `TELEGRAM_TOKEN`, `TELEGRAM_CHAT_ID` e `TELEGRAM_SEGREDO`. Registre o webhook com o mÃ©todo `setWebhook` do bot: envie `url` com o endereÃ§o HTTPS do domÃ­nio seguido de `/webhook/telegram` e `secret_token` com o mesmo valor de `TELEGRAM_SEGREDO`. O Telegram enviarÃ¡ esse segredo no header prÃ³prio. Inclua o bot no grupo definido em `TELEGRAM_CHAT_ID`.

Recrie o serviÃ§o apÃ³s alterar o ambiente: `docker compose up -d --force-recreate atende`. Os segredos das integraÃ§Ãµes sÃ£o diferentes do token da equipe em `nutricao.json`. NÃ£o publique o `.env` ou os dados dos pacientes.

### Tarefas e backup diÃ¡rio

As tarefas sÃ£o explÃ­citas: programe uma chamada local autenticada a `POST /api/tarefas/rodar` a cada minuto para confirmaÃ§Ãµes, retornos, lembretes e reenvios. O corpo Ã© `{}`; use o token do consultÃ³rio no header `X-Token`. NÃ£o informe `agora` em produÃ§Ã£o. Rodadas repetidas sÃ£o idempotentes.

Para criar uma cÃ³pia consistente do SQLite com o servidor ligado:

```sh
docker compose exec -T atende ./atende backup --dados /dados --saida /backups/estado.sqlite3
```

Exemplo de cron diÃ¡rio Ã s 03:15 (ajuste `/srv/atende-nutricao` para a pasta real; o escape de `%` Ã© necessÃ¡rio no crontab):

```cron
15 3 * * * cd /srv/atende-nutricao && docker compose exec -T atende ./atende backup --dados /dados --saida /backups/estado-$(date +\%F).sqlite3
```

Guarde tambÃ©m cÃ³pias protegidas de `dados/nutricao.json` e `.env`: o backup do banco nÃ£o inclui esses arquivos. Defina retenÃ§Ã£o e cÃ³pia segura fora da VPS. Para restaurar, pare o serviÃ§o antes e escolha o arquivo desejado:

```sh
docker compose stop atende
docker compose run --rm --no-deps atende ./atende restaurar --dados /dados --entrada /backups/estado.sqlite3
docker compose up -d atende
```

---

Projeto adaptado da comunidade INEMA TDS - Marca Arckhe