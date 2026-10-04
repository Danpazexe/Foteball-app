# ⚽ FOTEBALL

> Jogo de **gerência de futebol brasileiro** estilo Brasfoot, feito em React Native.

[![CI](https://github.com/Danpazexe/FOTEBALL/actions/workflows/ci.yml/badge.svg)](https://github.com/Danpazexe/FOTEBALL/actions/workflows/ci.yml)
![React Native](https://img.shields.io/badge/React%20Native-0.86-61DAFB?logo=react)
![TypeScript](https://img.shields.io/badge/TypeScript-5.8%20strict-3178C6?logo=typescript)

Assuma um clube, dispute o **Brasileirão** (Séries A a D), monte o time a cada
partida, negocie no mercado e busque o acesso — fugindo do rebaixamento.

---

## ✨ Funcionalidades

- **Partida narrada ao vivo** — cada minuto é simulado na hora; pausar e fazer
  substituições durante o jogo muda o resultado (a força do time é recalculada
  minuto a minuto).
- **Pirâmide de divisões** — Séries A, B, C e D com **acesso e rebaixamento** ao
  fim da temporada.
- **Mundo multi-país (em expansão)** — Argentina (Primera División) e Inglaterra
  (Premier League + Championship) com elencos reais e **mercado global** de
  transferências entre países ([procedência](src/data/seed/PROCEDENCIA_INTERNACIONAL.md)).
- **Identidade visual por divisão** — escudo de cada clube e emblema/cores da
  liga mudam conforme a competição.
- **Mercado de transferências** — propostas, contratações e vendas (operação
  atômica com janelas, finanças simétricas e reparo de formação).
- **Calendário** round-robin (turno e returno) com tabela de classificação.
- **Progressão** — moral, evolução de jogadores e finanças do clube.

## 🧱 Stack

| Camada | Tecnologia |
|--------|-----------|
| App | React Native **0.86** (bare CLI, **não** Expo) + React 19.2 |
| Linguagem | TypeScript 5.8 (strict) |
| Estado | Zustand v5 |
| Persistência | SQLite via `@op-engineering/op-sqlite` |
| UI | `react-native-svg` (placar/gráficos), `lucide-react-native` (ícones), `react-native-reanimated` (animações) |
| Testes | Jest |

## 📁 Estrutura

```
src/
├── engine/      # Lógica pura do jogo (simulação, temporada, transferências,
│                # finanças, progressão, táticas) — sem React, determinística
├── domain/      # Mundo canônico multi-país (WorldState, seletores, invariantes)
├── store/       # useGameStore (Zustand) + persistência (SQLite)
├── screens/     # Telas (partida, mercado, tabela, carreira…)
├── components/  # Componentes de UI reutilizáveis (ex.: Placar)
├── design-system/ # Tokens, componentes e primitivas visuais (v2)
├── api/         # Repositórios de dados + seed (JSON dos clubes/jogadores)
├── data/        # Seed por país: clubes/<pais>/<liga>/ e jogadores/<pais>/
├── assets/      # Escudos por país (escudos/<pasta>/) e emblemas das ligas (PNG)
├── audio/       # Sons (gol, fim de jogo…)
├── navigation/  # React Navigation
├── theme/       # Cores, espaçamentos, paletas
├── types/       # Tipos do domínio
└── utils/       # Helpers (formatadores etc.)
```

> **Convenção:** nomes de domínio em **português** (`calcularMoral`), termos
> técnicos genéricos em inglês. A `engine/` é pura e **determinística** (sem
> `Math.random`/`Date.now`; aleatoriedade via seed).

## 🚀 Começando

### Pré-requisitos

- **Node ≥ 22** e o ambiente de [React Native](https://reactnative.dev/docs/set-up-your-environment)
  configurado (Android Studio / Xcode conforme a plataforma).

### Instalação

```sh
npm install
# iOS (apenas macOS, na 1ª vez e após mudar deps nativas):
bundle install && bundle exec pod install
```

### Rodando

```sh
npm start          # inicia o Metro
npm run android    # builda e roda no Android (emulador/dispositivo)
npm run ios        # builda e roda no iOS (macOS)
```

## 🛠️ Scripts

| Comando | O que faz |
|---------|-----------|
| `npm start` | Inicia o Metro (bundler) |
| `npm run android` / `npm run ios` | Builda e roda o app |
| `npm run typecheck` | `tsc --noEmit` (checagem de tipos) |
| `npm run lint` | ESLint |
| `npm test` | Suíte de testes (Jest) |

**Barra de qualidade:** `typecheck` limpo + `lint` sem erros + Jest verde.

## 🤖 CI & build de APK

Dois workflows do GitHub Actions em [`.github/workflows`](.github/workflows):

### CI — [`ci.yml`](.github/workflows/ci.yml)
Roda a cada push/PR na `main`. Cada checagem é um **job próprio** (status
individual, fácil de rastrear), com **annotations inline** no código quando há
erro e um **gate final** (`CI status`) que resume tudo:

| Job | Comando |
|-----|---------|
| Typecheck | `tsc --noEmit` |
| Lint | `eslint .` |
| Testes | `jest` |

### Android APK — [`android.yml`](.github/workflows/android.yml)
Builda um APK instalável e o publica como **artifact** baixável. Disparo:

- **Manual:** aba **Actions → Android APK → Run workflow** (escolha `debug` ou `release`).
- **Por tag:** `git tag v0.1.0 && git push origin v0.1.0` (sempre `release`).

Baixe em **Actions → (execução) → Artifacts → `foteball-apk-<tipo>`**. Se o build
falhar, os relatórios do Gradle sobem como artifact (`android-build-reports`)
para diagnóstico.

> **Assinatura:** o build de **release** (push na `main`, tags e disparo manual
> `release`) é assinado com a **keystore de release** que vem dos secrets do
> repositório. Se algum secret faltar, o workflow **falha com erro claro**. Ele
> nunca cai para a debug key, que é pública. Só o build de `debug` manual usa
> `android/app/debug.keystore`. Veja abaixo como configurar.

### 🔐 Assinatura de release

**Por que isso importa:** o `debug.keystore` do template React Native é igual
em todo projeto, e a senha é pública (`android`). Um APK assinado com essa chave
pode ser substituído por qualquer APK modificado assinado com a mesma chave, e o
Android aceita a troca como "atualização".

#### 1. Gerar a keystore (uma vez, na sua máquina)

```sh
keytool -genkeypair -v \
  -keystore foteball-release.jks \
  -alias foteball \
  -keyalg RSA -keysize 4096 -validity 10000 \
  -storetype PKCS12
```

- O `keytool` pede uma senha e os dados do certificado.
- Em PKCS12 a senha da chave é a mesma da keystore. Use essa senha nos dois secrets.
- **Guarde o `.jks` e a senha fora do repositório**, num gerenciador de senhas
  e com backup. Se perder a keystore, não dá mais para publicar atualizações
  com a mesma assinatura.
- `*.jks`, `*.keystore` (exceto o `debug.keystore`) e `android/keystore.properties`
  estão no `.gitignore`.

#### 2. Cadastrar os secrets no GitHub

Em **Settings → Secrets and variables → Actions → New repository secret**:

| Secret | Valor |
|---|---|
| `ANDROID_KEYSTORE_BASE64` | conteúdo do `.jks` em base64: `base64 -i foteball-release.jks \| pbcopy` (macOS) ou `base64 -w0 foteball-release.jks` (Linux) |
| `ANDROID_KEYSTORE_PASSWORD` | senha da keystore |
| `ANDROID_KEY_ALIAS` | `foteball` (ou o alias que você usou) |
| `ANDROID_KEY_PASSWORD` | senha da chave (em PKCS12, a mesma da keystore) |

O workflow decodifica a keystore num arquivo temporário do runner e confere se
ela abre com a senha e o alias. Depois assina o APK e apaga o arquivo no fim. O
APK de release também é verificado com `apksigner`: se sair com a debug key, o
job falha.

#### 3. Build de release local (opcional)

O `android/app/build.gradle` lê `FOTEBALL_RELEASE_STORE_FILE`,
`FOTEBALL_RELEASE_STORE_PASSWORD`, `FOTEBALL_RELEASE_KEY_ALIAS` e
`FOTEBALL_RELEASE_KEY_PASSWORD`, nesta ordem:

1. de `android/keystore.properties` (ignorado pelo git);
2. de propriedades do Gradle (`-P…` ou `~/.gradle/gradle.properties`);
3. de variáveis de ambiente.

Exemplo de `android/keystore.properties`. O caminho é relativo a `android/` ou absoluto:

```properties
FOTEBALL_RELEASE_STORE_FILE=/caminho/seguro/foteball-release.jks
FOTEBALL_RELEASE_STORE_PASSWORD=...
FOTEBALL_RELEASE_KEY_ALIAS=foteball
FOTEBALL_RELEASE_KEY_PASSWORD=...
```

Sem isso, `npm run build:android` e `build:android:aab` falham avisando o que
falta. Os builds de debug (`npm run android`) não mudam.

#### 4. Primeira instalação com a chave nova

Como a assinatura muda, o Android **não** aceita o novo APK como atualização do
APK antigo, que foi assinado com a debug key. Cada jogador precisa **desinstalar
o app antigo e instalar o novo uma vez**. A desinstalação apaga os saves locais
(SQLite). Avise os jogadores antes.

## 📄 Licença

Projeto privado/pessoal — todos os direitos reservados.
