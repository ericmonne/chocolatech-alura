# HR Buddy — Roteiro Completo Detalhado (Revisado)

> Chatbot de RH com RAG, MySQL, OpenAI e Telegram, orquestrado pelo n8n.
> 3 aulas de 45 minutos cada.

---

## Preparação Geral para o Instrutor (Pré-Aula)
- **Checklist:**
  - [ ] Rodar o *Load Data Flow* (carga do manual de RH) toda vez que o n8n for reiniciado. O banco em memória (Simple Vector Store) é apagado quando a máquina virtual reinicia.
  - [ ] Verificar se o MySQL no Railway está Online (planos gratuitos dormem por inatividade).
  - [ ] Ter o Telegram web/mobile aberto para a demonstração final.
  - [ ] Garantir que credenciais OpenAI/Gemini (para Chat e Embeddings) e Telegram Bot estão ativas.

---

## AULA 01 — O Cérebro do HR Buddy (RAG, Chunking e n8n)

### 📌 Objetivo
Criar duas esteiras essenciais: uma para ler o PDF (Carga) e outra para responder perguntas baseadas nesse PDF (Conversação usando Vector Store).

### Conceitos Abordados

| Conceito | Analogia / Explicação |
|---|---|
| RAG | Dar um livro para a IA ler antes de responder |
| Chunking | Por que não mandar o documento inteiro (limites de token e custo) |
| Embeddings | Transformar texto em vetores para busca por semelhança |
| n8n | Ferramenta low-code visual, ideal para cursos e prototipagem |
| n8n Cloud vs. local | Cloud é gerenciado e mais fácil para começar; local tem mais controle |

### Fluxo 1: Carga de Dados (Load Data Flow)
Resumo: Puxa o arquivo TXT/PDF do GitHub, "fatia" em pedaços e salva na memória do Agente.

**Passo a Passo dos Nós: (resumo)**
1. **Manual Trigger:** Inicia o processo manualmente.
2. **HTTP Request:** Puxa o manual da empresa.
   - *Method:* `GET`
   - *URL:* `https://raw.githubusercontent.com/ericmonne/chocolatech-alura/main/Manual%20de%20RH%20ChocolaTech.txt`
   - *Response Format:* `Text` (em Options → Response. Por padrão o n8n tenta ler JSON e quebra).
3. **Simple Vector Store (Insert):** Guarda os vetores na memória.
   - Conecte ao conector "Document": **Default Data Loader** (para fatiar o texto automaticamente).
   - Conecte ao conector "Embedding": **Embeddings OpenAI** usando o modelo `text-embedding-3-small`.
   - Operation: `Retrieve Documents (As Tool for AI Agent)`
   - *Memory Key:* `vector_store_key`.
   - Description: `Busca informações sobre políticas de RH da empresa ChocolaTech`
4. **Embeddings OpenAI (ambos os nós):**
    - Modelo: `text-embedding-3-small`
    - Atenção: os dois nós de embeddings (Insert e Retrieve) DEVEM usar o mesmo modelo
5. **Default Data Loader:**
    - Type of Data: `JSON`
    - Text Splitting: `Simple`
    - Resultado: gerou 22 chunks a partir do documento

**Passo a passo detalhado para o Fluxo de Carga:**

### Passo 1: O Gatilho Manual
1. Clique com o botão direito na tela (ou no "+") e clique em **Add node**.
2. Pesquise por **`Manual Trigger`** (na versão em português pode aparecer como "On clicking 'execute'").
3. Funciona como um botão de "Play" para iniciar esse processo, pois ele só precisará rodar **uma única vez** cada vez que você for usar.

### Passo 2: O Download do Documento
1. Puxe a linha do conector direito do `Manual Trigger`.
2. Adicione e conecte um nó chamado **`HTTP Request`**.
3. **Configuração Crucial do nó HTTP Request:**
   - **Method:** `GET`
   - **URL:** Cole exatamente o caminho do GitHub ensinado no curso:
     `https://raw.githubusercontent.com/ericmonne/chocolatech-alura/main/Manual%20de%20RH%20ChocolaTech.txt`
   - ⚠️ **ATENÇÃO:** Logo abaixo, você precisa ir na seção **Options**, clicar em **Add Option**, selecionar **Response** e então **Response Format**. Mude a caixinha format de *JSON* (padrão) para **`Text`**. Sem isso, o nó falhará dizendo que não conseguiu interpretar o documento.

### Passo 3: Criando a Memória Vetorial (A Inserção)
1. Puxe a linha de saída principal do `HTTP Request`.
2. Adicione um nó chamado **`Simple Vector Store`**.
3. **Configuração do Vector Store:**
   - Note que ele é exatamente igual ao da aula anterior, mas com papel diferente.
   - **Operation:** Deixe como **`Insert Documents`** (inserir / salvar dados).
   - **Memory Key:** Escreva exatamente **`vector_store_key`**. É através dessa chave que o Agente do fluxo 2 saberá onde as políticas foram escondidas.

