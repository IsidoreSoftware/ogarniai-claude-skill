# Ogarni.AI API — Code Examples

## cURL

### List recent receipts
```bash
curl -X GET "https://api.ogarni.ai/api/PurchaseDocuments/my?sortBy=purchase_time&sortDirection=desc" \
  -H "X-API-Key: $OGARNIAI_API_TOKEN" \
  -H "Accept: application/json"
```

### Get receipts from January 2025
```bash
curl -X GET "https://api.ogarni.ai/api/PurchaseDocuments/my?from=2025-01-01&to=2025-01-31" \
  -H "X-API-Key: $OGARNIAI_API_TOKEN"
```

### Get weekly summary
```bash
curl -X GET "https://api.ogarni.ai/api/weekly-summaries/latest" \
  -H "X-API-Key: $OGARNIAI_API_TOKEN"
```

### Get current month summary
```bash
curl -X GET "https://api.ogarni.ai/api/summaries/presets?preset=current-month&granularity=Day" \
  -H "X-API-Key: $OGARNIAI_API_TOKEN"
```

### List unread notifications
```bash
curl -X GET "https://api.ogarni.ai/api/Notifications?isRead=false" \
  -H "X-API-Key: $OGARNIAI_API_TOKEN"
```

---

## Python

### Basic client
```python
import os
import requests

class OgarniAiClient:
    def __init__(self):
        self.token = os.environ["OGARNIAI_API_TOKEN"]
        self.base_url = "https://api.ogarni.ai"
        self.headers = {
            "X-API-Key": self.token,
            "Accept": "application/json",
        }

    def get(self, endpoint, params=None):
        response = requests.get(
            f"{self.base_url}{endpoint}",
            headers=self.headers,
            params=params,
        )
        response.raise_for_status()
        return response.json()

    def list_documents(self, from_date=None, to_date=None):
        return self.get("/api/PurchaseDocuments/my", {
            "from": from_date,
            "to": to_date,
        })

    def get_weekly_summary(self):
        return self.get("/api/weekly-summaries/latest")

    def get_categories(self):
        return self.get("/api/Categories")

    def get_current_month_summary(self):
        return self.get("/api/summaries/presets", {
            "preset": "current-month",
            "granularity": "Day",
        })
```

### Usage
```python
client = OgarniAiClient()

# Get this month's spending
summary = client.get_current_month_summary()
print(f"Total: {summary['totalAmount']} PLN")

for cat in summary["categoryTotals"]:
    print(f"  {cat['category']}: {cat['totalAmount']} PLN")

# List recent receipts
docs = client.list_documents()
for doc in docs["items"][:5]:
    print(f"{doc['storeName']}: {doc['totalAmount']} {doc['currency']}")
```

### With error handling and rate limit awareness
```python
import time

def safe_request(client, endpoint, params=None, max_retries=3):
    for attempt in range(max_retries):
        try:
            response = requests.get(
                f"{client.base_url}{endpoint}",
                headers=client.headers,
                params=params,
            )

            remaining = response.headers.get("X-RateLimit-Remaining")
            if remaining:
                print(f"Rate limit remaining: {remaining}")

            if response.status_code == 429:
                retry_after = int(response.headers.get("Retry-After", 60))
                print(f"Rate limited, waiting {retry_after}s...")
                time.sleep(retry_after)
                continue

            response.raise_for_status()
            return response.json()

        except requests.exceptions.RequestException as e:
            if attempt == max_retries - 1:
                raise
            time.sleep(2 ** attempt)

    raise Exception("Max retries exceeded")
```

---

## JavaScript / Node.js

### Basic client
```javascript
const API_TOKEN = process.env.OGARNIAI_API_TOKEN;
const BASE_URL = "https://api.ogarni.ai";

async function apiGet(endpoint, params = {}) {
  const url = new URL(`${BASE_URL}${endpoint}`);
  Object.entries(params).forEach(([key, value]) => {
    if (value !== undefined) url.searchParams.set(key, String(value));
  });

  const response = await fetch(url, {
    headers: {
      "X-API-Key": API_TOKEN,
      "Accept": "application/json",
    },
  });

  if (!response.ok) {
    throw new Error(`API error: ${response.status} ${response.statusText}`);
  }

  return response.json();
}

// List recent receipts
const docs = await apiGet("/api/PurchaseDocuments/my");
console.log(`Found ${docs.items.length} documents`);

// Get this month's spending
const summary = await apiGet("/api/summaries/presets", {
  preset: "current-month",
});
console.log(`Total: ${summary.totalAmount} PLN`);

// List categories
const categories = await apiGet("/api/Categories");
categories.categories.forEach((cat) => {
  console.log(`${cat.emoji} ${cat.englishName} (${cat.polishName})`);
});
```

### With axios
```javascript
import axios from "axios";

const client = axios.create({
  baseURL: "https://api.ogarni.ai",
  headers: {
    "X-API-Key": process.env.OGARNIAI_API_TOKEN,
    "Accept": "application/json",
  },
});

// Interceptor for rate limit logging
client.interceptors.response.use((response) => {
  const remaining = response.headers["x-ratelimit-remaining"];
  if (remaining) console.log(`Rate limit remaining: ${remaining}`);
  return response;
});

const { data: summary } = await client.get("/api/weekly-summaries/latest");
console.log(summary.summary);
```
