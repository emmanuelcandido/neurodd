# PADRÃO VISUAL Python CLI - GUIA COMPLETO PARA GEMINI CLI

## 🎯 CONTEXTO
Este guia estabelece padrões visuais rigorosos para aplicações Python CLI desenvolvidas com Gemini CLI. O Gemini CLI é um agente AI open-source que oferece acesso ao Gemini diretamente no terminal, usando loops de razão e ação (ReAct) com ferramentas integradas, exigindo interfaces consistentes e profissionais.

## 📋 REQUISITOS OBRIGATÓRIOS

### 1. BIBLIOTECAS E DEPENDÊNCIAS

# IMPORTS OBRIGATÓRIOS - ORDEM FIXA
import time
import os
import sys
from rich.console import Console
from rich.panel import Panel
from rich.table import Table
from rich.text import Text
from rich.box import ROUNDED
from rich.align import Align
from rich.progress import Progress, SpinnerColumn, TextColumn, BarColumn, TaskProgressColumn
from rich.prompt import Prompt, IntPrompt, Confirm
from pyfiglet import Figlet
from typing import List, Dict, Optional, Union

console = Console()  # SEMPRE global


### 2. INICIALIZAÇÃO LIMPA (OBRIGATÓRIA)

def initialize_app():
    """Inicializa app com tela limpa - SEM mensagens de dependências"""
    os.system('cls' if os.name == 'nt' else 'clear')
    # Verificações internas SILENCIOSAS
    # Inicialização direta do menu principal


### 3. PALETA DE CORES NORD (USO MANDATÓRIO)
- **bright_white**: Dados principais, valores importantes, texto de opções
- **white**: Labels, campos secundários, texto explicativo
- **bright_blue**: Títulos, bordas, elementos interativos, numeração
- **bright_cyan**: Arte ASCII, cabeçalhos especiais
- **bright_green**: Status positivo, sucessos, confirmações
- **bright_yellow**: Avisos, alertas, informações importantes
- **bright_red**: Erros, falhas, status crítico
- **dim white**: Texto auxiliar, unidades, informações secundárias, descrições

### 4. CORES PROIBIDAS
❌ NUNCA use: red, green, yellow, blue, cyan, magenta, purple, black

## 🔢 SISTEMA DE SELEÇÃO NUMÉRICA (OBRIGATÓRIO)

### 5. PADRÕES DE SELEÇÃO DO USUÁRIO

#### 5.1 QUANDO USAR SELEÇÃO NUMÉRICA (OBRIGATÓRIO)
Use seleção numérica em TODAS as situações onde há múltiplas opções para escolha:

**✅ SEMPRE use numeração para:**
- Listas de modelos/APIs/serviços
- Seleção de arquivos/diretórios
- Opções de configuração
- Listas de resultados de busca
- Seleção de templates/presets
- Escolha de métodos/algoritmos
- Seleção de formatos de saída
- QUALQUER lista com 2 ou mais opções pré-definidas

**❌ NUNCA peça para digitar nomes/strings quando há opções pré-definidas**

#### 5.2 IMPLEMENTAÇÃO DE SELEÇÃO NUMÉRICA

def create_numbered_selection(items: List[Dict], title: str, show_descriptions: bool = True) -> str:
    """Cria seleção numerada padronizada"""
    content = ""
    
    # Para itens agrupados por categoria
    if items and 'category' in items[0]:
        current_category = None
        for i, item in enumerate(items, 1):
            if item['category'] != current_category:
                current_category = item['category']
                content += f"\n[bold bright_blue]{current_category}:[/]\n"
            
            desc = f" [dim white]- {item['description']}[/]" if show_descriptions and 'description' in item else ""
            content += f"[bright_blue][{i}][/] [bright_white]{item['name']}[/]{desc}\n"
    
    # Para itens simples
    else:
        for i, item in enumerate(items, 1):
            if isinstance(item, dict):
                name = item.get('name', str(item))
                desc = f" [dim white]- {item['description']}[/]" if show_descriptions and 'description' in item else ""
                content += f"[bright_blue][{i}][/] [bright_white]{name}[/]{desc}\n"
            else:
                content += f"[bright_blue][{i}][/] [bright_white]{str(item)}[/]\n"
    
    return content.rstrip()

def get_numbered_choice(items: List, prompt: str = "Select option") -> Optional[Union[Dict, str, int]]:
    """Solicita escolha numerada com validação"""
    try:
        choice = IntPrompt.ask(
            f"\n[bold bright_white]➤ {prompt}[/]",
            choices=[str(i) for i in range(1, len(items) + 1)],
            show_choices=False
        )
        return items[choice - 1]
    except (KeyboardInterrupt, EOFError):
        console.print("\n[bright_yellow]⚠️ Selection cancelled[/]")
        return None


#### 5.3 EXEMPLOS DE IMPLEMENTAÇÃO

**Exemplo 1 - Seleção de Modelos por Empresa:**

def select_ai_model():
    models = [
        {"name": "GPT-4", "description": "Most capable model", "category": "OpenAI"},
        {"name": "GPT-3.5 Turbo", "description": "Fast and efficient", "category": "OpenAI"},
        {"name": "Claude 3 Opus", "description": "Best reasoning", "category": "Anthropic"},
        {"name": "Claude 3 Sonnet", "description": "Balanced performance", "category": "Anthropic"},
        {"name": "Gemini Pro", "description": "Google's flagship", "category": "Google"}
    ]
    
    console.print("\n[bold bright_blue]Available AI Models[/]")
    console.print("[bright_blue]" + "─" * 20 + "[/]")
    
    content = create_numbered_selection(models, "AI Models")
    panel = create_menu_panel(content, "Select AI Model")
    console.print(panel)
    
    selected = get_numbered_choice(models, "Choose model number")
    if selected:
        console.print(f"\n[bright_green]✓ Selected: {selected['name']}[/]")
        return selected


**Exemplo 2 - Seleção Simples de Arquivos:**

def select_file_from_directory(directory: str):
    files = ["config.json", "data.csv", "output.txt", "backup.zip"]
    
    content = create_numbered_selection(files, "Files", show_descriptions=False)
    panel = create_menu_panel(content, f"Files in {directory}")
    console.print(panel)
    
    return get_numbered_choice(files, "Select file number")