### Passo 4: O "Fatiador" (O Data Loader)
Aqui é onde preparamos o documento gigante em pequenos pedacinhos legíveis (Chunking).
1. Olhe para a base do seu novo `Simple Vector Store`.
2. Puxe uma linha do subconector direito, chamado **"Document"**.
3. Adicione um nó chamado **`Default Data Loader`**.
4. **Configuração do Data Loader:**
   - **Type of Data:** Marque `JSON` (sim, embora seja um arquivo texto, o campo de dados gerado pelo node anterior exige essa formatação).
   - **Text Splitting:** Selecione `Simple`. (Isso quebrará automaticamente o texto bruto em cerca de 22 blocos perfeitos para a IA processar).

### Passo 5: O Modelador Matemático (Embeddings)
Aqui vamos transformar o texto fatiado em coordenadas que os robôs entendem (*vetores*).
1. Volte à base do `Simple Vector Store`.
2. Puxe uma linha paralela a partir do subconector esquerdo, chamado **"Embedding"**.
3. Adicione o nó **`Embeddings OpenAI`**.
4. **Configuração do Embedding:**
   - **Credential:** Adicione a sua chave da OpenAI.
   - **Model:** Certifique-se de preencher com **`text-embedding-3-small`**. Isso é obrigatório.

### Como Executar
O seu canvas agora tem dois "desenhos" isolados. 
Para rodar esse Fluxo de Carga:
1. Posicione o mouse em cima do `Manual Trigger` recém-criado.
2. Vai aparecer o ícone clássico de "Play" (Test step). Aperte-o.
3. Se tudo estiver verde (Success), isso significa que o documento desceu do GitHub, foi lido, fatiado e salvo no "Cérebro" do seu n8n.
4. Agora você pode ir para a Janela do Chat e dizer um "Olá" para o Agente testar as informações!   

### Fluxo 2: Conversa Base Inicial (Retriever Flow)
Resumo: Janela de chat nativo do n8n que responde com base no que salvamos no passo de carga.

**Passo a Passo dos Nós: (resumo)**
*(Atenção: Este fluxo pressupõe que você já rodou o **Fluxo 1 (Carga)** pelo menos uma vez para que o texto das políticas já esteja salvo na memória vetorial sob a chave `vector_store_key`).*
1. **When chat message received:** Gatilho de chat nativo do n8n (usaremos isso apenas até a Aula 02).
2. **AI Agent:** Cérebro principal.
   - *Source for Prompt:* `Connected Chat Trigger Node`
   - Conecte por baixo (Model): **OpenAI Chat Model** (ex: `gpt-4o-mini`).
   - Conecte por baixo (Memory): **Simple Memory** (apenas um).
   - Conecte por baixo (Tool): **Simple Vector Store**.
     - *Operation:* `Retrieve Documents (As Tool for AI Agent)`.
     - *Dica Crítica:* Conecte a ele exatamente o **mesmo nó Embeddings OpenAI** (`text-embedding-3-small`) usado na carga. Misturar Google Gemini e OpenAI destrói a bússola vetorial do Agente!

---

**Passo a passo detalhado:**

### Passo 1: O Ponto de Entrada (Gatilho)
1. Clique no botão de **"+"** (Add node) na tela vazia do n8n.
2. Pesquise por **`When chat message received`** (também chamado de "Chat Trigger") e adicione-o.
3. Este nó não precisa de configurações adicionais. Ele cria aquela tela de bate-papo azul no rodapé do n8n para você testar.

### Passo 2: O Cérebro Principal
1. Clique no pequeno conector direito do nó "When chat message received" e arraste a linha.
2. Na lista que aparecer, pesquise e adicione o nó **`AI Agent`**.
3. **Configuração do nó AI Agent:**
   - **Source For Prompt:** Deixe como `Connected Chat Trigger Node` (isso fará com que o texto que você digitar no chat vá direto para o agente).
   - *Opcional:* Se você quiser, pode preencher o campo `System Message` dizendo como ele deve agir, por exemplo: *"Você é o HR Buddy, assistente de RH da ChocolaTech. Responda apenas sobre políticas da empresa."*

### Passo 3: O Modelo de Linguagem
1. O nó `AI Agent` tem pequenos conectores na base dele. Clique e arraste o conector marcado como **"Model"** (ou "Chat Model").
2. Pesquise e adicione o nó **`OpenAI Chat Model`**.
   - *(Nota: Se a sua OpenAI estiver sem créditos, como vimos anteriormente, você pode pesquisar por `Google Gemini Chat Model` ou `Groq Chat Model` aqui).*
