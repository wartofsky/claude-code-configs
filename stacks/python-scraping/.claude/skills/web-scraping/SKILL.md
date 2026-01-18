---
name: web-scraping
description: Web scraping patterns with httpx and BeautifulSoup. Use when crawling websites, extracting data from HTML, finding links, or parsing structured content. Triggers on scrape, crawl, BeautifulSoup, parse HTML keywords.
---

# Web Scraping Patterns

## Async HTTP Client

```python
import httpx
from typing import AsyncGenerator

class ScraperClient:
    def __init__(self):
        self.client = httpx.AsyncClient(
            timeout=30.0,
            follow_redirects=True,
            headers={
                "User-Agent": "Mozilla/5.0 (compatible; SchoolBot/1.0)",
                "Accept": "text/html,application/xhtml+xml",
                "Accept-Language": "en-US,en;q=0.9"
            }
        )
    
    async def __aenter__(self):
        return self
    
    async def __aexit__(self, *args):
        await self.client.aclose()
    
    async def get(self, url: str) -> httpx.Response:
        return await self.client.get(url)
    
    async def get_text(self, url: str) -> str:
        response = await self.get(url)
        response.raise_for_status()
        return response.text
```

## BeautifulSoup Parsing

```python
from bs4 import BeautifulSoup, Tag

def parse_html(html: str) -> BeautifulSoup:
    return BeautifulSoup(html, "html.parser")

# Select elements
soup = parse_html(html)

# CSS selectors (preferred)
rows = soup.select("table.staff tr")
links = soup.select("a[href^='mailto:']")
divs = soup.select("div.card, div.staff-member")

# Find methods
title = soup.find("h1")
all_links = soup.find_all("a", href=True)

# Get text content
text = element.get_text(strip=True)
text_with_sep = element.get_text(separator=" ", strip=True)

# Get attributes
href = link.get("href", "")
classes = element.get("class", [])
```

## Email Extraction

```python
import re
from urllib.parse import unquote

EMAIL_PATTERN = re.compile(r'[\w.+-]+@[\w-]+\.[\w.-]+')

def extract_emails(soup: BeautifulSoup) -> list[str]:
    emails = set()
    
    # From mailto links
    for link in soup.select("a[href^='mailto:']"):
        href = link["href"]
        email = href.replace("mailto:", "").split("?")[0]
        email = unquote(email).strip().lower()
        if EMAIL_PATTERN.match(email):
            emails.add(email)
    
    # From text content
    text = soup.get_text()
    for match in EMAIL_PATTERN.findall(text):
        # Filter out image files and common false positives
        if not any(ext in match.lower() for ext in ['.png', '.jpg', '.gif']):
            emails.add(match.lower())
    
    return list(emails)
```

## Staff Directory Parsing

```python
from dataclasses import dataclass

@dataclass
class StaffMember:
    name: str
    email: str | None
    title: str | None
    phone: str | None

def parse_staff_table(soup: BeautifulSoup) -> list[StaffMember]:
    """Parse staff from table format."""
    staff = []
    
    for row in soup.select("table tr"):
        cells = row.find_all(["td", "th"])
        if len(cells) < 2:
            continue
        
        name = cells[0].get_text(strip=True)
        if not name or name.lower() in ["name", "staff", "employee"]:
            continue
        
        staff.append(StaffMember(
            name=name,
            email=find_email_in_element(row),
            title=cells[1].get_text(strip=True) if len(cells) > 1 else None,
            phone=find_phone_in_element(row)
        ))
    
    return staff

def parse_staff_cards(soup: BeautifulSoup) -> list[StaffMember]:
    """Parse staff from card/div format."""
    staff = []
    
    selectors = [
        ".staff-card", ".team-member", ".directory-item",
        ".staff-member", ".employee-card", "[class*='staff']"
    ]
    
    for selector in selectors:
        for card in soup.select(selector):
            name_el = card.select_one("h2, h3, h4, .name, .title")
            if not name_el:
                continue
            
            staff.append(StaffMember(
                name=name_el.get_text(strip=True),
                email=find_email_in_element(card),
                title=find_title_in_element(card),
                phone=find_phone_in_element(card)
            ))
    
    return staff
```