#### 5.4 PADRÕES VISUAIS PARA SELEÇÕES
- **Numeração**: `[bright_blue][1][/]` (sempre azul brilhante)
- **Nome da opção**: `[bright_white]Nome[/]` (sempre branco brilhante)
- **Descrição**: `[dim white]- Descrição[/]` (sempre cinza escuro)
- **Categoria**: `[bold bright_blue]Categoria:[/]` (azul brilhante negrito)

## 🏗️ ESTRUTURA DE MENU ADAPTATIVA

### 6. DETECÇÃO INTELIGENTE DE ESTRUTURA

#### 6.1 ANÁLISE DE FUNCIONALIDADES

def analyze_app_structure(all_functions: List[Dict]) -> Dict[str, List[Dict]]:
    """
    Analisa funções da aplicação para determinar estrutura de menu ideal
    ADAPTATIVO: Não força templates quando não fazem sentido
    """
    
    # Classificação automática de funções
    core_functions = []
    config_functions = []
    utility_functions = []
    info_functions = []
    
    for func in all_functions:
        func_type = classify_function(func)
        
        if func_type == 'core':
            core_functions.append(func)
        elif func_type == 'config':
            config_functions.append(func)
        elif func_type == 'utility':
            utility_functions.append(func)
        elif func_type == 'info':
            info_functions.append(func)
    
    return {
        'core': core_functions,
        'config': config_functions,
        'utility': utility_functions,
        'info': info_functions
    }

def classify_function(function: Dict) -> str:
    """Classifica função baseado em nome/ação/contexto"""
    name_lower = function.get('name', '').lower()
    action_lower = function.get('action', '').lower()
    
    # Core: Funcionalidades principais da aplicação
    core_keywords = ['process', 'analyze', 'generate', 'create', 'build', 'run', 'execute', 'convert', 'transform', 'manage', 'monitor']
    
    # Config: Configurações e preferências
    config_keywords = ['setting', 'config', 'preference', 'option', 'setup', 'initialize']
    
    # Utility: Ferramentas auxiliares
    utility_keywords = ['clean', 'clear', 'backup', 'restore', 'export', 'import', 'log', 'history']
    
    # Info: Informações e ajuda
    info_keywords = ['about', 'help', 'info', 'version', 'documentation', 'guide']
    
    combined_text = f"{name_lower} {action_lower}"
    
    for keyword in core_keywords:
        if keyword in combined_text:
            return 'core'
    
    for keyword in config_keywords:
        if keyword in combined_text:
            return 'config'
    
    for keyword in utility_keywords:
        if keyword in combined_text:
            return 'utility'
    
    for keyword in info_keywords:
        if keyword in combined_text:
            return 'info'
    
    # Default: se não conseguir classificar, assume como core
    return 'core'


#### 6.2 ESCOLHA DE ESTRUTURA DE MENU

def determine_menu_structure(categorized_functions: Dict[str, List[Dict]]) -> str:
    """
    Determina estrutura de menu baseado no contexto da aplicação
    REGRAS ADAPTATIVAS:
    - 1 função core: Menu direto (sem agrupamentos)
    - 2-4 funções core + outras: Menu direto com todas as funções
    - 5+ funções core OU múltiplas categorias populadas: Menu agrupado
    """
    
    core_count = len(categorized_functions['core'])
    total_functions = sum(len(funcs) for funcs in categorized_functions.values())
    populated_categories = sum(1 for funcs in categorized_functions.values() if len(funcs) > 0)
    
    # Menu direto: Poucas funções ou apenas uma categoria
    if core_count <= 4 and populated_categories <= 2 and total_functions <= 8:
        return 'direct'
    
    # Menu agrupado: Muitas funções ou múltiplas categorias
    return 'grouped'

def render_adaptive_menu(all_functions: List[Dict]):
    """Renderiza menu com estrutura adaptativa baseada no contexto"""
    
    categorized = analyze_app_structure(all_functions)
    structure = determine_menu_structure(categorized)
    
    if structure == 'direct':
        render_direct_menu(all_functions)
    else:
        render_grouped_menu(categorized)


### 7. ESTRUTURA DE GRUPOS (APENAS QUANDO APLICÁVEL)

#### Grupos Disponíveis (NÃO obrigatórios - use apenas se fazem sentido):

AVAILABLE_GROUPS = {
    "core": {
        "title": "🎯 Core Functions",
        "description": "Primary application functions",
        "priority": 1
    },
    "config": {
        "title": "⚙️ Configuration", 
        "description": "System and user settings",
        "priority": 2
    },
    "data": {
        "title": "📊 Reports & Data",
        "description": "Data management and reporting",
        "priority": 3
    },
    "utils": {
        "title": "🔧 Utilities",
        "description": "Maintenance and diagnostic tools",
        "priority": 4
    },
    "info": {
        "title": "📖 Information",
        "description": "Help, documentation, and system info",
        "priority": 5
    }
}

def create_menu_groups(categorized_functions: Dict[str, List[Dict]]) -> List[Dict]:
    """Cria grupos de menu apenas para categorias que têm funções"""
    
    groups = []
    
    for category, functions in categorized_functions.items():
        if functions and category in AVAILABLE_GROUPS:  # Só adiciona se tem funções
            group_info = AVAILABLE_GROUPS[category].copy()
            group_info['functions'] = functions
            groups.append(group_info)
    
    # Ordena por prioridade
    groups.sort(key=lambda x: x['priority'])
    
    return groups


### 8. SISTEMA DE EMOJIS

#### Análise por Menu Completo:

def should_use_emojis_in_menu(menu_options):
    """
    Decide se deve usar emojis baseado na viabilidade do menu completo
    REGRA: Se 80%+ das opções têm emojis naturais → Use em todas
           Se <80% têm emojis naturais → Não use em nenhuma
    """
    viable_emojis = 0
    
    for option in menu_options:
        if find_natural_emoji(option.get('action', option.get('name', ''))) is not None:
            viable_emojis += 1
    
    return (viable_emojis / len(menu_options)) >= 0.8