3. **Configuração do Modelo:**
   - **Credential:** Selecione a sua conta de API (da OpenAI, Google, etc).
   - **Model:** Se for OpenAI, selecione o `gpt-4o-mini` (ele é o mais rápido e barato recomendado no curso).

### Passo 4: A Memória da Conversa
1. Volte ao nó `AI Agent` e arraste uma linha a partir do conector **"Memory"**.
2. Adicione **apenas um** nó chamado **`Simple Memory`** (ou `Window Buffer Memory` dependendo da versão do seu n8n).
3. **Configuração da Memória:**
   - **Session ID:** Deixe como `Connected Chat Trigger Node` (isso separa o histórico por sessão de chat).
   - Novamente, tenha certeza de que você possui **apenas um** nó de memória conectado ao seu agente!

### Passo 5: A Ponte com as Políticas da Empresa (Ferramenta Vector Store)
É aqui que a mágica do RAG acontece. Vamos dar o manual da empresa para o chatbot ler.
1. No nó `AI Agent`, arraste uma linha a partir do conector chamado **"Tool"**.
2. Adicione o nó **`Simple Vector Store`**.
3. **Configuração Crucial do Vector Store:**
   - **Operation:** É obrigatório mudar de Insert para **`Retrieve Documents (As Tool for AI Agent)`**. Se você colocar apenas Retrieve ou Insert, não vai funcionar.
   - **Memory Key:** Digite exatamente a mesma chave que você usou no fluxo de carga, que pelo roteiro é `vector_store_key`.
   - **Tool Description:** Este campo informa o modelo quando ele deve usar essa ferramenta. Digite algo como: *"Busca informações sobre políticas de RH, férias, horários e rotinas da empresa ChocolaTech"*. 

### Passo 6: O Tradutor Vetorial (Subnó de Embeddings)
1. O nó `Simple Vector Store` que você acabou de adicionar tem um subconector embaixo dele chamado **"Embedding"**.
2. Arraste uma linha desse conector e adicione o nó **`Embeddings OpenAI`**.
3. **Configuração do Embedding:**
   - **Model:** Obrigatoriamente use **`text-embedding-3-small`** (ou exatamente o mesmo que você usou no fluxo de carga anterior, se não os textos originais não serão encontrados).



   ### Erros e Inconsistências Encontrados

1. **Template RAG criou workflow duplicado**: Ao usar o "RAG starter template" do n8n, criou um workflow com ID duplicado causando erros em loop.
   - **Solução**: Arquivar o workflow problemático.

2. **Read/Write Files from Disk não funciona no n8n Cloud**: O nó lê arquivos do servidor do n8n, não da máquina local.
   - **Solução**: Usar HTTP Request para buscar o arquivo diretamente do GitHub.

3. **HTTP Request retornava JSON em vez de Text**: O campo Response Format não aparecia visualmente, mas estava disponível em Options → Response.
   - **Solução**: Mudar a opção para `Text`.

4. **Text Splitter não é um nó independente**: No n8n atual, o chunking é feito dentro do Default Data Loader, não como nó separado.

5. **Simple Vector Store "Retrieve" vs. "Retrieve for AI Agent"**: São opções diferentes dentro do mesmo nó.
   - **Solução**: Para usar como Tool do AI Agent, selecionar `Retrieve Documents for AI Agent as Tool`.

6. **Dois nós de Embeddings OpenAI são necessários**: Um para o fluxo de inserção (Load Data Flow) e outro para o fluxo de recuperação (Retriever Flow). Ambos DEVEM usar o mesmo modelo.

### Resultados dos Testes

- Pergunta: "Quantos dias de férias eu tenho direito?" → Resposta correta baseada no documento ✅
- Pergunta fora do escopo (ex.: capital da França) → Bot também respondeu ⚠️ (guardrails são introduzidos na Aula 03)

---

## AULA 02 — Memória Corporativa (Banco de Dados MySQL)
```sql
CREATE TABLE funcionarios (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    email VARCHAR(150) NOT NULL UNIQUE,
    departamento VARCHAR(100) NOT NULL,
    cargo VARCHAR(100) NOT NULL,
    data_admissao DATE NOT NULL,
    saldo_ferias INT NOT NULL DEFAULT 0,
    banco_horas DECIMAL(5,1) NOT NULL DEFAULT 0,
    regime VARCHAR(20) NOT NULL DEFAULT 'hibrido'
);
```

