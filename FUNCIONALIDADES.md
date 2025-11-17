# Tagarela - Funcionalidades Completas

## 🎯 Visão Geral
Aplicativo Vue.js 2 para conversão de texto em fala (TTS) com processamento de documentos PDF e Word.

---

## 📋 Funcionalidades Principais

### 1. **Sistema de Abas**
- 📄 **Upload de Arquivo**: Upload de PDF e Word com drag & drop
- ✍️ **Digitar Texto**: Área de texto para colar ou digitar diretamente
- Interface intuitiva com alternância entre modos

### 2. **Processamento de Documentos**
#### PDF
- Extração completa de texto usando `pdfjs-dist`
- Processamento página por página
- Normalização automática de encoding
- Debug detalhado de cada etapa

#### Word (.doc e .docx)
- Extração de texto usando `mammoth`
- Suporte a formatação
- Normalização automática de encoding

#### Normalização de Texto
Corrige automaticamente caracteres problemáticos:
- `Ɵ` → `ti`
- `İ` → `fí`
- `ƟƟ` → `tt`
- Ligaduras: `ﬁ` → `fi`, `ﬂ` → `fl`
- Aspas e travessões especiais
- Espaços múltiplos

### 3. **Text-to-Speech (TTS)**

#### Web Speech API (Método Principal)
- **Nativa do navegador** - funciona offline
- **Seleção de vozes**:
  - Vozes Premium/Melhoradas (Google, Enhanced, Natural)
  - Vozes Padrão do sistema
  - Filtradas por idioma
  - Indicação se é Local ou Online
- **Controles avançados**:
  - Velocidade: 0.5x a 2x
  - Tom (Pitch): 0.5x a 2x
  - Volume fixo em 100%
- **Seleção automática** de vozes premium quando disponível

#### Google TTS (Fallback)
- Usado quando Web Speech API não está disponível
- Divide texto em chunks automaticamente
- Reprodução sequencial

### 4. **Controles de Reprodução**

#### Timer com Progresso
- ⏰ Tempo decorrido (MM:SS)
- ⏳ Duração estimada
- 📊 Barra de progresso visual
- 🎯 Porcentagem de conclusão
- Cálculo baseado em palavras e velocidade

#### Botões de Controle
- **Play/Pause**: Único botão que alterna
  - ▶️ Verde quando parado
  - ⏸️ Laranja quando tocando (com animação pulsante)
  - Retoma de onde parou
- **Stop**: Para completamente e reseta timer
  - Desabilita erro "canceled"

### 5. **Console de Debug**
Sistema completo de logging que mostra:
- ✅ Operações bem-sucedidas (verde)
- ℹ️ Informações (azul)
- ⚠️ Avisos (laranja)
- ❌ Erros (vermelho)

**Informações registradas:**
- Upload e validação de arquivos
- Processamento página por página (PDF)
- Extração de texto (Word)
- Normalização de caracteres
- Carregamento de vozes
- Configuração de TTS
- Eventos de reprodução
- Erros com stack trace

**Recursos:**
- Timestamp em cada log
- Dados estruturados expandíveis
- Limit

e de 50 logs (auto-limpeza)
- Botão para limpar manualmente
- Scroll automático

### 6. **Interface e Design**

#### Design Minimalista
- Gradiente roxo no header
- Cards brancos com sombras suaves
- Animações e transições suaves
- Tipografia moderna (sistema de fontes)

#### Responsividade
- Layout adaptativo para mobile
- Controles reorganizados em telas pequenas
- Abas verticais no mobile
- Texto e botões otimizados

#### Feedback Visual
- Drag & drop com indicação visual
- Estados de botões (disabled, hover, active)
- Barra de progresso animada
- Animação pulsante no botão play
- Ícones SVG em todos os controles

### 7. **Tratamento de Erros**

#### Mensagens de Erro
- Título do erro em destaque
- Detalhes técnicos (quando disponível)
- Sugestões de correção
- Não mostra erros normais (canceled, interrupted)

#### Validações
- Tipo de arquivo (PDF, .doc, .docx)
- Tamanho do arquivo
- Conteúdo do texto
- Disponibilidade de vozes

---

## 🛠️ Tecnologias Utilizadas

### Core
- **Vue.js 2.6.14** - Framework JavaScript
- **Vite 5.0.0** - Build tool ultra-rápido
- **@vitejs/plugin-vue2** - Plugin oficial Vue 2

### Processamento
- **pdfjs-dist** - Extração de texto de PDFs
- **mammoth** - Extração de texto de Word

### TTS
- **Web Speech API** - TTS nativo do navegador (principal)
- **Google Translate TTS** - Fallback via URL direta

---

## 📊 Idiomas Suportados

- 🇧🇷 Português (Brasil)
- 🇵🇹 Português (Portugal)
- 🇺🇸 English (US)
- 🇬🇧 English (UK)
- 🇪🇸 Español
- 🇫🇷 Français

As vozes disponíveis variam conforme o sistema operacional.

---

## 🎨 Destaques de UX

1. **Fluxo Intuitivo**: Upload → Processar → Ouvir
2. **Feedback Constante**: Timer, progresso, logs
3. **Personalização**: Voz, velocidade, tom
4. **Acessibilidade**: Ícones + texto, cores contrastantes
5. **Performance**: Processamento assíncrono, chunking automático
6. **Confiabilidade**: Debug completo, fallbacks, validações

---

## 🚀 Como Usar

### Instalação
```bash
npm install
```

### Desenvolvimento
```bash
npm run dev
```
Abre em `http://localhost:8080`

### Build para Produção
```bash
npm run build
```
Gera pasta `dist/`

---

## 💡 Dicas de Uso

### Para voz mais natural:
1. Selecione vozes "Premium", "Enhanced" ou "Google"
2. Ajuste velocidade entre 1.0x e 1.2x
3. Ajuste tom entre 0.9x e 1.1x
4. Vozes "Online" geralmente são melhores

### Para PDFs com encoding ruim:
- O sistema corrige automaticamente
- Verifique os logs de normalização
- Edite manualmente se necessário

### Para textos longos:
- Web Speech API divide automaticamente
- Timer mostra progresso estimado
- Use pause/resume livremente

---

## 📝 Estrutura do Projeto

```
tagarela/
├── src/
│   ├── App.vue          # Componente principal
│   ├── main.js          # Entry point
│   └── styles.css       # Estilos globais
├── public/
│   └── (vazio - index.html na raiz)
├── index.html           # Template HTML
├── vite.config.js       # Configuração Vite
├── package.json         # Dependências
└── README.md           # Documentação básica
```

---

## 🐛 Debug e Troubleshooting

### Console de Debug mostra:
- Status de cada operação
- Erros com stack trace
- Informações de arquivos
- Configurações de TTS
- Eventos de reprodução

### Problemas comuns:
1. **Vozes não aparecem**: Aguarde alguns segundos, elas carregam após o mount
2. **Erro "canceled"**: Corrigido - agora é silencioso
3. **PDF com texto estranho**: Normalização automática aplicada
4. **Áudio não toca**: Verifique se o navegador suporta Web Speech API

---

## ✨ Características Técnicas

- ES Modules
- Async/Await
- Computed Properties
- Lifecycle Hooks
- Event Handling
- Reactive Data
- CSS Grid & Flexbox
- Media Queries
- SVG Icons
- Progress Tracking
- Timer Intervals
- File Reading
- ArrayBuffer Processing

---

Desenvolvido com ❤️ usando Vue.js 2 e Vite

