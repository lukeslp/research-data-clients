# Research Data Clients

Structured API clients for 17 external data sources. Provides consistent interfaces for retrieving data from academic, government, news, and web APIs.

## Features

- **Unified Factory Pattern**: Create any client with `ClientFactory.create_client('arxiv')`
- **Consistent Interface**: All clients follow similar patterns
- **Dataclasses**: Structured return types for papers, articles, snapshots
- **Convenience Functions**: Quick helpers for common operations
- **Full Type Hints**: IDE-friendly development

## Available Clients

| Client | Source | Requires API Key |
|--------|--------|------------------|
| ArxivClient | Academic papers from arXiv.org | No |
| SemanticScholarClient | Research papers with citations | No |
| PubMedClient | Biomedical literature from NCBI | No |
| ArchiveClient | Internet Archive / Wayback Machine | No |
| MultiArchiveClient | Wayback, Archive.is, Memento, 12ft | No |
| CensusClient | US Census Bureau data | Yes |
| FECClient | Campaign finance data | Yes |
| JudiciaryClient | Court records | No |
| GitHubClient | Repository and user data | Optional |
| NASAClient | Space imagery and data | Yes (free) |
| NewsClient | News articles | Yes |
| WikipediaClient | Wikipedia content | No |
| WeatherClient | Weather forecasts | Yes |
| OpenLibraryClient | Book metadata | No |
| YouTubeClient | Video metadata | Yes |
| FinanceClient | Stock data | Yes |
| MALClient | Anime/manga data | Yes |
| WolframAlphaClient | Computational queries | Yes |

## Installation

```bash
pip install research-data-clients
```

## Quick Start

### Using the Factory

```python
from research_data_clients import ClientFactory

# Create any client by name
arxiv = ClientFactory.create_client('arxiv')
papers = arxiv.search('machine learning', max_results=10)

github = ClientFactory.create_client('github', token='your-token')
repos = github.search_repositories('python framework')

# List all available clients
sources = ClientFactory.list_sources()
```

### ArxivClient - Academic Papers

```python
from research_data_clients import ArxivClient, search_arxiv

client = ArxivClient()
papers = client.search(
    query='quantum computing',
    max_results=10,
    sort_by='relevance'
)

for paper in papers:
    print(f"{paper.title}")
    print(f"Authors: {paper.authors}")
    print(f"PDF: {paper.pdf_url}")

# Convenience function
results = search_arxiv('neural networks', max_results=5)
```

### SemanticScholarClient - Citations

```python
from research_data_clients import SemanticScholarClient, get_paper_by_doi

client = SemanticScholarClient()
papers = client.search(
    query='transformers attention',
    limit=10,
    fields=['title', 'authors', 'citations']
)

# Get paper by DOI
paper = get_paper_by_doi('10.1000/xyz123')
print(f"Citations: {paper.citation_count}")
```

### PubMedClient - Medical Literature

```python
from research_data_clients import PubMedClient

client = PubMedClient()
articles = client.search('COVID-19 treatment', max_results=10)

# Specialized searches
trials = client.search_clinical_trials('diabetes', max_results=5)
reviews = client.search_reviews('cancer immunotherapy', max_results=5)
by_mesh = client.search_by_mesh('Alzheimer Disease', max_results=5)
by_author = client.search_by_author('Fauci AS', max_results=5)
```

### ArchiveClient - Wayback Machine

```python
from research_data_clients import ArchiveClient, archive_url

client = ArchiveClient()

# Save page to archive
snapshot = client.save_page('https://example.com')
print(f"Archived: {snapshot.archive_url}")

# Get historical snapshots
snapshots = client.get_snapshots(
    url='https://example.com',
    from_date='2020-01-01',
    to_date='2024-01-01'
)

# Convenience function
archived = archive_url('https://example.com')
```

### MultiArchiveClient - Multiple Providers

```python
from research_data_clients import MultiArchiveClient

client = MultiArchiveClient()

# Try specific provider
result = client.get_archive('https://example.com', provider='wayback')
result = client.get_archive('https://example.com', provider='archiveis')
result = client.get_archive('https://example.com', provider='memento')
result = client.get_archive('https://example.com', provider='12ft')

# Try all providers
all_results = client.get_all_archives('https://example.com')
for provider, result in all_results.items():
    if result.success:
        print(f"{provider}: {result.archive_url}")
```