-- Inserção de dados básicos para testes em aula
```
INSERT INTO funcionarios (nome, email, departamento, cargo, data_admissao, saldo_ferias, banco_horas, regime) VALUES
('João Silva', 'joao.silva@empresa.com', 'Engenharia', 'Engenheiro de Software', '2022-03-10', 20, 0.0, 'hibrido'),
('Maria Souza', 'maria.souza@empresa.com', 'Recursos Humanos', 'Analista de RH', '2021-05-15', 5, 12.5, 'hibrido'),
('Carlos Oliveira', 'carlos.oliveira@empresa.com', 'Financeiro', 'Analista Financeiro', '2023-01-20', 0, 0.0, 'presencial'),
('Ana Lima', 'ana.lima@empresa.com', 'Marketing', 'Especialista em Marketing', '2020-11-05', 15, -4.0, 'remoto'),
('Pedro Santos', 'pedro.santos@empresa.com', 'Vendas', 'Executivo de Vendas', '2022-08-01', 10, 8.0, 'hibrido'),
('Fernanda Costa', 'fernanda.costa@empresa.com', 'Operações', 'Gerente de Operações', '2019-02-12', 30, 0.0, 'presencial'),
('Rafael Mendes', 'rafael.mendes@empresa.com', 'TI', 'Analista de Suporte', '2023-06-10', 0, 15.5, 'hibrido'),
('Juliana Rocha', 'juliana.rocha@empresa.com', 'Engenharia', 'Desenvolvedora Front-end', '2021-09-25', 12, 0.0, 'remoto'),
('Bruno Alves', 'bruno.alves@empresa.com', 'Design', 'Designer UX/UI', '2022-04-18', 8, 3.5, 'hibrido'),
('Camila Ferreira', 'camila.ferreira@empresa.com', 'Atendimento', 'Analista de Atendimento', '2024-01-05', 0, 0.0, 'hibrido'),
('Eric Monné', 'eric.monne@chocolatech.com', 'Produto', 'Instrutor de Cursos', '2024-01-15', 25, 8.0, 'hibrido');
```

### 📌 Objetivo
Dar identificação ao funcionário. O robô vai buscar no banco MySQL (Railway) os dados de quem está conversando.

### Conceitos Abordados

| Conceito | Explicação |
|---|---|
| RAG vs. SQL | Busca vetorial encontra regras e políticas; MySQL encontra dados específicos do funcionário |
| Por que os dois | O PDF não sabe quem é o João; o MySQL não sabe as regras de férias |
| System Prompt com dados injetados | Como passar contexto dinâmico para o AI Agent |
| `$fromAI()` | Função do n8n para que o AI Agent preencha parâmetros de ferramentas dinamicamente |

### O Que Foi Adicionado ao AI Agent
- Nova Tool: **MySQL** (Select rows from `funcionarios`)
- System Message atualizado com lógica de identificação do funcionário
  
### Passo a Passo dos Nós (resumo)

1. Mantemos o Retriver Flow da Aula 01.
### Conceito 1: Por que RAG e SQL juntos?
* **RAG (Busca Vetorial na Aula 01):** É excelente para ler textos não estruturados e regras (ex: *"Como funciona o banco de horas?"*). Porém, o PDF do manual não tem a menor ideia de quem é a "Ana Lima".
* **SQL (Busca Estruturada na Aula 02):** É excelente para dados matemáticos e exatos de pessoas específicas (ex: *"A Ana tem saldo de 15.5 horas no banco"*). Porém, o MySQL não entende as regras subjetivas da empresa.
**A mágica do nosso Agente:** Ao juntar as duas ferramentas debaixo do mesmo robô, se a Ana perguntar *"Posso folgar amanhã usando meu banco?"*, o Agente vai no MySQL ler que ela tem `15.5 horas`, e depois vai no Vector Store ler que a política exige `pelo menos 8 horas para trocar por uma folga`. Ele cruza os dois e responde a pergunta dela com perfeição!

---
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
       - *Name:* `nome`
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
**Passo a passo detalhado**

---

### Passo 1: Adicionando a Ferramenta MySQL ao Agente
Você usará o mesmo fluxo de bate-papo que construiu na Aula 01.
1. Vá até o seu nó **`AI Agent`**.
2. Na parte de baixo do nó, onde está o conector chamado **"Tool"** (ferramenta), puxe uma nova linha e solte-a em um espaço vazio. *(Sim, você terá duas ferramentas ligadas no mesmo botão Tool: o Vector Store e agora o BD).*
3. Pesquise e adicione o nó **`Select rows from a table in MySQL`**.

### Passo 2: Configurando a Conexão no Banco na Nuvem
1. Dê um duplo clique no nó MySQL recém-criado.
2. No topo, em **Credential**, adicione uma nova conexão para o MySQL.
3. **Conceito de Public Networking:** O seu banco de dados está hospedado no Railway (um provedor na nuvem). O n8n precisa acessá-lo "por fora". Portanto, você deve ir no painel do Railway, habilitar o botão de rede pública (Public Networking/TCP Proxy Domain) e pegar o endereço (`Host`) e a `Porta` gerados por lá.
4. Preencha a credencial com o Host, a Porta, o Usuário (`root`), a Senha e o Database (`railway`). Salve a credencial.

