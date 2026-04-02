# uRBAN — Agente de Conformidade Urbanística para Belo Horizonte

## O que e o uRBAN

A ideia do uRBAN e ele poder servir como um consultor urbano digital
para projetos em Belo Horizonte. Voce tem um arquivo IFC de um projeto 
e quer saber se ele esta dentro das regras urbanisticas da cidade.
Em vez de precisar consultar manualmente a legislacao, abrir os mapas
da prefeitura e cruzar as informacoes na mao, o agente faz tudo isso
automaticamente.

Voce so digita uma pergunta em linguagem natural, o uRBAN le o IFC,
consulta os dados reais da PBH e te devolve um laudo tecnico.

Os dados urbanos escolhidos foram um pacote minimo pra testar o sistema.
O potencial aqui e muito maior, da pra expandir com muito mais camadas
de dados da prefeitura e responder perguntas complexas
sobre qualquer projeto na cidade desde que esteja georreferenciado.

---

## Perguntas que o uRBAN responde

O agente responde qualquer pergunta em linguagem natural sobre o projeto,
desde que a resposta possa ser construida a partir dos dados disponiveis.

**Sobre o projeto (extraido do IFC):**
- "Quantos pavimentos tem esse projeto?"
- "Qual e o uso declarado nessa edificacao?"

**Sobre o zoneamento:**
- "O projeto esta em uma zona que permite o uso declarado?"
- "Em qual zona urbanistica esse projeto esta localizado?"
- "O projeto condiz com o zoneamento?"
- "Quais usos sao permitidos nessa zona?"

**Sobre o gabarito:**
- "O numero de pavimentos respeita o gabarito maximo da zona?"
- "Qual e o gabarito maximo permitido para esse lote?"
- "Qual e a classificacao da via de frente ao lote?"

**Sobre a ADE:**
- "Existe alguma restricao de sobreposicao (ADE) no lote?"
- "Em qual setor da ADE o projeto esta inserido?"
- "Quais sao as restricoes da ADE para esse projeto?"

**Perguntas integradas:**
- "Esse projeto pode ser aprovado?"
- "Quais sao os conflitos urbanisticos desse projeto?"
- "O que precisa ser alterado para o projeto ficar em conformidade?"

**O que o uRBAN nao responde**

Perguntas que exigem dados alem dos carregados:
- Orcamento ou custo de construcao
- Conformidade com normas ABNT (NBR)
- Impacto ambiental ou estudos de vizinhanca

---

## Como o uRBAN funciona

O sistema usa dois tipos de dados complementares:

**1. Shapefiles (dados geograficos — consultados dinamicamente)**
Respondem onde o lote esta: em qual zona, se tem ADE, qual o tipo de via.
Isso vem direto dos dados reais da prefeitura. Esses arquivos nao treinam
o modelo — apenas servem como base de consulta geografica em tempo real.

**2. PARAMETROS_ZONA (dados normativos — extraidos da lei)**
Respondem o que e permitido naquela zona: gabarito, usos, CA.
Esses valores foram extraidos manualmente da Lei 11.181/2019 e inseridos
no codigo.
O fluxo do grafo com roteamento condicional:
```
Arquivo .ifc
     │
     ▼
No 1 — Extrator IFC (IfcOpenShell)
     │
     ▼
No 2 — Verificador de Uso ──→ [conflito] → Alerta Critico → FIM
     │
     ▼
No 3 — Verificador de Gabarito ──→ [conflito] → Alerta Gabarito → FIM
     │
     ▼
No 4 — Verificador de ADE
     │
     ▼
No 5 — Sintetizador (laudo final)
     │
     ▼
Log estruturado .txt
```

O grafo usa add_conditional_edges — se um conflito critico for detectado
o sistema para imediatamente e emite o alerta sem continuar os outros nos.

---

## Dados utilizados

| Dado | Fonte | Formato |
|---|---|---|
| Zoneamento Lei 11.181/2019 | Prefeitura de BH — BHMAP | Shapefile |
| Areas de Diretrizes Especiais (ADE) | Prefeitura de BH — BHMAP | Shapefile |
| ADE Setores | Prefeitura de BH — BHMAP | Shapefile |
| Classificacao Viaria | Prefeitura de BH — BHMAP | Shapefile |
| Modelo arquitetonico | Arquivo .ifc georreferenciado | IFC 2x3 |

