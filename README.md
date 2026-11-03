# Tech Challenge IADT Fase 4 - Sistema Multimodal Médico

## Visão Geral

Este projeto consiste em um **notebook Jupyter unificado** (`01_analise_video.ipynb`) que integra todas as funcionalidades do Tech Challenge da Fase 4:

- **Análise de Vídeo:** OpenPose-Light (MediaPipe) + YOLOv8 para detecção postural e de objetos críticos
- **Análise de Áudio:** Azure Cognitive Services - Speech to Text + Text Analytics
- **Detecção de Anomalias:** Séries temporais de sinais vitais, prescrições e padrões de movimentação
- **Workflow Integrado:** Decisão clínica final baseada na combinação de todas as fontes de dados

## 📋 Pré-requisitos

### 1. Contas/recursos Azure (necessário para o modo `azure`)

O notebook usa **dois serviços Azure** para a análise de áudio:
- **Azure AI Speech** → Speech to Text (`transcrever_audio`).
- **Azure AI Language** → Text Analytics, sentimento + entidades (`analisar_texto_azure`).

> Por que criar **individual** e não "multi-service"? O recurso **multi-service**
> normalmente **não oferece o tier F0 (Free)** e tem limite de 1 por região. Os
> recursos **individuais** oferecem **F0**, ideal para a demonstração.
> (F0 tem cota de minutos/chamadas — suficiente para o pitch.)

**Como criar (portal.azure.com):**
1. Faça login (conta gratuita disponível) e vá em **Create a resource**.
2. **Speech:** busque **"Azure AI Speech"** → **Create**:
   - *Name*, **Pricing tier = `F0 (Free)`**, **Region** (ex.: `westus2`), *Resource group* → **Review + create**.
   - Depois: **Keys and Endpoint** → copie **KEY 1** e a **Region**.
3. **Text Analytics:** busque **"Azure AI Language"** → **Create**:
   - *Name*, **Pricing tier = `F0 (Free)`**, **Region**, *Resource group* → **Create**.
   - Depois: **Keys and Endpoint** → copie **KEY 1**, a **Region** e o **Endpoint**
     (`https://<nome-do-recurso>.cognitiveservices.azure.com/`).
   - ⚠️ Recursos **novos** do Language **não** aceitam o endpoint regional legado
     (`https://<região>.api.cognitive.microsoft.com/`); usando o legado você recebe
     **HTTP 401**. Por isso o notebook aceita `AZURE_TEXT_ANALYTICS_ENDPOINT`.
4. Guarde **chave + região (Speech)** e **chave + região + endpoint (Language)**.

### 2. Bibliotecas Python

```bash
# Clonar o repositório (se houver)
git clone <repo-url> ./tech-challenge
cd tech-challenge

# Criar ambiente virtual .venv
python3 -m venv .venv

# Ativar o ambiente virtual
# No Linux/Mac:
source .venv/bin/activate
# No Windows:
.\.venv\Scriptsctivate

# Instalar dependências
pip install -r requirements.txt
```

### 3. Variáveis de Ambiente (recomendado: arquivo `.env`)

O notebook carrega automaticamente um `.env` via **python-dotenv** (`load_dotenv()`).
Copie o exemplo e preencha — é o método mais confiável, pois o kernel do Jupyter
nem sempre herda os `export` do terminal (comum no macOS):

```bash
cp .env.example .env
# edite o .env com suas chaves/regions/endpoint
```

Conteúdo do `.env`:
```
AZURE_SPEECH_KEY=<chave do Azure AI Speech>
AZURE_SPEECH_REGION=westus2

AZURE_TEXT_ANALYTICS_KEY=<chave do Azure AI Language>
AZURE_TEXT_ANALYTICS_REGION=westus2
AZURE_TEXT_ANALYTICS_ENDPOINT=https://<nome-do-recurso-language>.cognitiveservices.azure.com/
```
> `AZURE_TEXT_ANALYTICS_ENDPOINT` deve ser preenchido para recursos **Azure AI
> Language** (individuais). Deixe vazio **apenas** se usar multi-service legado.