### Passo 3: Criando as "Condições" e Explicando o `$fromAI()`
Com o nó MySQL aberto, vamos ensinar como ele fará a busca.
1. **Operation:** Selecione `Select` (queremos ler dados, não apagar nem inserir).
2. **Table:** Digite `funcionarios` (a tabela que criamos com o script SQL).
3. Na seção principal, adicione uma **Condition** (Condição de busca):
   * O nome da coluna (Field) é `nome`.
   * O operador (Operator) deve ser **`Like`** *(Conceito: usar "Like" previne que o banco dê erro caso a IA passe espaços em branco sem querer).*
4. **O segredo da IA: O comando `$fromAI()`**
   * No campo de Valor (Value), você não vai digitar um nome fixo. O banco precisa buscar dinamicamente o nome da pessoa com quem está conversando!
   * Passe o mouse no campo de valor e clique no ícone de brilho (✨) que é o botão **"From AI"**.
   * Nomeie o parâmetro de: `nome`.
   * Na **Descrição do parâmetro**, digite uma regra rígida: *"Sempre use o formato %nome% (com % no início e no fim) para busca parcial. Exemplo: %João Silva%"*.
   * **Conceito:** A função `{{ $fromAI(...) }}` é uma ponte mágica do n8n. Ela cria um formulário invisível onde a IA, ao ler o chat, "adivinha" qual nome o usuário digitou e preenche essa lacuna automaticamente antes de ir no banco de dados!

### Passo 4: O "System Prompt" (Dando a Identidade ao Robô)
Agora que a ferramenta está pronta, precisamos dar ordens explícitas ao "cérebro" principal do Agente para que ele saiba como usar esses dados novos.
1. Dê um duplo clique no nó principal, o **`AI Agent`**.
2. Vá até a caixa de texto grande chamada **System Message** (ou Instruções do Sistema).
3. Digite (com suas próprias palavras, mas mantendo a lógica do roteiro) como ele deve agir. Exemplo:
"Você é o HR Buddy, o assistente de RH da ChocolaTech. Responda APENAS dúvidas sobre RH, regras, e benefícios.
IDENTIFICAÇÃO: 
No início de cada conversa, pergunte: 
'Olá! Sou o HR Buddy. Qual é o seu nome completo?' 
Com o nome informado, USE A FERRAMENTA MYSQL para buscar os dados dessa pessoa na tabela. 
Use o saldo de férias, banco de horas, regime e cargo para dar respostas personalizadas a ela. 
- Se o funcionário for encontrado: use os dados retornados (saldo_ferias, banco_horas, regime, cargo, departamento) para personalizar as respostas.
- Se o funcionário NÃO for encontrado: informe 'Não encontrei seu cadastro no sistema. Posso responder apenas dúvidas gerais sobre as políticas de RH'
 Use a base de conhecimento para responder dúvidas sobre políticas e regras de RH."

### Pronto! Teste a Aula 02
- Teste com Pedro Santos: bot perguntou o nome, buscou no MySQL e respondeu com dados personalizados ✅
- Teste com nome inexistente: bot informou que só pode responder dúvidas gerais ✅
- Combinação RAG + MySQL: "Posso tirar férias agora? Como faço para solicitar?" — resposta usou políticas do documento E dados do banco ✅


### Erros e Inconsistências Encontrados

1. **MySQL interno vs. externo no Railway**: O host `mysql.railway.internal` só funciona dentro da rede Railway. O n8n Cloud não tem acesso a essa rede.
   - **Solução**: Habilitar Public Networking no Railway e usar o host público (`caboose.proxy.rlwy.net:23446`).

2. **Warning de versão no MySQL Workbench**: O Railway usa MySQL 9.4, e o Workbench foi homologado até a versão 8.0.
   - **Solução**: Clicar em "Continue Anyway" para prosseguir normalmente.

3. **Erro "No database selected" no Workbench**: Ocorre quando o schema não está selecionado na sidebar.
   - **Solução**: Clicar duas vezes no schema `railway` na sidebar antes de executar o script.

4. **Railway não tem editor SQL nativo funcional**: O campo de query na aba Database/Data serve apenas para visualização.
   - **Solução**: Usar o MySQL Workbench para executar scripts.

5. **`$fromAI()` retorna `undefined` no preview do n8n**: Comportamento esperado em tempo de design.
   - Em execução real, o AI Agent preenche o valor corretamente.

6. **MySQL timeout ao carregar colunas no n8n**: Em alguns momentos o Railway apresenta lentidão ao carregar metadados.
   - **Solução**: Digitar o nome da coluna manualmente em vez de aguardar o carregamento automático.
   - 
### ⚠️ Erro Clássico da Aula 02
No nó MySQL, o alerta vermelho *"No parameters are set up to be filled by AI"* avisa que a ferramenta está inútil. Esse aviso MATA o Agente, o obrigando a tentar adivinhar a tabela de outras formas e entrar em loop de Query. Ele só some quando o aluno clicar no aviso de mágica **(✨)** no campo Value, ativando corretamente a função `$fromAI()`.

