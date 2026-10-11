# 📝 Resposta do Laboratório: A Wiki Perdida dos Arquivos Corporativos

> Preencha este arquivo com a sua proposta de solução.
>
> Sua resposta deve explicar como transformar os documentos brutos da pasta `raw/` em uma Wiki Corporativa Inteligente, pesquisável e segura usando apenas serviços da AWS.

---

## 👤 Identificação

**Nome:**  
Alanna Cavalcante de Albuquerque

**Data:**  
11/10/2026

**Link do repositório:**  
https://github.com/Dev-Alanna/laboratorio-wiki-aws

---

# ✅ Quest 1: O Mapa dos Arquivos Perdidos

## 1.1 Formatos encontrados na pasta `raw/`

Descreva quais tipos de arquivos existem dentro da pasta `raw/`.

```md
Exemplo de como responder, com o formato e o que ele implica:
- <extensao>: <nasce digital ou precisa de OCR?>, <o que da para extrair>
```

> Abra a pasta e liste o que voce encontrou de fato. Esta quest avalia a sua
> leitura do acervo, entao a resposta certa e a que corresponde aos arquivos.

**Sua resposta:**

- **.pdf (`ata_reuniao_vendas_sa.pdf`)**: Documento digital de 5 páginas. Já possui camada de texto nativa, permitindo extrair diretamente o conteúdo, as decisões e os tópicos das reuniões sem necessidade de OCR.
- **.png (`ata_resultados_vendas_novos_dados.png`)**: Imagem digitalizada de uma página com anotações e escrita à mão. Precisa obrigatoriamente de OCR (reconhecimento óptico de caracteres) para ler os pixels e caligrafia e transformá-los em texto pesquisável.
- **.csv (`vendas_sa_dados_ficticios_laboratorio.csv`)**: Tabela estruturada de CRM com 240 oportunidades e 19 colunas. Já nasce digital, mas exige um tratamento para converter linhas e colunas em resumos textuais ou dados relacionais que a IA consiga entender facilmente.

---

## 1.2 Principais desafios encontrados

Explique quais dificuldades esses documentos podem apresentar.

```md
Exemplo:
- Arquivos sem padrão de nomenclatura
- Documentos escaneados com baixa qualidade
- Textos manuscritos ou parcialmente ilegíveis
- Atas com estruturas diferentes
- Informações importantes espalhadas em vários formatos
```

**Sua resposta:**

- **Formatos muito diferentes:** Mistura de texto corrido (PDF), imagem escaneada com escrita manual (PNG) e dados em tabela (CSV) na mesma pasta.
- **Leitura de caligrafia:** A imagem PNG contém anotações manuais, exigindo um OCR avançado capaz de lidar com escrita humana e possíveis ruídos na imagem.
- **Interpretação de tabela (CSV):** Consultar uma tabela com 19 colunas por linguagem natural é mais desafiador do que ler um texto corrido, pois exige relacionar métricas, clientes e valores.
- **Sem organização em pastas:** Todos os arquivos estão soltos juntos na pasta `raw/`, exigindo que o sistema faça a triagem e identificação automática de cada um.

---

## 1.3 Informações importantes a serem extraídas

Liste quais informações precisam ser identificadas para transformar os documentos em conhecimento pesquisável.

**Sua resposta:**

- **Decisões tomadas:** Quais acordos e definições foram estabelecidos nas reuniões.
- **Responsáveis e prazos:** Quem ficou encarregado de cada tarefa ou ação pendente.
- **Temas e tópicos:** Assuntos discutidos (ex.: orçamento, metas de vendas, segurança e fornecedores).
- **Dados comerciais (do CSV):** Nomes de clientes, valores das oportunidades, status de fechamento e datas de negociação.
- **Identificação da reunião:** Data do evento, participantes presentes e contexto geral.

---

## 1.4 Estratégia de classificação inicial

Como você classificaria os documentos sem depender de subpastas dentro de `raw/`?

**Sua resposta:**

Como não há subpastas dentro de `raw/`, a classificação inicial será feita de forma automática na chegada do arquivo, analisando a sua extensão (MIME type):
- Se for imagem (`.png` / `.jpg`): é direcionado para a esteira de OCR.
- Se for documento de texto (`.pdf`): passa por extração de texto digital.
- Se for tabela (`.csv`): é direcionado para a esteira de processamento de dados estruturados.

