# HR Buddy — Roteiro Completo Detalhado (Revisado)

> Chatbot de RH com RAG, MySQL, OpenAI e Telegram, orquestrado pelo n8n.
> 3 aulas de 45 minutos cada.

---

## Preparação Geral para o Instrutor (Pré-Aula)
- **Checklist:**
  - [ ] Rodar o *Load Data Flow* (carga do manual de RH) toda vez que o n8n for reiniciado. O banco em memória (Simple Vector Store) é apagado quando a máquina virtual reinicia.
  - [ ] Verificar se o MySQL no Railway está Online (planos gratuitos dormem por inatividade).
  - [ ] Ter o Telegram web/mobile aberto para a demonstração final.
  - [ ] Garantir que credenciais OpenAI (para Chat e Embeddings) e Telegram Bot estão ativas.

---

## AULA 01 — O Cérebro do HR Buddy (RAG, Chunking e n8n)

### 📌 Objetivo
Criar duas esteiras essenciais: uma para ler o PDF (Carga) e outra para responder perguntas baseadas nesse PDF (Conversação usando Vector Store).

### Fluxo 1: Carga de Dados (Load Data Flow)
Resumo: Puxa o arquivo TXT/PDF do GitHub, "fatia" em pedaços e salva na memória do Agente.

**Passo a Passo dos Nós:**
1. **Manual Trigger:** Inicia o processo manualmente.
2. **HTTP Request:** Puxa o manual da empresa.
   - *Method:* `GET`
   - *URL:* (Link bruto do GitHub do arquivo `Manual de RH ChocolaTech.txt`)
   - *Response Format:* `Text` (em Options → Response. Por padrão o n8n tenta ler JSON e quebra).
3. **Simple Vector Store (Insert):** Guarda os vetores na memória.
   - Conecte ao conector "Document": **Default Data Loader** (para fatiar o texto automaticamente).
   - Conecte ao conector "Embedding": **Embeddings OpenAI** usando o modelo `text-embedding-3-small`.
   - *Memory Key:* `vector_store_key`.

### Fluxo 2: Conversa Base Inicial (Retriever Flow)
Resumo: Janela de chat nativo do n8n que responde com base no que salvamos no passo de carga.

**Passo a Passo dos Nós:**
1. **When chat message received:** Gatilho de chat nativo do n8n (usaremos isso apenas até a Aula 02).
2. **AI Agent:** Cérebro principal.
   - *Source for Prompt:* `Connected Chat Trigger Node`
   - Conecte por baixo (Model): **OpenAI Chat Model** (ex: `gpt-4o-mini`).
   - Conecte por baixo (Memory): **Simple Memory** (apenas um).
   - Conecte por baixo (Tool): **Simple Vector Store**.
     - *Operation:* `Retrieve Documents (As Tool for AI Agent)`.
     - *Dica Crítica:* Conecte a ele exatamente o **mesmo nó Embeddings OpenAI** (`text-embedding-3-small`) usado na carga. Misturar Google Gemini e OpenAI destrói a bússola vetorial do Agente!

---

## AULA 02 — Memória Corporativa (Banco de Dados MySQL)

### 📌 Objetivo
Dar identificação ao funcionário. O robô vai buscar no banco MySQL (Railway) os dados de quem está conversando.

### Passo a Passo dos Nós
1. Mantemos o Retriver Flow da Aula 01.
2. A partir do conector "Tool" do **AI Agent**, puxe uma segunda linha e adicione o nó **`Select rows from a table in MySQL`**.
3. **Configuração da Credencial MySQL (Railway):**
   - Ative o "Public Networking / TCP Proxy Domain" no painel do Railway para acesso externo.
   - *Host:* `caboose.proxy.rlwy.net` (exemplo) | *Porta:* `23446` (exemplo)
   - *Usuário:* `root` | *Banco:* `railway`
4. **Configuração da Pesquisa (Nó MySQL):**
   - *Table:* `funcionarios` | *Operation:* `Select`
   - Em **Select Rows -> Conditions**, configure:
     - *Column:* `nome`
     - *Operator:* Mude para **`Like`** (se deixar `Equal`, a busca falha em pequenos desencontros de espaçamento ou sobrenomes).
     - *Value:* Clique no ícone de brilho mágico **(✨)** ao lado direito do campo para ativar o preenchimento pela IA via `$fromAI()`.
       - *Name:* `nome_funcionario`
       - *Description:* `"Apenas o nome do funcionário, envolvido por porcentagens para busca parcial. Exemplo: %Ana Lima%"`