---

## AULA 03 — Produto Real (Telegram e Guardrails)

### 📌 Objetivo
1. Transformar o projeto em um app real no celular.
2. Criar uma "Catraca" (Guardrails) bloqueando perguntas de curiosos (como receitas de bolo) ANTES delas ativarem o Agente de IA, economizando dinheiro.
3. Resolver o problema da Memória Sem Estado (Stateless) ensinando o Session ID isolado por pessoa.

### Conceitos Abordados

| Conceito | Explicação |
|---|---|
| Webhook | Porta de entrada pública que transforma o n8n em uma API REST |
| Guardrails | Filtro que classifica e bloqueia perguntas fora do escopo do chatbot |
| Stateless | Webhook não tem memória nativa entre chamadas |
| Chat Trigger vs. Webhook | Chat Trigger é conversacional (tem sessão); Webhook é API (sem sessão) |
| Telegram como interface | Gratuito, integração via BotFather, e o `chat.id` resolve o problema de sessão |
| Embeddings | Transformar texto em vetores numéricos para busca por semelhança |
| Default Data Loader | Preparar e fragmentar dados para o Vector Store |

### Conceitos Cruciais da Aula 03

1. **Telegram vs Chat Trigger (O problema da Memória Local):**
   Nas aulas anteriores, você usou a janela azul de bate-papo do próprio n8n. Ela tem uma "sessão" automática. Quando você vai para o mundo real usando **Webhooks** ou aplicativos como o **Telegram**, cada mensagem que chega é tratada como um evento único e isolado (conceito chamado de *Stateless* / Sem Estado). 
   **A Solução do Telegram:** O Telegram fornece um número único para cada usuário que fala com o robô, chamado de `chat.id`. Nós vamos injetar esse ID na memória do n8n para que o Agente consiga saber quem é quem no histórico!

2. **Guardrails (As Grades de Proteção):**
   Modelos de linguagem são caros. Se um funcionário começar a pedir receitas de bolo ou códigos de programação para o seu bot de RH, você vai gastar dinheiro da API à toa. O Guardrail é uma barreira estrutural: colocaremos um "classificador" barato e rápido logo na porta de entrada. Se a pergunta for sobre RH, o Agente principal é ativado. Se for fora do escopo, o fluxo é cortado antes mesmo de ativar o cérebro principal.

### Por Que o Telegram Resolve o Problema de Memória
O Webhook não tem sessão nativa. O Telegram fornece um `chat.id` único e persistente para cada conversa. Esse ID é usado como **session key** no nó Simple Memory, garantindo que o bot lembre do contexto da conversa de cada usuário.
---

### Passo a Passo dos Nós (resumo)
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

---

### Passo a Passo detalhado

#### Passo 1: Configurando o Telegram (O Gatilho)
A primeira coisa é criar o bot no aplicativo do Telegram.
1. Abra o seu Telegram e pesquise pelo robô oficial chamado **`@BotFather`**.
2. Mande o comando `/newbot`, escolha o nome de exibição do seu bot (ex: "HR Buddy ChocolaTech") e em seguida um nome de usuário que termine em `bot` (ex: `chocola_hr_bot`).
3. O BotFather vai te devolver um "Token" gigante. Copie-o.
4. No n8n (no canvas vazio), adicione o nó **`Telegram Trigger`**.
5. Crie uma nova Credencial (Telegram API) colando aquele Token.
6. Na configuração do gatilho, selecione **Updates:** `message` (para ele reagir apenas quando alguém mandar uma mensagem real de texto).

#### Passo 2: O Classificador (Guardrail)
Antes da mensagem chegar no Agente caro, vamos economizar tokens e tempo fazendo uma classificação burra, porém eficiente.
1. Puxe a linha do Telegram Trigger e adicione um nó de LLM puro, chamado **`Message a Model`** (ferramenta base do OpenAI).
2. Conecte sua credencial da OpenAI e escolha o modelo `gpt-4o-mini`.
3. No prompt, digite uma instrução exata:
   > *"Classifique a mensagem abaixo em uma das três categorias: SAUDACAO / RH / OUTRO.*
   > *Mensagem: `{{ $json.message.text }}`*
   > *Responda APENAS com uma palavra única."*

#### Passo 3: O Roteador / Catraca (O Nó Switch)
Agora, vamos fazer um desvio nos trilhos dependendo da palavra que o Classificador soltou.
1. Puxe a linha do classificador e adicione o nó **`Switch`**.
2. **Adicione as regras de roteamento (Rules):**
   - **Regra 1 (RH):** Teste se o output do nó anterior (geralmente em `$json.output[0].content[0].text` ou `$json.message.content` dependendo do nó) contém a palavra `RH`.
   - **Regra 2 (Saudação):** Teste se o output contém a palavra `SAUDACAO`.
   - **Fallback (Extra):** O Switch tem uma saída padrão para "todo o resto". Isso cuidará dos tópicos da categoria "OUTRO".

