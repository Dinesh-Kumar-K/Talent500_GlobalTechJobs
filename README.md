# Talent500 Job Data Extraction and Cleaning

This notebook outlines the process of extracting job data from Talent500, specifically for Albertsons India, and then cleaning the job descriptions.

## 1. Importing Libraries

First, we import the necessary Python libraries:
- `json`: For handling JSON data.
- `pandas`: For data manipulation and analysis, particularly with DataFrames.
- `bs4` (BeautifulSoup): For parsing HTML and XML documents.
- `requests`: For making HTTP requests to fetch data from web APIs.

```python
import json
import pandas as pd
from bs4 import BeautifulSoup
import requests
```

## 2. Fetching Job Listings

We define the necessary headers for making a request to the Talent500 API. These headers help in mimicking a browser request and ensuring the API responds as expected.

Then, we use the `requests` library to send a GET request to the Talent500 jobs search API. The URL specifies the company slug (`albertsonsindia`) and parameters for pagination (`offset` and `size`, and `search_after` for continued pagination).

```python
headers = {
    'accept': 'application/json, text/plain, */*',
    'accept-language': 'en-US,en;q=0.9,en-IN;q=0.8,ta;q=0.7',
    'cache-control': 'no-cache',
    'origin': 'https://talent500.com',
    'priority': 'u=1, i',
    'referer': 'https://talent500.com/',
    'sec-ch-ua': '"Not)A;Brand";v="8", "Chromium";v="138", "Microsoft Edge";v="138"',
    'sec-ch-ua-mobile': '?0',
    'sec-ch-ua-platform': '"Windows"',
    'sec-fetch-dest': 'empty',
    'sec-fetch-mode': 'cors',
    'sec-fetch-site': 'cross-site',
    'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/138.0.0.0 Safari/537.36 Edg/138.0.0.0',
}

response = requests.get(
    'https://prod-warmachine.talent500.co/api/v3/jobs/search/?company_slug=albertsonsindia&offset=1000&size=1000&search_after=[0,28.779936,%2267a2f1e9-1dee-43d2-a5b2-567e9c96c269%22]',
    headers=headers,
)
```

## 3. Normalizing JSON Data

The API response is in JSON format. We load the JSON data from the response text and then use `pandas.json_normalize` to convert it into a flat DataFrame. This makes it easier to work with the data.

```python
df = pd.json_normalize(json.loads(response.text)['data'])
```

## 4. Extracting Job Descriptions

To get the detailed description for each job, we define a function `return_desc` that takes a `job_id` (which we'll get from the 'slug' column in our DataFrame). It makes a new GET request to the specific job endpoint, parses the response, and returns the 'description' field from the JSON response.

```python
def return_desc(job_id):
    res = requests.get("https://prod-warmachine.talent500.co/api/jobs/" + job_id)
    soup = BeautifulSoup(res.text, 'lxml')
    return json.loads(res.text)['description']
```

## 5. Cleaning Job Descriptions

Job descriptions often contain HTML tags and other formatting. We create a `clean_text` function that uses BeautifulSoup to parse the text and then extract only the plain text content, removing any HTML markup.

```python
def clean_text(text):
    soup = BeautifulSoup(text, 'lxml')
    return soup.get_text()
```

## 6. Applying Functions and Cleaning Data

We apply the `return_desc` function to each 'slug' in our DataFrame to populate the 'description' column with the full job descriptions.

```python
df['description'] = df['slug'].apply(return_desc)
```

Subsequently, we apply the `clean_text` function to the newly populated 'description' column to clean up the extracted text.

```python
df['description'] = df['description'].apply(clean_text)
```

## 7. Exporting to Excel

Finally, we export the processed DataFrame, including the cleaned job descriptions, to an Excel file named `Talent500_GlobalTechJobs_Output_30-07-2025.xlsx`. The `index=False` argument prevents writing the DataFrame index as a column in the Excel file.

```python
df.to_excel('Talent500_GlobalTechJobs_Output_30-07-2025.xlsx', index=False)
```