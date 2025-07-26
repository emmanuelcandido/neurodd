### 🎯 Plano de Implementação: NeuroDD CLI Interativo Totalmente Funcional

**Visão Geral:**
Este plano será dividido em fases, garantindo que cada funcionalidade seja integrada à interface interativa e utilize os padrões visuais do `design.md` (cores Nord, seleção numérica, emojis, barras de progresso, etc.).

---

#### Fase 1: Aprimoramento da Interface CLI e Funções Básicas

**Objetivo:** Integrar completamente os utilitários de UI, implementar a lógica dos comandos `neuro start/stop/status/setup` e refinar as visualizações de operações e status.

**O que será implementado/refatorado:**

1.  **Integração Completa de `ui_utils.py`:**
    *   [x] Garantir que todas as renderizações de menu, painéis, títulos e inputs em `main_interface.py`, `operations_view.py` e `status_menu.py` utilizem as funções de `ui_utils.py` (`render_main_title`, `render_submenu_header`, `create_menu_panel`, `create_numbered_selection`, `get_numbered_choice`, `safe_input`, `get_status_text`, `create_status_table`, `execute_with_progress`, `multi_step_progress`).
    *   [x] Aplicar os decoradores `safe_operation` e `handle_errors` para tratamento robusto de interrupções e erros.

2.  **Implementação dos Comandos Globais (`neuro start/stop/status/setup`):**
    *   [x] **`neuro setup`**: Implementar a lógica de configuração inicial usando `ConfigManager` para criar/atualizar `settings.yaml`, `apis.json` e `directories.json`. Isso incluirá prompts interativos para o usuário inserir chaves de API e caminhos.
    *   [x] **`neuro start`**: Implementar a lógica para iniciar o `BackgroundDaemon` como um processo separado, utilizando `IPCHandler` para comunicação.
    *   [x] **`neuro stop`**: Implementar a lógica para enviar um comando de parada ao `BackgroundDaemon` via `IPCHandler`.
    *   [x] **`neuro status`**: Refinar `status_menu.py` para exibir informações mais detalhadas do sistema, incluindo status de conexão das IAs (usando `AIProcessor.verify_ai_status`) e caminhos configurados.

3.  **Refinamento da Visualização de Operações (`Visualizar Operações` no menu):**
    *   [x] `operations_view.py` será atualizado para buscar e exibir operações reais do `MemoryManager` (que por sua vez usa `DatabaseManager`).
    *   [ ] Apresentar as operações em uma tabela formatada com `create_status_table` e `get_status_text`, mostrando progresso e status em tempo real.

---

#### Fase 2: Pipeline de Processamento de Cursos (Core Function)

**Objetivo:** Implementar o fluxo completo de processamento de cursos através da interface interativa, incluindo a funcionalidade de transcrição que está faltando.

**O que será implementado/refatorado:**

1.  **Implementação de `processors/transcription.py`:**
    *   [x] Este é um módulo crítico que está faltando. Ele será responsável por:
        *   Receber arquivos de áudio (do `VideoConverter`).
        *   Utilizar a API do Whisper (OpenAI) ou um servidor Docker Whisper local para transcrever o áudio para texto.
        *   Lidar com o progresso da transcrição e possíveis erros.

2.  **Fluxo "Processar Curso" no `main_interface.py`:**
    *   [x] **Seleção de Diretório:** Solicitar ao usuário o caminho para o diretório do curso (usando `safe_input`).
    *   [x] **Conversão Vídeo para Áudio:** Chamar `video_converter.convert_course_videos` com barra de progresso (`execute_with_progress`).
    *   [x] **Transcrições:** Chamar `transcription.py` para transcrever os áudios gerados, também com barra de progresso.
    *   [x] **Processamento de IA:** Chamar `ai_processor.process_transcription` para processar o texto transcrito, permitindo ao usuário selecionar o prompt e a IA preferida (usando seleção numérica e `create_numbered_selection`).
    *   [x] **Seleção de IA**: Implementada a seleção de IA com base nas APIs configuradas, utilizando o sistema de seleção numérica do `design.md`.
    *   [x] **Cálculo de Timestamps:** Chamar `timestamp_calculator.calculate_hierarchical_timestamps` e `generate_timestamps_file`.
    *   [x] **Geração de XML:** Chamar `xml_generator.create_podcast_feed`.
    *   [x] **Geração de TTS e Embedding Obsidian:** Oferecer opções para o usuário gerar áudio TTS e incorporar em notas Obsidian.
    *   [x] **Upload para GitHub/Google Drive:** Oferecer opções para o usuário fazer upload dos arquivos gerados (`xml_generator`, textos processados, áudios TTS) para o GitHub (`github_client`) e Google Drive (`gdrive_client`), com barras de progresso.
    *   [x] **Registro da Operação:** Salvar todos os detalhes da operação no `MemoryManager` e `DatabaseManager`.

3.  **Menu de Operações Individuais:**
    *   [x] Criar um submenu para permitir que o usuário execute cada etapa do pipeline de forma isolada (ex: "Apenas Converter Vídeos", "Apenas Transcrever Áudios", "Apenas Processar com IA", etc.).

---

#### Fase 3: Funcionalidades Avançadas e Gerenciamento

**Objetivo:** Implementar as funcionalidades de gerenciamento de prompts, IAs, histórico e reprocessamento.

**O que será implementado/refatorado:**

1.  **Gerenciamento de Prompts:**
    *   [x] Criar um submenu para "Gerenciar Prompts".
    *   [ ] Opções para:
        *   [ ] Listar prompts existentes (`prompts/cursos/`, `prompts/livros/`).
        *   [ ] Visualizar o conteúdo de um prompt selecionado.
        *   [ ] Editar um prompt (abrir no editor padrão do sistema ou permitir edição via input multi-linha).
        *   [ ] Criar um novo prompt.

2.  **Gerenciamento de IAs:**
    *   [ ] Criar um submenu para "Gerenciar IAs".
    *   [ ] Opções para:
        *   [ ] Listar IAs disponíveis e seus status de conexão (`AIProcessor.verify_ai_status`).
        *   [ ] Testar a conexão com uma IA específica.
        *   [ ] Configurar chaves de API para IAs (direcionar para `neuro --setup` ou permitir edição via menu).
        *   [ ] Selecionar a IA preferida para processamento.

3.  **Gerenciamento de Histórico/Memória:**
    *   [ ] Criar um submenu para "Histórico de Operações".
    *   [ ] Opções para:
        *   [ ] Visualizar o histórico de todas as operações (`MemoryManager.get_history`).
        *   [ ] Ver detalhes de uma operação específica.
        *   [ ] "Esquecer" um curso específico (`MemoryManager.forget_course`).
        *   [ ] "Esquecer" todas as operações (`MemoryManager.forget_all`).

4.  **Versionamento e Reprocessamento:**
    *   [ ] Integrar lógica de versionamento no fluxo de processamento (ex: ao reprocessar um curso, criar uma nova versão dos arquivos de saída).
    *   [ ] Adicionar opção para "Reprocessar Curso Existente" no menu principal, permitindo selecionar um curso do histórico e reexecutar o pipeline.