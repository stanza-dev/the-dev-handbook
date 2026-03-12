---
source_course: "redis-probabilistic"
source_lesson: "redis-probabilistic-bloom-patterns"
---

# Bloom Filter Production Patterns

## Introduction

Bloom filters shine in production when used as a fast gate in front of expensive operations. This lesson covers four battle-tested patterns — URL deduplication, spam detection, cache pre-checks, and recommendation filtering — that you can adapt to your own systems.

## Key Concepts

- **Pre-filter Pattern**: Check the Bloom filter before hitting the database; skip the lookup entirely if the filter says "not present"
- **Write-Through**: Add items to the Bloom filter at the same time you write to primary storage, keeping them in sync
- **Filter Layering**: Use multiple Bloom filters for different purposes (e.g., one per time window, one per feature)
- **Rebuild Strategy**: Since Bloom filters cannot delete items, periodic rebuilds from the source of truth keep the filter accurate

## Real World Context

Web crawlers at scale (Googlebot, Common Crawl) must avoid re-crawling billions of URLs. Keeping all visited URLs in a hash table costs terabytes. A Bloom filter with 1% error rate for 10 billion URLs costs roughly 12 GB — orders of magnitude smaller. The occasional false positive (re-crawling a URL) is far cheaper than the memory cost of exact deduplication.

## Deep Dive

### Pattern 1: URL Deduplication for Web Crawlers

```python
import redis
from queue import Queue

r = redis.Redis()

class WebCrawler:
    def __init__(self, expected_urls=10_000_000, error_rate=0.001):
        self.filter_key = 'crawler:visited'
        try:
            r.execute_command('BF.RESERVE', self.filter_key,
                             error_rate, expected_urls, 'EXPANSION', 2)
        except redis.ResponseError:
            pass
        self.queue = Queue()

    def should_crawl(self, url):
        """Check if URL has already been crawled."""
        return r.execute_command('BF.EXISTS', self.filter_key, url) == 0

    def mark_crawled(self, url):
        """Record that URL has been crawled."""
        r.execute_command('BF.ADD', self.filter_key, url)

    def process_url(self, url):
        """Crawl URL if not seen before."""
        if not self.should_crawl(url):
            return  # Skip — already visited or false positive
        
        content = self.fetch(url)
        self.mark_crawled(url)
        
        for link in self.extract_links(content):
            if self.should_crawl(link):
                self.queue.put(link)
```

### Pattern 2: Email Spam Detection

```python
class SpamFilter:
    def __init__(self):
        self.filters = {
            'domains': 'spam:domains',
            'ips': 'spam:ips',
            'phrases': 'spam:phrases'
        }
        for key in self.filters.values():
            try:
                r.execute_command('BF.RESERVE', key, 0.001, 500000)
            except redis.ResponseError:
                pass

    def check_email(self, sender_domain, sender_ip, body_phrases):
        """Multi-layer spam check using Bloom filters."""
        if r.execute_command('BF.EXISTS', self.filters['domains'], sender_domain):
            return 'SPAM', 'known spam domain'
        if r.execute_command('BF.EXISTS', self.filters['ips'], sender_ip):
            return 'SPAM', 'known spam IP'
        for phrase in body_phrases:
            if r.execute_command('BF.EXISTS', self.filters['phrases'], phrase):
                return 'SUSPECT', f'suspicious phrase: {phrase}'
        return 'CLEAN', 'passed all checks'
```

### Pattern 3: Cache Warming Checks

```python
class CacheGate:
    def __init__(self):
        self.filter_key = 'cache:populated'
        try:
            r.execute_command('BF.RESERVE', self.filter_key, 0.01, 1_000_000)
        except redis.ResponseError:
            pass

    def get_item(self, item_id):
        """Check Bloom filter before hitting cache or database."""
        # If Bloom says 'no', item was never cached — go straight to DB
        if not r.execute_command('BF.EXISTS', self.filter_key, item_id):
            data = self.fetch_from_db(item_id)
            self.warm_cache(item_id, data)
            return data
        
        # Bloom says 'maybe' — check cache first
        cached = r.get(f'cache:{item_id}')
        if cached:
            return cached
        
        # Cache miss (evicted or false positive)
        data = self.fetch_from_db(item_id)
        self.warm_cache(item_id, data)
        return data

    def warm_cache(self, item_id, data):
        r.setex(f'cache:{item_id}', 3600, data)
        r.execute_command('BF.ADD', self.filter_key, item_id)
```

### Pattern 4: Recommendation Filtering

```python
class RecommendationFilter:
    def __init__(self, user_id):
        self.filter_key = f'seen:{user_id}'
        try:
            r.execute_command('BF.RESERVE', self.filter_key, 0.01, 50000)
        except redis.ResponseError:
            pass

    def filter_recommendations(self, candidates):
        """Remove already-seen items from recommendation list."""
        unseen = []
        for item_id in candidates:
            if not r.execute_command('BF.EXISTS', self.filter_key, item_id):
                unseen.append(item_id)
        return unseen

    def mark_seen(self, item_id):
        r.execute_command('BF.ADD', self.filter_key, item_id)
```

## Common Pitfalls

1. **Forgetting that Bloom filters are append-only** — You cannot remove items. If your use case requires removing entries (e.g., unblocking a domain), consider a Cuckoo filter or pair the Bloom filter with an explicit allow-list.
2. **Not planning for filter rebuilds** — Over time, old items accumulate and inflate the false positive rate. Schedule periodic rebuilds from the source of truth (e.g., nightly, rebuild the spam domain filter from the current blocklist).

## Best Practices

1. **Use BF.MEXISTS for batch checks** — When filtering a list of candidates (like recommendations), batch the checks to reduce round trips: `BF.MEXISTS key item1 item2 item3 ...`
2. **Set TTL on per-user filters** — User-scoped filters (e.g., seen items) should expire to avoid unbounded memory growth. Use `EXPIRE` after creation or let Redis eviction handle cleanup.

## Summary

- URL deduplication: check before crawling, add after crawling
- Spam detection: layer multiple Bloom filters (domain, IP, phrases) for multi-signal checks
- Cache warming: use the Bloom filter to skip unnecessary cache lookups
- Recommendation filtering: remove already-seen items from candidate lists with BF.MEXISTS
- Plan for rebuilds since Bloom filters are append-only

## Code Examples

**URL deduplication for a web crawler using a Bloom filter**

```python
import redis

r = redis.Redis()

class WebCrawler:
    def __init__(self, expected_urls=10_000_000, error_rate=0.001):
        self.filter_key = 'crawler:visited'
        try:
            r.execute_command('BF.RESERVE', self.filter_key,
                             error_rate, expected_urls, 'EXPANSION', 2)
        except redis.ResponseError:
            pass

    def should_crawl(self, url):
        return r.execute_command('BF.EXISTS', self.filter_key, url) == 0

    def mark_crawled(self, url):
        r.execute_command('BF.ADD', self.filter_key, url)

    def process_url(self, url):
        if not self.should_crawl(url):
            return
        content = self.fetch(url)
        self.mark_crawled(url)
        for link in self.extract_links(content):
            if self.should_crawl(link):
                self.queue.put(link)
```


## Resources

- [Bloom Filters](https://redis.io/docs/latest/develop/data-types/probabilistic/bloom-filter/) — Redis Bloom Filter documentation
- [BF.MEXISTS Command](https://redis.io/docs/latest/commands/bf.mexists/) — BF.MEXISTS command reference for batch membership checks

---

> 📘 *This lesson is part of the [Probabilistic Data Structures and Analytics](https://stanza.dev/courses/redis-probabilistic) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*