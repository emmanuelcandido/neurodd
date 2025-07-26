# CURSO PROCESSOR - Documentação Técnica

## 🏗️ ARQUITETURA GERAL
Sistema CLI interativo em Python para automação completa do pipeline educacional: vídeo→áudio→transcrição→processamento IA→podcast XML. Mantém memória persistente, oferece interface de monitoramento em tempo real e integração com múltiplas IAs e serviços de armazenamento.

## 📦 MÓDULOS E COMPONENTES
### core/
-   **Propósito**: Contém os módulos fundamentais para o funcionamento do sistema, como gerenciamento de memória, configurações, rastreamento de progresso e comunicação entre processos.
-   **Dependências**: SQLite3, JSON, Rich, threading, keyboard input, Click.

### processors/
-   **Propósito**: Responsável pelas etapas do pipeline de processamento de conteúdo, como conversão de vídeo, transcrição, processamento de IA, cálculo de timestamps e geração de XML.
-   **Dependências**: moviepy, edge-tts, ai_clients, prompts, config_manager, progress_tracker.

### integrations/
-   **Propósito**: Gerencia a comunicação com serviços e APIs externas, como ChatGPT, Claude, GitHub e Google Drive.
-   **Dependências**: openai, anthropic, google-generativeai, httpx, GitPython, google-api-python-client.

### cli/
-   **Propósito**: Implementa a interface de linha de comando interativa, o menu principal, a visualização de operações e o processo em segundo plano (daemon).
-   **Dependências**: Rich, Click, core modules, processors.

### data/
-   **Propósito**: Contém os modelos de dados (SQLite, Pydantic) e as migrações do banco de dados.
-   **Dependências**: SQLite3, Pydantic.

### prompts/
-   **Propósito**: Armazena os prompts personalizáveis para as interações com as IAs, organizados por tipo de conteúdo (cursos, livros).
-   **Dependências**: Nenhuma direta, mas usado por `ai_processor.py`.

### assets/
-   **Propósito**: Diretório de trabalho para armazenar arquivos temporários e gerados, como textos e áudios.
-   **Dependências**: Nenhuma direta, mas usado por `video_converter.py` e `tts_handler.py`.

### tests/
-   **Propósito**: Contém os testes unitários, de integração e fixtures para garantir a qualidade e o funcionamento do código.
-   **Dependências**: pytest, pytest-asyncio.

### data/
-   **Propósito**: Contém os modelos de dados (SQLite, Pydantic) e as migrações do banco de dados.
-   **Dependências**: SQLite3, Pydantic.

### prompts/
-   **Propósito**: Armazena os prompts personalizáveis para as interações com as IAs, organizados por tipo de conteúdo (cursos, livros).
-   **Dependências**: Nenhuma direta, mas usado por `ai_processor.py`.

### assets/
-   **Propósito**: Diretório de trabalho para armazenar arquivos temporários e gerados, como textos e áudios.
-   **Dependências**: Nenhuma direta, mas usado por `video_converter.py` e `tts_handler.py`.

### tests/
-   **Propósito**: Contém os testes unitários, de integração e fixtures para garantir a qualidade e o funcionamento do código.
-   **Dependências**: pytest, pytest-asyncio.

### config/
-   **Propósito**: Armazena os arquivos de configuração do sistema, como `settings.yaml` e `apis.json`.
-   **Dependências**: Nenhuma direta, mas usado por `config_manager.py`.

## ✨ PRINCÍPIOS DE DESIGN DA INTERFACE
A interface do NeuroDD CLI segue um conjunto rigoroso de princípios de design para garantir uma experiência de usuário consistente, intuitiva e eficiente. Estes princípios são detalhados no `design.md` na raiz do projeto e incluem:

-   **Paleta de Cores Nord**: Utilização de uma paleta de cores suaves e harmoniosas para uma estética agradável e legível.
-   **Seleção Numérica Universal**: Todas as interações de escolha são padronizadas para seleção numérica, agilizando a navegação e reduzindo erros.
-   **Feedback Visual Claro**: Mensagens de status, progresso e erros são comunicadas de forma imediata e visualmente distinta, utilizando cores e ícones apropriados.
-   **Menus Adaptativos**: A estrutura do menu se ajusta dinamicamente à complexidade da aplicação, apresentando opções de forma direta para funcionalidades simples e agrupada para cenários mais complexos.
-   **Eliminação de Ruídos**: Textos redundantes e informações desnecessárias são removidos para manter a interface limpa e focada na essência da interação.

## 🚀 COMO EXECUTAR
1.  **Instalação:** `pip install -e .`
2.  **Configuração Inicial:** `neuro --setup`
3.  **Interface Interativa:** `neuro`
4.  **Processo em Background:** `neuro start`, `neuro stop`, `neuro status`

## 🧪 COMO TESTAR
Execute todos os testes com `pytest tests/ -v --cov=curso_processor`.

## 🔧 COMO MODIFICAR
Consulte o `README.md` para instruções detalhadas sobre como adicionar novas funcionalidades ou modificar as existentes, mantendo a modularidade do projeto.