#### Passo 4: Respostas Diretas (Para Saudação e Fallback)
1. Nas saídas do nó `Switch` referentes a **SAUDACAO** e ao **Fallback**, adicione um nó **`Telegram`** comum (com a ação `Send a text message`).
2. **Configuração Exigida em TODOS os nós de envio do Telegram:**
   - **Chat ID:** Como o `Switch` apagou os dados originais do caminho, você deve forçar o n8n a ir lá buscar o ID inicial: `{{ $('Telegram Trigger').item.json.message.chat.id }}`
   - **Text (Saudação):** *"Olá! Eu sou o assistente de RH da ChocolaTech..."*
   - **Text (Fallback):** *"Sinto muito, só fui treinado para responder dúvidas relacionadas ao RH desta empresa."*
   - Fallback: opção de Extra Branch (ou Extra Output / Return items that don't match any rules)

#### Passo 5: O Cérebro Real (Apenas para Perguntas de RH)
Se a palavra classificada foi "RH", aí sim ativamos o fluxo pesado que construímos na Aula 02!
1. Na saída da Regra 1 (RH) do `Switch`, adicione e construa todo o seu nó **`AI Agent`** exatamente igual ao da aula passada (conectando o *Chat Model*, a ferramenta *Vector Store* e a ferramenta *MySQL*).
2. Puxe a saída verde do `AI Agent` e envie para **um último nó `Telegram`** (`Send a text message`), usando o mesmo `Chat ID` forçado do passo acima e definindo o campo de texto (Text) como a resposta bruta gerada pelo Agente Inteligente (`{{ $json.response }}` ou equivalente).
3. No campo Chat ID, passe o mouse do lado direito dele e clique no botão de "expressão" (o símbolo de engrenagem) e depois em Add Expression (Adicionar Expressão). Nodes -> Telegram Trigger -> Output Data -> Json -> Message ->Chat ->Id

#### Passo 6: O Segredo da Memória no Telegram!
Cuidado! Como o Agente de IA não está mais conectado diretamente a um Chat Trigger nativo (ele está no meio do fluxo), precisamos configurá-lo manualmente:
1. No nó **`Simple Memory`** conectado ao Agente, mude o campo *Session ID* de *Connected Node* para **`Define below`** (Definir abaixo).
2. Na chavinha que aparecer, coloque o ID da conversa do aplicativo da pessoa, para usar como número único da memória: `{{ $('Telegram Trigger').item.json.message.chat.id }}`.
3. No próprio nó do **`AI Agent`**, mude a opção *Source for Prompt* para **`Define below`** e no campo Prompt que surgir, force o preenchimento com o texto original da pessoa usando `{{ $('Telegram Trigger').item.json.message.text }}`.

### 7. Campo "Chat ID" (Para quem o bot vai enviar?)
Lembra daquele caminho de "pescaria" que ensinei antes? Aqui é **exatamente igual**.
* Abra o editor de Expressão (no quadradinho do lado direito de Chat ID).
* No painel esquerdo, expanda: **Nodes** > **Telegram Trigger** > **Output Data** > **JSON** > **message** > **chat**.
* Arraste a variável **`id`** para a caixa central.
* *Código gerado:* `{{ $('Telegram Trigger').item.json.message.chat.id }}`
*(Sempre buscamos lá do início para garantir que a mensagem chegue no celular da pessoa certa!)*

### 8. Campo "Text" (O que o bot vai dizer?)
Aqui, você precisa passar adiante a resposta inteligentíssima que o `AI Agent` acabou de inventar no nó anterior. Como o AI Agent é o nó que está colado (conectado na entrada) desse nó do Telegram, é bem mais fácil.
O *Código gerado é algo super curto:* `{{ $json.output }}`.

### Erros e Inconsistências Encontrados

1. **Simple Memory falha sem session ID correto com Telegram**: O nó espera o session ID do Chat Trigger nativo. Com Telegram Trigger, é necessário configurar manualmente.
   - **Solução**: Usar `$('Telegram Trigger').item.json.message.chat.id` como Key no nó Simple Memory.

2. **`chat_id` is empty nos nós Send a text message**: Após o Switch, o `$json` não carrega mais a referência ao `message.chat.id`.
   - **Solução**: Sempre referenciar o Telegram Trigger diretamente: `$('Telegram Trigger').item.json.message.chat.id`.

3. **Output do classificador OpenAI com estrutura diferente do esperado**: Ao usar o nó OpenAI (não o AI Agent), o output vem em `$json.output[0].content[0].text`, não em `$json.message.content`.
   - **Solução**: Ajustar o caminho da expressão no Switch.

4. **Respond to Webhook não interpreta expressões dentro de JSON**: Ao usar JSON com `{{ $json.output }}`, o n8n retornava a expressão como texto literal.
   - **Solução**: Usar a opção `First Incoming Item` no nó de resposta.

5. **Simple Memory incompatível com n8n Cloud Queue Mode**: Os dados de sessão são perdidos se o n8n reiniciar.
   - **Para produção**: Usar Redis ou PostgreSQL como backend de memória.

6. **Simple Vector Store perde dados ao reiniciar**: Por ser in-memory, o índice vetorial é apagado quando o n8n reinicia.
   - **Solução para o curso**: Sempre rodar o Load Data Flow antes de iniciar os testes. Comunicar essa limitação aos alunos.

7. **Workflow arquivado da versão Webhook genérico**: O workflow original `Prepare HR Buddy - Class 3` (baseado em Webhook genérico) foi substituído pela versão com Telegram e arquivado.

8. **AI Agent "No prompt specified" com Telegram Trigger**: O AI Agent configurado como "Connected Chat Trigger Node" procura um campo `chatInput` que só existe no Chat Trigger nativo — não no Telegram Trigger.
   - **Solução**: Mudar "Source for Prompt" para `Define below` e definir o Prompt como `{{ $('Telegram Trigger').item.json.message.text }}`.

9. **MySQL LIKE sem wildcards faz match exato**: Ao usar o operador `Like` no nó MySQL Tool, passar apenas o nome sem `%` equivale a um match exato — se o agente passar "João Silva" sem `%`, a query falha para qualquer variação.
   - **Solução**: No campo Description do Value do nó MySQL Tool, instruir o modelo: *"Sempre use o formato %nome% (com % no início e no fim) para busca parcial. Exemplo: %João Silva%"*

10. **Funcionário não encontrado mesmo com nome correto**: Se o nome não estiver cadastrado na tabela `funcionarios`, a query retorna vazio e o bot responde apenas com políticas gerais.
    - **Solução**: Inserir o funcionário no banco com o script abaixo e usar busca parcial com LIKE.

### Adicionando um Novo Funcionário ao Banco

Para adicionar um funcionário manualmente (ex.: para demonstração em aula), execute no MySQL Workbench:

```sql
USE hr_buddy;

INSERT INTO funcionarios (nome, email, departamento, cargo, data_admissao, saldo_ferias, banco_horas, regime) VALUES
('Christian Velasco', 'christian.velasco@techsolutions.com.br', 'Produto', 'Instrutor de Cursos', '2024-01-15', 25, 8.0, 'hibrido');
```

### 💡 Dica de Ouro Didática (A Técnica da "Esteira")
Quando ensinar aos alunos como resgatar o `Chat ID` e a mensagem original usando expressões cheias de pontuação (`$().item.json...`), evite focar em código e mostre o **n8n visual**:
**Roteiro de Fala:** *"Gente, o Switch substituiu nosso texto pela palavra 'RH', então o ID do usuário original se perdeu! Como o n8n funciona como uma esteira, vamos 'puxar um fio' lá do início. Usem a Engrenagem (Add Expression), abram o painel esquerdo: `Nodes -> Telegram Trigger -> Output Data -> JSON -> message -> chat`. Cliquem em `id` e apenas arrastem pra caixa! Vejam a mágica gerando o código feio sozinha!"*


## Notas Gerais para o Instrutor

### Checklist Antes de Cada Aula

- [ ] Rodar o Load Data Flow (Manual Trigger → HTTP Request → Simple Vector Store Insert) para garantir que o documento está carregado na memória
- [ ] Verificar se o serviço MySQL no Railway está online (o plano gratuito pode suspender por inatividade)
- [ ] Ter o Telegram aberto no celular ou no desktop para demonstração ao vivo
- [ ] Confirmar que as credenciais OpenAI e MySQL estão ativas no n8n

### Limitações Importantes para Mencionar em Aula

| Limitação | Impacto | Alternativa para Produção |
|---|---|---|
| Simple Vector Store é in-memory | Dados somem se o n8n reiniciar | Pinecone, Qdrant, pgvector |
| Railway plano gratuito | Latência ou suspensão por inatividade | Railway plano pago ou outro provedor |
| n8n Cloud plano gratuito | Limite de 50 execuções por mês | Plano pago ou n8n self-hosted |
| Simple Memory no n8n Cloud | Dados perdidos no reinício | Redis ou PostgreSQL |
| OpenAI tem custo | text-embedding-3-small e gpt-4o-mini são os mais baratos, mas cobram por token | Monitorar uso no painel da OpenAI |

### Repositório do Manual de RH

- Repositório: https://github.com/ericmonne/chocolatech-alura
- URL raw do arquivo: https://raw.githubusercontent.com/ericmonne/chocolatech-alura/main/Manual%20de%20RH%20ChocolaTech.txt

---
