# threatfox-checker
A simple python app to export THREATfox indicators to a CSV.

## Installation

1. Clone this repository:
```bash
git clone https://github.com/yourusername/threatfox-checker.git
cd threatfox-checker
```

2. Install the required Python packages:
```bash
pip install -r requirements.txt
```

## Setup

1. Rename `.env.example` to `.env`:
```bash
copy .env.example .env
```

2. Edit the `.env` file and paste your THREATfox API key:
```
THREATFOX_API_KEY=your_api_key_here
```

3. Rename `config.yaml.example` to `config.yaml`:
```bash
copy config.yaml.example config.yaml
```

4. Edit the `config.yaml` file to configure your queries:
```yaml
queries:
  - name: emotet
    type: malware_name
    value: emotet
    limiter: 100  # Get 100 most recent indicators

  - name: recent-phishing
    type: tag
    value: phishing
    limiter: 100  # Get 100 most recent indicators
```

## Query Configuration

Each query in the `config.yaml` file must have the following properties:

- `name`: A unique identifier for the query (used in the output filename)
- `type`: The type of search to perform. Must be one of:
  - `tag`: Search for indicators with a specific tag
  - `malware_name`: Search for indicators of a specific malware
- `value`: The actual value to search for (e.g., the malware name or tag)
- `limiter`: The number of most recent indicators to pull (e.g., `100` for the 100 most recent indicators)

## Usage

Run the application:
```bash
python main.py
```

### Test Mode

You can run the application in test mode to cache API responses, which is useful during development and testing:

```bash
python main.py --test
```

In test mode, API responses are cached in a `cache` directory. Subsequent runs with the same queries will use the cached responses instead of making API requests.

## Output

The app will output multiple CSV files for each query in the `output` directory:

1. A combined file with all indicators:
   - `output/query-name.csv`
   - Example: `output/emotet.csv`

2. Type-specific files for each indicator type:
   - `output/query-name-TYPE.csv`
   - Examples:
     - `output/emotet-domain.csv` (domain indicators only)
     - `output/emotet-url.csv` (URL indicators only) 
     - `output/emotet-ip-port.csv` (IP:port indicators)
     - `output/emotet-ip.csv` (IPs extracted from IP:port indicators)

### Auto-updating and Deduplication

Each time the app runs, it will:
- Read existing CSV files if present
- Fetch new indicators from the ThreatFox API
- Merge new indicators with existing ones
- Update any existing indicators with the most recent information
- Remove duplicates (based on the indicator ID)
- Write the combined dataset back to the CSV files

This approach ensures that your indicator files are always up-to-date while preserving historical data. You can run the script on a schedule (e.g., via cron job or Windows Task Scheduler) to regularly update your indicator files without worrying about duplicates.

The combined CSV contains the following fields:
- id
- ioc_type
- ioc_value
- threat_type
- threat_type_desc
- malware
- tags
- reporter
- confidence_level
- first_seen
- last_seen

Type-specific CSVs have the same fields except for `ioc_type` which is omitted since it's already indicated in the filename.