def find_natural_emoji(action_or_name: str) -> Optional[str]:
    """Encontra emoji natural para ação/nome"""
    text = action_or_name.lower()
    
    # Busca em EMOJI_LIBRARY
    for keyword, emoji in EMOJI_LIBRARY.items():
        if keyword in text:
            return emoji
    
    # Análise semântica adicional
    semantic_matches = {
        ('manage', 'organize', 'admin'): '🗂️',
        ('analyze', 'report', 'stats'): '📊',
        ('test', 'verify', 'check'): '🔍',
        ('connect', 'link', 'api'): '🔗',
        ('clean', 'clear', 'reset'): '🧹',
        ('install', 'setup', 'init'): '⚡',
        ('update', 'refresh', 'sync'): '🔄'
    }
    
    for keywords, emoji in semantic_matches.items():
        if any(keyword in text for keyword in keywords):
            return emoji
    
    return None


#### Biblioteca de Emojis Expandida:

EMOJI_LIBRARY = {
    # Ações principais
    'download': '⬇️', 'upload': '⬆️', 'search': '🔍', 'find': '🔍',
    'settings': '⚙️', 'config': '⚙️', 'configure': '⚙️',
    'delete': '🗑️', 'remove': '🗑️', 'edit': '✏️', 'modify': '✏️',
    'view': '👁️', 'show': '👁️', 'display': '👁️',
    'create': '➕', 'add': '➕', 'new': '➕',
    'copy': '📋', 'duplicate': '📋', 'clone': '📋',
    'save': '💾', 'store': '💾', 'load': '📂', 'open': '📂',
    'export': '📤', 'import': '📥', 'transfer': '🔄',
    'sync': '🔄', 'refresh': '🔄', 'update': '🔄',
    'backup': '💾', 'restore': '⏮️', 'recover': '⏮️',
    
    # Controles de mídia
    'play': '▶️', 'pause': '⏸️', 'stop': '⏹️', 'next': '⏭️',
    'previous': '⏮️', 'start': '🚀', 'begin': '🚀',
    'finish': '✅', 'complete': '✅', 'done': '✅',
    'cancel': '❌', 'abort': '❌', 'quit': '❌',
    
    # Navegação
    'back': '↩️', 'return': '↩️', 'forward': '➡️', 'next': '➡️',
    'exit': '🚪', 'quit': '🚪', 'home': '🏠', 'main': '🏠',
    'help': '❓', 'info': 'ℹ️', 'about': '📋', 'details': 'ℹ️',
    
    # Arquivos e dados
    'file': '📄', 'document': '📃', 'folder': '📁', 'directory': '📁',
    'image': '🖼️', 'picture': '🖼️', 'photo': '📷',
    'audio': '🎵', 'music': '🎵', 'sound': '🔊',
    'video': '🎬', 'movie': '🎬', 'book': '📚', 'text': '📝',
    'data': '📊', 'database': '🗄️', 'table': '📋',
    
    # Sistema e rede
    'network': '🌐', 'internet': '🌐', 'web': '🌐',
    'server': '🖥️', 'api': '🔗', 'connection': '🔗',
    'log': '📋', 'history': '📜', 'record': '📝',
    'monitor': '📊', 'stats': '📈', 'report': '📋',
    'analyze': '🔬', 'analysis': '🔬', 'test': '🧪',
    
    # Organização
    'organize': '🗂️', 'sort': '🗂️', 'arrange': '🗂️',
    'manage': '🗂️', 'admin': '👑', 'control': '🎛️',
    'queue': '📝', 'list': '📝', 'schedule': '📅',
    'calendar': '📅', 'time': '⏰', 'timer': '⏱️',
    
    # Status e ações especiais
    'clean': '🧹', 'clear': '🧹', 'reset': '🔄',
    'install': '⚡', 'setup': '⚡', 'init': '⚡',
    'build': '🔨', 'compile': '⚙️', 'process': '⚙️',
    'run': '▶️', 'execute': '▶️', 'launch': '🚀',
    
    # Comunicação e social
    'send': '📤', 'receive': '📥', 'message': '💬',
    'chat': '💬', 'mail': '📧', 'email': '📧',
    'share': '📤', 'publish': '📢', 'notify': '🔔',
    
    # Segurança
    'lock': '🔒', 'unlock': '🔓', 'secure': '🔒',
    'key': '🔑', 'password': '🔑', 'auth': '🔐',
    'verify': '✅', 'validate': '✅', 'check': '🔍',
    
    # Fallbacks universais
    'default': '📄', 'action': '⚡', 'option': '🔧', 
    'tool': '🛠️', 'utility': '🔧', 'function': '⚙️'
}


### 9. FUNÇÕES UTILITÁRIAS OBRIGATÓRIAS


def render_main_title(app_name: str):
    """Renderiza título principal com fonte 'big' - APENAS para menu principal"""
    os.system('cls' if os.name == 'nt' else 'clear')
    figlet = Figlet(font="big")
    art = figlet.renderText(app_name)
    centered_art = Align.center(Text(art, style="bold bright_cyan"))
    console.print(centered_art)
    console.print()

def render_submenu_header(menu_name: str, emoji: str = "⚙️"):
    """Cabeçalho simples para submenus - SEM arte ASCII"""
    os.system('cls' if os.name == 'nt' else 'clear')
    header = f"{emoji} {menu_name}"
    console.print(f"\n[bold bright_cyan]{header}[/]")
    console.print("[bright_blue]" + "─" * len(header) + "[/]")
    console.print()

def create_menu_panel(content: str, title: str) -> Panel:
    """Painel de menu sem emoji na borda"""
    return Panel(
        content,
        title=f"[bold bright_blue]{title}[/]",
        border_style="bright_blue",
        box=ROUNDED,
        padding=(1, 2)
    )

def get_menu_choice(prompt: str = "Enter your choice", max_option: int = None) -> str:
    """Input padrão com navegação ESC e validação numérica"""
    try:
        if max_option:
            # Para menus numerados, use IntPrompt com validação
            choice = IntPrompt.ask(
                f"\n[bold bright_white]➤ {prompt} (0 to go back)[/]",
                choices=[str(i) for i in range(0, max_option + 1)],
                show_choices=False
            )
            return str(choice)
        else:
            # Para inputs gerais
            choice = console.input(f"\n[bold bright_white]➤ {prompt} (ESC to go back): [/]").strip()
            return choice
    except KeyboardInterrupt:
        console.print("\n[bright_yellow]⚠️ Returning to previous menu (ESC/Ctrl+C)[/]")
        return "0"
    except EOFError:
        console.print("\n[bright_yellow]⚠️ Input ended (Ctrl+D)[/]")
        return "0"

