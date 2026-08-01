---
name: browser-use-skill
description: >
  Panduan lengkap penggunaan library `browser-use` untuk mengotomasi browser web
  menggunakan AI agent. Mencakup instalasi, konfigurasi LLM, penggunaan Agent API,
  kustomisasi controller, dan integrasi Playwright.
  Aktifkan skill ini ketika diminta mengerjakan task otomasi web, web scraping cerdas,
  form filling otomatis, QA testing berbasis AI, atau mengintegrasikan browser agent
  ke dalam project Python.
---

# 🌐 Browser Use Skill

**Browser Use** adalah library Python yang memungkinkan AI agent mengontrol browser web
seperti layaknya manusia — membuka halaman, klik tombol, isi form, dan ekstrak data.

- **GitHub:** https://github.com/browser-use/browser-use
- **Docs:** https://docs.browser-use.com
- **Versi terinstall:** `browser-use==0.13.6`, `playwright==1.61.0`

---

## 📦 Instalasi (sudah terinstall di mesin ini)

```bash
pip install browser-use
python -m playwright install   # download Chromium, Firefox, WebKit
```

**Dependensi utama yang ikut terinstall:**
| Package | Versi | Fungsi |
|---|---|---|
| `browser-use` | 0.13.6 | Core library |
| `playwright` | 1.61.0 | Browser automation engine |
| `cdp-use` | 1.4.5 | Chrome DevTools Protocol |
| `anthropic` | 0.76.0 | Anthropic SDK (Claude) |
| `openai` | 2.16.0 | OpenAI SDK (GPT) |
| `google-genai` | 1.65.0 | Google Gemini SDK |
| `mcp` | 1.26.0 | Model Context Protocol |

---

## ⚡ Quickstart Minimal

```python
import asyncio
from browser_use import Agent
from langchain_openai import ChatOpenAI

async def main():
    agent = Agent(
        task="Go to google.com and search for 'browser-use python'",
        llm=ChatOpenAI(model="gpt-4o"),
    )
    result = await agent.run()
    print(result)

asyncio.run(main())
```

---

## 🔑 Konfigurasi API Key (.env)

Buat file `.env` di root project:

```bash
# Pilih salah satu provider LLM:
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
GOOGLE_API_KEY=AIza...
GROQ_API_KEY=gsk_...

# Opsional - Browser Use Cloud key:
BROWSER_USE_API_KEY=your-key
```

Load dengan `python-dotenv`:

```python
from dotenv import load_dotenv
load_dotenv()
```

---

## 🤖 Mendukung Berbagai LLM Provider

### OpenAI (GPT)
```python
from langchain_openai import ChatOpenAI
llm = ChatOpenAI(model="gpt-4o", temperature=0)
```

### Anthropic (Claude)
```python
from langchain_anthropic import ChatAnthropic
llm = ChatAnthropic(model="claude-opus-4-5", temperature=0)
```

### Google Gemini
```python
from langchain_google_genai import ChatGoogleGenerativeAI
llm = ChatGoogleGenerativeAI(model="gemini-2.0-flash")
```

### Groq (Fast inference)
```python
from langchain_groq import ChatGroq
llm = ChatGroq(model="llama-3.3-70b-versatile")
```

### Ollama (Lokal, gratis)
```python
from langchain_ollama import ChatOllama
llm = ChatOllama(model="qwen2.5:7b")
```

---

## 🧩 Agent API

### Inisialisasi Dasar
```python
from browser_use import Agent

agent = Agent(
    task="Deskripsi task yang harus dikerjakan agent",
    llm=llm,                    # LLM provider
    browser=None,               # None = buat browser baru otomatis
    max_actions_per_step=10,    # Maks aksi per langkah
    use_vision=True,            # Gunakan screenshot untuk konteks visual
)
```

### Menjalankan Agent
```python
# Jalankan sampai selesai:
result = await agent.run()

# Jalankan dengan batas langkah maksimum:
result = await agent.run(max_steps=20)

# Pause/resume support:
await agent.run()
await agent.pause()
await agent.resume()
await agent.stop()
```

### Result Object
```python
result = await agent.run()
print(result.final_result())       # Output akhir
print(result.is_done())            # True jika selesai
print(result.has_errors())         # True jika ada error
print(result.history)              # List semua aksi yang dilakukan
```

---

## 🌐 Konfigurasi Browser

### Default (Playwright baru)
```python
agent = Agent(task="...", llm=llm)  # auto-buat browser baru
```

### Custom Browser Config
```python
from browser_use import Agent, Browser, BrowserConfig

browser = Browser(
    config=BrowserConfig(
        headless=False,            # True = tanpa UI, False = tampilkan browser
        disable_security=False,    # Hati-hati untuk production
        extra_chromium_args=["--no-sandbox"],
        chrome_instance_path="/path/to/chrome",  # Gunakan Chrome yang sudah ada
    )
)

agent = Agent(task="...", llm=llm, browser=browser)
```

### Connect ke Browser Existing (CDP)
```python
from browser_use import Agent, Browser, BrowserConfig

# Jalankan Chrome dengan: --remote-debugging-port=9222
browser = Browser(
    config=BrowserConfig(cdp_url="http://localhost:9222")
)
agent = Agent(task="...", llm=llm, browser=browser)
```

---

## 🎯 Controller (Custom Actions)

Tambah aksi custom ke agent:

```python
from browser_use import Agent, Controller
from pydantic import BaseModel

controller = Controller()

# Custom action dengan return value
@controller.action("Extract all product prices from the page")
async def extract_prices(browser):
    page = await browser.get_current_page()
    prices = await page.evaluate("() => Array.from(document.querySelectorAll('.price')).map(el => el.textContent)")
    return prices

# Custom action dengan parameter model
class LoginCredentials(BaseModel):
    username: str
    password: str

@controller.action("Login to the website", param_model=LoginCredentials)
async def login(credentials: LoginCredentials, browser):
    page = await browser.get_current_page()
    await page.fill("#username", credentials.username)
    await page.fill("#password", credentials.password)
    await page.click("#submit")

agent = Agent(task="...", llm=llm, controller=controller)
```

---

## 📋 Use Cases & Contoh

### 1. Form Filling Otomatis
```python
agent = Agent(
    task="""
    Buka https://example.com/application,
    isi form dengan data berikut:
    - Name: John Doe
    - Email: john@example.com
    - Message: Hello World
    Klik tombol Submit.
    """,
    llm=llm,
)
result = await agent.run()
```

### 2. Data Extraction / Web Scraping
```python
agent = Agent(
    task="""
    Pergi ke https://news.ycombinator.com,
    kumpulkan 10 artikel teratas beserta poin dan komentar-nya,
    return sebagai JSON list.
    """,
    llm=llm,
)
result = await agent.run()
data = result.final_result()
```

### 3. QA Testing Otomatis
```python
agent = Agent(
    task="""
    Test website http://localhost:3000:
    1. Cek apakah homepage load dengan benar
    2. Coba login dengan user test
    3. Navigasi ke halaman dashboard
    4. Report semua error, broken link, atau masalah UI
    """,
    llm=llm,
    use_vision=True,  # Penting untuk visual QA
)
result = await agent.run()
```

### 4. E-commerce Automation
```python
agent = Agent(
    task="""
    Pergi ke tokopedia.com,
    cari 'laptop gaming',
    filter harga 5-10 juta,
    kumpulkan 5 produk teratas dengan nama, harga, dan rating.
    """,
    llm=llm,
)
```

### 5. Multi-tab / Multi-step Task
```python
agent = Agent(
    task="""
    1. Buka gmail.com dan login
    2. Cari email terbaru dari noreply@github.com
    3. Buka email tersebut
    4. Klik link verifikasi di dalamnya
    5. Konfirmasi bahwa akun berhasil terverifikasi
    """,
    llm=llm,
    browser=Browser(config=BrowserConfig(headless=False)),
)
```

---

## 🔧 Advanced Configuration

### System Prompt Custom
```python
from browser_use.agent.prompts import SystemPrompt

class MySystemPrompt(SystemPrompt):
    def important_rules(self) -> str:
        rules = super().important_rules()
        rules += """
        ATURAN TAMBAHAN:
        - Selalu gunakan Bahasa Indonesia dalam laporan
        - Jangan klik tombol berbayar
        - Screenshot sebelum dan sesudah setiap aksi penting
        """
        return rules

agent = Agent(task="...", llm=llm, system_prompt_class=MySystemPrompt)
```

### Sensitive Data (Jangan masuk ke prompt/log)
```python
from browser_use import Agent, SensitiveData

agent = Agent(
    task="Login ke website dengan kredensial saya",
    llm=llm,
    sensitive_data=SensitiveData(
        username="user@example.com",
        password="rahasia123"
    )
)
```

### Hooks & Callbacks
```python
from browser_use.agent.views import AgentOutput

async def on_step_start(state, agent):
    print(f"Memulai langkah: {state.step_number}")

async def on_step_end(state, agent):
    print(f"Selesai langkah: {state.step_number}")

agent = Agent(
    task="...",
    llm=llm,
    register_new_step_callback=on_step_start,
    register_done_callback=on_step_end,
)
```

---

## 🛡️ Tips & Best Practices

### DO ✅
- Gunakan `headless=False` saat development agar bisa monitor agent
- Tambahkan `max_steps` untuk cegah infinite loop
- Gunakan `sensitive_data` untuk password/token — jangan taruh di task string
- Test dengan task sederhana dulu sebelum task kompleks
- Gunakan `use_vision=True` untuk task yang membutuhkan pemahaman visual UI

### DON'T ❌
- Jangan hardcode credentials di task string (masuk ke LLM context/log)
- Jangan gunakan `disable_security=True` di production
- Jangan jalankan di headless jika website memiliki CAPTCHA yang kompleks
- Jangan abaikan error handling — browser agent bisa crash

### Error Handling
```python
try:
    result = await agent.run(max_steps=30)
    if result.has_errors():
        print("Agent errors:", result.errors())
except Exception as e:
    print(f"Agent gagal: {e}")
finally:
    await browser.close()  # Selalu close browser
```

---

## 🔍 Debugging

```python
import logging

# Enable verbose logging
logging.basicConfig(level=logging.DEBUG)

# Atau khusus browser-use:
logging.getLogger("browser_use").setLevel(logging.DEBUG)
```

Gunakan `headless=False` dan jalankan dengan slow-motion:
```python
from playwright.async_api import async_playwright

async with async_playwright() as p:
    browser = await p.chromium.launch(headless=False, slow_mo=500)
```

---

## 📂 Lokasi Playwright Browsers (Mesin Ini)

```
C:\Users\rizky\AppData\Local\ms-playwright\
├── firefox-1532\     (Firefox 151.0)
└── webkit-2311\      (WebKit 26.5)
```

---

## 🔗 Referensi

- **GitHub:** https://github.com/browser-use/browser-use
- **Docs:** https://docs.browser-use.com
- **Examples:** https://github.com/browser-use/browser-use/tree/main/examples
- **Discord:** https://link.browser-use.com/discord
- **Blog:** https://browser-use.com/posts