Alternativa (sem `.env`), exportando antes de abrir o Jupyter:
```bash
export AZURE_SPEECH_KEY=...
export AZURE_SPEECH_REGION=westus2
export AZURE_TEXT_ANALYTICS_KEY=...
export AZURE_TEXT_ANALYTICS_REGION=westus2
export AZURE_TEXT_ANALYTICS_ENDPOINT=https://<nome>.cognitiveservices.azure.com/
jupyter notebook            # precisa ser aberto DESTE mesmo terminal
```

## 🐍 Ambiente Virtual Python

Recomenda-se criar um ambiente virtual isolado para o projeto. Este exemplo cria a pasta `env` local:

```bash
# Entrar na pasta do projeto
cd /caminho/para/o/fase4_entrega

# Criar a pasta do ambiente virtual (nome 'env' - sem o ponto inicial)
python3 -m venv env

# Ativar o ambiente virtual
# No Linux/Mac:
source env/bin/activate
# No Windows:
.\env\Scripts\activate

# Após ativar, instalar as dependências
pip install -r requirements.txt

# Para desativar o ambiente virtual mais tarde:
deactivate
```

## 📦 Estrutura do Projeto

```
fiap-tech-chalenge-fase4/
├── 01_analise_video.ipynb             (notebook único de análise multimodal)
├── dados/
│   ├── sinais.csv                     (entrada real: série fc/spo2 por paciente)
│   └── prescricoes.json               (entrada real: prescrições ao longo do tempo)
├── requirements.txt                   (bibliotecas Python)
├── .env.example                       (modelo das chaves Azure -> copie para .env)
├── .env                               (suas chaves — NÃO versionado)
├── .gitignore
├── README.md                          (este arquivo)
├── capturas/                          (saída: vídeo/áudio gravados — ignorado)
├── modelos/                           (saída: modelo Vosk baixado — ignorado)
└── logs/                              (saída: log de execução e retornos — ignorado)
```

## 🚀 Como Executar o Projeto

### 1. Iniciar Jupyter Notebook

```bash
jupyter notebook notebooks/01_analise_video.ipynb
```

### 2. Executar as Células em Ordem

O notebook foi estruturado para ser executado sequencialmente:

**Seção 1: Análise de Vídeo (≈ 5 min)**
1. Cell 1-4: Importações e configuração de modelos (YOLOv8 + MediaPipe Pose)
2. Cell 5-7: Funções de detecção de objetos e postura
3. Cell 6: `processar_video_clinico()` - processa vídeos clínicos

**Seção 2: Análise de Áudio (≈ 5 min)**
4. Cell 8: Configuração Azure Cognitive Services
5. Cell 9: `transcrever_audio()` - transcreve áudio de consultas médicas
6. Cell 10: `analisar_texto_azure()` - análise de sentimento e entidades
7. Cell 11: `detectar_alteracoes_vocais()` - detecta fadiga, dor, dificuldade respiratória

**Seção 3: Detecção de Anomalias (≈ 5 min)**
8. Cell 12: `detectar_anomalias_sinais_vitais()` - FC e SpO2
9. Cell 13: `detectar_anomalias_prescricoes()` - mudanças em medicação
10. Cell 14: `detectar_anomalias_movimentacao()` - padrões de movimento/cadela
11. Cell 15: `workflow_integrado()` - decisão clínica final

**Seção 4: Relatório Final (≈ 5 min)**
12. Cell 16: `print('=' * 70)` ... → Relatório final de 15 minutos para apresentação

### 3. Modo Offline (sem chaves Azure)

Se você não tiver chaves Azure (ou `.env` vazio), o notebook funciona em modo
limitado, escolhido no `input()` de execução (`MODO`):

