# Roteiro de Build e Instalação — Nice Code (VS Code Extension)

Este roteiro descreve o processo completo para preparar o ambiente, buildar e instalar a extensão `ibict.nice-code` em uma máquina nova. Siga os passos na ordem indicada.

---

## Pré-requisitos

- Node.js >= 20.x (recomendado via nvm ou asdf)
- Python 3.x disponível no PATH (necessário para alguns scripts)
- VS Code instalado com o comando `code` disponível no terminal
- Acesso à internet (para baixar binários pré-compilados)

Verifique:
```bash
node --version    # >= 20
python3 --version # qualquer 3.x
code --version
```

---

## Estrutura do Monorepo

O projeto é um monorepo. A extensão VS Code está em `extensions/vscode/` mas depende de:

- `core/` — lógica principal
- `gui/` — interface React (compilada para `gui/dist/`)
- `packages/config-types/` — tipos de configuração
- `packages/config-yaml/` — parser de config YAML
- `packages/terminal-security/` — segurança de terminal
- `packages/openai-adapters/` — adaptadores OpenAI
- `packages/fetch/` — utilitário de fetch
- `packages/llm-info/` — informações de modelos LLM

---

## Passo 1 — Instalar dependências do `core`

O `core` contém binários nativos (`onnxruntime-node`, `sqlite3`). Use `--ignore-scripts` para evitar compilação nativa (o sqlite3 será baixado pré-compilado depois):

```bash
cd core
npm install --ignore-scripts
cd ..
```

> **Por quê `--ignore-scripts`?** O pacote `sqlite3` precisa de Python com `distutils` para compilar. No Python 3.12+ `distutils` foi removido. O script de prepackage baixa um binário pré-compilado do sqlite3, então a compilação local não é necessária.

---

## Passo 2 — Buildar os pacotes locais do monorepo

A GUI e a extensão dependem destes pacotes. Devem ser buildados **nesta ordem** (há dependência entre eles):

```bash
# 1. config-types (base para config-yaml)
cd packages/config-types
npm install --ignore-scripts
npm run build
cd ../..

# 2. config-yaml (depende de config-types)
cd packages/config-yaml
npm install --ignore-scripts
npm run build
cd ../..

# 3. Demais pacotes (independentes entre si)
cd packages/terminal-security && npm install --ignore-scripts && npm run build && cd ../..
cd packages/openai-adapters   && npm install --ignore-scripts && npm run build && cd ../..
cd packages/fetch             && npm install --ignore-scripts && npm run build && cd ../..
cd packages/llm-info          && npm install --ignore-scripts && npm run build && cd ../..
```

---

## Passo 3 — Buildar a GUI (React)

```bash
cd gui
npm install
npm run build
cd ..
```

O resultado fica em `gui/dist/`. Verifique que `gui/dist/assets/index.js` e `gui/dist/assets/index.css` existem antes de continuar.

---

## Passo 4 — Instalar dependências da extensão VS Code

```bash
cd extensions/vscode
npm install
```

---

## Passo 5 — Binário do ripgrep (`rg`)

O pacote `@vscode/ripgrep` tenta baixar o binário `rg` do GitHub durante o `postinstall`. Se a rede bloquear o acesso ao GitHub, copie o binário local:

```bash
# Verificar se o binário já foi baixado
ls extensions/vscode/node_modules/@vscode/ripgrep/bin/rg

# Se não existir e o GitHub estiver acessível:
node extensions/vscode/node_modules/@vscode/ripgrep/lib/postinstall.js

# Se o GitHub NÃO estiver acessível (rede restrita), use o ripgrep local:
which rg   # confirmar que existe (ex: /opt/homebrew/bin/rg no macOS)
mkdir -p extensions/vscode/node_modules/@vscode/ripgrep/bin
cp $(which rg) extensions/vscode/node_modules/@vscode/ripgrep/bin/rg
```

---

## Passo 6 — Buildar e empacotar a extensão

```bash
cd extensions/vscode
npm run package
```

Este comando executa em sequência:
1. `prepackage` — copia GUI, binários nativos (onnxruntime, sqlite3, lancedb, ripgrep) para os diretórios corretos
2. `vscode:prepublish` (esbuild com minify) — gera `out/extension.js`
3. `vsce package` — empacota tudo em `build/nice-code-1.3.40.vsix`

O arquivo `.vsix` gerado fica em `extensions/vscode/build/`.

---

## Passo 7 — Instalar a extensão no VS Code

```bash
code --install-extension extensions/vscode/build/nice-code-1.3.40.vsix
```

> **Atenção:** Se a extensão original `Continue.continue` estiver instalada, ela vai conflitar com `ibict.nice-code` pois ambas registram os mesmos IDs de views e comandos (`continue.continueGUIView`, etc.). Remova-a antes:
> ```bash
> rm -rf ~/.vscode/extensions/continue.continue-*/
> ```

Após instalar, **reinicie o VS Code**.

---

## Bugs Conhecidos e Correções Aplicadas

### `getExtensionUri` com ID hardcoded

**Arquivo:** `extensions/vscode/src/util/vscode.ts` — linha 25

**Problema:** A função `getExtensionUri()` estava hardcoded com o ID da extensão original:
```typescript
// ERRADO — causa "An error occurred while loading view: continue.continueGUIView"
return vscode.extensions.getExtension("Continue.continue")!.extensionUri;
```

**Correção já aplicada:**
```typescript
// CORRETO
return vscode.extensions.getExtension("ibict.nice-code")!.extensionUri;
```

Se o ID da extensão mudar (publisher ou name no `package.json`), esta linha deve ser atualizada para corresponder ao formato `"publisher.name"`.

---

## Solução de Problemas

| Erro | Causa | Solução |
|------|-------|---------|
| `gyp ERR! find Python` | Python não encontrado para compilar sqlite3 | Use `npm install --ignore-scripts` no `core/` |
| `No module named 'distutils'` | Python 3.12+ removeu distutils | Mesmo que acima |
| `File node_modules/@vscode/ripgrep/bin/rg does not exist` | Binário rg não baixado | Ver Passo 5 |
| `Cannot find module '@continuedev/config-yaml'` | Pacotes locais não buildados | Executar Passo 2 |
| `gui build did not produce index.js` | GUI não buildada | Executar Passo 3 |
| `An error occurred while loading view: continue.continueGUIView` | `getExtensionUri` com ID errado ou extensão Continue conflitando | Ver seção Bugs Conhecidos e remover `continue.continue-*` |
| `A view with id continue.continueGUIView is already registered` | Extensão `Continue.continue` instalada junto | `rm -rf ~/.vscode/extensions/continue.continue-*/` |
