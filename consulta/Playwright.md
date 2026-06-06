# 🎭 Guia Completo: Playwright (Automação e Web Scraping)

O Playwright é a ferramenta mais moderna e poderosa para automação de navegadores, superando o Selenium em velocidade, confiabilidade e facilidade de uso.

---

## 📑 Sumário
1. [🛠️ Instalação](#-instalação)
2. [🚀 Uso Básico (Síncrono e Assíncrono)](#-uso-básico-síncrono-e-assíncrono)
3. [🧩 Navegadores e Contextos](#-navegadores-e-contextos)
4. [🔍 Seletores e Locators](#-seletores-e-locators)
5. [⏳ Esperas e Estabilidade](#-esperas-e-estabilidade)
6. [🕵️ Evitando Detecção (Stealth Mode)](#-evitando-detecção-stealth-mode)
7. [🧩 Lidando com CAPTCHAs](#-lidando-com-captchas)
8. [📦 Downloads, Uploads e Arquivos](#-downloads-uploads-e-arquivos)
9. [📡 Interceptação de Rede e Cookies](#-interceptação-de-rede-e-cookies)
10. [⚙️ Configuração de Playwright](#-configuração-de-playwright)
11. [🎭 Diálogos, Frames e Popups](#-diálogos-frames-e-popups)
12. [📊 Exemplo Completo de Scraping](#-exemplo-completo-de-scraping)
13. [🧪 Testes e CI](#-testes-e-ci)
14. [⚡ Depuração, Tracing e Logs](#-depuração-tracing-e-logs)
15. [✅ Checklist de Playwright](#-checklist-de-playwright)
16. [💡 Dicas de Ouro](#-dicas-de-ouro)

---

## 🛠️ Instalação

```bash
pip install playwright
# Instala os navegadores necessários (Chromium, Firefox, WebKit)
playwright install
```

---

## 🚀 Uso Básico (Síncrono e Assíncrono)

### **Versão Síncrona (Mais simples)**
```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=False) # headless=False abre o navegador visível
    page = browser.new_page()
    page.goto("https://www.google.com")
    print(page.title())
    browser.close()
```

### **Versão Assíncrona (Recomendada para alta performance)**
```python
import asyncio
from playwright.async_api import async_playwright

async def main():
    async with async_playwright() as p:
        browser = await p.chromium.launch()
        page = await browser.new_page()
        await page.goto("https://reflex.dev")
        await page.screenshot(path="reflex.png")
        await browser.close()

asyncio.run(main())
```

---

## 🧩 Navegadores e Contextos

O Playwright suporta Chromium, Firefox e WebKit. Cada navegador tem seu próprio motor e compatibilidade.

### Lançando navegadores
```python
browser = await p.chromium.launch(headless=True)
# ou
browser = await p.firefox.launch(headless=True)
```

### Browser contexts
Use `browser.new_context()` para isolar cookies, cache e armazenamento local.

```python
context = await browser.new_context()
page = await context.new_page()
```

### Persistent context
Para usar um perfil de usuário real e manter sessão entre execuções:

```python
context = await p.chromium.launch_persistent_context(
    user_data_dir='./perfil_chrome',
    headless=False,
    viewport={'width': 1280, 'height': 800},
)
```

### Emulação de dispositivo e geolocalização
```python
iphone = playwright.devices['iPhone 13']
context = await browser.new_context(**iphone)
await context.grant_permissions(['geolocation'])
await context.set_geolocation({'latitude': -23.5505, 'longitude': -46.6333})
```

---

## 🔍 Seletores e Locators

### Locators
Use `locator()` para interações mais estáveis e encadeadas.

```python
button = page.locator('button#login')
await button.click()
await button.fill('meu_usuario')
```

### Seletores comuns
- Texto: `page.locator('text=Enviar')`
- CSS: `page.locator('.btn-primary')`
- XPath: `page.locator('//div[@class="card"]')`
- Atributo: `page.locator('[data-test="submit"]')`

### Interações essenciais
- `await page.click('button#login')`
- `await page.fill('input[name="username"]', 'meu_usuario')`
- `await page.type('input#senha', 'minha_senha')`
- `await page.check('input[type="checkbox"]')`
- `await page.uncheck('input[type="checkbox"]')`
- `await page.select_option('select', value='opcao1')`

### Extrair dados
```python
texto = await page.text_content('h1')
valor = await page.input_value('input#email')
```

---

## 🕵️ Evitando Detecção (Stealth Mode)
Sites modernos usam scripts para detectar se você é um robô. Para evitar isso, use o plugin **Stealth**.

```bash
pip install playwright-stealth
```

```python
from playwright.sync_api import sync_playwright
from playwright_stealth import stealth_sync

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    context = browser.new_context(
        user_agent="Mozilla/5.0 (Windows NT 10.0; Win64; x64) ..."
    )
    page = context.new_page()
    
    # Aplica o modo furtivo para esconder sinais de automação
    stealth_sync(page)
    
    page.goto("https://bot.sannysoft.com") # Site para testar detecção
    page.screenshot(path="stealth_test.png")
```

---

## 🧩 Lidando com CAPTCHAs

O Playwright não "quebra" CAPTCHAs sozinho, mas fornece as ferramentas para integrá-lo a soluções.

### **1. Estratégia de Evitação (A melhor)**
- Use **Modo Stealth** (visto acima).
- Use **Proxies Residenciais** para rotacionar o IP.
- Use **Sessões Reais**: Carregue cookies de um perfil real do Chrome.
```python
context = p.chromium.launch_persistent_context(user_data_dir="./perfil_chrome", headless=False)
```

### **2. Solução via API de Terceiros (2Captcha / Anti-Captcha)**
Para CAPTCHAs obrigatórios (reCAPTCHA, hCaptcha), você deve enviar o `site_key` para uma API que resolve e te devolve o token.

```python
# Exemplo conceitual de preenchimento do token resolvido
token_resolvido = "TOKEN_DA_API_DE_TERCEIROS"
page.evaluate(f'document.getElementById("g-recaptcha-response").innerHTML="{token_resolvido}";')
page.click("#botao-enviar")
```

### **3. Intervenção Manual**
Em scripts de uso único, você pode pausar o script, resolver manualmente e continuar.
```python
page.pause() # Abre um console de inspeção e pausa a execução
```

---

## 📦 Downloads, Uploads e Arquivos

### Download de arquivos
```python
async with page.expect_download() as download_info:
    await page.click('a#download')
download = await download_info.value
await download.save_as('arquivo.zip')
```

### Upload de arquivos
```python
await page.set_input_files('input[type="file"]', 'upload.pdf')
```

### Capturas de tela e PDF
```python
await page.screenshot(path='captura.png', full_page=True)
await page.pdf(path='pagina.pdf', format='A4')
```

### Configurações de viewport e screenshot
```python
await page.set_viewport_size({'width': 1920, 'height': 1080})
```

---

## 📡 Interceptação de Rede e Cookies

### Interceptação de requests
```python
async def handle_route(route, request):
    if 'analytics' in request.url:
        await route.abort()
    else:
        await route.continue_()

await page.route('**/*', handle_route)
await page.goto(url)
```

### Modificar headers
```python
await page.set_extra_http_headers({'X-Forwarded-For': '1.2.3.4'})
```

### Cookies e local storage
```python
await context.add_cookies([{'name': 'session', 'value': 'abc123', 'url': 'https://exemplo.com'}])
cookies = await context.cookies()
```

### Estado de sessão persistente
```python
await context.storage_state(path='state.json')
context = await browser.new_context(storage_state='state.json')
```

---

## ⚙️ Configuração de Playwright

### Arquivo `playwright.config.py`
```python
# Exemplo de configuração do Playwright Test Runner
# A maioria dos projetos usa structure simples como abaixo.

timeout = 30000
use = {
    'headless': True,
    'viewport': {'width': 1280, 'height': 720},
}
projects = [
    {
        'name': 'chromium',
        'use': {'browserName': 'chromium'},
    },
    {
        'name': 'firefox',
        'use': {'browserName': 'firefox'},
    },
]
```

### Comandos úteis
- `playwright install --with-deps` (CI / containers Linux)
- `playwright test --project=chromium`
- `playwright show-trace trace.zip`
- `playwright codegen https://site.com`

### Headed vs Headless
- `headless=True` para CI e raspagem silenciosa.
- `headless=False` para debug e inspeção visual.

---

## 🎭 Diálogos, Frames e Popups

### Diálogos
```python
async def handle_dialog(dialog):
    print(dialog.message)
    await dialog.accept()

page.on('dialog', handle_dialog)
await page.click('#deleta')
```

### Frames
```python
frame = page.frame(name='iframe-name')
await frame.fill('input#campo', 'valor')
```

### Popups e novas janelas
```python
async with context.expect_page() as nova_pagina_info:
    await page.click('a[target="_blank"]')
nova_pagina = await nova_pagina_info.value
```

---

## 🧪 Testes e CI

- Use o Playwright Test Runner (`pip install pytest-playwright`) para testes end-to-end.
- Configure `playwright.config.py` com browsers, retries e reporters.
- Execute em CI com `playwright install --with-deps` e `pytest`.
- Separe testes por `tests/` e use fixtures para estados de login.
- Use `expect` e assertions claras para validação de fluxo.

---

## ⚡ Depuração, Tracing e Logs

### Tracing
```python
await context.tracing.start(screenshots=True, snapshots=True, sources=True)
await page.goto(url)
await context.tracing.stop(path='trace.zip')
```

### Codegen
```bash
playwright codegen https://site.com
```

### Debug
- Use `page.pause()` para abrir ferramenta de depuração.
- Habilite `PWDEBUG=1` para lançar o navegador com painel de devtools.
- Ative logs de rede e snapshots para reproduzir falhas.

---

## ✅ Checklist de Playwright

- [ ] Instalação de browsers e dependências correta com `playwright install`.
- [ ] Use `browser.new_context()` para isolar testes e sessões.
- [ ] Prefira `locator()` em vez de `query_selector()` para estabilidade.
- [ ] Configure timeouts e waits explícitos para evitar flaky tests.
- [ ] Use `page.route()` para interceptar rede e bloquear recursos não essenciais.
- [ ] Salve downloads e faça upload com `set_input_files()`.
- [ ] Capture screenshots ou traces em falhas.
- [ ] Teste em headless e headed conforme o ambiente.
- [ ] Configure CI com `playwright install --with-deps` e execução automatizada.
- [ ] Proteja scripts de detecção com user agent e stealth quando necessário.

---

## 📊 Exemplo Completo de Scraping

```python
import asyncio
from playwright.async_api import async_playwright

async def scrape_produtos():
    async with async_playwright() as p:
        # Lança navegador com viewport realista
        browser = await p.chromium.launch(headless=True)
        context = await browser.new_context(viewport={'width': 1920, 'height': 1080})
        page = await context.new_page()

        await page.goto("https://exemplo.com/produtos")

        # Rola a página para carregar lazy loading
        await page.mouse.wheel(0, 5000)
        await page.wait_for_timeout(2000)

        produtos = await page.query_selector_all(".product-card")
        
        dados = []
        for p in produtos:
            nome = await p.query_selector(".title")
            preco = await p.query_selector(".price")
            
            dados.append({
                "nome": await nome.inner_text(),
                "preco": await preco.inner_text()
            })

        print(dados)
        await browser.close()

asyncio.run(scrape_produtos())
```

---

## 💡 Dicas de Ouro
- **Tracing**: Use `context.tracing.start(screenshots=True, snapshots=True)` para gravar uma execução e depurar erros visuais depois.
- **Codegen**: Use `playwright codegen https://site.com` no terminal. Ele abrirá o navegador e **gerará o código Python automaticamente** enquanto você clica no site.