- **Áudio:** se `MODO=offline`, a transcrição de um arquivo real usa o **Vosk
  (pt-BR)** local; sem arquivo real usa uma transcrição simulada. Sem STT, o
  sentimento cai em `neutro` (heurística local).
- **Vídeo:** YOLO/MediaPipe rodam localmente; sem arquivo, usa frames simulados.
- **Anomalias:** funcionam normalmente com dados de entrada.

```python
# Como o código decide ligar o Azure (célula da Seção 2):
_azure_on = bool(AZURE_ON and _HAS_AZURE_LIBS and speech_key and text_key)
# AZURE_ON vem de MODO; as demais condições evitam 401 quando falta pacote/chave.
```

## 📊 Dados de entrada (pasta `dados/`)

A **detecção de anomalias** é orientada a dados reais: ela lê arquivos em
`dados/` e, **só se não existirem**, usa uma série de **exemplo** (sempre
rotulada como `EXEMPLO` no log/relatório — o relatório imprime `Fontes ->`).
Os exemplos no repositório representam um paciente que **esteve sintomático e
está bem no fim** (últimas leituras normais → decisão “Monitoramento normal”).

- **`dados/sinais.csv`** → série temporal de sinais vitais. Colunas:
  `timestamp,fc,spo2` e, **opcionalmente**, `pa_sist` e `temp` (mais colunas
  = mais parâmetros avaliados). As últimas linhas definem o estado atual; a
  janela recente (média/mín/máx) aparece no `resumo`.
  ```csv
  timestamp,fc,spo2,pa_sist,temp
  2026-11-01T08:00,112,90,150,38.9
  ...
  2026-11-01T15:30,76,99,116,36.8
  ```
  Critérios (último valor): FC 60–100 (alerta <50/>110), SpO2 ≥95,
  PA sist 90–140, temp 35,5–38,0.
- **`dados/prescricoes.json`** → **linha do tempo** de prescrições (lista de
  `{medicamento: dosagem}`). A última é comparada com a anterior (novo/suspenso/
  dosagem/interação) e a série completa é resumida (iniciados/suspensos).
  Se as duas últimas forem iguais → `estavel` (sem alerta).
  ```json
  [ {"Paracetamol": "750mg"},
    {"Paracetamol": "750mg", "Oxigenio": "2L/min", "Soro Fisiologico": "1000ml"},
    {"Paracetamol": "750mg", "Oxigenio": "1L/min", "Soro Fisiologico": "1000ml"} ]
  ```
- **`dados/consulta.txt`** (opcional) → transcrição simulada da fala para o
  modo **offline sem gravação**. A detecção de alterações vocais é
  **consciente de negação** (“nega dor”, “sem falta de ar”) e usa **fronteira de
  palavra** (evita falso‑positivo p.ex. `dor` dentro de `dor**miu**`).

**Movimentação** (`detectar_anomalias_movimentacao`): usa os **keypoints reais
do vídeo** quando há captura (fonte `VIDEO`); sem vídeo, gera trajetória
sintética (fonte `SINTETICO`). Por padrão `estavel` (caminhada suave, varia por
semente mas sem alarme); use `MOVIMENTO_CENARIO=queda` para disparar o padrão
de queda (movimento brusco + imobilidade).

### Trocar de paciente / testar (variáveis de ambiente)
| Variável | Efeito |
|---|---|
| `CAMINHO_SINAIS` | CSV de sinais vitais (padrão `dados/sinais.csv`) |
| `CAMINHO_PRESCRICOES` | JSON de prescrições (padrão `dados/prescricoes.json`) |
| `MOVIMENTO_CENARIO` | `estavel` (padrão) ou `queda` |
| `SEMENTE` | torna o movimento sintético determinístico (ex.: `42`) |

