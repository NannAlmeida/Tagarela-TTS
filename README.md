# Tagarela - Text to Speech

Aplicativo Vue.js 2 para conversão de texto em fala (TTS) com suporte para documentos PDF e Word.

## Funcionalidades

- 📄 Upload de arquivos PDF e Word (.doc, .docx)
- ✍️ Digitar ou colar texto diretamente (sem necessidade de arquivo)
- 📝 Extração automática de texto dos documentos
- 🔊 Conversão de texto em fala usando Web Speech API (nativa do navegador)
- 🌍 Suporte a múltiplos idiomas (Português, Inglês, Espanhol, Francês)
- ⚡ Controle de velocidade de reprodução (0.5x a 2x)
- 🐛 Console de debug para acompanhar o processamento
- 🎨 Design minimalista e moderno
- 📱 Interface responsiva
- 🎯 Sistema de abas para alternar entre upload e digitação

## Instalação

```bash
npm install
```

## Desenvolvimento

```bash
npm run dev
```

O aplicativo estará disponível em `http://localhost:8080`

## Build para Produção

```bash
npm run build
```

Os arquivos compilados estarão na pasta `dist/`.

## Tecnologias Utilizadas

- **Vue.js 2** - Framework JavaScript
- **Web Speech API** - Para conversão de texto em fala (nativa do navegador)
- **pdfjs-dist** - Para processamento de arquivos PDF
- **mammoth** - Para processamento de arquivos Word
- **Vite** - Para build e desenvolvimento (ultra-rápido!)
- **Google TTS API** - Como fallback caso Web Speech API não esteja disponível

## Como Usar

### Opção 1: Upload de Arquivo
1. Clique na aba "Upload de Arquivo"
2. Arraste um arquivo PDF ou Word para a área de upload (ou clique para selecionar)
3. Clique em "Processar Documento" para extrair o texto
4. Ajuste o idioma e velocidade conforme necessário
5. Clique em "Reproduzir" para ouvir o texto

### Opção 2: Digitar Texto
1. Clique na aba "Digitar Texto"
2. Digite ou cole o texto desejado na área de texto
3. Clique em "Usar Este Texto"
4. Ajuste o idioma e velocidade conforme necessário
5. Clique em "Reproduzir" para ouvir o texto

## Debug

O aplicativo possui um console de debug integrado que aparece automaticamente quando há logs. Ele mostra:
- Todas as operações realizadas (upload, processamento, reprodução)
- Erros detalhados com stack trace
- Informações sobre arquivos processados
- Status de cada etapa do processamento

Isso facilita identificar problemas durante o uso.

## Notas

- **Web Speech API**: Funciona offline e é nativa do navegador. Suporta múltiplas vozes dependendo do sistema operacional
- **Google TTS**: Usado como fallback caso a Web Speech API não esteja disponível
- O processamento de PDFs pode demorar um pouco dependendo do tamanho do arquivo
- Para melhor experiência, use documentos com texto selecionável (não imagens escaneadas)
- O console de debug ajuda a identificar erros durante o processamento de arquivos

