<template>
  <div id="app">
    <div class="container">
      <header class="header">
        <h1 class="title">Tagarela</h1>
        <p class="subtitle">Converta documentos em fala (by Paulo Renan)</p>
      </header>

      <main class="main-content">
        <!-- Sistema de Abas -->
        <div class="tabs">
          <button 
            @click="activeTab = 'upload'" 
            :class="['tab', { 'active': activeTab === 'upload' }]">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor">
              <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path>
              <polyline points="17 8 12 3 7 8"></polyline>
              <line x1="12" y1="3" x2="12" y2="15"></line>
            </svg>
            Upload de Arquivo
          </button>
          <button 
            @click="activeTab = 'text'" 
            :class="['tab', { 'active': activeTab === 'text' }]">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor">
              <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path>
              <polyline points="14 2 14 8 20 8"></polyline>
              <line x1="16" y1="13" x2="8" y2="13"></line>
              <line x1="16" y1="17" x2="8" y2="17"></line>
              <polyline points="10 9 9 9 8 9"></polyline>
            </svg>
            Digitar Texto
          </button>
        </div>

        <!-- Aba: Upload de Arquivo -->
        <div v-if="activeTab === 'upload'" class="upload-section">
          <div class="upload-area" 
               :class="{ 'drag-over': isDragOver, 'has-file': selectedFile }"
               @dragover.prevent="handleDragOver"
               @dragleave.prevent="handleDragLeave"
               @drop.prevent="handleDrop">
            <input 
              type="file" 
              ref="fileInput" 
              @change="handleFileSelect" 
              accept=".pdf,.doc,.docx"
              class="file-input"
              id="file-input">
            <label for="file-input" class="upload-label">
              <svg class="upload-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor">
                <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path>
                <polyline points="17 8 12 3 7 8"></polyline>
                <line x1="12" y1="3" x2="12" y2="15"></line>
              </svg>
              <span v-if="!selectedFile">Arraste um arquivo aqui ou clique para selecionar</span>
              <span v-else class="file-name">{{ selectedFile.name }}</span>
              <span v-if="selectedFile" class="file-size">({{ formatFileSize(selectedFile.size) }})</span>
            </label>
          </div>

          <div v-if="selectedFile" class="actions">
            <button @click="processFile" :disabled="processing" class="btn btn-primary">
              <span v-if="!processing">Processar Documento</span>
              <span v-else>Processando...</span>
            </button>
            <button @click="clearFile" class="btn btn-secondary">Limpar</button>
          </div>
        </div>

        <!-- Aba: Digitar Texto -->
        <div v-if="activeTab === 'text'" class="text-input-section">
          <div class="text-input-header">
            <h3>Digite ou cole o texto aqui</h3>
            <button @click="clearTextInput" class="btn btn-secondary btn-small">Limpar</button>
          </div>
          <textarea 
            v-model="manualText" 
            class="text-input-area"
            placeholder="Digite ou cole seu texto aqui..."
            :rows="8">
          </textarea>
          <div class="actions">
            <button 
              @click="useManualText" 
              :disabled="!manualText || !manualText.trim()" 
              class="btn btn-primary">
              Usar Este Texto
            </button>
          </div>
        </div>

        <div v-if="extractedText" class="text-section">
          <div class="text-header">
            <h2>Texto Extraído</h2>
            <div class="text-actions">
              <button @click="copyText" class="btn btn-icon" title="Copiar texto">
                <svg viewBox="0 0 24 24" fill="none" stroke="currentColor">
                  <rect x="9" y="9" width="13" height="13" rx="2" ry="2"></rect>
                  <path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"></path>
                </svg>
              </button>
            </div>
          </div>
          <div class="text-content">
            <textarea 
              v-model="extractedText" 
              readonly 
              class="text-area"
              :rows="textRows">
            </textarea>
          </div>
        </div>

        <div v-if="extractedText" class="tts-section">
          <div class="tts-controls">
            <div class="control-group full-width">
              <label for="language">Idioma:</label>
              <select v-model="selectedLanguage" id="language" class="select" @change="loadVoices">
                <option value="pt-BR">Português (Brasil)</option>
                <option value="pt-PT">Português (Portugal)</option>
                <option value="en-US">English (US)</option>
                <option value="en-GB">English (UK)</option>
                <option value="es-ES">Español</option>
                <option value="fr-FR">Français</option>
              </select>
            </div>
            <div class="control-group full-width" v-if="availableVoices.length > 0">
              <label for="voice">Voz:</label>
              <select v-model="selectedVoice" id="voice" class="select">
                <option :value="null">Voz padrão do sistema</option>
                <optgroup v-for="(group, groupIndex) in groupedVoices" :key="`group-${groupIndex}`" :label="group.label">
                  <option 
                    v-for="(voice, voiceIndex) in group.voices" 
                    :key="`${groupIndex}-${voiceIndex}-${voice.name}`" 
                    :value="voice.name">
                    {{ voice.name }} {{ voice.localService ? '(Local)' : '(Online)' }}
                  </option>
                </optgroup>
              </select>
            </div>
            <div class="control-group">
              <label for="speed">Velocidade:</label>
              <input 
                type="range" 
                id="speed" 
                v-model="speed" 
                min="0.5" 
                max="2" 
                step="0.1"
                class="slider">
              <span class="speed-value">{{ speed }}x</span>
            </div>
            <div class="control-group">
              <label for="pitch">Tom:</label>
              <input 
                type="range" 
                id="pitch" 
                v-model="pitch" 
                min="0.5" 
                max="2" 
                step="0.1"
                class="slider">
              <span class="speed-value">{{ pitch }}x</span>
            </div>
          </div>
          
          <!-- Timer -->
          <div v-if="playing || elapsedTime > 0" class="timer-section">
            <div class="timer-display">
              <svg viewBox="0 0 24 24" fill="none" stroke="currentColor">
                <circle cx="12" cy="12" r="10"></circle>
                <polyline points="12 6 12 12 16 14"></polyline>
              </svg>
              <span class="timer-text">{{ formatTime(elapsedTime) }}</span>
              <span v-if="estimatedDuration > 0" class="timer-total"> / {{ formatTime(estimatedDuration) }}</span>
            </div>
            <div class="progress-bar">
              <div class="progress-fill" :style="{ width: progressPercentage + '%' }"></div>
            </div>
          </div>

          <div class="playback-controls">
            <button 
              @click="playTTS" 
              :disabled="!extractedText"
              :class="['btn', 'btn-play', { 'btn-playing': playing && !paused }]">
              <svg v-if="!playing || paused" viewBox="0 0 24 24" fill="currentColor">
                <path d="M8 5v14l11-7z"/>
              </svg>
              <svg v-else viewBox="0 0 24 24" fill="currentColor">
                <path d="M6 4h4v16H6V4zm8 0h4v16h-4V4z"/>
              </svg>
              <span>{{ playing && !paused ? 'Pausar' : 'Reproduzir' }}</span>
            </button>
            <button 
              @click="stopTTS" 
              :disabled="!playing"
              class="btn btn-stop">
              <svg viewBox="0 0 24 24" fill="currentColor">
                <rect x="6" y="6" width="12" height="12"/>
              </svg>
              <span>Parar</span>
            </button>
          </div>
        </div>

        <div v-if="error" class="error-message">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor">
            <circle cx="12" cy="12" r="10"></circle>
            <line x1="12" y1="8" x2="12" y2="12"></line>
            <line x1="12" y1="16" x2="12.01" y2="16"></line>
          </svg>
          <div class="error-content">
            <span class="error-title">{{ error }}</span>
            <span v-if="errorDetails" class="error-details">{{ errorDetails }}</span>
          </div>
        </div>

        <!-- Debug Console (desenvolvimento) -->
        <div v-if="debugLogs.length > 0 && debug" class="debug-console">
          <div class="debug-header">
            <h3>Console de Debug</h3>
            <button @click="clearDebugLogs" class="btn btn-icon btn-small">Limpar</button>
          </div>
          <div class="debug-logs">
            <div 
              v-for="(log, index) in debugLogs" 
              :key="index" 
              :class="['debug-log', `debug-${log.type}`]">
              <span class="debug-time">{{ log.time }}</span>
              <span class="debug-message">{{ log.message }}</span>
              <pre v-if="log.data" class="debug-data">{{ JSON.stringify(log.data, null, 2) }}</pre>
            </div>
          </div>
        </div>
      </main>
      
      <footer class="footer">
        <div class="footer-content">
          <p class="footer-text">
            Desenvolvido por 
            <a 
              href="https://www.linkedin.com/in/rennanalmeida/" 
              target="_blank" 
              rel="noopener noreferrer"
              class="footer-link">
              <svg viewBox="0 0 24 24" fill="currentColor" class="linkedin-icon">
                <path d="M19 3a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h14m-.5 15.5v-5.3a3.26 3.26 0 0 0-3.26-3.26c-.85 0-1.84.52-2.32 1.3v-1.11h-2.79v8.37h2.79v-4.93c0-.77.62-1.4 1.39-1.4a1.4 1.4 0 0 1 1.4 1.4v4.93h2.79M6.88 8.56a1.68 1.68 0 0 0 1.68-1.68c0-.93-.75-1.69-1.68-1.69a1.69 1.69 0 0 0-1.69 1.69c0 .93.76 1.68 1.69 1.68m1.39 9.94v-8.37H5.5v8.37h2.77z"/>
              </svg>
              Paulo Renan Almeida
            </a>
          </p>
          <p class="footer-year">© {{ new Date().getFullYear() }} - Todos os direitos reservados</p>
        </div>
      </footer>
    </div>
  </div>
