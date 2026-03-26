# OpenAlex_Crawler
A lightweight OpenAlex paper collection tool for keyword-based searches.
By entering specific search keywords (query keywords) selected by the user in a certain domain, it collects the bibliographic information of the papers that appear in the search results for those query keywords.


This code currently includes results collected from 17 query keywords(See the file named _'keywords.txt'_) in the Transportation field, and it is also possible to input any academic keywords from other fields to get results.


## Features
- Keyword-based collection from a `.txt` file
- Single combined CSV output
- Duplicate removal based on `openalex_id`
- Resume support with `checkpoint.json`
- Failed query logging
- Page-level progress logs
- Optional use of OpenAlex API key
- Designed for Jupyter Notebook execution


## Requirements
1) Python 3.10+

2) Libraries
    - `requests`
    - `tqdm`
  
   Install dependencies with:
```bash
pip install -r requirements.txt
```

3) Api key (or e-mail)
   : Although data can be collected via email, this code is optimized for collection using an individual API key.

4) Input file
   : Prepare a keyword file in plain text format. (Example: keywords.txt)
   : One keyword per line (Empty lines are ignored). 


## Output files
The notebook generates the following files:

- `EV_pilot.csv`: combined final output file
- `checkpoint.json`: resume state for interrupted collection
- `failed_queries.txt`: log of failed queries


## Files
```bash
OpenAlex_Crawler/
├─ .gitattributes
├─ .gitignore
├─ README.md
├─ requirements.txt
├─ keyword_pilot.txt
├─ checkpoint.json
├─ failed_queries.txt
├─ dataset/
│  ├─ Transportation_pilot.zip
│  ├─ Transportation_pilot_abbrev_only.zip
│  └─ Transportation_pilot_full_phrase_queries.csv
└─ notebooks/
   └─ openalex_collector.ipynb
```


## How It Works
The collector:
1. Loads keywords from a text file.
2. Sends paginated requests to OpenAlex.
3. Extracts key metadata such as:
     - title
     - DOI
     - publication year
     - journal/source
     - authors
     - abstract
     - OpenAlex ID
     - citation count
4. Appends only new records to the output CSV.
5. Saves checkpoint state after each page so collection can resume later.

## Function Definition
* `load_keywords()`: Loads search keywords(query) from 'keywords.txt'.
  - Input: your keyword file path
* `parse_abstract()`: Reconstructs plain abstract text from OpenAlex inverted index format.
* `get_journal_name()`: Extracts the journal or source name from one OpenAlex work record.
* `normalize_work()`: Converts one raw API result into one flat CSV row.
  - Extract title, DOI number, publication year, journal, authors, ... etc.
  - Standardizes missing values.
* `build_session()`: Creates a reusable HTTP session for API requiests.
    - Input: None
    - Sets a custom **User-Agent**.
    - Adds `mailto` information <span style="color:red">if email is available</span>.
* `build_params()`: Builds query parameters for one OpenAlex requests.
    - Input: keywords(str), cursor(str)
    - Adds paging and selected fields
    - Uses either legacy `title_and_abstract.search` or current `search`.
    - Adds `api_key` and `mailto` <span style="color:red">if available</span>.
* `get_rate_limit_status()`: Requests current daily budget information from OpenAlex.
    - Returns budget information **if api key exists**.
    - Returns **None** if no key is available or the requests fails.
* `handle_limit()`: Handles rate-limit responses.
    - Waits before retrying.
      + If an api key exists, checks whether the daily free budget is exhausted.
      + Raises an error if the daily budget is fully used.
* `fetch_page()`: Fetches one page of results.
    - Sends one paginated API request.
    - Retries temporary failures.
      + `results`: current page results
      + `next_cursor`: cursor for the next page
* `initialize_output()`: Creates the combined output CSV if it does not exist.
    - Creates the output directory if needed.
    - Writes the CSV header once.
* `load_existing_ids()`: Loads already saved OpenAlex IDs from the output file.
    - Used for deduplications.
* `load_checkpoint()`: Loads completed query names from the checkpoint file.
    - Saves completed query names to disk.
    - Writes completed queries as JSON.
* `log_failed_query()`: Records failed queries in a log file.
    - Appends one failed query per line.
* `append_rows()`: Appends collected rows to the combined CSV file.
    - Writes only new rows.
* `collect_keywords()`: Collects all pages for one keyword.
    - Repeats API requests until no ntext cursor exists.
    - Deduplicates by `openalex_id`.

## User setting (Running code)
* Change User settings blocks personally.
* `USE_LEGACY_TITLE_ABSTRACT_SEARCH` (optional setting)
    - True: `title_and_abstract.search` (include title and abstract)
    - False: `filter=field.search` (include title, abstract, and full-text)
    - For related information, please visit (https://developers.openalex.org).


## Notes
* Large keyword queries may require multiple runs if the daily API budget is exhausted
* Resume works best when the same search settings are kept across runs
* If you change the keyword file, output file, year filter, or search mode, previous checkpoint state may no longer match