---

## Dependencias
```bash
pip install langgraph langchain langchain-anthropic anthropic \
            geopandas shapely pyproj pandas requests ifcopenshell
```

---

## Como executar

### 1. Clone o repositorio
```bash
git clone https://github.com/thamiressoouza/ZIGURAT-M5-T2.git
```

### 2. Configure a API Key da Anthropic

No Google Colab va em Secrets (icone de chave) e adicione:
- Nome: ANTHROPIC_API_KEY
- Valor: sua chave em console.anthropic.com

### 3. Baixe o arquivo IFC

O arquivo IFC o notebook faz o download automaticamente via gdown, 
basta apenas executar a celula:
```python
!gdown 1UqY3FXceVHn6ndLe9M_wMcTNaiQox57p
```

### 4. Suba os Shapefiles da PBH

Os Shapefiles são baixados automaticamente pelo notebook
via gdown direto do Google Drive, sem precisar fazer
upload manual de nada. Basta apenas executar a celula.

> Fonte: Prefeitura de Belo Horizonte — BHMAP
> Legislacao: Lei 11.181/2019 — Plano Diretor Municipal

### 5. Execute o notebook

Abra Agentic_Conformidade_uRBAN.ipynb no Google Colab e execute
as celulas na ordem de cima para baixo.

### 6. Faca sua pergunta

Na celula de interface digite sua pergunta sobre o projeto
e o uRBAN retorna o laudo tecnico em linguagem natural.

### 7. Exporte o resultado

Execute a celula de exportacao para baixar o log estruturado .txt
com o laudo completo. O nome do arquivo e gerado automaticamente
a partir do nome do site no IFC.

---

## Arquivos do repositorio
```
/
├── Agentic_Conformidade_uRBAN.ipynb                            # notebook principal
├── espirito_santo_507.ifc                                      # arquivo IFC de input
├── log_rua_espirito_santo_507_belo_horizonte_mg.txt            # exemplo de output gerado
├── README.md                                                   # este arquivo
├── ADE_11181.zip                                               # shapefile ADE
├── ADE_SETORES_11181.zip                                       # shapefile ADE Setores
├── CLASSIFICACAO_VIARIA_11181.zip                              # shapefile Classificacao Viaria
└── ZONEAMENTO_11181.zip                                        # shapefile Zoneamento
```

---

## Stack tecnologica

| Tecnologia | Papel no sistema |
|---|---|
| LangGraph | Orquestracao do grafo multi-agente |
| LangChain | Integracao com o LLM |
| Claude Sonnet (Anthropic) | Modelo de linguagem para os laudos |
| IfcOpenShell | Leitura e extracao de dados do arquivo IFC |
| GeoPandas + Shapely | Consultas geoespaciais nos Shapefiles |
| Shapefiles PBH | Dados urbanisticos oficiais de BH |

---

## Exemplo de output
```
============================================================
uRBAN — CONFORMIDADE URBANISTICA BH
============================================================
Data:          01/04/2026 14:32
Arquivo IFC:   espirito_santo_507.ifc
Site:          Rua Espirito Santo, 507 — Belo Horizonte/MG
Coordenadas:   -19.9278, -43.9394
Dados urbanos: Shapefiles PBH — Lei 11.181/2019
------------------------------------------------------------

PERGUNTA DO USUARIO:
  O projeto condiz com o zoneamento?

DADOS EXTRAIDOS DO IFC:
  Uso declarado:    RESIDENTIAL
  Pavimentos:       2
  Altura total:     3.0m

VERIFICACOES:
  Zona:              OP-3
  Uso permitido:     Sim
  Via:               ARTERIAL
  Gabarito conforme: Sim (2 de 4 permitidos)
  ADE identificada:  ADE Avenida do Contorno
  Conflitos:         0

LAUDO FINAL:
  O projeto apresenta conformidade com o zoneamento OP-3...
============================================================
```

---

## Gerado por

- Sistema: uRBAN — Multi-Agente LangGraph
- Framework: LangGraph + IfcOpenShell + GeoPandas
- LLM: Claude Sonnet (Anthropic)