Cada arquivo recebe tags/metadados com seu tipo documental (ex.: `tipo: ata_reuniao`, `formato: pdf`, `origem: crm`), permitindo filtrar e organizar tudo logicamente sem mexer na estrutura original dos arquivos.

---

# ✅ Quest 2: O Portal de Entrada na AWS

## 2.1 Armazenamento dos arquivos brutos

Explique como os arquivos da pasta `raw/` seriam enviados e armazenados na AWS.

Serviços que você pode considerar:

- Amazon S3
- AWS IAM
- AWS KMS
- Amazon S3 Versioning
- Amazon S3 Lifecycle

**Sua resposta:**

Os arquivos da pasta `raw/` seriam enviados para um bucket no **Amazon S3** (ex.: `bucket-wiki-corporativa/raw/`). 

Para garantir organização e segurança na entrada:
- **AWS IAM (Identity and Access Management):** Políticas de acesso com privilégio mínimo, garantindo que apenas usuários e serviços autorizados façam upload ou leiam os arquivos.
- **AWS KMS (Key Management Service):** Criptografia em repouso ativada (SSE-KMS), garantindo que todos os documentos fiquem criptografados no S3.
- **Amazon S3 Lifecycle:** Configuração de ciclo de vida para transição de documentos antigos para classes de armazenamento mais econômicas (como S3 Glacier) após determinado período.

---

## 2.2 Preservação dos arquivos originais

Explique como garantir que os arquivos originais sejam mantidos intactos e rastreáveis.

**Sua resposta:**

Para garantir que os arquivos originais permaneçam intactos e rastreáveis:
- **Amazon S3 Versioning:** Habilitado no bucket para manter o histórico de versões de cada arquivo, impedindo que uma sobreposição ou exclusão acidental apague o documento original.
- **S3 Object Lock (ou MFA Delete):** Impede a exclusão definitiva de documentos corporativos e atas por tempo determinado (modo WORM - Write Once, Read Many).
- **Rastreabilidade e Logs:** Ativação do **AWS CloudTrail** e dos logs de acesso do S3 para auditar quem fez upload, quem leu e quando qualquer arquivo foi acessado.

---

## 2.3 Extração de texto dos documentos

Explique como cada tipo de arquivo seria processado.

Considere:

- PDFs escaneados;
- Imagens;
- PDFs digitais;
- Arquivos `.txt`;
- Arquivos `.docx`;
- Arquivos `.md`.

Serviços que você pode considerar:

- Amazon Textract
- AWS Lambda
- AWS Step Functions
- Amazon S3
- Amazon CloudWatch

**Sua resposta:**

O processamento é orquestrado de forma serverless pelo **AWS Step Functions** acionado por eventos do S3 através de uma função **AWS Lambda**:

- **Imagens e PDFs escaneados (`.png` / `.jpg` e PDFs sem camada de texto):** Enviados para o **Amazon Textract**. O Textract executa OCR avançado capaz de ler texto impresso, tabelas e anotações manuscritas (como na ata digitalizada).
- **PDFs digitais, `.txt`, `.md` e `.docx`:** Processados diretamente via **AWS Lambda** usando bibliotecas leves para extrair a camada de texto nativa de forma rápida e com menor custo.
- **Arquivos tabulares (`.csv`):** Processados via **AWS Lambda** para ler as linhas de oportunidades do CRM e convertê-las em registros resumidos em linguagem natural (ex.: "Oportunidade da empresa X no valor de Y fechada na data Z").

Todo o texto bruto extraído é salvo em um bucket intermediário no S3 (ex.: `bucket-wiki-corporativa/processed/`).

---

## 2.4 Tratamento de falhas

Explique como sua solução identificaria e registraria erros de processamento.

**Sua resposta:**

Para identificar e registrar falhas no fluxo:
- **Amazon CloudWatch:** Coleta logs detalhados e métricas de execução de cada Lambda e chamada do Textract, gerando alarmes em caso de taxa de erro elevada.
- **Dead-Letter Queues (DLQ) com Amazon SQS:** Caso um arquivo falhe repetidamente na extração (ex.: arquivo corrompido ou formato não suportado), a mensagem vai para uma fila de DLQ para reprocessamento ou análise manual.
- **Notificações:** O **Amazon SNS** envia alertas automáticos por e-mail para a equipe técnica quando ocorrem erros críticos na ingestão.

---

# ✅ Quest 3: A Relíquia dos Metadados

