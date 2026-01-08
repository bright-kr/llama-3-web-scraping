# LLaMA 3로 Webスクレイピング하기

[![Bright Data Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.co.kr/)

이 가이드는 LLaMA 3를 사용하여 큰 HTML을 구조화되고, 깔끔하며, 활용 가능한 JSON으로 변환하는 방법을 설명합니다:

- [Web Scraping에 LLaMA 3를 선택해야 하는 이유](#why-choose-llama-3-for-web-scraping)
- [시스템 요구 사항](#system-requirements)
- [Ollama 설정하기](#setting-up-ollama)
- [적절한 LLaMA 모델 선택하기](#selecting-the-right-llama-model)
- [모델 다운로드 및 실행](#downloading-and-running-the-model)
- [LLM 기반 Amazon 스크레이퍼 구축](#building-an-amazon-scraper-powered-by-llms)
- [アンチボット 보호 처리](#handling-anti-bot-protection)
- [스크레이퍼 고도화 및 확장](#enhancing-and-expanding-your-scraper)

## Why Choose LLaMA 3 for Web Scraping

[Meta's LLaMA 3](https://ai.meta.com/blog/meta-llama-3/) (2024년 4월 공개)는 8B부터 405B 파라미터까지 확장되는 오픈 웨이트 LLM 시리즈로, 폭넓은 작업과 하드웨어 구성에 적합합니다. 3.1부터 3.3까지의 업데이트를 통해 기능이 더욱 강화되었습니다.

전통적인 スクレイピング 기법—[XPath 또는 CSS](https://brightdata.co.kr/blog/web-data/xpath-vs-css-selectors)를 사용하는 방식—은 웹사이트 레이아웃 변경에 취약합니다. 그러나 LLaMA 3는 사람처럼 콘텐츠를 이해하므로, 업데이트가 있어도 신뢰성을 유지하는 지능적이고 탄력적인 スクレイピング을 제공합니다.

따라서 다음과 같은 경우에 이상적입니다:

- Amazon과 같은 대형 리테일 사이트
- 복잡한 데이터 파싱
- 견고하고 내구성 있는 스크레이퍼
- 민감한 데이터의 사내 유지 보장

AI 기반 Webスクレイピング에 대해 더 알아보려면 [이전 가이드](https://brightdata.co.kr/blog/web-data/ai-web-scraping)를 참고하시기 바랍니다.

## System Requirements

[LLM 기반 スクレイピング](https://brightdata.co.kr/blog/web-data/web-scraping-with-scrapegraphai) 프로젝트를 시작하기 전에 다음을 갖추었는지 확인하시기 바랍니다:

- [Python 3](https://www.python.org/downloads/)
- Python에 대한 기본 이해
- 다음 OS 구성 중 하나:
  - macOS 11 Big Sur 이상
  - Linux
  - Windows 10 이상
- 충분한 머신 리소스(자세한 내용은 아래 참고)

## Setting Up Ollama

Ollama는 대규모 언어 모델을 로컬에서 설치, 실행, 관리하는 과정을 간소화합니다.

![Ollama installation page](https://github.com/luminati-io/llama-3-web-scraping/blob/main/images/ollama-llm-download-installation-page.png)

시작 방법은 다음과 같습니다:

1. [Ollama 공식 사이트](https://ollama.com/)로 이동합니다
2. OS에 맞는 버전을 다운로드합니다
3. **중요**: 설치 중 명령 실행을 요청받는데, 모델을 선택하기 전까지는 실행하지 마시기 바랍니다.

## Selecting the Right LLaMA Model

적절한 버전을 찾기 위해 [Ollama 모델 라이브러리](https://ollama.com/library)를 살펴보시기 바랍니다.

일반적인 머신을 사용 중이라면 `llama3.1:8b`가 훌륭한 선택입니다. 컴팩트하고 효율적이며, 디스크 공간 약 4.9 GB와 RAM 6–8 GB 정도가 필요합니다. 최신 노트북 대부분에서 무리 없이 실행됩니다.

더 강력한 하드웨어를 보유하고 있다면 `70B` 또는 `405B` 같은 대형 버전이 더 뛰어난 추론 능력과 더 긴 컨텍스트 윈도우를 제공하지만, 하드웨어 요구 사항이 큽니다.

## Downloading and Running the Model

다음 명령으로 LLaMA 3.1 (8B) 모델을 가져옵니다:

```sh
ollama run llama3.1:8b
```

대화형 프롬프트가 표시됩니다:

```sh
>>> Send a message (/? for help)
```

테스트해 보시기 바랍니다:

```sh
>>> who are you?
I am LLaMA, *an AI assistant developed by Meta AI...*
```

정상 동작이 확인되면 Ollama 서버를 시작합니다:

```sh
ollama serve
```

이는 `http://127.0.0.1:11434/`에서 로컬 서버를 실행합니다. 이 창은 계속 열어 두시기 바랍니다.

브라우저로 접속하면 **“Ollama is running.”** 메시지가 표시되어야 합니다.

## Building an Amazon Scraper Powered by LLMs

이제 Amazon에서 제품 상세 정보를 추출하는 스크레이퍼를 만들어 보겠습니다. Amazon은 [동적 콘텐츠](https://brightdata.co.kr/blog/how-tos/scrape-dynamic-websites-python)와 강력한 アンチボット 보호로 인해 가장 까다로운 대상 중 하나입니다.

![Amazon product page](https://github.com/luminati-io/llama-3-web-scraping/blob/main/images/amazon-office-chair-product-page-1.png)

다음 항목을 추출합니다:

- 제목
- 가격
- 할인
- 평점 및 리뷰 수
- 설명 및 특징
- 재고 상태 및 [ASINs](https://brightdata.co.kr/blog/web-data/how-to-scrape-amazon-asin)

### Smart Multi-Stage Approach

LLaMA 기반 스크레이퍼는 다음과 같은 스마트한 다단계 워크플로를 따릅니다:

1. Selenium을 통한 **브라우저 자동화**
2. 타깃 섹션에서의 **HTML 추출**
3. 입력을 간소화하기 위한 **Markdown 변환**
4. 데이터를 구조화하기 위한 **LLM 처리**
5. 추가 분석을 위한 **결과 저장**

워크플로의 시각적 구성은 다음과 같습니다:

![Workflow diagram](https://github.com/luminati-io/llama-3-web-scraping/blob/main/images/llama-web-scraping-workflow-diagram.png)

여기서는 **Python**을 사용하지만, [JavaScript](https://brightdata.co.kr/blog/web-data/best-languages-web-scraping) 등 다른 언어로도 적용할 수 있습니다.

### Step 1 – Install Required Libraries

먼저 필요한 Python 라이브러리를 설치합니다:

```sh
pip install requests selenium webdriver-manager markdownify
```

- `requests` – LLM 서비스에 API 호출을 보내기 위한 [최고의 Python HTTP 클라이언트](https://brightdata.co.kr/blog/web-data/best-python-http-clients)
- `selenium` – 브라우저를 자동화하며, JavaScript 비중이 큰 웹사이트에 적합합니다
- `webdriver-manager` – 올바른 ChromeDriver 버전을 자동으로 다운로드 및 관리합니다
- `markdownify` – HTML을 Markdown으로 변환합니다

### Step 2 – Initialize the Headless Browser

Selenium으로 [헤드리스 브라우저](https://brightdata.co.kr/blog/proxy-101/what-is-a-headless-browser)를 설정합니다:

```python
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.chrome.options import Options
from webdriver_manager.chrome import ChromeDriverManager

options = Options()
options.add_argument("--headless")

driver = webdriver.Chrome(
    service=Service(ChromeDriverManager().install()),
    options=options
)
```

### Step 3 – Extract the Product HTML

Amazon 제품 상세 정보는 동적으로 렌더링되며 `<div id="ppd">` 컨테이너 내부에 래핑됩니다. 스크립트는 해당 섹션이 로드될 때까지 기다린 뒤, HTML을 추출합니다:

```python
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

wait = WebDriverWait(driver, 15)
product_container = wait.until(
    EC.presence_of_element_located((By.ID, "ppd"))
)

# Extract the full HTML of the product container
page_html = product_container.get_attribute("outerHTML")
```

이 접근 방식은 다음과 같은 장점이 있습니다:

- JavaScript로 렌더링되는 콘텐츠(가격, 평점 등)를 기다립니다
- 헤더, 푸터, 사이드바를 무시하고 관련 제품 섹션만 타깃팅합니다

_[Python으로 Amazon 제품 데이터를 スクレイピング하는 방법](https://brightdata.co.kr/blog/how-tos/how-to-scrape-amazon)에 대한 전체 가이드를 확인해 보시기 바랍니다._

### Step 4 – Convert HTML to Markdown

Amazon 페이지는 HTML 중첩이 매우 깊어 LLM이 처리하기에 비효율적입니다. 따라서 이 HTML을 깔끔한 Markdown으로 변환하여 불필요한 데이터를 제거하는 것이 좋습니다. 이렇게 하면 토큰 수가 줄고 이해도가 향상됩니다.

전체 스크립트를 실행하면 `amazon_page.html`과 `amazon_page.md` 두 파일이 생성됩니다. 두 파일을 [Token Calculator Tool](https://token-calculator.net/)에 각각 붙여 넣어 토큰 수를 비교해 보시기 바랍니다.

HTML은 약 **270,000 토큰**을 포함합니다:

![token-calculator-html-tokens](https://github.com/luminati-io/llama-3-web-scraping/blob/main/images/token-calculator-html-tokens.png)

Markdown 버전은 **~11,000 토큰**만 포함합니다:

![token-calculator-markdown-tokens](https://github.com/luminati-io/llama-3-web-scraping/blob/main/images/token-calculator-markdown-tokens.png)

이 **96% 감소**는 다음으로 이어집니다:

- **비용 효율성** – 토큰이 적을수록 API 또는 연산 비용이 낮아집니다
- **더 빠른 처리** – 입력 데이터가 적을수록 LLM 응답이 빨라집니다
- **정확도 향상** – 더 깔끔하고 평탄한 텍스트는 모델이 구조화 데이터를 더 정확히 추출하도록 돕습니다

_[AI 에이전트가 HTML보다 Markdown을 선호하는 이유](https://hackernoon.com/why-are-the-new-ai-agents-choosing-markdown-over-html)에 대해 더 읽어보시기 바랍니다._

Python에서 변환하는 방법은 다음과 같습니다:

```python
from markdownify import markdownify as md

clean_text = md(page_html, heading_style="ATX")
```

### Step 5 – Create the Data Extraction Prompt

일관되고 깔끔한 JSON 출력을 얻기 위해서는 잘 구조화된 프롬프트가 매우 중요합니다. 아래 프롬프트는 모델이 사전에 정의된 형식으로 유효한 JSON만 반환하도록 지시합니다:

```python
PROMPT = (
    "You are an expert Amazon product data extractor. Your task is to extract product data from the provided content. "
    "Return ONLY valid JSON with EXACTLY the following fields and formats:\n\n"
    "{\n"
    '  "title": "string – the product title",\n'
    '  "price": number – the current price (numerical value only)",\n'
    '  "original_price": number or null – the original price if available,\n'
    '  "discount": number or null – the discount percentage if available,\n'
    '  "rating": number or null – the average rating (0–5 scale),\n'
    '  "review_count": number or null – total number of reviews,\n'
    '  "description": "string – main product description",\n'
    '  "features": ["string"] – list of bullet point features,\n'
    '  "availability": "string – stock status",\n'
    '  "asin": "string – 10-character Amazon ID"\n'
    "}\n\n"
    "Return ONLY the JSON without any additional text."
)
```

### Step 6 – Call the LLM API

Markdown 텍스트를 HTTP API를 통해 LLaMA 인스턴스로 전송합니다:

```python
import requests
import json

response = requests.post(
    "<http://localhost:11434/api/generate>",
    json={
        "model": "llama3.1:8b",
        "prompt": f"{PROMPT}\n\n{clean_text}",
        "stream": False,
        "format": "json",
        "options": {
            "temperature": 0.1,
            "num_ctx": 12000,
        },
    },
    timeout=250,
)

raw_output = response.json()["response"].strip()
product_data = json.loads(raw_output)
```

각 옵션의 의미는 다음과 같습니다:

- `temperature` – 결정적 출력(특히 JSON 포맷팅에 이상적)을 위해 0.1로 설정합니다
- `num_ctx` – 최대 컨텍스트 길이를 정의합니다. 12,000 토큰이면 대부분의 Amazon 제품 페이지에 충분합니다
- `stream` – `False`일 때 API는 처리 후 전체 응답을 반환합니다
- `format` – 출력 형식(JSON)을 지정합니다
- `model` – 사용할 LLaMA 버전을 지정합니다

물론입니다! 아래는 더 깔끔하게 다듬은 설명입니다:

변환된 Markdown은 종종 약 11,000 토큰 정도이므로, 컨텍스트 윈도우(`num_ctx`)를 그에 맞게 설정하시기 바랍니다. 값을 늘리면 더 긴 입력을 지원할 수 있지만, 더 많은 RAM을 사용하고 처리 속도가 느려집니다. 필요하거나 리소스가 충분한 경우에만 늘리시기 바랍니다.

### Step 7 – Save the Results

마지막으로 구조화된 제품 데이터를 JSON 파일로 저장합니다:

```python
with open("product_data.json", "w", encoding="utf-8") as f:
    json.dump(product_data, f, indent=2, ensure_ascii=False)
```

### Step 8: Execute the Script

스크레이퍼를 실행하려면 Amazon 제품 URL을 제공하고 スクレイピング 함수를 호출합니다:

```python
if __name__ == "__main__":
    url = "<https://www.amazon.com/Black-Office-Chair-Computer-Adjustable/dp/B00FS3VJAO>"

    # Call your function to scrape and extract product data
    scrape_amazon_product(url)
```

### Step 9 – Full Code Example

아래는 전체 Python 스크립트입니다:

```python
import json
import logging
import time
from typing import Final, Optional, Dict, Any

import requests
from markdownify import markdownify as html_to_md
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.ui import WebDriverWait
from webdriver_manager.chrome import ChromeDriverManager

# Configuration constants
LLM_API_CONFIG: Final[Dict[str, Any]] = {
    "endpoint": "<http://localhost:11434/api/generate>",
    "model": "llama3.1:8b",
    "temperature": 0.1,
    "context_window": 12000,
    "stream": False,
    "timeout_seconds": 220,
}

DEFAULT_PRODUCT_DATA: Final[Dict[str, Any]] = {
    "title": "",
    "price": 0.0,
    "original_price": None,
    "discount": None,
    "rating": None,
    "review_count": None,
    "description": "",
    "features": [],
    "availability": "",
    "asin": "",
}

PRODUCT_DATA_EXTRACTION_PROMPT: Final[str] = (
    "You are an expert Amazon product data extractor. Your task is to extract product data from the provided content. "
    "Return ONLY valid JSON with EXACTLY the following fields and formats:\n\n"
    "{\n"
    '  "title": "string - the product title",\n'
    '  "price": number - the current price (numerical value only),\n'
    '  "original_price": number or null - the original price if available,\n'
    '  "discount": number or null - the discount percentage if available,\n'
    '  "rating": number or null - the average rating (0-5 scale),\n'
    '  "review_count": number or null - total number of reviews,\n'
    '  "description": "string - main product description",\n'
    '  "features": ["string"] - list of bullet point features,\n'
    '  "availability": "string - stock status",\n'
    '  "asin": "string - 10-character Amazon ID"\n'
    "}\n\n"
    "Return ONLY the JSON without any additional text."
)

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s - %(levelname)s - %(message)s",
    handlers=[logging.StreamHandler()],
)

def initialize_web_driver(headless: bool = True) -> webdriver.Chrome:
    """Initialize and return a configured Chrome WebDriver instance."""
    options = Options()
    if headless:
        options.add_argument("--headless=new")

    service = Service(ChromeDriverManager().install())
    return webdriver.Chrome(service=service, options=options)

def fetch_product_container_html(product_url: str) -> Optional[str]:
    """Retrieve the HTML content of the Amazon product details container."""
    driver = initialize_web_driver()
    try:
        logging.info(f"Accessing product page: {product_url}")
        driver.set_page_load_timeout(15)
        driver.get(product_url)

        # Wait for the product container to appear
        container = WebDriverWait(driver, 5).until(
            EC.presence_of_element_located((By.ID, "ppd"))
        )
        return container.get_attribute("outerHTML")
    except Exception as e:
        logging.error(f"Error retrieving product details: {str(e)}")
        return None
    finally:
        driver.quit()

def extract_product_data_via_llm(markdown_content: str) -> Optional[Dict[str, Any]]:
    """Extract structured product data from markdown text using LLM API."""
    try:
        logging.info("Extracting product data via LLM API...")
        response = requests.post(
            LLM_API_CONFIG["endpoint"],
            json={
                "model": LLM_API_CONFIG["model"],
                "prompt": f"{PRODUCT_DATA_EXTRACTION_PROMPT}\n\n{markdown_content}",
                "format": "json",
                "stream": LLM_API_CONFIG["stream"],
                "options": {
                    "temperature": LLM_API_CONFIG["temperature"],
                    "num_ctx": LLM_API_CONFIG["context_window"],
                },
            },
            timeout=LLM_API_CONFIG["timeout_seconds"],
        )
        response.raise_for_status()

        raw_output = response.json()["response"].strip()
        # Clean JSON output if it's wrapped in markdown code blocks
        if raw_output.startswith(("```json", "```")):
            raw_output = raw_output.split("```")[1].strip()
            if raw_output.startswith("json"):
                raw_output = raw_output[4:].strip()

        return json.loads(raw_output)

    except requests.exceptions.RequestException as e:
        logging.error(f"LLM API request failed: {str(e)}")
        return None
    except json.JSONDecodeError as e:
        logging.error(f"Failed to parse LLM response: {str(e)}")
        return None
    except Exception as e:
        logging.error(f"Unexpected error during data extraction: {str(e)}")
        return None

def scrape_amazon_product(
    product_url: str, output_file: str = "product_data.json"
) -> None:
    """Scrape an Amazon product page and save extracted data along with HTML and Markdown to files."""
    start_time = time.time()
    logging.info(f"Starting scrape for: {product_url}")

    # Step 1: Fetch product page HTML
    product_html = fetch_product_container_html(product_url)
    if not product_html:
        logging.error("Failed to retrieve product page content")
        return

    # Optional: save HTML for debugging
    with open("amazon_product.html", "w", encoding="utf-8") as f:
        f.write(product_html)

    # Step 2: Convert HTML to Markdown
    product_markdown = html_to_md(product_html)

    # Optional: save Markdown for debugging
    with open("amazon_product.md", "w", encoding="utf-8") as f:
        f.write(product_markdown)

    # Step 3: Extract structured data via LLM
    product_data = (
        extract_product_data_via_llm(product_markdown) or DEFAULT_PRODUCT_DATA.copy()
    )

    # Step 4: Save JSON results
    try:
        with open(output_file, "w", encoding="utf-8") as json_file:
            json.dump(product_data, json_file, indent=2, ensure_ascii=False)
        logging.info(f"Successfully saved product data to {output_file}")
    except IOError as e:
        logging.error(f"Failed to save JSON results: {str(e)}")

    elapsed_time = time.time() - start_time
    logging.info(f"Completed in {elapsed_time:.2f} seconds")

if __name__ == "__main__":
    # Example usage
    test_url = (
        "<https://www.amazon.com/Black-Office-Chair-Computer-Adjustable/dp/B00FS3VJAO>"
    )
    scrape_amazon_product(test_url)
```

이 스크립트는 추출된 제품 데이터를 `product_data.json`이라는 파일명으로 저장합니다. 출력은 다음과 유사합니다:

```json
{
    "title": "Home Office Chair Ergonomic Desk Chair Mesh Computer Chair with Lumbar Support Armrest Executive Rolling Swivel Adjustable Mid Back Task Chair for Women Adults, Black",
    "price": 36.98,
    "original_price": 41.46,
    "discount": 11,
    "rating": 4.3,
    "review_count": 58112,
    "description": 'Office chair comes with all hardware and tools, and is easy to assemble in about 10–15 minutes. The high-density sponge cushion offers flexibility and comfort, while the mid-back design and rectangular lumbar support enhance ergonomics. All components are BIFMA certified, supporting up to 250 lbs. The chair includes armrests and an adjustable seat height (17.1"–20.3"). Its ergonomic design ensures a perfect fit for long-term use.',
    "features": [
        "100% mesh material",
        "Quick and easy assembly",
        "High-density comfort seat",
        "BIFMA certified quality",
        "Includes armrests",
        "Ergonomic patented design",
    ],
    "availability": "In Stock",
    "asin": "B00FS3VJAO",
}
```

## Handling Anti-Bot Protection

위의 [web scraping bot](https://brightdata.co.kr/blog/how-tos/what-is-a-scraping-bot)을 실행하면 CAPTCHA 챌린지와 같은 Amazon의 アンチボット 조치를 마주칠 가능성이 큽니다:

![amazon-captcha-anti-bot-challenge](https://github.com/luminati-io/llama-3-web-scraping/blob/main/images/amazon-captcha-anti-bot-challenge.png)

LLaMA 3는 파싱을 훌륭하게 수행하지만, 사이트 보호를 우회하는 것은 여전히 까다롭습니다. [Bright Data’s Scraping Browser](https://brightdata.co.kr/products/scraping-browser)는 강력한 해결책을 제공합니다.

### Why Use Bright Data Scraping Browser

[Bright Data Scraping Browser](https://brightdata.co.kr/products/scraping-browser)는 최신 Webスクレイピング 프로젝트를 스케일링하기 위해 설계된 헤드리스, 클라우드 기반 브라우저입니다. 내장 プロキシ 인프라와 고급 언블로킹 기능을 제공하며, [Bright Data Unlocker scraping suite](https://docs.brightdata.com/scraping-automation/introduction)의 일부입니다.

선택해야 하는 이유는 다음과 같습니다:

- 신뢰할 수 있는 TLS 지문과 스텔스 회피
- [150M+ residential IP proxy network](https://brightdata.co.kr/proxy-types/residential-proxies)를 통한 내장 IP 로테이션
- 자동 CAPTCHA 해결
- 인프라 비용 절감 – 클라우드 설정이나 유지보수가 필요 없습니다
- Playwright, Puppeteer, Selenium 네이티브 지원
- 대량 추출을 위한 무제한 확장성

무엇보다도, 몇 줄의 코드만으로 워크플로에 통합할 수 있습니다.

### Setting Up Scraping Browser

Scraping Browser를 시작하려면:

[Bright Data 계정 생성](https://brightdata.co.kr/)을 진행합니다(신규 사용자는 결제 수단 추가 후 $5 크레딧을 받습니다). 그런 다음 대시보드에서 **Proxies & Scraping**으로 이동해 **Get started**를 클릭합니다.

![brightdata-scraping-solutions-dashboard](https://github.com/luminati-io/llama-3-web-scraping/blob/main/images/brightdata-scraping-solutions-dashboard.png)

새 zone(예: _test\_browser_)을 생성하고, _Premium domains_ 및 [CAPTCHA solver](https://brightdata.co.kr/products/web-unlocker/captcha-solver) 같은 기능을 활성화합니다.

![brightdata-create-scraping-browser-zone](https://github.com/luminati-io/llama-3-web-scraping/blob/main/images/brightdata-create-scraping-browser-zone.png)

다음으로, 대시보드에서 Selenium URL을 복사합니다.

![brightdata-selenium-connection-credentials](https://github.com/luminati-io/llama-3-web-scraping/blob/main/images/brightdata-selenium-connection-credentials.png)

### Modifying Your Code for Scraping Browser

Scraping Browser를 통해 연결하도록 `initialize_web_driver` 함수를 업데이트합니다:

```python
from selenium.webdriver import Remote
from selenium.webdriver.chrome.options import Options as ChromeOptions
from selenium.webdriver.chromium.remote_connection import ChromiumRemoteConnection

SBR_WEBDRIVER = "<https://username:password@host>:port"

def initialize_web_driver():
    options = ChromeOptions()
    sbr_connection = ChromiumRemoteConnection(SBR_WEBDRIVER, "goog", "chrome")
    driver = Remote(sbr_connection, options=options)
    return driver
```

이제 스크레이퍼는 Bright Data 인프라를 통해 라우팅되며, Amazon 및 기타 アンチボット 시스템을 손쉽게 처리합니다.

## Enhancing and Expanding Your Scraper

향후 다음과 같은 개선 사항을 추가할 수 있습니다:

- URL 및 프롬프트 인자를 구성 가능하게 만들기
- `.env` 파일에서 자격 증명을 안전하게 로드하기
- 멀티 페이지 スクレイピング 및 [pagination handling](https://brightdata.co.kr/blog/web-data/pagination-web-scraping) 지원
- [other marketplaces](https://brightdata.co.kr/blog/how-tos/ecommerce-web-scraping-guide)로 スクレイピング 확장
- Google 서비스에서 데이터 추출:
  - [Google Flights](https://github.com/luminati-io/google-flights-api)
  - [Google Search](https://github.com/luminati-io/google-search-api)
  - [Google Trends](https://github.com/luminati-io/google-trends-api)
- 다음과 같은 다양한 LLM 통합 탐색:
  - [Gemini](https://brightdata.co.kr/blog/web-data/web-scraping-with-gemini)
  - [Perplexity](https://brightdata.co.kr/blog/web-data/web-scraping-with-perplexity)
  - [Crawl4AI and DeepSeek](https://brightdata.co.kr/blog/web-data/crawl4ai-and-deepseek-web-scraping)

## Conclusion

이 가이드는 LLaMA 3로 신뢰할 수 있고 지능적인 스크레이퍼를 만들 수 있도록 구성되어 있습니다.

최고의 スクレイピング 성과를 위해 LLaMA의 추론 능력과 [Bright Data’s Scraping Browser](https://brightdata.co.kr/products/scraping-browser) 같은 도구의 인프라를 결합하시기 바랍니다.

한 단계 더 성장할 준비가 되셨습니까? 무료로 [Bright Data의 전체 스크레이핑 스위트](https://brightdata.co.kr/)를 사용해 보시기 바랍니다!