def create_progress_bar(description: str = "Processing"):
    """Cria barra de progresso padrão"""
    return Progress(
        SpinnerColumn(),
        TextColumn("[bold bright_blue]{task.description}[/]"),
        BarColumn(bar_width=40, style="bright_blue", complete_style="bright_green"),
        TaskProgressColumn(),
        console=console,
        transient=False
    )

def safe_input(prompt: str, input_type: str = "text", **kwargs) -> Union[str, int, bool, None]:
    """Input seguro com tratamento de interrupções"""
    try:
        if input_type == "int":
            return IntPrompt.ask(f"[bold bright_white]➤ {prompt}[/]", **kwargs)
        elif input_type == "bool":
            return Confirm.ask(f"[bold bright_white]➤ {prompt}[/]", **kwargs)
        else:
            return Prompt.ask(f"[bold bright_white]➤ {prompt}[/]", **kwargs)
    except (KeyboardInterrupt, EOFError):
        console.print("\n[bright_yellow]⚠️ Input cancelled[/]")
        return None


### 10. SISTEMA DE BARRAS DE PROGRESSO

#### Quando Usar:
- **Operações > 2 segundos**: SEMPRE implementar
- **Downloads/Uploads**: SEMPRE implementar
- **Processamento de dados**: SEMPRE implementar
- **Operações de rede**: SEMPRE implementar
- **Análise de arquivos**: SEMPRE implementar
- **Seleções de API/modelo**: Para operações de teste/validação

#### Implementação Avançada:

def execute_with_progress(operation_func, description: str, *args, **kwargs):
    """Executa operação com barra de progresso"""
    with create_progress_bar(description) as progress:
        task = progress.add_task(description, total=100)
        
        try:
            result = operation_func(progress, task, *args, **kwargs)
            progress.update(task, completed=100)
            time.sleep(0.5)  # Permite visualizar 100%
            return result
        except Exception as e:
            progress.update(task, completed=100)
            console.print(f"\n[bright_red]✗ Error: {str(e)}[/]")
            return None

def multi_step_progress(steps: List[Dict[str, any]]):
    """Progresso multi-etapas"""
    total_steps = len(steps)
    
    with create_progress_bar("Multi-step Operation") as progress:
        for i, step in enumerate(steps):
            task = progress.add_task(step['description'], total=100)
            
            try:
                result = step['function'](**step.get('args', {}))
                progress.update(task, completed=100)
                
                if not result and step.get('required', True):
                    console.print(f"\n[bright_red]✗ Failed at step: {step['description']}[/]")
                    return False
                    
            except Exception as e:
                console.print(f"\n[bright_red]✗ Error in {step['description']}: {str(e)}[/]")
                return False
        
    console.print("\n[bright_green]✅ All steps completed successfully[/]")
    return True


### 11. RENDERIZAÇÃO DE MENU ADAPTATIVA


def render_direct_menu(all_functions: List[Dict]):
    """Menu direto - todas as funções em uma lista simples"""
    
    use_emojis = should_use_emojis_in_menu(all_functions)
    content = ""
    
    for i, func in enumerate(all_functions, 1):
        emoji = f"{find_natural_emoji(func.get('action', func.get('name', '')))} " if use_emojis else ""
        desc = f" [dim white]- {func['description']}[/]" if 'description' in func else ""
        content += f"[bright_blue][{i}][/] {emoji}[bright_white]{func['name']}[/]{desc}\n"
    
    panel = create_menu_panel(content.rstrip(), "Main Menu")
    console.print(panel)

def render_grouped_menu(categorized_functions: Dict[str, List[Dict]]):
    """Menu agrupado - funções organizadas por categoria"""
    
    groups = create_menu_groups(categorized_functions)
    option_counter = 1
    
    for group in groups:
        use_emojis = should_use_emojis_in_menu(group['functions'])
        content = ""
        
        for func in group['functions']:
            emoji = f"{find_natural_emoji(func.get('action', func.get('name', '')))} " if use_emojis else ""
            desc = f" [dim white]- {func['description']}[/]" if 'description' in func else ""
            content += f"[bright_blue][{option_counter}][/] {emoji}[bright_white]{func['name']}[/]{desc}\n"
            option_counter += 1
        
        panel = create_menu_panel(content.rstrip(), group['title'])
        console.print(panel)
        console.print()


### 12. NAVEGAÇÃO ESC E VALIDAÇÃO (ATUALIZADA)

def handle_menu_navigation(choice: str, max_option: int) -> Union[int, str]:
    """Manipula navegação de menu com validação"""
    if choice.lower() in ['0', 'back', 'b']:
        return "back"
    
    try:
        num_choice = int(choice)
        if 1 <= num_choice <= max_option:
            return num_choice
        else:
            console.print(f"[bright_yellow]⚠️ Please enter a number between 1 and {max_option}[/]")
            return "invalid"
    except ValueError:
        console.print("[bright_yellow]⚠️ Please enter a valid number[/]")
        return "invalid"

def menu_loop(menu_function, options: List[Dict], title: str = "Menu"):
    """Loop de menu padrão com navegação"""
    while True:
        menu_function()
        choice = get_menu_choice("Select option", len(options))
        
        result = handle_menu_navigation(choice, len(options))
        
        if result == "back":
            break
        elif result == "invalid":
            continue
        else:
            # Executar função selecionada
            selected_option = options[result - 1]
            try:
                selected_option['function']()
            except KeyboardInterrupt:
                console.print("\n[bright_yellow]⚠️ Operation cancelled[/]")
                continue


### 13. SISTEMA DE STATUS EXPANDIDO