## 3.1 Padronização dos textos processados

Explique como os textos extraídos seriam limpos, normalizados e preparados para consulta.

**Sua resposta:**

Os textos brutos extraídos passam por uma etapa de limpeza e normalização executada por uma função **AWS Lambda**:
- **Limpeza de ruídos:** Remoção de quebras de linha excessivas, cabeçalhos/rodapés repetitivos de páginas escaneadas e caracteres especiais indesejados gerados pelo OCR.
- **Normalização:** Conversão de codificação para UTF-8 padrão e padronização de formatos de data (ex.: DD/MM/AAAA para AAAA-MM-DD).
- **Chunking (Divisão em blocos):** O texto de documentos longos (como o PDF de 5 páginas) é quebrado em blocos menores com sobreposição (overlapping), preservando o contexto dos parágrafos para que a busca semântica funcione com precisão.

---

## 3.2 Metadados propostos

Defina quais metadados você extrairia de cada documento.

| Metadado | Por que ele é importante? |
|---|---|
| Nome do documento | Permite identificar a fonte original e citar exatamente de onde a resposta da IA veio. |
| Tipo do documento | Facilita a filtragem entre atas de reuniões, anotações manuscritas ou dados de CRM. |
| Data identificada | Permite ordenar respostas cronologicamente e responder sobre decisões de períodos específicos (ex.: "último trimestre"). |
| Tema principal | Ajuda a agrupar e buscar documentos por assunto (ex.: segurança, orçamento, vendas). |
| Participantes | Permite descobrir em quais reuniões determinadas pessoas ou equipes estiveram presentes. |
| Decisões tomadas | Crucial para responder perguntas da liderança sobre o que foi deliberado e acordado. |
| Responsáveis | Essencial para responder perguntas do tipo "quem ficou responsável pela ação X". |
| Próximos passos | Permite rastrear pendências, prazos e tarefas que ficaram em aberto. |
| Nível de confidencialidade | Garante governança e conformidade, restringindo a busca conforme as permissões do usuário. |
| Caminho do arquivo original | Garante a rastreabilidade direta do arquivo no bucket do Amazon S3. |

---

## 3.3 Uso de IA para enriquecimento dos documentos

Explique como o Amazon Bedrock poderia ajudar a identificar temas, decisões, responsáveis, pendências e resumos dos documentos.

**Sua resposta:**

O **Amazon Bedrock** seria utilizado logo após a extração e limpeza do texto para atuar como um "analista automatizado" dos documentos:

- **Resumo Executivo:** O Bedrock gera um resumo conciso de 2 a 3 parágrafos destacando os objetivos principais e os resultados de cada ata ou anotação.
- **Extração Estruturada (Decisões e Pendências):** Através de prompts específicos com saída em formato JSON, o modelo identifica trechos onde decisões foram firmadas e lista tarefas pendentes junto com seus respectivos responsáveis.
- **Mapeamento de Temas e Tags:** Analisa o contexto completo do documento e classifica os assuntos tratados (ex.: "Infraestrutura", "Segurança", "Comercial"), gerando tags automáticas que alimentam os metadados.
- **Interpretação Contextual do Manuscrito:** Ajuda a interpretar eventuais trechos fragmentados vindos do OCR da imagem PNG, reconstruindo o sentido das frases no contexto empresarial.

---

## 3.4 Armazenamento dos metadados

Explique onde os metadados seriam armazenados e como seriam conectados aos documentos originais.

Serviços que você pode considerar:

- Amazon S3
- Amazon DynamoDB
- AWS Glue Data Catalog
- Amazon Bedrock Knowledge Bases

**Sua resposta:**

Os metadados serão armazenados no **Amazon DynamoDB** e integrados diretamente ao **Amazon Bedrock Knowledge Bases**:
- **Amazon DynamoDB:** Banco NoSQL de baixa latência onde cada item armazena o `document_id`, o caminho do arquivo original no S3 (`s3_uri`), data, autores, tags e resumo. Essa base permite consultas relacionais e filtros estruturados rápidos.
- **Amazon Bedrock Knowledge Bases:** Cada trecho de texto enviado para a base vetorial carrega consigo um arquivo `.metadata.json` associado no S3. Isso permite que a IA faça buscas semânticas combinadas com filtros de metadados (ex.: buscar apenas por documentos do tipo `ata_reuniao` do ano de 2024).

---

