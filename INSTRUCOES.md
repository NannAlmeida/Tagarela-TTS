# Instruções para executar o projeto

## Passo 1: Limpar instalação anterior
```bash
rm -rf node_modules package-lock.json .vite dist
```

## Passo 2: Instalar dependências
```bash
npm install
```

## Passo 3: Executar o projeto
```bash
npm run dev
```

## Se ainda houver erro

Verifique se o erro persiste e me informe a mensagem de erro completa.

## Mudanças realizadas

- Alterado de `vite-plugin-vue2` para `@vitejs/plugin-vue2` (plugin oficial)
- Atualizado Vite para versão 5.0.0
- Configuração ajustada para ES modules
- __dirname configurado corretamente para ES modules