Ex. de execução sintomática (só com env, sem editar o repo):
```bash
printf 'timestamp,fc,spo2,pa_sist,temp\n...,130,87,162,39.8\n' > /tmp/ruim.csv
CAMINHO_SINAIS=/tmp/ruim.csv MOVIMENTO_CENARIO=queda jupyter notebook
```

> Os datasets sugeridos no enunciado (**PhysioNet** e **AudioSet**) podem ser
> usados para **gerar** `dados/sinais.csv`/áudios de teste; o repositório já traz
> versões fictícias.

## 🖥️ Apresentação de 15 Minutos

O notebook foi desenhado para uma apresentação de 15 minutos:

| Tempo | Conteúdo |
|-------|----------|
| 0-5 min | Demonstração de análise de vídeo: YOLOv8 + OpenPose-Light |
| 5-10 min | Demonstração de análise de áudio: Azure Speech + Text Analytics |
| 10-15 min | Workflow integrado e decisão clínica final |

### Para gravar a apresentação:
1. Inicie o Jupyter Notebook
2. Execute todas as células na ordem
3. Use OBS Studio ou a função "Gravar tela" do seu sistema
4. Ou use o arquivo `apresentacao_15min.md` como guia

## 🔧 Solução de Problemas

### Erro: "ModuleNotFoundError: No module named 'mediapipe'"
```bash
pip install mediapipe>=0.10.0
```

### Erro: "ModuleNotFoundError: No module named 'ultralytics'"
```bash
pip install ultralytics>=8.0.0
```

### Erro no YOLOv8 (lento em CPU)
- YOLOv8 roda mais lentamente em CPU (sem GPU)
- Para demonstração, o notebook inclui simulações offline
- Considere rodar em uma máquina com GPU para melhor performance

### Erro no Azure (401 / 403 / chaves inválidas)
- **`HTTP 401 "invalid subscription key or wrong API endpoint"`**: na maioria das
  vezes é **endpoint** — recursos novos de **Azure AI Language** não aceitam o
  endpoint regional legado. Preencha `AZURE_TEXT_ANALYTICS_ENDPOINT` no `.env` com
  o **endpoint próprio** do recurso (`https://<nome>.cognitiveservices.azure.com/`).
- **`HTTP 403 Forbidden`**: quase sempre é **endpoint de outro produto**. Ex. real:
  copiar o endpoint de um **Azure AI SEARCH** (`https://<nome>.search.windows.net`)
  no lugar do Text Analytics → a chave não autoriza ali e retorna 403. O **endpoint
  correto** é do recurso **Azure AI Language**: `https://<nome>.cognitiveservices.azure.com/`
  (ou `...services.ai.azure.com`). O notebook já **avisa** na célula da Seção 2 quando
  o endpoint não parece de Text Analytics. Se o endpoint estiver certo e ainda der 403,
  cheque o **Networking** do recurso (firewall "All networks"/IP liberado).
- Confirme que **não trocou** a chave do Speech com a do Language (são recursos distintos).
- Veja o **diagnóstico impresso** na célula da Seção 2: ele mostra libs, chaves,
  regiões e o **endpoint em uso** (`.env` ou legado) + avisos de endpoint errado.
- Garanta **região = local do recurso** e que o `.env` foi carregado (`load_dotenv()`).
- Use o tier **F0 (Free)** nos recursos individuais para testes.

### Notebook não executa todas as células
- Execute as células na ordem numerada (01 a 21)
- Algumas células dependem de variáveis definidas em células anteriores

## 📝 Licença

Este projeto foi desenvolvido como parte do Tech Challenge IADT Fase 4 do curso Pós-Tech em IA para Devs.

## 📞 Suporte

- Revisar as referências das Aulas 01-04 e 10-14 do PDF do curso
- Consultar o documento `IADT - Fase 4 - Tech challenge.pdf` (5 páginas)
- Dúvidas: consultar o plano `PLAN_Tech_Challenge_IADT_Fase4.md`