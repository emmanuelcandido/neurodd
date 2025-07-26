# Briefing Técnico: CURSO PROCESSOR - Sistema CLI de Processamento Educacional

## Análise da Ideia
### Funcionalidades Aprovadas
✅ **Sistema de Memória e Persistência** - Histórico completo com opções granulares de "esquecer"
✅ **Interface CLI Interativa** - Seção "Operações" estilo qBittorrent com progresso em tempo real
✅ **Sistema de Comandos Background** - `neuro start/stop/status/cli` via entry_point global
✅ **Menu de Status Centralizado** - Todas as configurações e variáveis acessíveis
✅ **Sistema de Prompts Personalizáveis** - Estrutura `prompts/cursos/` totalmente editável
✅ **Integração Multi-IA** - ChatGPT, Claude, Gemini, Ollama com verificação de status
✅ **Core Function: Processamento de Cursos** - Pipeline completo vídeo→áudio→transcrição→processamento
✅ **Sistema de Timestamps Hierárquicos** - Cálculo baseado em duração com formato específico
✅ **Geração de Feeds XML** - Podcasts com metadados completos e timestamps integrados
✅ **Edge-TTS e Embedding** - Áudio natural com organização no cofre Obsidian
✅ **Integração GitHub/Google Drive** - Upload automático com estrutura específica
✅ **Versionamento e Reprocessamento** - Sistema v1, v2 sem duplicação
✅ **Estrutura de Diretórios Organizada** - `assets/textos/`, `assets/audios/` configuráveis
✅ **Execução Individual de Operações** - Cada etapa do pipeline independente

### Inconsistências Detectadas e Resolvidas
🔧 **Conflito de Dependências**: Whisper API vs Docker - **Solução**: Suporte a ambos com fallback automático
🔧 **Gerenciamento de Estado**: CLI interativo + background process - **Solução**: Comunicação via socket IPC
🔧 **Cross-platform Paths**: Windows/Mac/Linux - **Solução**: `pathlib.Path` para compatibilidade universal
🔧 **Encoding Issues**: Caracteres especiais - **Solução**: UTF-8 forçado em todas as operações I/O

### Integração com Projeto Existente
**Projeto Novo** - Não há projeto existente para integração

## Arquitetura Técnica

### Stack Tecnológica Escolhida
- **Linguagem**: Python 3.9+ (compatibilidade universal Windows/Mac/Linux)
- **Framework CLI**: Click 8.0+ (interface interativa robusta, entry_points nativos)
- **Interface de Progresso**: Rich (barras de progresso estilo qBittorrent, formatação avançada)
- **Persistência**: SQLite + JSON (memória local, sem dependências externas)
- **IPC**: Socket Unix/Named Pipes (comunicação CLI↔background process)
- **APIs**: httpx (async HTTP client), openai, anthropic, google-generativeai
- **Áudio/Vídeo**: moviepy, edge-tts, whisper-api
- **Git Integration**: GitPython
- **Google Drive**: google-api-python-client

**Justificativas**:
- **Cross-platform nativo**: Python roda identicamente em todos os SOs
- **Baixa manutenção**: Dependências estáveis, SQLite sem configuração
- **ROI máximo**: Rich interface + Click = desenvolvimento rápido com UX profissional
- **Sem Docker**: Dependências Python puras, instalação via pip

### Estrutura Modular