def get_status_text(status: str, context: str = "") -> str:
    """Sistema de status expandido com contexto"""
    status_map = {
        # Status positivos
        "success": "[bright_green]✓ Success[/]",
        "completed": "[bright_green]✓ Completed[/]",
        "active": "[bright_green]● Active[/]",
        "online": "[bright_green]● Online[/]",
        "connected": "[bright_green]🔗 Connected[/]",
        "available": "[bright_green]✓ Available[/]",
        
        # Status de progresso
        "processing": "[bright_blue]⟳ Processing[/]",
        "loading": "[bright_blue]⟳ Loading[/]",
        "syncing": "[bright_blue]🔄 Syncing[/]",
        "downloading": "[bright_blue]⬇️ Downloading[/]",
		"uploading": "[bright_blue]⬆️ Uploading[/]",
        
        # Status de aviso
        "warning": "[bright_yellow]⚠ Warning[/]",
        "pending": "[bright_yellow]○ Pending[/]",
        "waiting": "[bright_yellow]⏳ Waiting[/]",
        "partial": "[bright_yellow]◐ Partial[/]",
        
        # Status de erro
        "error": "[bright_red]✗ Error[/]",
        "failed": "[bright_red]✗ Failed[/]",
        "inactive": "[bright_red]● Inactive[/]",
        "offline": "[bright_red]● Offline[/]",
        "disconnected": "[bright_red]🔗 Disconnected[/]",
        "unavailable": "[bright_red]✗ Unavailable[/]",
        
        # Status neutros
        "unknown": "[white]? Unknown[/]",
        "none": "[dim white]— None[/]",
        "disabled": "[dim white]○ Disabled[/]"
    }
    
    base_status = status_map.get(status.lower(), f"[white]{status}[/]")
    return f"{base_status} {context}".strip() if context else base_status

def create_status_table(items: List[Dict], title: str = "Status") -> Table:
    """Cria tabela de status padronizada"""
    table = Table(title=f"[bold bright_blue]{title}[/]", box=ROUNDED)
    table.add_column("Item", style="bright_white", no_wrap=True)
    table.add_column("Status", style="white")
    table.add_column("Details", style="dim white")
    
    for item in items:
        table.add_row(
            item.get('name', 'Unknown'),
            get_status_text(item.get('status', 'unknown')),
            item.get('details', '')
        )
    
    return table


### 14. REGRAS DE ESPAÇAMENTO REFINADAS

# HIERARQUIA DE ESPAÇAMENTO (ORDEM DE PRIORIDADE)

# 1. Após arte ASCII - SEMPRE uma linha
console.print()

# 2. Entre grupos de menu - SEMPRE uma linha
console.print()

# 3. Antes de input - SEMPRE uma linha
console.print()

# 4. Após mensagens de status/erro - UMA linha
console.print()

# 5. Entre tabelas/painéis - UMA linha se relacionados, DUAS se diferentes contextos
console.print()  # relacionados
console.print("\n")  # contextos diferentes

# 6. NUNCA mais de duas linhas em branco consecutivas
# 7. NUNCA espaço no início de funções (primeira linha sempre conteúdo)


### 15. TRATAMENTO DE INTERRUPÇÕES E ERROS AVANÇADO

def safe_operation(operation_func, description: str = "Operation", show_progress: bool = True):
    """Wrapper seguro para operações com tratamento completo de erros"""
    def wrapper(*args, **kwargs):
        try:
            if show_progress and hasattr(operation_func, '__code__') and operation_func.__code__.co_argcount > 2:
                return execute_with_progress(operation_func, description, *args, **kwargs)
            else:
                return operation_func(*args, **kwargs)
                
        except KeyboardInterrupt:
            console.print(f"\n[bright_yellow]⚠️ {description} cancelled by user (Ctrl+C)[/]")
            return None
        except EOFError:
            console.print(f"\n[bright_yellow]⚠️ Input ended unexpectedly (Ctrl+D)[/]")
            return None
        except FileNotFoundError as e:
            console.print(f"\n[bright_red]📄 File not found: {str(e)}[/]")
            return None
        except PermissionError as e:
            console.print(f"\n[bright_red]🔒 Permission denied: {str(e)}[/]")
            return None
        except ConnectionError as e:
            console.print(f"\n[bright_red]🌐 Connection error: {str(e)}[/]")
            return None
        except TimeoutError as e:
            console.print(f"\n[bright_red]⏱️ Operation timed out: {str(e)}[/]")
            return None
        except ValueError as e:
            console.print(f"\n[bright_red]📊 Invalid value: {str(e)}[/]")
            return None
        except Exception as e:
            console.print(f"\n[bright_red]💥 Unexpected error in {description}: {str(e)}[/]")
            return None
    
    return wrapper

# Decorador para funções principais
def handle_errors(description: str = "Operation"):
    """Decorador para tratamento automático de erros"""
    def decorator(func):
        return safe_operation(func, description, show_progress=False)
    return decorator


### 16. ELIMINAÇÃO DE TEXTOS REDUNDANTES (EXPANDIDA)

#### Proibições Rigorosas:
❌ **"Welcome to [App Name]"** - Título ASCII já identifica
❌ **"Please select an option"** - Interface já é clara
❌ **"The following options are available"** - Lista já mostra opções
❌ **"Configuration settings"** - Título do painel já indica
❌ **"Processing your request..."** - Barra de progresso já mostra
❌ **"Operation completed successfully"** - Status visual já indica
❌ **"Invalid input, try again"** - Validação já trata
❌ **"Press Enter to continue"** - Interação óbvia
❌ **"Type the number of your choice"** - Prompt já indica isso

#### Exceções Permitidas:
✅ **Contexto técnico específico**: "Using API key: sk-***1234"
✅ **Resultados quantificados**: "Found 15 files in directory"
✅ **Status operacional crítico**: "Database connection lost"
✅ **Guias para ações complexas**: "Select models to compare (space to toggle)"
✅ **Confirmações de ações destrutivas**: "This will permanently delete 45 files"

### 17. INTEGRAÇÃO COM GEMINI CLI

#### 17.1 Padrões Específicos para Gemini CLI

