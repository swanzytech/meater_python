# Meater Temperature Logging

This project allows you to log temperature data from your Meater devices using the Meater Cloud Public REST API. The script retrieves data at regular intervals and logs it to a file.

## Prerequisites

1. Python 3.x
2. `requests` library for making HTTP requests

You can install the `requests` library using pip:

```bash
pip install requests
```

## Getting Started

### Step 1: Obtain Your Meater API Key

To interact with the Meater API, you need an API key. Follow these steps to obtain your API key:

1. Open your terminal.
2. Use the following `curl` command to get your API key:

```bash
curl --location --request POST 'https://public-api.cloud.meater.com/v1/login' \
--header 'Content-Type: application/json' \
--data-raw '{
  "email": "<MEATER Cloud Email Address>",
  "password": "<MEATER Cloud Password>"
}'
```

Replace `<MEATER Cloud Email Address>` and `<MEATER Cloud Password>` with your actual Meater Cloud email address and password. The response will contain your API key.

### Step 2: Set Up the Script

Save the following script as `meater_logger.py`:

```python
#!/usr/bin/env python3

import requests
import time
import logging

# Replace with your actual Meater API endpoint and access token
MEATER_API_URL = "https://public-api.cloud.meater.com/v1/devices"
ACCESS_TOKEN = "<YOUR API KEY>"

# Configure logging
LOG_FILE = "/<pathto>/meater.log"
logging.basicConfig(filename=LOG_FILE, level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

def celsius_to_fahrenheit(celsius):
    return (celsius * 9/5) + 32

def get_meater_data():
    headers = {
        "Authorization": f"Bearer {ACCESS_TOKEN}"
    }
    response = requests.get(MEATER_API_URL, headers=headers)
    if response.status_code == 200:
        return response.json()
    else:
        logging.error("Failed to retrieve data: %s %s", response.status_code, response.text)
        return None

def main():
    while True:
        data = get_meater_data()
        if data:
            logging.info("API Response: %s", data)  # Log API response
            if 'data' in data and 'devices' in data['data']:
                for device in data['data']['devices']:
                    ambient_temp_c = device['temperature']['ambient']
                    internal_temp_c = device['temperature']['internal']
                    ambient_temp_f = celsius_to_fahrenheit(ambient_temp_c)
                    internal_temp_f = celsius_to_fahrenheit(internal_temp_c)
                    logging.info(f"Device ID: {device['id']}")
                    logging.info(f"Ambient Temperature: {ambient_temp_f:.2f} °F")
                    logging.info(f"Internal Temperature: {internal_temp_f:.2f} °F")
                    logging.info("--------------------------")
            else:
                logging.info("No devices found in the response.")
        else:
            logging.warning("No data received.")
        time.sleep(30)  # Wait for 30 seconds before fetching data again

if __name__ == "__main__":
    main()
```

### Step 3: Configure the Script

1. Replace `<YOUR API KEY>` with your actual Meater API key.
2. Replace `<pathto>` in `LOG_FILE` with the path where you want to save your log file.

### Step 4: Run the Script

Make the script executable and run it:

```bash
chmod +x meater_logger.py
./meater_logger.py
```

The script will log temperature data from your Meater devices every 30 seconds. Check the log file at the specified path to see the logged data.

## Acknowledgements

- [Meater Cloud Public REST API](https://github.com/apption-labs/meater-cloud-public-rest-api/tree/master) for providing the API access.
- [Python Requests](https://requests.readthedocs.io/en/master/)