curso_processor/
├── core/
│   ├── __init__.py
│   ├── memory_manager.py      # Persistência e histórico
│   ├── config_manager.py      # Configurações centralizadas
│   ├── progress_tracker.py    # Interface de progresso Rich
│   └── ipc_handler.py         # Comunicação CLI↔background
├── processors/
│   ├── __init__.py
│   ├── video_converter.py     # Conversão vídeo→áudio
│   ├── transcription.py       # Whisper API/local
│   ├── ai_processor.py        # Multi-IA com prompts
│   ├── timestamp_calculator.py # Timestamps hierárquicos
│   ├── xml_generator.py       # Feeds de podcast
│   └── tts_handler.py         # Edge-TTS + embedding
├── integrations/
│   ├── __init__.py
│   ├── github_client.py       # Git operations
│   ├── gdrive_client.py       # Google Drive upload
│   ├── ai_clients/
│   │   ├── __init__.py
│   │   ├── chatgpt.py
│   │   ├── claude.py
│   │   ├── gemini.py
│   │   └── ollama.py
├── cli/
│   ├── __init__.py
│   ├── main_interface.py      # Menu principal interativo
│   ├── operations_view.py     # Seção "Operações" qBittorrent-style
│   ├── status_menu.py         # Menu de configurações
│   └── background_daemon.py   # Processo headless
├── data/
│   ├── database.py           # SQLite models
│   ├── schemas.py            # Pydantic models
│   └── migrations/
├── prompts/
│   ├── cursos/
│   │   ├── processamento.md
│   │   ├── resumo.md
│   │   └── criterios.md
│   └── livros/              # Preparado para futuro
├── assets/                  # Diretório de trabalho
│   ├── textos/
│   └── audios/
├── tests/
│   ├── unit/
│   ├── integration/
│   └── fixtures/
├── config/
│   ├── settings.yaml
│   └── apis.json
└── setup.py                # Entry points: neuro command


### Módulos e Responsabilidades

- **core/memory_manager.py**: 
  - Função específica: Persistência SQLite, histórico de operações, cache inteligente
  - Reutilização: Base para todos os módulos que precisam de estado
  - Dependências: SQLite3, JSON
  - Testabilidade: Mocks de database, fixtures de estado

- **core/progress_tracker.py**: 
  - Função específica: Interface Rich estilo qBittorrent, barras de progresso
  - Reutilização: Todos os processors usam para mostrar progresso
  - Dependências: Rich, threading para updates assíncronos
  - Testabilidade: Capture de output Rich, mocks de progresso

- **processors/video_converter.py**: 
  - Função específica: Conversão vídeo→áudio via moviepy, preserva hierarquia
  - Reutilização: Pode ser usado para outros tipos de conteúdo audiovisual
  - Dependências: moviepy, pathlib
  - Testabilidade: Arquivos de vídeo pequenos como fixtures

- **processors/ai_processor.py**: 
  - Função específica: Orquestra múltiplas IAs, aplica prompts personalizáveis
  - Reutilização: Base para qualquer processamento com IA (cursos, livros, etc.)
  - Dependências: ai_clients/, prompts/ structure
  - Testabilidade: Mocks de API responses, prompts de teste

- **cli/operations_view.py**: 
  - Função específica: Interface "Operações" com controles pausar/cancelar
  - Reutilização: Interface padrão para qualquer operação longa
  - Dependências: Rich tables/progress, keyboard input handling
  - Testabilidade: Simulação de input do usuário, capture de interface

## Instruções de Desenvolvimento para Gemini CLI

### Ordem de Implementação
1. **Base Infrastructure** (core/, data/): Sistema de memória, configurações, IPC
   - **Justificativa**: Fundação para todas as outras funcionalidades
2. **CLI Interface** (cli/): Menu principal, interface de operações, entry points
   - **Justificativa**: Interface permite testar funcionalidades incrementalmente
3. **Video Processing** (processors/video_converter.py): Conversão vídeo→áudio
   - **Justificativa**: Primeira etapa do pipeline, sem dependências externas complexas
4. **AI Integration** (integrations/ai_clients/, processors/ai_processor.py): Multi-IA
   - **Justificativa**: Core da funcionalidade, permite testar processamento
5. **Advanced Features** (timestamps, XML, TTS, GitHub/Drive): Funcionalidades complementares
   - **Justificativa**: Dependem da base sólida já implementada

### Configuração de Testes
- **Framework de testes**: pytest + pytest-asyncio (padrão Python para async operations)
- **Testes unitários**: Cada módulo com mocks de dependências externas (APIs, filesystem)
- **Testes de integração**: Pipeline completo com arquivos pequenos de teste
- **Testes alternativos**: CLI interface testing com pexpect, API integration tests
- **Comando de execução**: `pytest tests/ -v --cov=curso_processor`

### Deploy e Ambiente
- **Desenvolvimento local**: `pip install -e .` (editable install)
- **Ambiente virtual**: `python -m venv venv && source venv/bin/activate` (ou `venv\Scripts\activate` no Windows)
- **Instalação global**: `pip install .` (instala comando `neuro` globalmente)
- **Configuração**: Primeira execução cria estrutura `~/.curso_processor/` com configs
- **Dependências**: `requirements.txt` com versões fixas, `setup.py` com entry_points