# Configuração para compatibilidade com Gemini CLI
def setup_gemini_cli_compatibility():
    """Configura ambiente para máxima compatibilidade com Gemini CLI"""
    
    # Rich configuration para terminais diversos
    console._environ = {
        'TERM': 'xterm-256color',  # Força suporte a cores
        'COLORTERM': 'truecolor'   # Suporte a cores true
    }
    
    # Configurações de encoding para saída limpa
    if sys.platform == "win32":
        import codecs
        sys.stdout = codecs.getwriter("utf-8")(sys.stdout.detach())
        sys.stderr = codecs.getwriter("utf-8")(sys.stderr.detach())

def optimize_for_automation():
    """Otimizações para uso automatizado via Gemini CLI"""
    
    # Detecta se está sendo executado via Gemini CLI
    is_automated = any(arg in sys.argv for arg in ['--automated', '--gemini-cli', '--batch'])
    
    if is_automated:
        # Reduz timeouts
        console._force_terminal = True
        
        # Simplifica prompts para parsing automatizado
        return True
    
    return False


#### 17.2 Comandos CLI Integrados

def add_cli_arguments():
    """Adiciona argumentos CLI padrão compatíveis com Gemini"""
    import argparse
    
    parser = argparse.ArgumentParser(
        description="NeuroDeamon Application",
        formatter_class=argparse.RawDescriptionHelpFormatter,
        epilog="""
Examples:
  python app.py                    # Interactive mode
  python app.py --automated        # Gemini CLI mode
  python app.py --config path.json # Custom config
        """
    )
    
    parser.add_argument('--automated', action='store_true',
                       help='Run in automated mode (Gemini CLI compatible)')
    parser.add_argument('--config', type=str,
                       help='Path to configuration file')
    parser.add_argument('--output-format', choices=['json', 'table', 'plain'],
                       default='table', help='Output format')
    parser.add_argument('--no-color', action='store_true',
                       help='Disable colored output')
    parser.add_argument('--verbose', '-v', action='count', default=0,
                       help='Increase verbosity level')
    
    return parser.parse_args()


### 18. EXEMPLOS COMPLETOS DE IMPLEMENTAÇÃO

#### Exemplo 1 - Aplicação com Poucas Funções (Menu Direto):

@handle_errors("Simple File Manager")
def main():
    """Aplicação simples com menu direto"""
    
    # Setup inicial
    setup_gemini_cli_compatibility()
    args = add_cli_arguments()
    
    # Definir funções da aplicação
    all_functions = [
        {"name": "List Files", "description": "Show all files in directory", "action": "list", "function": list_files},
        {"name": "Copy File", "description": "Copy selected file", "action": "copy", "function": copy_file},
        {"name": "Delete File", "description": "Remove selected file", "action": "delete", "function": delete_file},
        {"name": "Settings", "description": "Application settings", "action": "settings", "function": show_settings}
    ]
    
    # Inicialização e análise
    initialize_app()
    categorized = analyze_app_structure(all_functions)
    structure = determine_menu_structure(categorized)
    
    # Renderizar menu apropriado
    render_main_title("File Manager")
    
    if structure == 'direct':
        render_direct_menu(all_functions)
    else:
        render_grouped_menu(categorized)
    
    # Loop principal
    menu_loop(
        lambda: (render_main_title("File Manager"), render_direct_menu(all_functions) if structure == 'direct' else render_grouped_menu(categorized)),
        all_functions,
        "File Manager"
    )

def list_files():
    """Lista arquivos com seleção numérica"""
    render_submenu_header("File List", "📄")
    
    # Simular lista de arquivos
    files = ["document.pdf", "image.jpg", "data.csv", "backup.zip", "notes.txt"]
    
    content = create_numbered_selection(files, "Files")
    panel = create_menu_panel(content, "Files in Current Directory")
    console.print(panel)
    
    selected = get_numbered_choice(files, "Select file to view details")
    if selected:
        console.print(f"\n[bright_green]✓ Selected: {selected}[/]")
        console.print("[dim white]Size: 1.2 MB | Modified: 2024-01-15[/]")
    
    safe_input("Press Enter to continue")


#### Exemplo 2 - Aplicação Complexa (Menu Agrupado):

@handle_errors("Advanced Data Processor")
def main():
    """Aplicação complexa com menu agrupado"""
    
    setup_gemini_cli_compatibility()
    args = add_cli_arguments()
    
    # Muitas funções core - forçará menu agrupado
    all_functions = [
        # Core functions (muitas)
        {"name": "Process Data", "description": "Main data processing", "action": "process", "function": process_data},
        {"name": "Analyze Results", "description": "Statistical analysis", "action": "analyze", "function": analyze_data},
        {"name": "Generate Report", "description": "Create PDF reports", "action": "report", "function": generate_report},
        {"name": "Batch Convert", "description": "Convert multiple files", "action": "convert", "function": batch_convert},
        {"name": "Merge Datasets", "description": "Combine data sources", "action": "merge", "function": merge_data},
        {"name": "Export Results", "description": "Export in various formats", "action": "export", "function": export_data},
        
        # Config functions
        {"name": "Settings", "description": "Application preferences", "action": "config", "function": show_settings},
        {"name": "Import Config", "description": "Load configuration file", "action": "import", "function": import_config},
        
        # Utility functions
        {"name": "Clean Cache", "description": "Clear temporary files", "action": "clean", "function": clean_cache},
        {"name": "View Logs", "description": "Show processing logs", "action": "log", "function": view_logs},
        
        # Info functions
        {"name": "Help", "description": "User documentation", "action": "help", "function": show_help},
        {"name": "About", "description": "Version and credits", "action": "about", "function": show_about}
    ]
    
    initialize_app()
    categorized = analyze_app_structure(all_functions)
    structure = determine_menu_structure(categorized)  # Será 'grouped'
    
    render_main_title("Data Processor")
    render_grouped_menu(categorized)
    
    # Criar lista plana para o menu loop
    flat_functions = []
    for category_funcs in categorized.values():
        flat_functions.extend(category_funcs)
    
    menu_loop(
        lambda: (render_main_title("Data Processor"), render_grouped_menu(categorized)),
        flat_functions,
        "Data Processor"
    )