# ✅ Quest 4: O Oráculo da Wiki Inteligente

## 4.1 Estratégia de indexação

Explique como os documentos seriam divididos em trechos menores e preparados para busca semântica.

**Sua resposta:**

Para preparar os documentos para a busca semântica, aplicamos uma estratégia de **Chunking (divisão em blocos de texto)**:
- **Tamanho dos blocos:** Os textos são divididos em blocos de aproximadamente 300 a 500 tokens (cerca de 200 a 350 palavras).
- **Sobreposição (Overlap):** Aplicamos uma sobreposição de 10% a 20% (50 tokens) entre blocos consecutivos para garantir que nenhuma frase ou contexto de decisão seja cortado na transição de um pedaço para o outro.
- **Vínculo com a fonte:** Cada bloco preserva o ID do documento, número da página de onde saiu e o link para o arquivo original no S3.

---

## 4.2 Busca semântica e base vetorial

Explique como embeddings seriam gerados e onde seriam armazenados.

Serviços que você pode considerar:

- Amazon Bedrock Knowledge Bases
- Amazon OpenSearch Serverless
- Amazon Aurora PostgreSQL com pgvector
- Amazon S3 Vectors
- Modelos de embeddings no Amazon Bedrock

**Sua resposta:**

- **Geração de Embeddings:** Usamos o modelo **Amazon Titan Text Embeddings** (disponível no Amazon Bedrock) para converter os blocos de texto em vetores numéricos de alta dimensão que representam o significado semântico do conteúdo.
- **Base Vetorial:** Utilizamos o **Amazon OpenSearch Serverless** como armazenamento vetorial gerenciado pelo **Amazon Bedrock Knowledge Bases**. Ele indexa os vetores e os metadados sem necessidade de gerenciar servidores, permitindo buscas por similaridade de cosseno em milissegundos.

---

## 4.3 Geração de respostas com IA

Explique como a Wiki responderia perguntas em linguagem natural com base nos documentos originais.

Considere explicar:

- Como a pergunta do usuário seria recebida;
- Como os trechos relevantes seriam recuperados;
- Como o Amazon Bedrock geraria a resposta;
- Como a resposta indicaria as fontes utilizadas.

**Sua resposta:**

A solução utiliza o padrão **RAG (Retrieval-Augmented Generation)**:
1. **Recepção da pergunta:** O usuário pergunta em linguagem natural (ex.: *"Quem ficou responsável pela ação de segurança?"*).
2. **Conversão e Busca (Retrieval):** A pergunta é convertida em vetor pelo Titan Embeddings e o OpenSearch Serverless busca os 3 a 5 trechos de documentos mais relevantes no acervo.
3. **Geração (Generation):** O **Amazon Bedrock** (utilizando modelos como Claude 3 Sonnet ou Amazon Titan) recebe a pergunta + os trechos recuperados em um prompt protegido, instruindo o modelo a responder **apenas** com base nas evidências fornecidas.
4. **Citação de fontes:** A resposta final inclui expressamente a referência do documento original (ex.: *"Segundo a Ata de Reunião de Vendas (pág. 3), o responsável foi João Silva"*), garantindo total rastreabilidade.

---

## 4.4 Interface de consulta

Proponha como os usuários acessariam essa Wiki Inteligente.

Serviços que você pode considerar:

- Amazon Q Business
- AWS Amplify
- Amazon API Gateway
- AWS Lambda
- Amazon Cognito

**Sua resposta:**

Propomos uma interface amigável baseada em:
- **Amazon Q Business / AWS Amplify:** Uma aplicação web moderna e responsiva (estilo portal de Wiki corporativo com chat conversacional).
- **Amazon Cognito:** Autenticação segura de usuários corporativos com Single Sign-On (SSO) e perfis de acesso.
- **Amazon API Gateway + AWS Lambda:** Camada de API que conecta o frontend aos serviços do Amazon Bedrock de forma segura, rápida e desacoplada.

---

## 4.5 Segurança, auditoria e monitoramento

Explique como controlar acesso, proteger dados, auditar consultas e monitorar custos, erros e qualidade das respostas.

Serviços que você pode considerar:

- AWS IAM
- AWS KMS
- Amazon Cognito
- AWS CloudTrail
- Amazon CloudWatch
- Amazon Macie
- AWS Cost Explorer

**Sua resposta:**

