# VibeCoderAI

**An AI coding assistant that runs on your Android phone, offline.**
Describe an app in chat, the AI writes it, you preview it in the app, and you can turn it into an installable APK right on the phone. No PC needed.

📥 **Download:** open the [Releases](https://github.com/kinggend/VibeCoderAI/releases) page and install the latest `.apk` (Android 7.0+, 64-bit ARM phones).

---

## What makes it different

| | |
|---|---|
| 🔒 **Works fully offline** | The AI model runs on your phone with llama.cpp. Your code and chats never leave the device unless you choose a remote server. |
| 📦 **Builds APKs on the phone** | Turn a web project (`index.html`) into a signed, installable Android app in seconds, without a PC or Android Studio. |
| 🤖 **A real agent, not just chat** | It writes, edits and patches files, runs commands, builds, checks for errors and repairs them itself. |
| ✋ **You stay in control** | Every file change and command can ask for your approval first, with a diff you can review. |
| 🧠 **Any GGUF model** | Search and download models from Hugging Face inside the app (Qwen, Gemma, Llama, and more). Each model uses its own chat format. |
| ☁️ **Optional cloud power** | For bigger apps, connect Ollama, LM Studio, llama-server, OpenRouter, Groq, Gemini or OpenAI. |
| 🔐 **Encrypted API keys** | Keys are locked with the Android Keystore and never backed up. See below. |
| 🗣️ **Understands mixed languages** | Type Telugu or Hindi in English letters (Tenglish/Hinglish) and it replies the same way. |

---

### 📸 Screenshots

<p align="center">
  <img src="https://raw.githubusercontent.com/kinggend/kinggend/main/assets/vibecoder_chat.jpg" width="200" alt="Chat Interface" />
  &nbsp;&nbsp;
  <img src="https://raw.githubusercontent.com/kinggend/kinggend/main/assets/vibecoder_projects.jpg" width="200" alt="Projects & Editor" />
  &nbsp;&nbsp;
  <img src="https://raw.githubusercontent.com/kinggend/kinggend/main/assets/vibecoder_models.jpg" width="200" alt="Local Models" />
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/kinggend/kinggend/main/assets/vibecoder_terminal.jpg" width="200" alt="Sandboxed Terminal" />
  &nbsp;&nbsp;
  <img src="https://raw.githubusercontent.com/kinggend/kinggend/main/assets/vibecoder_settings.jpg" width="200" alt="Settings" />
</p>


## Features

- **Chat**: streaming replies, markdown, code blocks with Copy and Insert, multiple chats with history, regenerate, voice input, file attach.
- **Agent tools**: WriteFile, EditFile, ApplyPatch, RunCommand, BuildProject, RunCheck, AnalyzeErrors.
- **Agent team**: Builder, Debugger, Reviewer and Designer roles, plus an autonomous mode (plan → build → test → self-repair).
- **Live preview**: run your web app inside VibeCoderAI, with console errors and a "Fix with AI" button.
- **Editor**: file tabs, syntax highlighting, line numbers, undo/redo, find and replace.
- **Projects and terminal**: project manager, version snapshots (undo), terminal with approval.
- **Models**: Hugging Face search, background downloads that keep going when the app is closed, import from storage, RAM fit check before loading.
- **Smart memory use**: checks free RAM before loading a model and unloads it when the phone runs low.
- **Fast replies**: reuses the model's prompt cache between messages; optional GPU (Vulkan) build.
- **Knows the time**: the phone's date and time are given to the model, so answers about "today" are right.

---

## 🔐 How your API key is protected

When you use a remote server, the **API key, server URL and model name** are encrypted before they are saved.

```
 You paste the key
        │
        ▼
 Android Keystore makes a secret AES-256 lock key
 (kept in the phone's secure hardware; the app itself can't read it out)
        │
        ▼
 Key is encrypted with AES-256-GCM  →  "v1:<iv>:<scrambled text>"
        │
        ▼
 Saved in the app's private storage (vibecoder_secrets)
 Excluded from Google backup and phone-to-phone transfer
        │
        ▼
 Decrypted in memory only when a request is sent
        │
        ▼
 Sent only to the Server URL you entered (Authorization: Bearer …)
```

- The plain key is **never written to disk**. After you tap **Save key**, the app only shows a masked preview (`sk-••••••abcd`) with **Change** and **Remove**.
- Uses **StrongBox** (a separate security chip) on phones that have one, otherwise the hardware-backed Keystore.
- Keys saved by older versions (plain text) are **moved into encrypted storage automatically** on first launch.
- Uninstalling the app deletes both the lock key and the encrypted key.
- If the server URL is `http://` on the internet, the app warns you, because the key would travel unencrypted. Use `https://`.

Code: `app/src/main/java/com/example/data/SecretCipher.kt` and `SettingsStore.kt` inside the project zip.

---

## Which model should I use?

| Free RAM on phone | Good choice |
|---|---|
| ~3 GB | 1.5B – 3B, Q4 (e.g. Qwen2.5-Coder 1.5B/3B Q4_K_M) |
| 6 GB+ | 4B – 7B, Q4 |

Bigger is smarter but slower. Expect a few words per second on a mid-range phone. For large multi-file apps, a remote server works better.

---

### Architecture

- **Single-activity approach:** Built entirely with Jetpack Compose UI.
- **Core Flow:** `Compose UI` → `AgentOrchestrator (agent loop)` → `tools/engine layers`.
- **Packages:** 
  - `agent` (roles, loop, router, repair)
  - `llama` (JNI engine, memory estimator)
  - `phone` (accessibility agent + overlay)
  - `project`, `terminal`, `build`, `patch`, `github`
  - `data` (settings, Room), `security`, `transparency`, `ui`
- **Dependency Injection:** Manual DI container (`AppGraph`).
- **Testing:** Pure-Kotlin logic (router, memory math, export) is unit-tested separately.
- **Modules:** `app` + `webshell` (WebView shell APK builder).

### Tech stack

- **Core:** Kotlin, Jetpack Compose + Material 3, Coroutines/Flow, Room (KSP).
- **Native:** `llama.cpp` + `mtmd` via CMake/JNI (arm64-v8a only).
- **Android APIs:** AccessibilityService (`takeScreenshot`, `dispatchGesture`, overlay windows), SpeechRecognizer, TextToSpeech.
- **Security & System:** Android Keystore (AES-256-GCM), apksig + WebView shell for on-device APK building.
- **Build:** Min SDK 24, Target/Compile SDK 36, Gradle Kotlin DSL, GitHub Actions for automated builds.

---

## Build it yourself

This repo holds the Android project as `vibecoderai-apk-build.zip` and the workflow in `.github/workflows/`.
Push a change and GitHub Actions unzips the project, fetches llama.cpp, and builds debug and signed release APKs. Download them from the run's **Artifacts**.

## Limits (honest list)

- On-device APK building works for **web projects** (HTML/CSS/JS). Native Android/Gradle projects need a PC or GitHub Actions.
- Text models only; GGUF format only.
- Small local models are best for small apps and edits.
- Chats and project files are stored in app storage but are not encrypted; only remote server settings are.

---

**Version:** 1.6.3
