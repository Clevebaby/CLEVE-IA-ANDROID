# Guia de Exportação e Geração de APK Android (Capacitor)
**CLEVE IA – OTC SMART SCANNER / POCKET AI PRO OTC**

Este documento explica passo a passo como usar o código-fonte exportado em `.zip` para gerar um **APK Android instalável** com **Capacitor**, preservando todas as funcionalidades existentes (análise OTC 1M/5M, importação CSV, análise visual de capturas, histórico isolado por par/período e integração Telegram).

---

## 1. Arquitetura do Projeto

O projeto está dividido em duas camadas complementares:

1. **Frontend Web / Android WebView (`src/` + `index.html` → `dist/`)**:
   - React 19, TypeScript, Vite 8, Tailwind CSS v4, Lucide Icons.
   - Compilado para HTML/CSS/JS estático em `dist/` e empacotado no Android através do **Capacitor** (`capacitor.config.ts` + pasta `android/`).
2. **Backend Express (`server.ts`)**:
   - Endpoints REST `/api/*` (análise OTC, importação CSV, histórico de sinais e capturas, Telegram Bot API e análise visual de capturas via `@google/genai` com `GEMINI_API_KEY` protegida no servidor).
   - **Nota de Segurança**: A chave `GEMINI_API_KEY` nunca deve ser colocada dentro do APK. O APK comunica com este servidor Express através do `VITE_API_BASE_URL` (definido no build) ou configurado na própria app em **Configurações → Exportação e APK Android**.

---

## 2. Pré-requisitos para Compilar o APK

Num computador (Windows, macOS ou Linux) com o `.zip` extraído:

- **Node.js 20+** e **npm**
- **Java JDK 17 ou 21** (definido em `JAVA_HOME`)
- **Android Studio** instalado com **Android SDK** (API 34/35) e **Android SDK Platform-Tools** (`adb`)

---

## 3. Passo a Passo para Gerar o APK Android

### Passo 1 — Extrair e instalar dependências
No terminal, dentro da pasta extraída do `.zip`:

```bash
npm install
```

### Passo 2 — Definir o URL do servidor Backend (Opcional no build, recomendável para APK)
Se já tiver o servidor Express (`server.ts`) alojado num endereço HTTPS (ex.: Cloud Run / Render / Railway / VPS):

- Crie um ficheiro `.env` (baseado em `.env.example`) com:
```env
VITE_API_BASE_URL="https://SEU-SERVIDOR-EXPRESS"
```
*(Também pode alterar ou definir este URL diretamente dentro da aplicação em **Configurações → Servidor Backend para APK Android**).*

### Passo 3 — Compilar o frontend e sincronizar com o Capacitor Android
```bash
npm run build
npx cap sync android
```
*(Caso a pasta `android/` ainda não exista no ambiente local, execute primeiro `npx cap add android` e depois `npx cap sync android`.)*

### Passo 4 — Gerar o ficheiro `.apk`

#### Opção A — Linha de comandos (APK Debug pronto a testar no telemóvel)
```bash
cd android
./gradlew assembleDebug
```
*(No Windows PowerShell/CMD use `gradlew.bat assembleDebug`.)*

O APK gerado ficará em:
```text
android/app/build/outputs/apk/debug/app-debug.apk
```

#### Opção B — Android Studio (APK Debug ou Release Assinado)
```bash
npx cap open android
```
No Android Studio:
1. Aguarde a sincronização do **Gradle**.
2. Para testes rápidos: **Build → Build Bundle(s) / APK(s) → Build APK(s)**.
3. Para publicação/distribuição: **Build → Generate Signed Bundle / APK... → APK**.

---

## 4. Permissões Android e Funcionalidades Incluídas

No ficheiro `android/app/src/main/AndroidManifest.xml`, certifique-se de manter a permissão de rede ativa (incluída por defeito no template Capacitor):

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

### Funcionalidades preservadas no APK
- Seleção individual dos 7 pares Pocket Option OTC (`EUR/USD OTC`, `GBP/USD OTC`, `USD/JPY OTC`, `AUD/CAD OTC`, `EUR/GBP OTC`, `USD/CHF OTC`, `EUR/JPY OTC`).
- Análise de 1M e 5M com RSI, MACD, EMA 9 e EMA 21.
- Importação de ficheiros CSV de velas OTC reais.
- Carregamento e análise visual de capturas de ecrã (velas de força, rejeição, doji, reversão, suportes/resistências, rompimentos e histórico comparativo por par e período).
- Envio manual e automático de alertas para o Telegram.
- Modo **DEMO** ativo por defeito e **execução automática de operações permanentemente desativada**.

---

## 5. Executar o Servidor Backend (`server.ts`) separadamente

Para que o APK utilize análise de capturas e alertas no Telegram:

```bash
# Definir GEMINI_API_KEY nas variáveis de ambiente do servidor
npm install
npm run build
npm start
```
O servidor escuta na porta `3000` (`0.0.0.0:3000`) com suporte CORS habilitado para pedidos vindos da WebView Capacitor (`https://localhost` / `capacitor://localhost`).