- **Controle de Acesso e Permissões:** Políticas estritas via **AWS IAM** e autenticação de usuários via **Amazon Cognito**.
- **Criptografia:** Todos os dados em trânsito (TLS 1.3) e em repouso no S3 e OpenSearch são criptografados com chaves gerenciadas pelo **AWS KMS**.
- **Auditoria de Consultas:** **AWS CloudTrail** registra todas as ações e chamadas de API; logs do **Amazon CloudWatch** monitoram perguntas feitas e tempo de resposta.
- **Proteção de Dados:** **Amazon Macie** para varredura e descoberta contínua de dados confidenciais ou não-conformidades nos buckets do S3.
- **Monitoramento de Custos:** Acompanhamento via **AWS Cost Explorer** e alertas de orçamento configurados no **AWS Budgets**.

---

# 🧩 Arquitetura Final da Solução

Agora reúna tudo em uma visão única.

## 1. Visão geral

Explique em poucas linhas a ideia central da sua arquitetura.

**Sua resposta:**

A arquitetura transforma documentos corporativos brutos e desestruturados em uma Wiki Corporativa Inteligente através de um pipeline Serverless na AWS baseado no padrão RAG (Retrieval-Augmented Generation). Ela une armazenamento seguro no S3, extração automatizada de texto e OCR com Amazon Textract, indexação vetorial no Amazon Bedrock Knowledge Bases com OpenSearch Serverless, e geração de respostas confiáveis com citação direta das fontes originais.

---

## 2. Serviços AWS utilizados

| Serviço AWS | Papel na solução |
|---|---|
| Amazon S3 | Armazenamento seguro dos arquivos brutos (raw), processados e backups. |
| Amazon Textract | OCR inteligente para extração de texto impresso, tabelas e manuscritos em imagens e PDFs. |
| Amazon Bedrock | Execução de modelos de IA generativa e geração de embeddings vetoriais. |
| Amazon Bedrock Knowledge Bases | Orquestração completa do RAG e sincronização entre documentos e base vetorial. |
| Amazon OpenSearch Serverless | Banco de dados vetorial para busca semântica de alta performance. |
| AWS Lambda | Funções serverless para triagem por extensão, limpeza de texto e chamadas de API. |
| AWS Step Functions | Orquestração do pipeline de ingestão e processamento de documentos com tratamento de erros. |
| Amazon DynamoDB | Armazenamento centralizado de metadados catalogados dos documentos. |
| Amazon CloudWatch | Coleta de métricas, logs e alarmes de falha do sistema. |
| AWS IAM | Controle de acesso granular baseado em privilégio mínimo. |
| AWS KMS | Criptografia gerenciada para proteção de dados em repouso. |

Adicione, remova ou ajuste os serviços conforme sua proposta.

---

## 3. Fluxo de dados de ponta a ponta

Descreva o caminho dos dados desde a pasta `raw/` até a Wiki Inteligente.

```md
Exemplo de estrutura:

1. Arquivos estão inicialmente na pasta raw/
2. Arquivos são enviados para o Amazon S3
3. Documentos escaneados passam pelo Amazon Textract
4. Arquivos digitais têm seus textos extraídos
5. Textos são limpos e padronizados
6. Metadados são extraídos
7. Conteúdos são indexados em uma base pesquisável
8. Usuário pesquisa na Wiki
9. IA responde com base nos documentos originais
```

**Sua resposta:**

1. Os documentos chegam à pasta `raw/` no Amazon S3.
2. O evento de upload aciona o AWS Step Functions, que faz a triagem pelo formato (MIME type).
3. Documentos em imagem ou escaneados vão para o Amazon Textract (OCR); documentos digitais e CSVs são lidos pelo AWS Lambda.
4. O texto extraído é limpo, normalizado e dividido em blocos (chunks com overlap).
5. Metadados e resumos são gerados e salvos no Amazon DynamoDB.
6. Os blocos de texto são convertidos em embeddings no Amazon Bedrock e indexados no Amazon OpenSearch Serverless.
7. O usuário acessa a Wiki corporativa e faz uma pergunta em linguagem natural.
8. O Amazon Bedrock Knowledge Bases busca os blocos mais relevantes por busca semântica e gera a resposta com IA citando os documentos de origem.

---

## 4. Diagrama textual da arquitetura

Crie um diagrama simples usando texto.

```md
Exemplo:

raw/ → Amazon S3 → Lambda/Step Functions → Textract → S3 Processado → Bedrock Knowledge Bases → Interface de Consulta → Usuário Final
```

