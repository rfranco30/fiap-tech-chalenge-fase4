# Tech Challenge IADT Fase 4 - Sistema Multimodal Médico

## Visão Geral

Este projeto consiste em um **notebook Jupyter unificado** (`01_analise_video.ipynb`) que integra todas as funcionalidades do Tech Challenge da Fase 4:

- **Análise de Vídeo:** OpenPose-Light (MediaPipe) + YOLOv8 para detecção postural e de objetos críticos
- **Análise de Áudio:** Azure Cognitive Services - Speech to Text + Text Analytics
- **Detecção de Anomalias:** Séries temporais de sinais vitais, prescrições e padrões de movimentação
- **Workflow Integrado:** Decisão clínica final baseada na combinação de todas as fontes de dados

## 📋 Pré-requisitos

### 1. Conta Azure (necessário para análise de áudio)
Para utilizar as funcionalidades de transcrição e análise de sentimento:

1. Acesse [Azure Portal](https://portal.azure.com/)
2. Crie uma conta gratuita ou faça login
3. Navegue até "Cognitive Services"
4. Crie dois recursos:
   - **Speech** - Para Speech to Text
   - **Text Analytics** - Para análise de sentimento e entidades
5. Anote as chaves e regiões (ex: `westus2`, `eastus`)

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

### 3. Variáveis de Ambiente

Configure as chaves Azure como variáveis de ambiente:

```bash
# No Linux/Mac
export AZURE_SPEECH_KEY=sua_chave_de_speech
export AZURE_SPEECH_REGION=sua_regiao_speech (ex: westus2)
export AZURE_TEXT_ANALYTICS_KEY=sua_chave_de_text_analytics
export AZURE_TEXT_ANALYTICS_REGION=sua_regiao_text_analytics (ex: westus2)

# No Windows
set AZURE_SPEECH_KEY=sua_chave_de_speech
set AZURE_SPEECH_REGION=westus2
set AZURE_TEXT_ANALYTICS_KEY=sua_chave_de_text_analytics
set AZURE_TEXT_ANALYTICS_REGION=westus2
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
/fase4_entrega/
├── PLAN_Tech_Challenge_IADT_Fase4.md  (plano detalhado)
├── notebooks/
│   └── 01_analise_video.ipynb         (notebook unificado - 21 células)
├── dados/
│   ├── physionet/                     (dataset opcional - sinais vitais)
│   └── audioset/                      (dataset opcional - eventos sonoros)
├── requirements.txt                   (bibliotecas Python)
├── README.md                          (este arquivo)
└── apresentacao_15min.md              (guia de apresentação)
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

Se você não tiver chaves Azure, o notebook funcionará em modo limitado:

- As células de áudio retornarão valores padrão/padrão neutro
- Objetos de vídeo usarão simulações em vez de detecção real
- As funções de anomalia funcionarão normalmente com dados de entrada

```python
# Exemplo de como o código lida com chaves ausentes:
speech_key = os.getenv('AZURE_SPEECH_KEY', 'SUA_CHAVE_AQUI')
# Se AZURE_SPEECH_KEY for 'SUA_CHAVE_AQUI', o modo offline é ativado automaticamente
```

## 📊 Datasets Opcionais

### PhysioNet (Sinais Vitais)
- Acesse: https://physionet.org/
- Dataset MIMIC-III ou similar para validar `detectar_anomalias_sinais_vitais`
- Coloque os dados na pasta `dados/physionet/`

### AudioSet (Eventos Sonoros)
- Acesse: https://research.google.com/audioset/
- Para fine-tuning de detecção de alterações vocais
- Coloque os dados na pasta `dados/audioset/`

Caso os datasets não estejam disponíveis, o código utiliza dados sintéticos para demonstração.

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

### Erro no Azure (chaves inválidas)
- Verifique se as variáveis de ambiente estão corretas
- Certifique-se de que os recursos Azure foram criados
- Use o tier gratuito (free tier) para testes iniciais

### Notebook não executa todas as células
- Execute as células na ordem numerada (01 a 21)
- Algumas células dependem de variáveis definidas em células anteriores

## 📝 Licença

Este projeto foi desenvolvido como parte do Tech Challenge IADT Fase 4 do curso Pós-Tech em IA para Devs.

## 📞 Suporte

- Revisar as referências das Aulas 01-04 e 10-14 do PDF do curso
- Consultar o documento `IADT - Fase 4 - Tech challenge.pdf` (5 páginas)
- Dúvidas: consultar o plano `PLAN_Tech_Challenge_IADT_Fase4.md`