## Link Discovery

```python
from urllib.parse import urljoin, urlparse

def extract_links(soup: BeautifulSoup, base_url: str) -> list[str]:
    """Extract all valid links from page."""
    links = []
    
    for anchor in soup.find_all("a", href=True):
        href = anchor["href"]
        
        # Skip javascript, anchors, mailto
        if href.startswith(("#", "javascript:", "mailto:", "tel:")):
            continue
        
        # Make absolute
        absolute_url = urljoin(base_url, href)
        
        # Validate
        parsed = urlparse(absolute_url)
        if parsed.scheme in ("http", "https"):
            links.append(absolute_url)
    
    return list(set(links))

def find_directory_links(soup: BeautifulSoup, base_url: str) -> list[str]:
    """Find links that might be staff directories."""
    keywords = ["staff", "directory", "faculty", "team", "about", "contact"]
    directory_links = []
    
    for link in soup.find_all("a", href=True):
        href = link["href"].lower()
        text = link.get_text().lower()
        
        if any(kw in href or kw in text for kw in keywords):
            absolute = urljoin(base_url, link["href"])
            directory_links.append(absolute)
    
    return directory_links
```

## Rate Limiting

```python
import asyncio
from collections import defaultdict
from urllib.parse import urlparse

class RateLimiter:
    def __init__(self, requests_per_second: float = 1.0):
        self.delay = 1.0 / requests_per_second
        self.last_request: dict[str, float] = defaultdict(float)
        self._lock = asyncio.Lock()
    
    async def acquire(self, url: str):
        domain = urlparse(url).netloc
        
        async with self._lock:
            now = asyncio.get_event_loop().time()
            elapsed = now - self.last_request[domain]
            
            if elapsed < self.delay:
                await asyncio.sleep(self.delay - elapsed)
            
            self.last_request[domain] = asyncio.get_event_loop().time()

# Usage
rate_limiter = RateLimiter(requests_per_second=2.0)

async def scrape_with_rate_limit(url: str):
    await rate_limiter.acquire(url)
    return await client.get(url)
```

## Crawling

```python
async def crawl_site(
    start_url: str,
    max_pages: int = 50,
    same_domain_only: bool = True
) -> list[str]:
    """Crawl a website and return all page URLs."""
    visited = set()
    to_visit = [start_url]
    start_domain = urlparse(start_url).netloc
    
    async with ScraperClient() as client:
        while to_visit and len(visited) < max_pages:
            url = to_visit.pop(0)
            
            if url in visited:
                continue
            
            if same_domain_only and urlparse(url).netloc != start_domain:
                continue
            
            try:
                await rate_limiter.acquire(url)
                html = await client.get_text(url)
                visited.add(url)
                
                soup = parse_html(html)
                links = extract_links(soup, url)
                
                for link in links:
                    if link not in visited and link not in to_visit:
                        to_visit.append(link)
                        
            except Exception as e:
                logging.warning(f"Failed to crawl {url}: {e}")
    
    return list(visited)
```

## Error Handling

```python
class ScrapingError(Exception):
    def __init__(self, url: str, message: str):
        self.url = url
        super().__init__(f"{url}: {message}")

async def safe_scrape(url: str) -> str | None:
    """Scrape with graceful error handling."""
    try:
        response = await client.get(url)
        
        if response.status_code == 404:
            return None
        
        response.raise_for_status()
        return response.text
        
    except httpx.TimeoutException:
        raise ScrapingError(url, "Request timed out")
    except httpx.HTTPStatusError as e:
        raise ScrapingError(url, f"HTTP {e.response.status_code}")
    except httpx.RequestError as e:
        raise ScrapingError(url, str(e))
```