def select_processing_models():
    """Exemplo de seleção múltipla com categorias"""
    render_submenu_header("Model Selection", "🤖")
    
    models = [
        {"name": "Linear Regression", "description": "Basic linear model", "category": "Statistical"},
        {"name": "Random Forest", "description": "Ensemble method", "category": "Statistical"},
        {"name": "Neural Network", "description": "Deep learning model", "category": "AI/ML"},
        {"name": "Transformer", "description": "Attention-based model", "category": "AI/ML"},
        {"name": "Decision Tree", "description": "Rule-based model", "category": "Statistical"},
        {"name": "SVM", "description": "Support Vector Machine", "category": "Statistical"}
    ]
    
    console.print("\n[bold bright_blue]Available Processing Models[/]")
    console.print("[bright_blue]" + "─" * 30 + "[/]")
    
    content = create_numbered_selection(models, "Models", show_descriptions=True)
    panel = create_menu_panel(content, "Select Processing Model")
    console.print(panel)
    
    selected = get_numbered_choice(models, "Choose model")
    
    if selected:
        # Executar com barra de progresso
        def initialize_model(progress, task):
            steps = ["Loading model", "Initializing parameters", "Validating configuration", "Ready to process"]
            for i, step in enumerate(steps):
                progress.update(task, description=step, completed=(i+1) * 25)
                time.sleep(0.8)
            return True
        
        result = execute_with_progress(
            initialize_model,
            f"Initializing {selected['name']}"
        )
        
        if result:
            console.print(f"\n[bright_green]✅ {selected['name']} initialized successfully[/]")
            console.print(f"[dim white]Category: {selected['category']} | {selected['description']}[/]")
            
            # Mostrar opções de configuração do modelo
            config_options = ["Default Settings", "Custom Parameters", "Load Preset"]
            
            console.print(f"\n[bold bright_blue]Configuration Options for {selected['name']}[/]")
            console.print("[bright_blue]" + "─" * 40 + "[/]")
            
            config_content = create_numbered_selection(config_options, "Configuration")
            config_panel = create_menu_panel(config_content, "Model Configuration")
            console.print(config_panel)
            
            config_choice = get_numbered_choice(config_options, "Select configuration")
            if config_choice:
                console.print(f"\n[bright_green]✓ Using: {config_choice}[/]")
    
    safe_input("Press Enter to continue")


### 19. PADRÕES DE SELEÇÃO AVANÇADA

#### Exemplo 3 - Seleção com Toggle Multiple:

def select_multiple_datasets():
    """Seleção múltipla com toggles visuais e validação"""
    render_submenu_header("Dataset Selection", "📊")
    
    datasets = [
        {"name": "Sales Data Q1", "selected": False, "size": "2.1 MB", "category": "Sales"},
        {"name": "Sales Data Q2", "selected": False, "size": "2.3 MB", "category": "Sales"},
        {"name": "Customer Info", "selected": True, "size": "5.7 MB", "category": "Customer"},
        {"name": "Product Catalog", "selected": False, "size": "1.8 MB", "category": "Product"},
        {"name": "Market Research", "selected": False, "size": "12.4 MB", "category": "Research"},
        {"name": "Competitor Analysis", "selected": False, "size": "8.2 MB", "category": "Research"}
    ]
    
    while True:
        # Render current selection state
        console.print("\n[bold bright_blue]Select Datasets for Processing[/]")
        console.print("[bright_blue]" + "─" * 50 + "[/]")
        
        content = ""
        current_category = None
        
        for i, dataset in enumerate(datasets, 1):
            if dataset['category'] != current_category:
                current_category = dataset['category']
                content += f"\n[bold bright_blue]{current_category} Data:[/]\n"
            
            # Visual toggle indicator
            toggle = "[bright_green]●[/]" if dataset['selected'] else "[dim white]○[/]"
            size_info = f"[dim white]({dataset['size']})[/]"
            content += f"[bright_blue][{i}][/] {toggle} [bright_white]{dataset['name']}[/] {size_info}\n"
        
        # Summary and confirmation option
        selected_count = sum(1 for d in datasets if d['selected'])
        total_size = sum(float(d['size'].split()[0]) for d in datasets if d['selected'])
        
        content += f"\n[bright_blue][0][/] [bright_white]Process Selected[/] [dim white]({selected_count} datasets, {total_size:.1f} MB)[/]"
        
        panel = create_menu_panel(content.rstrip(), "Dataset Selection (Toggle with numbers)")
        console.print(panel)
        
        choice = get_menu_choice("Select number to toggle (0 to process)", len(datasets))
        
        if choice == "0":
            selected_datasets = [d for d in datasets if d['selected']]
            if selected_datasets:
                console.print(f"\n[bright_green]✅ Processing {len(selected_datasets)} datasets[/]")
                
                # Mostrar selecionados
                for dataset in selected_datasets:
                    console.print(f"  [bright_white]• {dataset['name']}[/] [dim white]({dataset['size']})[/]")
                
                # Simular processamento
                def process_datasets(progress, task):
                    for i, dataset in enumerate(selected_datasets):
                        progress.update(task, description=f"Processing {dataset['name']}", completed=((i+1)/len(selected_datasets))*100)
                        time.sleep(1.2)
                    return True
                
                execute_with_progress(process_datasets, "Processing Selected Datasets")
                return selected_datasets
            else:
                console.print("[bright_yellow]⚠️ Please select at least one dataset[/]")
                continue
        else:
            try:
                idx = int(choice) - 1
                if 0 <= idx < len(datasets):
                    datasets[idx]['selected'] = not datasets[idx]['selected']
                    # Clear screen and show updated selection
                    os.system('cls' if os.name == 'nt' else 'clear')
                    render_submenu_header("Dataset Selection", "📊")
            except (ValueError, IndexError):
                console.print("[bright_yellow]⚠️ Invalid selection[/]")
                time.sleep(1)


### 20. SISTEMA DE MENU PRINCIPAL INTELIGENTE


