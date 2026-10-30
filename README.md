# DENTAL IA - Serviço de WhatsApp (Render)

Este pequeno servidor é a **ponte** entre o WhatsApp da clínica e o cérebro do robô.
A IA, a agenda e os pacientes ficam no Readdy Backend. O que roda aqui é só o "cabo":

1. Conecta o número escaneando um QR Code.
2. Envia cada mensagem recebida para a IA analisar.
3. Devolve a resposta no WhatsApp e entrega as mensagens escritas pelo painel.

> Nenhuma chave de IA fica aqui. Este serviço guarda apenas a senha da ponte.

---

## Passo 1 - Subir para o GitHub

1. Descompacte o arquivo **dentalia-wa-service.zip** que você baixou do painel.
2. No GitHub, crie um repositório novo (pode ser privado), por exemplo **dentalia-whatsapp**.
3. Envie **todo o conteúdo da pasta wa-service** (package.json, render.yaml, src/, README.md e .env.example) para a raiz do repositório.

Pelo site do GitHub: **Add file > Upload files** e arraste o conteúdo da pasta.
Atenção: o arquivo **package.json** precisa ficar na raiz do repositório, e não dentro de uma subpasta.

---

## Passo 2 - Criar o serviço no Render

### Opção A - Blueprint (mais rápido, usa o render.yaml)

1. Entre em https://dashboard.render.com
2. Clique em **New > Blueprint**
3. Escolha o repositório que você acabou de criar
4. O Render lê o **render.yaml**, cria o serviço e o disco automaticamente
5. Preencha as variáveis pedidas e confirme

### Opção B - Web Service manual

1. Clique em **New > Web Service** e escolha o repositório
2. Preencha:

| Campo | Valor |
|---|---|
| Name | dentalia-whatsapp |
| Root Directory | (deixe vazio) |
| Runtime | Node |
| Build Command | npm install |
| Start Command | npm start |
| Plan | Starter (necessário para o disco da sessão) |

3. Em **Advanced**, adicione um **Disk**:

| Campo | Valor |
|---|---|
| Name | wa-session |
| Mount Path | /var/data |
| Size | 1 GB |

> O disco é o que faz o WhatsApp **não pedir QR novo** a cada deploy ou reinício.
> No plano gratuito não existe disco: conecta, mas cai quando o serviço reinicia.

---

## Passo 3 - Variáveis de ambiente

Em **Environment Variables**, adicione:

| Nome | Valor |
|---|---|
| BRIDGE_URL | https://vouf0jzspa5glm7d1r7w.helloreaddy.com/functions/v1/wa-agent |
| WA_BRIDGE_SECRET | a MESMA senha que você cadastrou como WA_BRIDGE_SECRET no backend |
| SUPABASE_ANON_KEY | sb_publishable_iDnRFJLBJDSeCuJiKlSvMes3qKbxwzWJ |
| AUTH_DIR | /var/data/auth |
| TZ | America/Sao_Paulo |

Clique em **Create Web Service** e espere o deploy terminar.
Quando os logs mostrarem **ouvindo na porta** e **conectado como ...**, deu certo.

---

## Passo 4 - Conectar o WhatsApp

1. Abra o painel do DENTAL IA e vá em **Robô WhatsApp**.
2. Informe a chave de acesso do painel (WA_PANEL_TOKEN).
3. A aba **Conexão** mostra o QR Code assim que o serviço gerar.
4. No celular da clínica: **WhatsApp > Configurações > Dispositivos conectados > Conectar dispositivo** e aponte para o QR.
5. O status vira **Conectado** e o número aparece no painel.

O QR é renovado automaticamente a cada ~20 segundos. Se expirar, atualize a tela.

---

## Passo 5 - Testar de verdade

1. De outro celular, mande um "oi" para o número conectado.
2. A IA responde, pergunta o nome e oferece horários realmente livres da agenda.
3. Ao escolher um horário, a consulta é gravada em **appointments** com origem **ai**.
4. O painel mostra a conversa e a agenda mostra o agendamento.

---

## Problemas comuns

**O painel diz que o serviço não está online**
O Render ainda está construindo, ou a BRIDGE_URL está errada. Confira o endpoint **/health** do seu serviço.

**Acesso negado ao conectar**
O WA_BRIDGE_SECRET do Render está diferente do que está salvo no backend.

**A IA não responde**
Falta a chave GEMINI_API_KEY no backend. Sem ela a mensagem é gravada, mas fica sem resposta.

**Fica pedindo QR toda hora**
Falta o disco em /var/data ou o serviço está no plano gratuito.

**A IA inventou preço**
Nunca deveria. Cadastre os procedimentos e valores reais; ela é instruída a usar apenas essa tabela.

---

## Aviso importante

Esta conexão usa o WhatsApp Web (não oficial). É a forma mais rápida de começar, mas
o WhatsApp pode bloquear números usados para disparo em massa. Para produção séria,
o caminho é migrar para a Cloud API oficial da Meta: o cérebro do robô continua igual,
só troca este arquivo de transporte.