### GitHubClient - Repositories

```python
from research_data_clients import GitHubClient

client = GitHubClient(token='your-github-token')

# Search repos
repos = client.search_repositories(
    query='machine learning',
    language='python',
    sort='stars'
)

# Get repo details
repo = client.get_repository('owner/repo')
print(f"Stars: {repo['stargazers_count']}")
```

### WikipediaClient - Articles

```python
from research_data_clients import WikipediaClient

client = WikipediaClient()

# Search
results = client.search('quantum physics')

# Get article
article = client.get_article('Quantum_mechanics')
print(article['summary'])

# Different language
article_de = client.get_article('Quantenmechanik', lang='de')
```

### NASAClient - Space Data

```python
from research_data_clients import NASAClient

client = NASAClient(api_key='your-nasa-key')

# Astronomy Picture of the Day
apod = client.get_apod(date='2024-01-01')

# Mars rover photos
photos = client.get_mars_photos(
    rover='curiosity',
    sol=1000,
    camera='FHAZ'
)

# Near Earth Objects
neos = client.get_near_earth_objects(
    start_date='2024-01-01',
    end_date='2024-01-07'
)
```

### WolframAlphaClient - Computation

```python
from research_data_clients import WolframAlphaClient, wolfram_query

client = WolframAlphaClient(api_key='your-wolfram-key')

# Simple query
result = client.query('What is the population of France?')
print(result.result)

# Mathematical calculation
calc = client.calculate('integrate x^2 from 0 to 1')

# Unit conversion
converted = client.convert('100', 'miles', 'kilometers')

# Full query with all pods
full = client.query_full('derivative of sin(x)')
```

### CensusClient - Demographics

```python
from research_data_clients import CensusClient

client = CensusClient(api_key='your-census-key')

# Population data
data = client.get_population(
    year=2020,
    geography='state',
    state='CA'
)

# Economic data
econ = client.get_economic_data(
    dataset='acs/acs5',
    year=2020,
    variables=['B01001_001E'],
    geography='county:*',
    state='CA'
)
```

### Other Clients

```python
# News
news = ClientFactory.create_client('news', api_key='key')
articles = news.search('AI regulation', language='en')

# Weather
weather = ClientFactory.create_client('weather', api_key='key')
forecast = weather.get_forecast(lat=37.77, lon=-122.42, days=7)

# YouTube
youtube = ClientFactory.create_client('youtube', api_key='key')
videos = youtube.search('python tutorial', max_results=10)

# Finance
finance = ClientFactory.create_client('finance', api_key='key')
stock = finance.get_quote('AAPL')

# Open Library
books = ClientFactory.create_client('openlibrary')
results = books.search('science fiction')
book = books.get_book_by_isbn('9780451524935')
```

## Data Classes

```python
from research_data_clients import (
    ArxivPaper,
    SemanticScholarPaper,
    PubMedArticle,
    ArchivedSnapshot,
    ArchiveResult,
    WolframResult
)

# Structured return types with attributes
paper = ArxivPaper(
    id='2103.12345',
    title='Paper Title',
    authors=['Author 1', 'Author 2'],
    abstract='...',
    published='2021-03-15',
    pdf_url='https://arxiv.org/pdf/...'
)
```

## API Key Management

Set via environment variables:

```bash
export CENSUS_API_KEY=...
export NASA_API_KEY=...
export NEWS_API_KEY=...
export WEATHER_API_KEY=...
export YOUTUBE_API_KEY=...
export ALPHAVANTAGE_API_KEY=...
export WOLFRAM_APP_ID=...
export GITHUB_TOKEN=...
```

Or pass directly:

```python
client = NASAClient(api_key='your-key-here')
```

## Error Handling

```python
import requests

try:
    client = ArxivClient()
    papers = client.search('query')
except requests.exceptions.RequestException as e:
    print(f"Network error: {e}")
except ValueError as e:
    print(f"Invalid parameters: {e}")
```

## Dependencies

Core: `requests`

Optional (for specific clients):
- `arxiv` - ArxivClient
- `feedparser` - RSS parsing

## License

MIT License - see LICENSE file

## Author

Luke Steuber