</template>

<script>
import * as pdfjsLib from 'pdfjs-dist';
import mammoth from 'mammoth';

// Configurar worker do PDF.js
if (typeof window !== 'undefined') {
  pdfjsLib.GlobalWorkerOptions.workerSrc = `https://cdnjs.cloudflare.com/ajax/libs/pdf.js/${pdfjsLib.version}/pdf.worker.min.js`;
}

// Função para TTS usando Web Speech API ou Google TTS como fallback
function getGoogleTTSUrl(text, lang) {
  // URL da API do Google Translate TTS
  const baseUrl = 'https://translate.google.com/translate_tts';
  const params = new URLSearchParams({
    ie: 'UTF-8',
    q: text,
    tl: lang,
    client: 'tw-ob'
  });
  return `${baseUrl}?${params.toString()}`;
}

export default {
  name: 'App',
  data() {
    return {
      activeTab: 'upload',
      selectedFile: null,
      extractedText: '',
      manualText: '',
      processing: false,
      isDragOver: false,
      error: null,
      errorDetails: null,
      selectedLanguage: 'pt-BR',
      selectedVoice: null,
      availableVoices: [],
      speed: 1.0,
      pitch: 1.0,
      playing: false,
      paused: false,
      currentAudio: null,
      currentUtterance: null,
      textRows: 10,
      debugLogs: [],
      debug: false,
      elapsedTime: 0,
      estimatedDuration: 0,
      timerInterval: null,
      startTime: null
    };
  },
  computed: {
    groupedVoices() {
      const filtered = this.availableVoices.filter(voice => {
        return voice.lang.startsWith(this.selectedLanguage.substring(0, 2));
      });
      
      // Agrupar por qualidade
      const premium = filtered.filter(v => 
        v.name.includes('Premium') || 
        v.name.includes('Enhanced') || 
        v.name.includes('Google') ||
        v.name.includes('Natural') ||
        !v.localService
      );
      
      const standard = filtered.filter(v => 
        !premium.includes(v) && v.localService
      );
      
      const groups = [];
      
      if (premium.length > 0) {
        groups.push({
          label: 'Vozes Premium/Melhoradas',
          voices: premium
        });
      }
      
      if (standard.length > 0) {
        groups.push({
          label: 'Vozes Padrão',
          voices: standard
        });
      }
      
      return groups;
    },
    progressPercentage() {
      if (this.estimatedDuration === 0) return 0;
      return Math.min((this.elapsedTime / this.estimatedDuration) * 100, 100);
    }
  },
  mounted() {
    this.loadVoices();
    // Carregar vozes quando elas estiverem disponíveis
    if ('speechSynthesis' in window) {
      window.speechSynthesis.onvoiceschanged = () => {
        this.loadVoices();
      };
    }
  },
  methods: {
    loadVoices() {
      if ('speechSynthesis' in window) {
        const voices = window.speechSynthesis.getVoices();
        this.availableVoices = voices;
        
        this.addDebugLog('Vozes carregadas', 'info', {
          totalVoices: voices.length,
          languageVoices: voices.filter(v => v.lang.startsWith(this.selectedLanguage.substring(0, 2))).length
        });
        
        // Tentar selecionar automaticamente uma voz premium/melhorada
        if (!this.selectedVoice && voices.length > 0) {
          const langPrefix = this.selectedLanguage.substring(0, 2);
          const premiumVoice = voices.find(v => 
            v.lang.startsWith(langPrefix) && 
            (v.name.includes('Premium') || v.name.includes('Enhanced') || v.name.includes('Google') || !v.localService)
          );
          
          if (premiumVoice) {
            this.selectedVoice = premiumVoice.name;
            this.addDebugLog('Voz premium selecionada automaticamente', 'success', {
              voiceName: premiumVoice.name
            });
          }
        }
      }
    },
    addDebugLog(message, type = 'info', data = null) {
      const time = new Date().toLocaleTimeString();
      this.debugLogs.push({ time, message, type, data });
      console.log(`[${time}] [${type.toUpperCase()}] ${message}`, data || '');
      // Manter apenas os últimos 50 logs
      if (this.debugLogs.length > 50) {
        this.debugLogs.shift();
      }
    },
    clearDebugLogs() {
      this.debugLogs = [];
    },
    handleDragOver(e) {
      this.isDragOver = true;
      this.addDebugLog('Drag over detectado', 'info');
    },
    handleDragLeave(e) {
      this.isDragOver = false;
      this.addDebugLog('Drag leave detectado', 'info');
    },
    handleDrop(e) {
      this.isDragOver = false;
      const files = e.dataTransfer.files;
      this.addDebugLog(`Arquivo solto: ${files.length} arquivo(s)`, 'info', { 
        fileCount: files.length,
        fileNames: Array.from(files).map(f => f.name)
      });
      if (files.length > 0) {
        this.selectFile(files[0]);
      }
    },
    handleFileSelect(e) {
      const files = e.target.files;
      this.addDebugLog(`Arquivo selecionado: ${files.length} arquivo(s)`, 'info', {
        fileCount: files.length,
        fileNames: Array.from(files).map(f => f.name)
      });
      if (files.length > 0) {
        this.selectFile(files[0]);
      }
    },
    selectFile(file) {
      this.addDebugLog('Iniciando validação do arquivo', 'info', {
        fileName: file.name,
        fileSize: file.size,
        fileType: file.type
      });

      const validTypes = [
        'application/pdf',
        'application/msword',
        'application/vnd.openxmlformats-officedocument.wordprocessingml.document'
      ];
      
      if (!validTypes.includes(file.type)) {
        this.addDebugLog('Tipo de arquivo inválido', 'error', {
          fileName: file.name,
          fileType: file.type,
          validTypes
        });
        this.error = 'Por favor, selecione um arquivo PDF ou Word (.pdf, .doc, .docx)';
        this.errorDetails = `Tipo recebido: ${file.type || 'desconhecido'}`;
        return;
      }
      
      this.addDebugLog('Arquivo validado com sucesso', 'success', {
        fileName: file.name,
        fileSize: file.size,
        fileType: file.type
      });
      
      this.selectedFile = file;
      this.extractedText = '';
      this.error = null;
      this.errorDetails = null;
    },
    async processFile() {
      if (!this.selectedFile) {
        this.addDebugLog('Tentativa de processar sem arquivo selecionado', 'warning');
        return;
      }
      
      this.addDebugLog('Iniciando processamento do arquivo', 'info', {
        fileName: this.selectedFile.name,
        fileSize: this.selectedFile.size,
        fileType: this.selectedFile.type
      });
      
      this.processing = true;
      this.error = null;
      this.errorDetails = null;
      this.extractedText = '';
      
      try {
        const fileType = this.selectedFile.type;
        this.addDebugLog('Tipo de arquivo identificado', 'info', { fileType });
        
        if (fileType === 'application/pdf') {
          this.addDebugLog('Processando como PDF', 'info');
          await this.processPDF();
        } else if (fileType === 'application/msword' || 
                   fileType === 'application/vnd.openxmlformats-officedocument.wordprocessingml.document') {
          this.addDebugLog('Processando como Word', 'info');
          await this.processWord();
        } else {
          throw new Error(`Tipo de arquivo não suportado: ${fileType}`);
        }
        
        this.addDebugLog('Arquivo processado com sucesso', 'success', {
          textLength: this.extractedText.length,
          textPreview: this.extractedText.substring(0, 100) + '...'
        });
      } catch (error) {
        this.addDebugLog('Erro ao processar arquivo', 'error', {
          errorMessage: error.message,
          errorStack: error.stack,
          errorName: error.name
        });
        console.error('Erro ao processar arquivo:', error);
        this.error = 'Erro ao processar o arquivo. Por favor, tente novamente.';
        this.errorDetails = `Detalhes: ${error.message || 'Erro desconhecido'}`;
      } finally {
        this.processing = false;
        this.addDebugLog('Processamento finalizado', 'info');
      }
    },
    normalizeText(text) {
      this.addDebugLog('Normalizando texto extraído', 'info', {
        originalLength: text.length
      });
      
      // Mapa de correções de caracteres problemáticos comuns em PDFs
      const charMap = {
        'Ɵ': 'ti',
        'ƟƟ': 'tt',
        'İ': 'fí',
        'Ə': 'e',
        'ı': 'i',
        'ﬁ': 'fi',
        'ﬂ': 'fl',
        'ﬀ': 'ff',
        'ﬃ': 'ffi',
        'ﬄ': 'ffl',
        '': 'ti',
        '': 'fi',
        '–': '-',
        '—': '-',
        '\'': "\'",
        '\'': "\'",
        '"': '"',
        '"': '"',
        '…': '...',
        '‐': '-',
        '‑': '-',
        '‒': '-',
        '―': '-'
      };
      
      let normalizedText = text;
      let replacementCount = 0;
      
      // Aplicar substituições
      // for (const [wrong, correct] of Object.entries(charMap)) {
      //   const count = (normalizedText.match(new RegExp(wrong, 'g')) || []).length;
      //   if (count > 0) {
      //     normalizedText = normalizedText.replace(new RegExp(wrong, 'g'), correct);
      //     replacementCount += count;
      //     this.addDebugLog(`Corrigido: "${wrong}" → "${correct}"`, 'info', {
      //       occurrences: count
      //     });
      //   }
      // }
      
      // Normalizar espaços múltiplos
      normalizedText = normalizedText.replace(/ {2,}/g, ' ');
      
      // Normalizar quebras de linha múltiplas
      normalizedText = normalizedText.replace(/\n{3,}/g, '\n\n');
      
      this.addDebugLog('Normalização concluída', 'success', {
        totalReplacements: replacementCount,
        finalLength: normalizedText.length
      });
      
      return normalizedText;
    },
    async processPDF() {
      try {
        this.addDebugLog('Convertendo arquivo para ArrayBuffer', 'info');
        const arrayBuffer = await this.fileToArrayBuffer(this.selectedFile);
        this.addDebugLog('ArrayBuffer criado', 'success', {
          bufferSize: arrayBuffer.byteLength
        });
        
        this.addDebugLog('Carregando documento PDF', 'info');
        const loadingTask = pdfjsLib.getDocument({ data: arrayBuffer });
        const pdf = await loadingTask.promise;
        this.addDebugLog('PDF carregado', 'success', {
          numPages: pdf.numPages
        });
        
        let fullText = '';
        for (let i = 1; i <= pdf.numPages; i++) {
          this.addDebugLog(`Processando página ${i} de ${pdf.numPages}`, 'info');
          try {
            const page = await pdf.getPage(i);
            const textContent = await page.getTextContent();
            const pageText = textContent.items.map(item => item.str).join(' ');
            fullText += pageText + '\n\n';
            this.addDebugLog(`Página ${i} processada`, 'success', {
              textLength: pageText.length,
              itemsCount: textContent.items.length
            });
          } catch (pageError) {
            this.addDebugLog(`Erro ao processar página ${i}`, 'error', {
              error: pageError.message
            });
            throw pageError;
          }
        }
        
        // Normalizar o texto antes de atribuir
        const rawText = fullText.trim();
        this.extractedText = this.normalizeText(rawText);
        this.updateTextRows();
        this.addDebugLog('PDF processado completamente', 'success', {
          totalTextLength: this.extractedText.length
        });
      } catch (error) {
        this.addDebugLog('Erro no processamento do PDF', 'error', {
          errorMessage: error.message,
          errorStack: error.stack
        });
        throw error;
      }
    },
    async processWord() {
      try {
        this.addDebugLog('Convertendo arquivo Word para ArrayBuffer', 'info');
        const arrayBuffer = await this.fileToArrayBuffer(this.selectedFile);
        this.addDebugLog('ArrayBuffer criado', 'success', {
          bufferSize: arrayBuffer.byteLength
        });
        
        this.addDebugLog('Extraindo texto do Word usando Mammoth', 'info');
        const result = await mammoth.extractRawText({ arrayBuffer });
        this.addDebugLog('Texto extraído do Word', 'success', {
          textLength: result.value.length,
          messagesCount: result.messages.length
        });
        
        // Normalizar o texto antes de atribuir
        const rawText = result.value.trim();
        this.extractedText = this.normalizeText(rawText);
        this.updateTextRows();
        
        if (result.messages.length > 0) {
          this.addDebugLog('Avisos ao processar Word', 'warning', {
            messages: result.messages
          });
          console.warn('Avisos ao processar Word:', result.messages);
        }
        
        this.addDebugLog('Word processado completamente', 'success', {
          totalTextLength: this.extractedText.length
        });
      } catch (error) {
        this.addDebugLog('Erro no processamento do Word', 'error', {
          errorMessage: error.message,
          errorStack: error.stack
        });
        throw error;
      }
    },
    fileToArrayBuffer(file) {
      return new Promise((resolve, reject) => {
        this.addDebugLog('Iniciando leitura do arquivo com FileReader', 'info');
        const reader = new FileReader();
        reader.onload = () => {
          this.addDebugLog('FileReader: arquivo lido com sucesso', 'success', {
            resultType: reader.result.constructor.name,
            resultSize: reader.result.byteLength
          });
          resolve(reader.result);
        };
        reader.onerror = (error) => {
          this.addDebugLog('FileReader: erro ao ler arquivo', 'error', {
            error: error.toString()
          });
          reject(error);
        };
        reader.onprogress = (event) => {
          if (event.lengthComputable) {
            const percent = Math.round((event.loaded / event.total) * 100);
            this.addDebugLog(`FileReader: progresso ${percent}%`, 'info', {
              loaded: event.loaded,
              total: event.total
            });
          }
        };
        reader.readAsArrayBuffer(file);
      });
    },
    updateTextRows() {
      const lines = this.extractedText.split('\n').length;
      this.textRows = Math.min(Math.max(lines, 5), 20);
    },
    formatFileSize(bytes) {
      if (bytes === 0) return '0 Bytes';
      const k = 1024;
      const sizes = ['Bytes', 'KB', 'MB', 'GB'];
      const i = Math.floor(Math.log(bytes) / Math.log(k));
      return Math.round(bytes / Math.pow(k, i) * 100) / 100 + ' ' + sizes[i];
    },
    copyText() {
      navigator.clipboard.writeText(this.extractedText).then(() => {
        alert('Texto copiado para a área de transferência!');
      }).catch(err => {
        console.error('Erro ao copiar:', err);
      });
    },
    formatTime(seconds) {
      const mins = Math.floor(seconds / 60);
      const secs = Math.floor(seconds % 60);
      return `${mins}:${secs.toString().padStart(2, '0')}`;
    },
    startTimer() {
      this.stopTimer();
      this.startTime = Date.now() - (this.elapsedTime * 1000);
      this.timerInterval = setInterval(() => {
        if (!this.paused) {
          this.elapsedTime = (Date.now() - this.startTime) / 1000;
        }
      }, 100);
    },
    stopTimer() {
      if (this.timerInterval) {
        clearInterval(this.timerInterval);
        this.timerInterval = null;
      }
    },
    resetTimer() {
      this.stopTimer();
      this.elapsedTime = 0;
      this.estimatedDuration = 0;
      this.startTime = null;
    },
    calculateEstimatedDuration() {
      // Estimar duração baseado no número de palavras e velocidade
      const words = this.extractedText.split(/\s+/).length;
      const wordsPerMinute = 150 * this.speed; // Média de palavras por minuto ajustada pela velocidade
      this.estimatedDuration = (words / wordsPerMinute) * 60;
      
      this.addDebugLog('Duração estimada calculada', 'info', {
        words,
        wordsPerMinute,
        estimatedSeconds: this.estimatedDuration
      });
    },
    async playTTS() {
      if (!this.extractedText) return;
      
      // Se já está tocando, pausar/retomar
      if (this.playing) {
        if ('speechSynthesis' in window && window.speechSynthesis.speaking) {
          if (this.paused) {
            // Retomar
            window.speechSynthesis.resume();
            this.paused = false;
            this.startTime = Date.now() - (this.elapsedTime * 1000);
            this.addDebugLog('Reprodução retomada', 'info');
          } else {
            // Pausar
            window.speechSynthesis.pause();
            this.paused = true;
            this.addDebugLog('Reprodução pausada', 'info');
          }
          return;
        } else if (this.currentAudio) {
          // Pausar/retomar áudio
          if (this.paused) {
            this.currentAudio.play();
            this.paused = false;
            this.startTime = Date.now() - (this.elapsedTime * 1000);
            this.addDebugLog('Reprodução retomada', 'info');
          } else {
            this.currentAudio.pause();
            this.paused = true;
            this.addDebugLog('Reprodução pausada', 'info');
          }
          return;
        }
      }
      
      try {
        this.error = null;
        this.paused = false;
        this.resetTimer();
        this.calculateEstimatedDuration();
        
        this.addDebugLog('Iniciando TTS', 'info', {
          textLength: this.extractedText.length,
          language: this.selectedLanguage,
          speed: this.speed
        });
        
        // Verificar se Web Speech API está disponível
        if ('speechSynthesis' in window) {
          this.addDebugLog('Usando Web Speech API (nativa do navegador)', 'info');
          this.playWithWebSpeechAPI();
        } else {
          this.addDebugLog('Web Speech API não disponível, usando Google TTS', 'warning');
          // Dividir texto em chunks menores (limite da API)
          const maxLength = 200;
          const chunks = this.splitTextIntoChunks(this.extractedText, maxLength);
          
          if (chunks.length === 0) return;
          
          this.playing = true;
          this.startTimer();
          await this.playChunks(chunks, 0);
        }
        
      } catch (error) {
        this.addDebugLog('Erro ao reproduzir TTS', 'error', {
          errorMessage: error.message,
          errorStack: error.stack
        });
        console.error('Erro ao reproduzir TTS:', error);
        this.error = 'Erro ao reproduzir áudio. Por favor, tente novamente.';
        this.errorDetails = error.message;
        this.playing = false;
        this.resetTimer();
      }
    },
    playWithWebSpeechAPI() {
      // Cancelar qualquer fala em andamento
      window.speechSynthesis.cancel();
      
      const utterance = new SpeechSynthesisUtterance(this.extractedText);
      
      // Configurar idioma
      utterance.lang = this.selectedLanguage;
      utterance.rate = parseFloat(this.speed);
      utterance.pitch = parseFloat(this.pitch);
      utterance.volume = 1;
      
      // Selecionar voz específica se escolhida
      if (this.selectedVoice) {
        const voices = window.speechSynthesis.getVoices();
        const voice = voices.find(v => v.name === this.selectedVoice);
        if (voice) {
          utterance.voice = voice;
          this.addDebugLog('Voz selecionada', 'info', {
            voiceName: voice.name,
            voiceLang: voice.lang,
            isLocal: voice.localService
          });
        }
      }
      
      this.addDebugLog('Configurando Web Speech API', 'info', {
        lang: utterance.lang,
        rate: utterance.rate,
        pitch: utterance.pitch,
        voiceName: utterance.voice ? utterance.voice.name : 'padrão',
        textLength: this.extractedText.length
      });
      
      utterance.onstart = () => {
        this.playing = true;
        this.paused = false;
        this.startTimer();
        this.addDebugLog('Reprodução iniciada', 'success');
      };
      
      utterance.onend = () => {
        this.playing = false;
        this.paused = false;
        this.stopTimer();
        this.addDebugLog('Reprodução finalizada', 'success');
      };
      
      utterance.onerror = (event) => {
        // Ignorar erro "canceled" que acontece ao clicar em parar
        if (event.error === 'canceled' || event.error === 'interrupted') {
          this.addDebugLog('Reprodução cancelada pelo usuário', 'info', {
            error: event.error
          });
          this.playing = false;
          this.paused = false;
          this.stopTimer();
          return;
        }
        
        this.playing = false;
        this.paused = false;
        this.stopTimer();
        this.addDebugLog('Erro na Web Speech API', 'error', {
          error: event.error,
          errorMessage: event.message
        });
        this.error = `Erro ao reproduzir áudio: ${event.error}`;
        this.errorDetails = event.message || 'Erro desconhecido';
      };
      
      utterance.onpause = () => {
        this.paused = true;
        this.addDebugLog('Reprodução pausada via evento', 'info');
      };
      
      utterance.onresume = () => {
        this.paused = false;
        this.addDebugLog('Reprodução retomada via evento', 'info');
      };
      
      this.currentUtterance = utterance;
      window.speechSynthesis.speak(utterance);
    },
    splitTextIntoChunks(text, maxLength) {
      const sentences = text.match(/[^.!?]+[.!?]+/g) || [text];
      const chunks = [];
      let currentChunk = '';
      
      for (const sentence of sentences) {
        if ((currentChunk + sentence).length <= maxLength) {
          currentChunk += sentence;
        } else {
          if (currentChunk) chunks.push(currentChunk.trim());
          if (sentence.length <= maxLength) {
            currentChunk = sentence;
          } else {
            // Se a sentença for muito longa, dividir por palavras
            const words = sentence.split(' ');
            for (const word of words) {
              if ((currentChunk + ' ' + word).length <= maxLength) {
                currentChunk += ' ' + word;
              } else {
                if (currentChunk) chunks.push(currentChunk.trim());
                currentChunk = word;
              }
            }
          }
        }
      }
      
      if (currentChunk) chunks.push(currentChunk.trim());
      return chunks.filter(chunk => chunk.length > 0);
    },
    async playChunks(chunks, index) {
      if (index >= chunks.length) {
        this.playing = false;
        this.addDebugLog('Reprodução de todos os chunks concluída', 'success');
        return;
      }
      
      try {
        this.addDebugLog(`Gerando áudio para chunk ${index + 1} de ${chunks.length}`, 'info', {
          chunkText: chunks[index].substring(0, 50) + '...',
          language: this.selectedLanguage
        });
        
        // Usar Google TTS diretamente
        const url = getGoogleTTSUrl(chunks[index], this.selectedLanguage);
        
        this.addDebugLog('URL do áudio gerada', 'success', {
          url: url.substring(0, 100) + '...'
        });
        
        const audio = new Audio(url);
        audio.playbackRate = this.speed;
        
        audio.onended = () => {
          this.addDebugLog(`Chunk ${index + 1} reproduzido com sucesso`, 'success');
          this.playChunks(chunks, index + 1);
        };
        
        audio.onerror = (error) => {
          this.addDebugLog('Erro no áudio', 'error', {
            error: error.toString(),
            url: url.substring(0, 100)
          });
          console.error('Erro no áudio:', error);
          this.playing = false;
          this.error = 'Erro ao reproduzir áudio. Por favor, tente novamente.';
          this.errorDetails = 'Não foi possível carregar o áudio do Google TTS';
        };
        
        this.currentAudio = audio;
        this.addDebugLog('Iniciando reprodução do áudio', 'info');
        await audio.play();
        
      } catch (error) {
        this.addDebugLog('Erro ao reproduzir chunk', 'error', {
          errorMessage: error.message,
          errorStack: error.stack,
          chunkIndex: index
        });
        console.error('Erro ao reproduzir chunk:', error);
        this.playing = false;
        this.error = 'Erro ao reproduzir áudio. Por favor, tente novamente.';
        this.errorDetails = error.message;
      }
    },
    stopTTS() {
      this.addDebugLog('Parando reprodução', 'info');
      
      // Parar Web Speech API se estiver ativa
      if ('speechSynthesis' in window && window.speechSynthesis.speaking) {
        window.speechSynthesis.cancel();
        this.addDebugLog('Web Speech API cancelada', 'info');
      }
      
      // Parar áudio se estiver ativo
      if (this.currentAudio) {
        this.currentAudio.pause();
        this.currentAudio.currentTime = 0;
        this.currentAudio = null;
        this.addDebugLog('Áudio parado', 'info');
      }
      
      this.playing = false;
      this.paused = false;
      this.resetTimer();
    },
    clearFile() {
      this.addDebugLog('Limpando arquivo selecionado', 'info');
      this.selectedFile = null;
      this.extractedText = '';
      this.error = null;
      this.errorDetails = null;
      this.stopTTS();
      if (this.$refs.fileInput) {
        this.$refs.fileInput.value = '';
      }
    },
    useManualText() {
      if (!this.manualText || !this.manualText.trim()) {
        this.addDebugLog('Tentativa de usar texto vazio', 'warning');
        return;
      }
      
      this.addDebugLog('Usando texto manual', 'info', {
        textLength: this.manualText.length
      });
      
      this.extractedText = this.manualText.trim();
      this.updateTextRows();
      this.error = null;
      this.errorDetails = null;
      this.addDebugLog('Texto manual aplicado com sucesso', 'success', {
        textLength: this.extractedText.length
      });
    },
    clearTextInput() {
      this.addDebugLog('Limpando texto manual', 'info');
      this.manualText = '';
      this.extractedText = '';
      this.error = null;
      this.errorDetails = null;
    }
  }
};
</script>

