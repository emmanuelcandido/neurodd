# CURSO PROCESSOR - Guia Completo Para Uso

## 🚀 COMEÇAR A USAR (Para leigos)
1.  **Instalação:** Abra o terminal e execute:
    ```bash
    pip install -e .
    ```
    Isso instala o `curso-processor` no seu ambiente de desenvolvimento.

2.  **Primeira Configuração:** Execute o comando abaixo para iniciar o assistente de configuração. Ele criará os arquivos e diretórios necessários.
    ```bash
    neuro --setup
    ```

3.  **Iniciar a Interface Interativa:** Para usar o sistema interativamente, digite:
    ```bash
    neuro
    ```
    O aplicativo iniciará em modo interativo, apresentando um menu principal com todas as funcionalidades. Navegue usando os números das opções.

## ✅ COMO SABER SE ESTÁ FUNCIONANDO
-   **Configuração Inicial:** Após `neuro --setup`, verifique se a pasta `~/.curso_processor/` (ou `C:\Users\<user>\.curso_processor\` no Windows) foi criada com os arquivos `settings.yaml`, `apis.json` e `directories.json`.
-   **Interface CLI:** Ao executar `neuro`, o menu principal deve aparecer de forma interativa e responsiva, com opções numeradas e cores vibrantes.
-   **Processamento de Curso:** Após processar um curso, verifique se os arquivos de áudio (`assets/audios/`), textos (`assets/textos/`), `timestamps.md` e `cursos.xml` foram gerados corretamente na estrutura de diretórios do curso.

## ✨ PRINCÍPIOS DE DESIGN DA INTERFACE
A interface do NeuroDD CLI foi desenvolvida com foco em clareza, consistência e usabilidade, seguindo um padrão visual moderno e intuitivo:

-   **Paleta de Cores Nord**: Utiliza uma paleta de cores suaves e agradáveis para uma experiência visual confortável.
-   **Seleção Numérica**: Todas as interações de escolha são feitas através de números, tornando a navegação rápida e eficiente.
-   **Feedback Visual Claro**: Mensagens de status, progresso e erros são apresentadas de forma clara, com cores e ícones que facilitam a compreensão.
-   **Menus Adaptativos**: A estrutura do menu se adapta à complexidade da funcionalidade, apresentando opções de forma direta ou agrupada, conforme a necessidade.
-   **Eliminação de Ruídos**: Textos redundantes foram removidos para manter a interface limpa e focada na informação essencial.

## 🧪 COMO TESTAR FUNCIONALIDADES

Para executar os testes automatizados do projeto, use o `pytest`.

### Executar Todos os Testes

```bash
pytest tests/ -v --cov=curso_processor --cov-report=html
```
**Resultado esperado**: Todos os testes devem passar, e um relatório de cobertura HTML será gerado na pasta `htmlcov/`.

### Testes por Categoria
-   **Unitários**: `pytest tests/unit/ -v`
-   **Integração**: `pytest tests/integration/ -v`

## 🆘 RESOLUÇÃO DE PROBLEMAS COMUNS
### Problema: `neuro` comando não encontrado
**Solução**: Certifique-se de que você instalou o pacote em modo editável (`pip install -e .`) e que seu ambiente virtual está ativado (se estiver usando um).

### Problema: Erro de conexão com API de IA
**Solução**: Verifique suas chaves de API em `~/.curso_processor/config/apis.json`. Elas podem estar incorretas ou expiradas.

### Problema: Vídeos não convertem para áudio
**Solução**: Verifique se o `ffmpeg` está instalado e acessível no seu PATH. O `moviepy` (usado para conversão) depende dele.

## 🔧 CONFIGURAÇÕES NECESSÁRIAS
As configurações são gerenciadas automaticamente pelo comando `neuro --setup`. No entanto, você pode editar os arquivos manualmente em `~/.curso_processor/config/` (ou `C:\Users\<user>\.curso_processor\config\` no Windows):
-   `settings.yaml`: Configurações gerais do sistema.
-   `apis.json`: Suas chaves de API para serviços externos (IA, GitHub, etc.).
-   `directories.json`: Caminhos para diretórios de trabalho e saída.