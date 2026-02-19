# research-data-clients

[![PyPI](https://img.shields.io/pypi/v/research-data-clients.svg)](https://pypi.org/project/research-data-clients/)
[![Python 3.9+](https://img.shields.io/badge/python-3.9+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Python clients for 18 external data APIs. No LLM dependencies. Just clean access to academic literature, government data, news, web archives, and more.

Each client follows the same pattern: instantiate, call a method, get a typed result back.

## Install

```bash
pip install research-data-clients

# arXiv support requires an extra dependency
pip install research-data-clients[arxiv]

# Everything
pip install research-data-clients[all]
```

## Clients

| Client | Source | API Key |
|--------|--------|---------|
| `ArxivClient` | Academic preprints | No |
| `SemanticScholarClient` | Papers with citation counts | No |
| `PubMedClient` | Biomedical literature | No (optional for higher limits) |
| `WikipediaClient` | Article content, multilingual | No |
| `ArchiveClient` | Wayback Machine snapshots | No |
| `MultiArchiveClient` | Wayback, Archive.is, Memento, 12ft | No |
| `WeatherClient` | NOAA forecasts and alerts | No |
| `OpenLibraryClient` | Book metadata by ISBN/title | No |
| `CensusClient` | ACS, SAIPE, Decennial Census | Yes (free) |
| `NASAClient` | APOD, Mars photos, NEO, Earth imagery | Yes (free, or `DEMO_KEY`) |
| `GitHubClient` | Repos, code search, issues | Optional |
| `FECClient` | Campaign finance, contributions | Yes (free) |
| `NewsClient` | Headlines and article search | Yes |
| `YouTubeClient` | Video metadata | Yes |
| `FinanceClient` | Stock and forex data (Alpha Vantage) | Yes |
| `WolframAlphaClient` | Computation and factual queries | Yes |
| `MyAnimeListClient` | Anime and manga metadata | Yes |
| `JudiciaryClient` | Federal judiciary disclosures | No |

## Quick start

```python
from research_data_clients import ArxivClient

client = ArxivClient()
papers = client.search("transformer attention mechanism", max_results=5)

for paper in papers:
    print(paper.title)
    print(paper.authors)
    print(paper.pdf_url)
    print()
```

### Factory

To instantiate by name instead of importing directly:

```python
from research_data_clients import ClientFactory

arxiv = ClientFactory.create_client("arxiv")
github = ClientFactory.create_client("github", api_key="your-token")
weather = ClientFactory.create_client("weather")

# See all available names
ClientFactory.list_sources()
```

## Usage by client

### ArxivClient

Requires `pip install research-data-clients[arxiv]`.

```python
from research_data_clients import ArxivClient, search_arxiv, get_paper_by_id

client = ArxivClient()

# Text search
papers = client.search("protein folding", max_results=10, sort_by="relevance")
# sort_by can be "relevance" or "date"

# By author
papers = client.search_by_author("Yann LeCun", max_results=5)

# By category
papers = client.search_by_category("cs.LG", max_results=10, sort_by="date")

# Fetch specific paper
paper = client.get_by_id("2301.07041")
print(paper.title)
print(paper.categories)   # ['cs.LG', 'cs.AI']
print(paper.doi)          # None or DOI string

# Convenience function (returns plain dicts)
results = search_arxiv("neural networks", max_results=5)
```

`ArxivPaper` fields: `title`, `authors`, `summary`, `published`, `updated`, `arxiv_id`, `pdf_url`, `categories`, `entry_id`, `doi`, `comment`, `journal_ref`, `primary_category`.

### SemanticScholarClient

```python
from research_data_clients import SemanticScholarClient, search_papers, get_paper_by_doi

client = SemanticScholarClient()

# Search
papers = client.search("BERT language model", limit=10)

for paper in papers:
    print(paper.title)
    print(f"Citations: {paper.citation_count}")
    print(f"Influential citations: {paper.influential_citation_count}")

# Lookup by DOI
paper = client.get_by_doi("10.18653/v1/N19-1423")

# Lookup by arXiv ID
paper = client.get_by_arxiv_id("2301.07041")

# Async version
papers = await client.search_async("graph neural networks", limit=5)
```

`SemanticScholarPaper` fields: `title`, `authors`, `year`, `abstract`, `doi`, `keywords`, `venue`, `url`, `paper_id`, `citation_count`, `reference_count`, `influential_citation_count`.

### PubMedClient

```python
from research_data_clients import PubMedClient, search_pubmed

client = PubMedClient()
# Optional: PubMedClient(api_key="your-ncbi-key", email="you@example.com")

# Basic search
articles = client.search("COVID-19 mRNA vaccine", max_results=10)

# Specialized searches
trials = client.search_clinical_trials("type 2 diabetes", max_results=5)
reviews = client.search_reviews("CRISPR gene editing", max_results=5)
by_mesh = client.search_by_mesh("Alzheimer Disease", max_results=5)
by_author = client.search_by_author("Fauci AS", max_results=10)

# Filter by journal and date range
articles = client.search(
    "sepsis treatment",
    journal="New England Journal of Medicine",
    date_range="2022/01/01:2024/01/01"
)

# Fetch by PMID
article = client.get_by_id("33495306")
print(article.url)   # https://pubmed.ncbi.nlm.nih.gov/33495306/
```

`PubMedArticle` fields: `pmid`, `title`, `authors`, `journal`, `publication_date`, `abstract`, `doi`, `publication_types`, `keywords`, `mesh_terms`. The `.url` property builds the PubMed link automatically.

### WikipediaClient

```python
from research_data_clients import WikipediaClient

client = WikipediaClient()                  # English
client_de = WikipediaClient(language="de")  # German

# Search
results = client.search("quantum entanglement", limit=5)
# Returns list of {title, description, url}

# Summary (intro section only)
data = client.get_summary("Quantum entanglement")
print(data["summary"])
print(data["image"])

# Full article text
data = client.get_full_content("Quantum entanglement")
print(f"{data['word_count']} words")

# Random articles
data = client.get_random(limit=3)
```

### ArchiveClient

```python
from research_data_clients import ArchiveClient, archive_url, get_latest_archive

client = ArchiveClient()

# Get most recent snapshot
snapshot = client.get_latest_snapshot("https://example.com")
if snapshot:
    print(snapshot.archive_url)
    print(snapshot.timestamp)

# Request a new archive
result = client.archive_url("https://example.com", wait_for_completion=True)
if result.success:
    print(result.archive_url)

# Snapshot nearest to a date
from datetime import datetime
snapshot = client.get_snapshot_at_timestamp("https://example.com", datetime(2020, 6, 1))

# Full history
snapshots = client.get_all_snapshots("https://example.com", limit=50)

# Convenience functions
archived_url = get_latest_archive("https://example.com")
result = archive_url("https://example.com")
```

### MultiArchiveClient

Tries multiple archive providers: Wayback Machine, Archive.is, Memento, and 12ft.io.

```python
from research_data_clients import MultiArchiveClient, get_archive

client = MultiArchiveClient()

result = client.get_archive("https://example.com", provider="wayback")
result = client.get_archive("https://example.com", provider="archiveis")
result = client.get_archive("https://example.com", provider="memento")
result = client.get_archive("https://example.com", provider="12ft")

# Try all providers at once
results = client.get_all_archives("https://example.com")
for provider, result in results.items():
    if result.success:
        print(f"{provider}: {result.archive_url}")

# Convenience function
url = get_archive("https://example.com", provider="wayback")
```

### WeatherClient

Uses NOAA's public API. No key required.

```python
from research_data_clients import WeatherClient

client = WeatherClient()

# Current conditions (US coordinates)
weather = client.get_current_weather(latitude=40.7128, longitude=-74.0060)

# Forecast
forecast = client.get_forecast(latitude=37.7749, longitude=-122.4194)
```

### CensusClient

Returns pandas DataFrames.

```python
from research_data_clients import CensusClient

client = CensusClient(api_key="your-census-key", use_cache=True)
# API key is free at https://api.census.gov/data/key_signup.html

# Population by county
df = client.fetch_population(year=2022)
print(df[["name", "total_population"]].head())

# Any ACS variables
df = client.fetch_acs(
    year=2022,
    variables={
        "B27001_001E": "total_pop",
        "B27001_005E": "uninsured_pop"
    },
    geography="county:*"
)

# State-level poverty estimates
df = client.fetch_saipe(year=2022, geography="state")
print(df[["name", "poverty_rate"]].head())

# Limit to one state (FIPS code)
ca_df = client.fetch_population(year=2022, geography="county:*", state="06")

# Metadata and caching
client.save_metadata("census_run.json")
client.clear_cache()
```

### NASAClient

```python
from research_data_clients import NASAClient

client = NASAClient()  # Uses DEMO_KEY if NASA_API_KEY not set
# Free key at https://api.nasa.gov/

# Astronomy Picture of the Day
apod = client.get_apod()                    # Today
apod = client.get_apod(date="2024-01-01")  # Specific date
apod = client.get_apod(count=5)            # 5 random APODs

# Mars rover photos
result = client.get_mars_photos(sol=1000, rover="curiosity", camera="MAST")
for photo in result["photos"]:
    print(photo["img_src"])

# Near Earth Objects
neos = client.get_neo(start_date="2024-01-01", end_date="2024-01-07")
print(f"{neos['element_count']} objects tracked")

# Earth satellite imagery
imagery = client.get_earth_imagery(lat=29.78, lon=-95.37, date="2020-01-01")
```

### GitHubClient

```python
from research_data_clients import GitHubClient

client = GitHubClient()  # Anonymous: 60 req/hour
client = GitHubClient(api_key="ghp_your_token")  # Authenticated: 5000 req/hour
# Also reads GITHUB_TOKEN or GITHUB_API_KEY from environment

# Search repositories
result = client.search_repositories(
    query="language:python stars:>1000 topic:machine-learning",
    sort="stars",
    order="desc",
    per_page=20
)
for repo in result["repositories"]:
    print(f"{repo['name']} ({repo['stars']} stars)")

# Get specific repo
repo = client.get_repository("owner", "repo-name")

# Code search
result = client.search_code("addClass in:file language:js")

# Issue search
result = client.search_issues("repo:python/cpython is:open label:bug")
```

### FECClient

```python
from research_data_clients import FECClient

client = FECClient(api_key="your-fec-key")
# Free key at https://api.open.fec.gov/developers/

# Search candidates
result = client.search_candidates(
    name="Smith",
    office="S",         # H=House, S=Senate, P=President
    state="CA",
    party="DEM",
    cycle=2024
)

# Financial totals for a candidate
totals = client.get_candidate_totals("P80001571", cycle=2024)
print(f"Raised: ${totals['receipts']:,.0f}")
print(f"Spent: ${totals['disbursements']:,.0f}")

# Committee info and spending
info = client.get_committee_info("C00000059")
disbursements = client.get_disbursements("C00000059", min_amount=10000)

# Individual contributions
contribs = client.search_individual_contributions(
    contributor_name="Bezos",
    min_amount=2500
)
```

### NewsClient

```python
from research_data_clients import NewsClient

client = NewsClient(api_key="your-news-api-key")
# Key at https://newsapi.org/

# Top headlines
result = client.get_top_headlines(country="us", category="technology")

# Search all articles
result = client.search_everything(
    query="climate policy",
    from_date="2024-01-01",
    to_date="2024-06-01",
    sort_by="publishedAt"
)

# List available sources
sources = client.get_sources(category="science", language="en")
```

### WolframAlphaClient

```python
from research_data_clients import WolframAlphaClient, wolfram_query, wolfram_calculate

client = WolframAlphaClient(api_key="your-app-id")
# Key at https://developer.wolframalpha.com/

# Short text answer
result = client.query("population of Tokyo")
print(result.result)  # "13.96 million people (2023)"

# Math
result = client.calculate("integrate x^2 from 0 to 1")
result = client.convert("100", "miles", "kilometers")
result = client.define("perihelion")

# Spoken-style answer
result = client.query_spoken("what is the speed of light")

# Full structured response with all pods
result = client.query_full("derivative of sin(x)*cos(x)")
for pod in result.pods:
    print(pod["title"])
    for subpod in pod["subpods"]:
        print(f"  {subpod['plaintext']}")

# Convenience functions
answer = wolfram_query("boiling point of water in fahrenheit")
answer = wolfram_calculate("sqrt(2) * pi")
```

### FinanceClient (Alpha Vantage)

```python
from research_data_clients import FinanceClient

client = FinanceClient(api_key="your-alphavantage-key")
# Free key at https://www.alphavantage.co/support/#api-key

# Daily stock prices (returns list of {date, open, high, low, close, volume})
result = client.get_daily_time_series("AAPL")
result = client.get_daily_time_series("TSLA", output_size="full")  # full history
for day in result["prices"][:5]:
    print(f"{day['date']}: close={day['adjusted_close']}")

# Forex
rate = client.get_fx_rate("USD", "EUR")
print(rate["exchange_rate"])

# Crypto
quote = client.get_crypto_quote("BTC", market="USD")
```

### YouTubeClient

```python
from research_data_clients import YouTubeClient

client = YouTubeClient(api_key="your-youtube-key")
# Key at https://console.cloud.google.com/ (YouTube Data API v3)

# Search videos
result = client.search_videos(
    "python tutorial",
    max_results=10,
    order="viewCount",         # relevance, viewCount, date, rating
    video_duration="medium"    # any, short, medium, long
)
for video in result.get("items", []):
    print(video["title"], video["url"])

# Channel statistics
stats = client.get_channel_statistics("UC_x5XG1OV2P6uZZ5FSM9Ttw")

# Playlist items
items = client.get_playlist_items("PLxxxxxxxx", max_results=20)
```

### MyAnimeListClient

```python
from research_data_clients.mal_client import MyAnimeListClient

client = MyAnimeListClient(api_key="your-mal-client-id")
# Register at https://myanimelist.net/apiconfig

# Search anime
results = client.search_anime("Cowboy Bebop", limit=5)

# Get full details by ID
details = client.get_anime_details(1)  # ID 1 = Cowboy Bebop

# Seasonal anime
seasonal = client.get_season_anime(year=2024, season="winter")
# season: winter, spring, summer, fall
```

### OpenLibraryClient

```python
from research_data_clients import OpenLibraryClient

client = OpenLibraryClient()

# Search by title, author, or subject
result = client.search_books("Dune", limit=5)

# Look up by ISBN
book = client.get_book_by_isbn("9780441013593")

# Author details (use Open Library author key like /authors/OL26320A)
author = client.get_author("/authors/OL26320A")

# Browse by subject
result = client.get_subjects("science fiction", limit=10)
```

## API keys

Most keys are free to obtain. Set them as environment variables:

```bash
export CENSUS_API_KEY=...        # https://api.census.gov/data/key_signup.html
export NASA_API_KEY=...          # https://api.nasa.gov/
export NEWS_API_KEY=...          # https://newsapi.org/
export ALPHAVANTAGE_API_KEY=...  # https://www.alphavantage.co/support/#api-key
export WOLFRAMALPHA_APP_ID=...   # https://developer.wolframalpha.com/
export FEC_API_KEY=...           # https://api.open.fec.gov/developers/
export GITHUB_TOKEN=...          # https://github.com/settings/tokens
export MAL_API_KEY=...           # https://myanimelist.net/apiconfig
export YOUTUBE_API_KEY=...       # https://console.cloud.google.com/
```

Or pass directly to the constructor:

```python
client = NASAClient(api_key="...")
```

Clients without API keys (arXiv, Semantic Scholar, PubMed, Wikipedia, NOAA, OpenLibrary, Wayback Machine) work out of the box.

## Data classes

A few clients return typed dataclasses instead of plain dicts:

```python
from research_data_clients import (
    ArxivPaper,          # title, authors, summary, pdf_url, categories, doi, ...
    SemanticScholarPaper, # title, authors, citation_count, venue, ...
    PubMedArticle,       # pmid, title, journal, abstract, .url property
    ArchivedSnapshot,    # url, timestamp, status_code, archive_url
    ArchiveResult,       # success, archive_url, snapshot, error
    WolframResult,       # success, query, result, result_type, pods
)

# All support .to_dict()
paper = ArxivClient().get_by_id("2301.07041")
d = paper.to_dict()
```

## Error handling

```python
import requests
from research_data_clients import ArxivClient

try:
    client = ArxivClient()
    papers = client.search("quantum computing")
except requests.exceptions.RequestException as e:
    print(f"Network error: {e}")
except ValueError as e:
    print(f"Bad parameter: {e}")
```

Clients that return dicts (GitHub, NASA, Wikipedia, etc.) include an `"error"` key when something goes wrong rather than raising, so check for it:

```python
result = github.search_repositories("python framework")
if "error" in result:
    print(result["error"])
else:
    for repo in result["repositories"]:
        ...
```

## Requirements

- Python 3.9+
- `requests` (only required dependency for most clients)
- `arxiv>=2.0.0` — required for `ArxivClient` (`pip install research-data-clients[arxiv]`)
- `feedparser>=6.0.0` — required for RSS-based news clients (`pip install research-data-clients[rss]`)
- `pandas` — required for `CensusClient`

## License

MIT — see [LICENSE](LICENSE)

## Author

Luke Steuber — [lukesteuber.com](https://lukesteuber.com)