### System Prompt do AI Agent (O Segredo)
No nó `AI Agent`, atualize a caixa **System Message** com regras para dar liberdade ao robô buscar exatamente o que o usuário digita:
```text
Você é o HR Buddy, assistente virtual de RH da ChocolaTech.
REGRAS:
1. Sempre responda em português.
2. Responda APENAS dúvidas relacionadas a RH.
IDENTIFICAÇÃO DO FUNCIONÁRIO:
- Se o usuário não disser quem é, pergunte o nome completo dele logo na primeira mensagem.
- Use a ferramenta MySQL para buscar na tabela funcionarios usando SEMPRE o NOME COMPLETO informado pelo usuário na conversa.
- Se encontrado: use os saldos de férias e banco de horas.
- Se não encontrado: não invente dados pessoais. Responda apenas com base nas políticas gerais de RH do Vector Store.
Use a base de conhecimento para dúvidas gerais.
```

### ⚠️ Erro Clássico da Aula 02
No nó MySQL, o alerta vermelho *"No parameters are set up to be filled by AI"* avisa que a ferramenta está inútil. Esse aviso MATA o Agente, o obrigando a tentar adivinhar a tabela de outras formas e entrar em loop de Query. Ele só some quando o aluno clicar no aviso de mágica **(✨)** no campo Value, ativando corretamente a função `$fromAI()`.

---

## AULA 03 — Produto Real (Telegram e Guardrails)

### 📌 Objetivo
1. Transformar o projeto em um app real no celular.
2. Criar uma "Catraca" (Guardrails) bloqueando perguntas de curiosos (como receitas de bolo) ANTES delas ativarem o Agente de IA, economizando dinheiro.
3. Resolver o problema da Memória Sem Estado (Stateless) ensinando o Session ID isolado por pessoa.

### Passo a Passo dos Nós
*Dica para Aula: Abandone o antigo fluxo da Aula 02 e crie um **Workflow Novo** "HR Buddy - Telegram" para não poluir a didática.*

1. **Telegram Trigger (A Porta de Entrada):**
   - Adicione e crie as credenciais (Token do resgatado com o @BotFather). *Updates:* `message`.
2. **Message a Model (O Classificador / Porteiro):**
   - Conecte ao Telegram Trigger. Use um Model OpenAI puro e barato (`gpt-4o-mini`).
   - *Prompt:* `"Classifique a mensagem abaixo em três blocos: SAUDACAO, RH ou OUTRO. Mensagem: {{ $json.message.text }}. Responda apenas com UMA PALAVRA."`
3. **Switch (A Catraca):**
   - Roteia o caminho dependendo da palavra classificada:
     - *Rule 1 (RH):* Se `{{ $json.output[0].content[0].text }}` for igual a `RH`.
     - *Rule 2 (SAUDACAO):* Se igual a `SAUDACAO`.
     - *Fallback (OUTRO):* Na seção Options (abaixo), defina "Fallback Output" como `Extra Branch`.
4. **Respostas Diretas (Para Saudação e Fallback):**
   - Adicione nós **Send a text message (Telegram)** nas saídas secundárias.
   - Textos: *"Olá, sou o assistente de RH!"* e *"Desculpe, só respondo questões de RH"*.
   - **ID do Telegram:** Código: `{{ $('Telegram Trigger').item.json.message.chat.id }}`. Ensine pelo método visual de arrastar (veja dica no final).
5. **A Rota Principal de RH (O AI Agent Completo):**
   - Na saída "RH" do Switch, reconstrua seu **AI Agent** (usando as ferramentas da Aula 02: Vector Store Mapeado e MySQL Tool).
   - Como ele não está conectado mais ao Chat Nativo azul:
     - *Source for Prompt:* `Define below`.
     - *Prompt:* Digite `{{ $('Telegram Trigger').item.json.message.text }}` *Ou arraste a palavra `text` do nó Telegram Trigger usando o histórico.*
   - No **Simple Memory**, configure a separação de histórico por usuário:
     - *Session ID:* `Define below`
     - *Key:* Digite o id do chat do Telegram: `{{ $('Telegram Trigger').item.json.message.chat.id }}`.
6. **Despacho Final:**
   - Encerre a ponta verde do AI Agent ligando ele a um último nó **Send a text message**.
   - *Chat ID:* O mesmo id resgatado.
   - *Text:* Digite literalmente `{{ $json.output }}` (Onde repousa o recado elaborado e super inteligente gerado pelo Agente final).

### 💡 Dica de Ouro Didática (A Técnica da "Esteira")
Quando ensinar aos alunos como resgatar o `Chat ID` e a mensagem original usando expressões cheias de pontuação (`$().item.json...`), evite focar em código e mostre o **n8n visual**:
**Roteiro de Fala:** *"Gente, o Switch substituiu nosso texto pela palavra 'RH', então o ID do usuário original se perdeu! Como o n8n funciona como uma esteira, vamos 'puxar um fio' lá do início. Usem a Engrenagem (Add Expression), abram o painel esquerdo: `Nodes -> Telegram Trigger -> Output Data -> JSON -> message -> chat`. Cliquem em `id` e apenas arrastem pra caixa! Vejam a mágica gerando o código feio sozinha!"*