def create_intelligent_main_menu(app_name: str, all_functions: List[Dict]):
    """
    Cria menu principal com detecção inteligente de estrutura
    ADAPTATIVO: Escolhe automaticamente entre direto e agrupado
    """
    
    # Análise da aplicação
    categorized = analyze_app_structure(all_functions)
    structure = determine_menu_structure(categorized)
    
    # Renderização adaptativa
    render_main_title(app_name)
    
    if structure == 'direct':
        console.print(f"[dim white]Simple interface mode - {len(all_functions)} functions[/]")
        console.print()
        render_direct_menu(all_functions)
        return all_functions, 'direct'
    else:
        populated_groups = [group for group in categorized.values() if group]
        console.print(f"[dim white]Organized interface mode - {len(populated_groups)} categories[/]")
        console.print()
        render_grouped_menu(categorized)
        
        # Flatten for menu_loop
        flat_functions = []
        for category_funcs in categorized.values():
            flat_functions.extend(category_funcs)
        return flat_functions, 'grouped'

def main_application_template(app_name: str, functions: List[Dict]):
    """
    Template principal para aplicações NeuroDeamon
    USO UNIVERSAL - Adapta-se automaticamente a qualquer estrutura
    """
    
    # Setup padrão
    setup_gemini_cli_compatibility()
    args = add_cli_arguments()
    initialize_app()
    
    # Menu inteligente
    menu_functions, menu_type = create_intelligent_main_menu(app_name, functions)
    
    # Loop principal com re-renderização apropriada
    def refresh_menu():
        if menu_type == 'direct':
            render_main_title(app_name)
            render_direct_menu(functions)
        else:
            categorized = analyze_app_structure(functions)
            render_main_title(app_name)
            render_grouped_menu(categorized)
    
    menu_loop(refresh_menu, menu_functions, app_name)


## 🎯 CHECKLIST DE CONFORMIDADE FINAL ATUALIZADO

### Checklist Técnico:
- [ ] Imports obrigatórios na ordem correta
- [ ] Inicialização limpa sem mensagens desnecessárias
- [ ] Arte ASCII com fonte 'big' apenas no menu principal
- [ ] Paleta de cores Nord rigorosamente respeitada
- [ ] **Sistema de seleção numérica implementado para TODAS escolhas múltiplas (OBRIGATÓRIO)**
- [ ] **Detecção inteligente de estrutura de menu (adaptativo)**
- [ ] Análise de viabilidade de emojis por menu (regra 80%)
- [ ] Biblioteca de emojis expandida com fallbacks
- [ ] Barras de progresso para operações > 2 segundos
- [ ] Navegação ESC/Ctrl+C tratada em todos inputs
- [ ] Tratamento robusto de erros e interrupções
- [ ] Validação numérica em menus
- [ ] Sistema de status expandido
- [ ] Compatibilidade com Gemini CLI

### Checklist de UX Atualizado:
- [ ] **Numeração sequencial crescente em TODOS os menus (OBRIGATÓRIO)**
- [ ] **Seleção numérica SEMPRE ao invés de texto livre (OBRIGATÓRIO)**
- [ ] **Menu adaptativo baseado no contexto da aplicação**
- [ ] **Grupos de funções apenas quando fazem sentido (não forçar)**
- [ ] Feedback visual claro para seleções/status
- [ ] Eliminação completa de textos redundantes (incluindo "digite o número")
- [ ] Espaçamento consistente e hierárquico
- [ ] Prompts claros e concisos
- [ ] Confirmações para ações destrutivas
- [ ] Progress feedback para operações longas

### Checklist de Estrutura Adaptativa:
- [ ] **Análise automática de funções por categoria**
- [ ] **Detecção automática: direct vs grouped menu baseado em contexto**
- [ ] **Template de grupos usado apenas quando aplicável**
- [ ] **Menu direto para aplicações simples (≤4 core functions)**
- [ ] **Menu agrupado para aplicações complexas (5+ core functions ou múltiplas categorias)**
- [ ] Submenus com headers apropriados (sem ASCII art)
- [ ] Menu principal com título ASCII centralizado
- [ ] Painéis consistentes com bordas azuis
- [ ] Integração com argumentos CLI
- [ ] Modo automatizado para Gemini CLI
- [ ] Configuração otimizada para terminais diversos

## 🚀 INSTRUÇÕES FINAIS PARA GEMINI CLI ATUALIZADAS

1. **SELEÇÃO NUMÉRICA UNIVERSAL**: NUNCA peça para o usuário digitar nomes quando há opções pré-definidas - SEMPRE use numeração
2. **ESTRUTURA ADAPTATIVA**: Analise o contexto da aplicação e escolha automaticamente entre menu direto e agrupado
3. **GRUPOS INTELIGENTES**: Use agrupamento apenas quando a aplicação realmente tem múltiplas categorias populadas
4. **ANÁLISE DE EMOJIS INTELIGENTE**: Faça esforço real para encontrar emojis apropriados, use a regra 80%
5. **PROGRESSO VISUAL**: Implemente barras de progresso para qualquer operação que possa demorar
6. **COMPATIBILIDADE CLI**: Configure adequadamente para funcionar tanto interativamente quanto via Gemini CLI
7. **TRATAMENTO ROBUSTO**: Trate todas as possíveis interrupções e erros graciosamente
8. **FEEDBACK CONSISTENTE**: Use o sistema de status padronizado para todas as operações
9. **NAVEGAÇÃO INTUITIVA**: Sempre permita voltar com 0/ESC/Ctrl+C
10. **ELIMINAÇÃO DE RUÍDO**: Remova qualquer texto que não adicione valor informativo real
11. **TEMPLATE UNIVERSAL**: Use `main_application_template()` como base para máxima adaptabilidade

**ESTAS DIRETRIZES SÃO OBRIGATÓRIAS E GARANTEM MÁXIMA AUTONOMIA DO GEMINI CLI NA CONSTRUÇÃO DE INTERFACES CONSISTENTES, ADAPTATIVAS E PROFISSIONAIS QUE SE AJUSTAM AUTOMATICAMENTE AO CONTEXTO DA APLICAÇÃO.**

**PRINCIPAIS MUDANÇAS IMPLEMENTADAS:**
- ✅ Sistema de seleção numérica obrigatório para todas as opções múltiplas
- ✅ Estrutura de menu adaptativa que não força templates inadequados
- ✅ Detecção inteligente entre menu direto vs agrupado baseado no contexto
- ✅ Eliminação de textos redundantes como "digite o número"
- ✅ Template universal que se adapta a qualquer estrutura de aplicação
```