## Dados e Dependências

### Dados Disponíveis para Implementação Imediata
- **API ChatGPT**: [REMOVIDO]
- **API Claude**: [REMOVIDO]
- **GitHub Token**: [REMOVIDO]
- **Cofre Obsidian**: C:\Users\emman\Temporário\Second Brain - Vault de Testes
- **Repositório Feeds**: https://github.com/emmanuelcandido/feeds.git
- **Docker Whisper**: onerahmet/openai-whisper-asr-webservice (endpoint http://localhost:9000)
- **Prompt de processamento**: 8.000+ caracteres com regras específicas TDAH/INTP

### Dependências Externas
- **APIs necessárias**: OpenAI Whisper API (primary), Docker Whisper (fallback), ChatGPT API, Claude API
- **Serviços externos**: GitHub API, Google Drive API (configuração postergada)
- **Bibliotecas críticas**: 
  - click>=8.0.0 (CLI framework)
  - rich>=12.0.0 (interface de progresso)
  - moviepy>=1.0.3 (conversão vídeo)
  - edge-tts>=6.0.0 (síntese de voz)
  - httpx>=0.24.0 (HTTP client async)
  - GitPython>=3.1.0 (Git operations)
- **Credenciais necessárias**: API keys armazenadas em ~/.curso_processor/apis.json criptografado

## Especificações Cross-Platform

### Compatibilidade Universal
- **Windows**: Suporte a paths com `\`, Named Pipes para IPC, encoding CP1252 fallback
- **macOS**: Unix sockets, paths case-sensitive, Homebrew Python compatibility
- **Linux**: Unix sockets, systemd integration opcional, distribuições diversas
- **Paths**: `pathlib.Path` exclusivamente, normalização automática
- **Encoding**: UTF-8 forçado em todas as operações I/O

### Instalação e Configuração
- **Instalação**: `pip install curso-processor` (ou local `pip install -e .`)
- **Primeira execução**: Wizard de configuração cria `~/.curso_processor/`
- **Dependências sistema**: Python 3.9+, ffmpeg (para moviepy)
- **Configuração automática**: Detecta SO e ajusta configurações IPC

## Critérios de Sucesso Técnico
- **Funcionalidade**: Todas as 12 funcionalidades da ideia implementadas e testadas
- **Testes**: 90%+ cobertura, testes unitários + integração + CLI passando
- **Performance**: Conversão vídeo <2x tempo real, transcrição <1.5x tempo real
- **Cross-platform**: Funcionamento idêntico Windows/Mac/Linux
- **Interface**: Seção "Operações" responsiva com controles funcionais
- **Persistência**: Zero perda de dados em interrupções, recuperação completa
- **APIs**: Verificação de status e fallbacks funcionando
- **Memória**: Sistema "esquecer" curso/tudo funcionando sem corrupção

## Instruções Específicas para Gemini CLI

### Prioridades de Desenvolvimento
1. **CRÍTICO**: 
   - Sistema de memória persistente (SQLite + JSON)
   - Interface CLI com Rich (menu principal + seção operações)
   - Conversão vídeo→áudio com preservação de hierarquia
   - Integração ChatGPT + Claude com prompts personalizáveis
   - Entry points `neuro` command funcionais

2. **IMPORTANTE**: 
   - Background process com IPC
   - Sistema de timestamps hierárquicos
   - Geração XML de feeds válidos
   - GitHub integration para upload feeds
   - Edge-TTS com embedding Obsidian

3. **DESEJÁVEL**: 
   - Google Drive integration
   - Ollama + Gemini API integration
   - Sistema de versionamento avançado
   - Configurações de idioma/voz customizáveis

### Comandos e Configurações

# Instalação e setup
pip install -e .
neuro --setup  # Primeira configuração

# Comandos principais
neuro cli      # Interface interativa principal
neuro start    # Background daemon
neuro stop     # Para daemon
neuro status   # Status do sistema

# Desenvolvimento
pytest tests/ -v --cov=curso_processor
python -m curso_processor.cli.main_interface  # Debug mode


### Validação de Funcionalidade
- **Antes de considerar funcional**: Executar TODOS os testes (unitários, integração, CLI)
- **Critério de aprovação**: 100% dos testes críticos + 90% cobertura geral
- **Teste de integração**: Processar curso pequeno (3 vídeos) do início ao fim
- **Teste cross-platform**: Executar em pelo menos 2 SOs diferentes

### Nomenclaturas Obrigatórias (Manter Exatas)
- `cursos.xml` - arquivo XML feed (configurável)
- `.edge-tts` - subdiretório no cofre Obsidian
- `Audios`, `Resumos` - pastas criadas no processamento
- `Resumo.md` - arquivo unificado final
- `timestamps.md` - arquivo separado com timestamps
- `neuro` - comando CLI principal
- `Processed: true` - campo YAML obrigatório
- `assets/textos/`, `assets/audios/` - estrutura de trabalho
- `prompts/cursos/`, `prompts/livros/` - estrutura de prompts
- `YALM header` - manter grafia exata do usuário (não YAML)
- `nome_do_curso.mp3` - formato arquivo áudio unificado

### Sistema de Continuação IA Obrigatório

# Implementar em ai_processor.py
def process_with_continuation(prompt, content):
    response = send_to_ai(prompt + content)
    
    while response.endswith('[CONTINUA]'):
        response = response.replace('[CONTINUA]', '').strip()
        continuation = send_to_ai('[CONTINUAR]')
        response += '\n' + continuation
        
    # Remove marcadores finais
    response = response.replace('[FIM]', '').strip()
    return response


---

# CURSO PROCESSOR - Documentação Técnica

## Arquitetura Geral
### Propósito do Sistema
Sistema CLI interativo em Python para automação completa do pipeline educacional: vídeo→áudio→transcrição→processamento IA→podcast XML. Mantém memória persistente, oferece interface de monitoramento em tempo real e integração com múltiplas IAs e serviços de armazenamento.

### Estrutura Modular
Arquitetura baseada em módulos independentes comunicando via interfaces bem definidas:
- **Core**: Memória, configurações, progresso, IPC
- **Processors**: Pipeline de transformação de conteúdo
- **Integrations**: APIs externas (IA, GitHub, Drive)
- **CLI**: Interface do usuário e daemon background

### Princípios de Design
- Responsabilidade única por módulo
- Baixo acoplamento, alta coesão
- Testabilidade individual com mocks
- Reutilização maximizada entre funcionalidades
- Compatibilidade cross-platform nativa
- Interface Rich para UX profissional
- Persistência robusta com recuperação de falhas

## Módulos do Sistema

### core/memory_manager.py
- **Função**: Gerencia estado persistente, histórico de operações, cache inteligente
- **Localização**: `curso_processor/core/memory_manager.py`
- **Dependências**: SQLite3, JSON, pathlib
- **Interface pública**: 
  
  class MemoryManager:
      def save_operation(self, operation_id, data)
      def load_operation(self, operation_id)
      def forget_course(self, course_name)
      def forget_all(self)
      def get_history(self, limit=50)
  
- **Entrada**: Operation objects, course names, configuration data
- **Saída**: Stored data, operation history, boolean success flags
- **Testes**: Mock SQLite database, fixtures com estados diversos
- **Reutilização**: Base para qualquer funcionalidade que precisa de persistência

### core/progress_tracker.py
- **Função**: Interface Rich estilo qBittorrent, barras de progresso, controles
- **Localização**: `curso_processor/core/progress_tracker.py`
- **Dependências**: Rich, threading, keyboard input
- **Interface pública**:
  
  class ProgressTracker:
      def start_operation(self, name, total_steps)
      def update_progress(self, operation_id, current, status)
      def pause_operation(self, operation_id)
      def cancel_operation(self, operation_id)
      def show_operations_view(self)
  
- **Entrada**: Operation metadata, progress updates, user commands
- **Saída**: Rich UI components, operation status
- **Testes**: Capture Rich output, simulate user input, mock threading
- **Reutilização**: Todos os processors usam para mostrar progresso

### processors/video_converter.py
- **Função**: Conversão vídeo→áudio via moviepy, preserva hierarquia de pastas
- **Localização**: `curso_processor/processors/video_converter.py`
- **Dependências**: moviepy, pathlib, progress_tracker
- **Interface pública**:
  
  class VideoConverter:
      def convert_course_videos(self, course_dir, output_dir)
      def convert_single_video(self, video_path, audio_path)
      def get_video_duration(self, video_path)
  
- **Entrada**: Video file paths, output directory paths
- **Saída**: Audio files, duration metadata, conversion success status
- **Testes**: Small video fixtures, mock moviepy for speed
- **Reutilização**: Qualquer processamento de conteúdo audiovisual

### processors/ai_processor.py
- **Função**: Orquestra múltiplas IAs, aplica prompts personalizáveis, sistema de continuação
- **Localização**: `curso_processor/processors/ai_processor.py`
- **Dependências**: ai_clients/, prompts/ structure, config_manager
- **Interface pública**:
  
  class AIProcessor:
      def process_transcription(self, text, prompt_template, criteria)
      def process_with_continuation(self, prompt, content)
      def select_ai_for_task(self, task_type)
      def verify_ai_status(self, ai_name)
  
- **Entrada**: Text content, prompt templates, AI selection criteria
- **Saída**: Processed text with markdown, continuation handling
- **Testes**: Mock AI responses, test prompts, continuation scenarios
- **Reutilização**: Base para processamento de cursos, livros, qualquer conteúdo

### processors/timestamp_calculator.py
- **Função**: Calcula timestamps hierárquicos baseado em duração, formato "1:00" antes da numeração
- **Localização**: `curso_processor/processors/timestamp_calculator.py`
- **Dependências**: video_converter (para durações), pathlib
- **Interface pública**:
  
  class TimestampCalculator:
      def calculate_hierarchical_timestamps(self, course_structure)
      def format_timestamp(self, seconds)
      def generate_timestamps_file(self, course_dir, timestamps)
  
- **Entrada**: Course directory structure, video durations
- **Saída**: Hierarchical timestamps, formatted strings, timestamps.md file
- **Testes**: Mock video durations, test course structures
- **Reutilização**: Qualquer conteúdo que precisa de navegação temporal

### integrations/ai_clients/chatgpt.py
- **Função**: Cliente específico ChatGPT com handling de rate limits e erros
- **Localização**: `curso_processor/integrations/ai_clients/chatgpt.py`
- **Dependências**: openai, httpx, config_manager
- **Interface pública**:
  
  class ChatGPTClient:
      def send_message(self, prompt, model="gpt-4")
      def verify_connection(self)
      def get_available_models(self)
  
- **Entrada**: Text prompts, model selection, API configuration
- **Saída**: AI responses, connection status, model list
- **Testes**: Mock OpenAI API, test rate limiting, error scenarios
- **Reutilização**: Template para outros clientes IA (Claude, Gemini, Ollama)

### cli/main_interface.py
- **Função**: Menu principal interativo, navegação entre funcionalidades
- **Localização**: `curso_processor/cli/main_interface.py`
- **Dependências**: Rich, Click, all processors and integrations
- **Interface pública**:
  
  @click.command()
  def main():
      # Entry point principal
  
  class MainInterface:
      def show_main_menu(self)
      def handle_user_input(self, choice)
      def process_course_workflow(self)
  
- **Entrada**: User keyboard input, menu selections
- **Saída**: Rich interface, workflow orchestration
- **Testes**: Simulate user input sequences, test menu navigation
- **Reutilização**: Template para outras interfaces CLI complexas

### cli/operations_view.py
- **Função**: Seção "Operações" estilo qBittorrent com controles pause/cancel
- **Localização**: `curso_processor/cli/operations_view.py`
- **Dependências**: Rich tables, keyboard handling, progress_tracker
- **Interface pública**:
  
  class OperationsView:
      def display_operations_table(self)
      def handle_operation_controls(self)
      def update_real_time_progress(self)
  
- **Entrada**: Operation data, user control commands
- **Saída**: Real-time table updates, operation control responses
- **Testes**: Mock operations data, simulate control inputs
- **Reutilização**: Interface padrão para qualquer operação longa do sistema

## Exemplos de Uso

### Caso de Uso 1: Processamento Completo de Curso


# Via CLI interativa
neuro cli
# Selecionar "1 - Processar Curso Completo"
# Informar diretório: /path/to/course
# Sistema processa automaticamente: vídeo→áudio→transcrição→IA→XML

# Via código (para testes)
from curso_processor.processors.video_converter import VideoConverter
from curso_processor.processors.ai_processor import AIProcessor
from curso_processor.processors.xml_generator import XMLGenerator

# Pipeline completo
converter = VideoConverter()
ai_processor = AIProcessor()
xml_generator = XMLGenerator()

# Converter vídeos
audio_files = converter.convert_course_videos("/path/to/course", "/path/to/audios")

# Processar com IA
processed_content = ai_processor.process_transcription(
    transcription_text, 
    "prompts/cursos/processamento.md",
    "prompts/cursos/criterios.md"
)

# Gerar XML feed
xml_generator.create_podcast_feed(processed_content, "Meu Curso")


### Caso de Uso 2: Execução Individual de Operações


# Apenas converter vídeos
neuro cli
# Selecionar "2 - Operações Individuais" → "1 - Converter Vídeos"

# Apenas processar transcrições existentes
neuro cli
# Selecionar "2 - Operações Individuais" → "3 - Processar com IA"

# Via background daemon
neuro start  # Inicia processo headless
neuro status # Verifica operações ativas
neuro stop   # Para daemon


### Caso de Uso 3: Personalização de Prompts


# Editar prompt de processamento
# Arquivo: prompts/cursos/processamento.md
# Sistema automaticamente usa novo prompt na próxima execução

# Adicionar critérios personalizados
# Arquivo: prompts/cursos/criterios.md
# Enviado junto com prompt principal para contexto


## Instruções para Testes

### Executar Todos os Testes

# Testes completos com cobertura
pytest tests/ -v --cov=curso_processor --cov-report=html

# Testes rápidos (apenas unitários)
pytest tests/unit/ -v

# Testes de integração (mais lentos)
pytest tests/integration/ -v


### Testes por Categoria
- **Unitários**: `pytest tests/unit/ -v`
- **Integração**: `pytest tests/integration/ -v`
- **CLI Interface**: `pytest tests/cli/ -v --capture=no`

### Interpretar Resultados

# Sucesso: todos os testes passaram
========== 45 passed in 12.34s ==========

# Falha: identificar módulo problemático
========== 2 failed, 43 passed in 12.34s ==========
FAILED tests/unit/test_video_converter.py::test_convert_video
FAILED tests/integration/test_full_pipeline.py::test_course_processing

# Cobertura: verificar módulos não testados
Coverage HTML report: htmlcov/index.html


## Instruções para Modificações

### Adicionar Nova Funcionalidade
1. **Criar módulo**: Seguir estrutura `processors/nova_funcionalidade.py`
2. **Implementar interface**: Métodos públicos bem definidos
3. **Adicionar testes**: `tests/unit/test_nova_funcionalidade.py`
4. **Integrar ao CLI**: Adicionar opção em `cli/main_interface.py`
5. **Atualizar documentação**: Adicionar seção neste arquivo

### Modificar Funcionalidade Existente
1. **Identificar módulo**: Usar estrutura modular para localizar responsabilidade
2. **Manter interface pública**: Não quebrar contratos existentes
3. **Atualizar testes**: Modificar testes correspondentes ao módulo
4. **Testar integração**: Executar `pytest tests/integration/`

### Manter Modularidade
- **Regra 1**: Cada módulo tem responsabilidade única e bem definida
- **Regra 2**: Comunicação apenas via interfaces públicas documentadas
- **Regra 3**: Dependências injetadas via construtor ou parâmetros
- **Regra 4**: Mocks obrigatórios para dependências externas em testes
- **Regra 5**: Nenhum módulo deve conhecer detalhes de implementação de outros

## Deploy e Ambiente

### Desenvolvimento Local

# Setup inicial
git clone <repository>
cd curso_processor
python -m venv venv
source venv/bin/activate  # Linux/Mac
# ou venv\Scripts\activate  # Windows

# Instalação editável
pip install -e .

# Primeira configuração
neuro --setup


### Configuração Cross-Platform

# Windows
pip install curso-processor
neuro --setup  # Cria C:\Users\<user>\.curso_processor\

# macOS
pip install curso-processor
neuro --setup  # Cria ~/.curso_processor/

# Linux
pip install curso-processor
neuro --setup  # Cria ~/.curso_processor/


### Estrutura de Configuração

~/.curso_processor/
├── config/
│   ├── settings.yaml      # Configurações gerais
│   ├── apis.json          # Credenciais criptografadas
│   └── directories.json   # Paths personalizados
├── database/
│   └── memory.db         # SQLite com histórico
├── logs/
│   ├── operations.log    # Log de operações
│   └── errors.log        # Log de erros
└── temp/
    └── processing/       # Arquivos temporários


## Histórico de Alterações
- **2025-01-XX**: Criação inicial da arquitetura modular
- **2025-01-XX**: Implementação do sistema de memória persistente
- **2025-01-XX**: Interface Rich com seção "Operações" estilo qBittorrent

## Cross-Platform e Compatibilidade

### Plataformas Suportadas
- **Windows**: Windows 10+ com Python 3.9+
- **macOS**: macOS 10.15+ com Python 3.9+
- **Linux**: Distribuições com Python 3.9+ (Ubuntu 20.04+, CentOS 8+, etc.)

### Dependências do Sistema

# Windows (via chocolatey ou manual)
choco install python ffmpeg

# macOS (via homebrew)
brew install python@3.9 ffmpeg

# Linux (Ubuntu/Debian)
sudo apt update
sudo apt install python3.9 python3-pip ffmpeg

# Linux (CentOS/RHEL)
sudo yum install python39 python3-pip ffmpeg


### Tratamento de Paths Cross-Platform

# SEMPRE usar pathlib.Path
from pathlib import Path

# Correto
course_dir = Path(user_input).resolve()
audio_file = course_dir / "Audios" / f"{video.stem}.mp3"

# Incorreto - não usar
course_dir = os.path.join(user_input, "Audios")  # Não fazer isso


### Encoding e Caracteres Especiais

# Forçar UTF-8 em todas as operações I/O
with open(file_path, 'r', encoding='utf-8') as f:
    content = f.read()

# Tratar nomes de arquivos com acentos/caracteres especiais
safe_filename = unidecode(original_name)  # Fallback se necessário


## Configurações Específicas por SO

### Windows
- **Paths**: Usar `Path.resolve()` para expandir paths relativos
- **IPC**: Named Pipes (`\\.\pipe\neuro_ipc`)
- **Encoding**: UTF-8 com fallback CP1252
- **Executável**: `neuro.exe` via entry_points

### macOS
- **Paths**: Case-sensitive filesystem
- **IPC**: Unix sockets (`/tmp/neuro.sock`)
- **Permissions**: Verificar acesso a diretórios protegidos
- **Executável**: `neuro` via entry_points

### Linux
- **Paths**: Case-sensitive, links simbólicos
- **IPC**: Unix sockets (`/tmp/neuro.sock`)
- **Systemd**: Integração opcional para daemon
- **Executável**: `neuro` via entry_points

## Critérios de Validação Final

### Funcionalidades Core (100% Obrigatório)
- ✅ Sistema de memória persistente com SQLite
- ✅ Interface CLI Rich com seção "Operações"
- ✅ Conversão vídeo→áudio preservando hierarquia
- ✅ Integração ChatGPT + Claude funcionais
- ✅ Sistema de prompts personalizáveis
- ✅ Geração XML de feeds válidos
- ✅ Entry points `neuro` funcionando cross-platform

### Testes de Aceitação

# Teste 1: Instalação limpa
pip install curso-processor
neuro --setup
neuro status  # Deve mostrar configuração inicial

# Teste 2: Processamento básico
neuro cli
# Processar curso com 3 vídeos pequenos
# Verificar: Audios/, Resumos/, timestamps.md, cursos.xml

# Teste 3: Cross-platform
# Executar mesmo comando em Windows + Linux/Mac
# Verificar resultados idênticos


### Performance Mínima
- **Conversão vídeo**: <2x tempo real (vídeo 10min → conversão <20min)
- **Transcrição**: <1.5x tempo real (áudio 10min → transcrição <15min)
- **Processamento IA**: <30s por transcrição de 5min
- **Interface**: Responsiva <100ms para navegação de menus