**Sua resposta:**

[Upload raw/] → [Amazon S3 (raw)] 
       ↓ 
[AWS Step Functions + Lambda (Triagem)]
   ├── Imagem/Manuscrito ──→ [Amazon Textract (OCR)]
   └── PDF Digital / CSV ──→ [AWS Lambda (Extração)]
       ↓
[Limpeza e Chunking (Lambda)] ──→ [Amazon DynamoDB (Metadados)]
       ↓
[Amazon Bedrock (Embeddings)] ──→ [OpenSearch Serverless (Base Vetorial)]
       ↓
[Amazon Bedrock Knowledge Bases (RAG)] ←── [API Gateway + Lambda] ←── [Portal Web Wiki] ←── [Usuário]

---

## 5. Riscos e limitações

Liste possíveis desafios da sua solução.

```md
Exemplo:
- Documentos ilegíveis podem prejudicar a extração de texto.
- OCR pode gerar erros em documentos com baixa qualidade.
- Custos podem aumentar conforme o volume de documentos.
- Metadados inferidos por IA podem precisar de validação humana.
- Respostas geradas por IA devem sempre referenciar documentos de origem.
```

**Sua resposta:**

- **Qualidade de manuscrito:** Caligrafias excessivamente ilegíveis na imagem PNG podem demandar conferência humana dos textos extraídos.
- **Interpretação de planilhas complexas:** O CSV do CRM possui muitas colunas técnicas; perguntas analíticas complexas podem exigir consultas SQL específicas via Amazon Athena além do RAG textual.
- **Custo de chamadas de IA:** O uso contínuo de modelos de linguagem e OCR em larga escala deve ser monitorado com limites no AWS Budgets para evitar surpresas financeiras.
- **Alucinação de IA:** Necessidade de configurar temperatura baixa e regras estritas no prompt do Bedrock para garantir que a IA nunca invente respostas fora dos documentos oficiais.

---

## 6. Melhorias futuras

Descreva como a solução poderia evoluir.

```md
Exemplo:
- Criar uma interface web para consulta.
- Criar um chat interno para perguntas sobre atas.
- Adicionar controle de acesso por departamento.
- Criar dashboard de decisões e pendências.
- Gerar alertas automáticos sobre ações em aberto.
- Integrar com ferramentas corporativas.
```

**Sua resposta:**

- **Painel Analítico para o CSV:** Integrar o Amazon QuickSight ou Amazon Athena para gerar dashboards visuais automáticos das oportunidades de vendas do CRM.
- **Chatbot Integrado:** Disponibilizar o bot de consulta diretamente no Slack ou Microsoft Teams da empresa.
- **Human-in-the-Loop:** Usar o Amazon Augmented AI (A2I) para permitir que revisores humanos aprovem a extração de documentos com baixa pontuação de confiança no OCR.
- **Alertas Proativos:** Criar rotinas automáticas com Amazon EventBridge para avisar gestores sobre prazos e pendências encontrados nas atas que estão vencendo.

---

# 🧠 Checklist Final

Antes de entregar, confirme se sua solução responde:

- [x] Como transformar documentos escaneados em texto?
- [x] Como lidar com diferentes formatos dentro da mesma pasta `raw/`?
- [x] Como armazenar os documentos originais?
- [x] Como preservar a rastreabilidade entre resposta e documento fonte?
- [x] Como organizar metadados?
- [x] Como criar busca semântica?
- [x] Como usar Amazon Bedrock na solução?
- [x] Como proteger documentos sensíveis?
- [x] Como monitorar falhas?
- [x] Como a empresa usaria essa Wiki no dia a dia?

---

# 🏁 Conclusão

Escreva uma breve conclusão defendendo sua solução como se estivesse apresentando para uma liderança técnica ou de negócio.

**Sua resposta:**

A solução proposta entrega uma resposta moderna, escalável e segura para o desafio da gestão de conhecimento corporativo. Ao combinar serviços nativos da AWS como Amazon S3, Amazon Textract e o poder de IA Generativa do Amazon Bedrock com busca vetorial (RAG), eliminamos o retrabalho de busca manual e transformamos arquivos esquecidos em decisões pesquisáveis. A arquitetura 100% serverless garante que a empresa pague apenas pelo que utilizar, com alta disponibilidade, governança estrita e rastreabilidade total das fontes de informação.
