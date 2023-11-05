import requests
from bs4 import BeautifulSoup
import logging
import json
import time

# Configure the logging module to save logs in a .log file
logging.basicConfig(filename='search_results.log', level=logging.INFO, format='%(message)s')

# Define the Tor proxy address
tor_proxy = 'socks5h://localhost:9150'  # Replace with your Tor proxy address if needed

# Define the search engine URL
search_engine_url = 'http://juhanurmihxlp77nkq76byazcldy2hlmovfu2epvl5ankdibsot4csyd.onion'  # Replace with the URL of your chosen search engine

# Define the query you want to search for
query = 'Axis'

# Set up the proxies dictionary to route requests through Tor
proxies = {
    'http': tor_proxy,
    'https': tor_proxy
}

# Run the script in a loop for real-time monitoring
while True:
    try:
        # Construct the URL with the /search/ endpoint and the query parameter
        url = f'{search_engine_url}/search/?q={query}'

        # Make an HTTP GET request to the search engine using the Tor proxy and the constructed URL
        response = requests.get(url, proxies=proxies)

        # Check the response status code
        if response.status_code == 200:
            # Parse the HTML content of the response using BeautifulSoup
            soup = BeautifulSoup(response.text, 'html.parser')

            # Find and extract the search results
            results = soup.find_all('li', class_='result')

            for result in results:
                url_element = result.find('a')  # Find the <a> tag within the 'li' element
                url = url_element['href']  # Extract the 'href' attribute for the URL
                description = url_element.get_text(strip=True)  # Get the text inside the <a> tag as the description

                # Create a dictionary to hold the data
                result_data = {
                    'URL': url,
                    'Description': description,
                }

                # Convert the dictionary to a JSON string
                json_data = json.dumps(result_data)

                # Log the JSON data to the .log file
                logging.info(json_data)

        else:
            logging.error(f"Request to the search engine failed with status code {response.status_code}")

    except requests.exceptions.RequestException as e:
        logging.error(f"An error occurred: {e}")

    # Wait for 6 hours before the next search
    time.sleep(6 * 60 * 60)  # 6 hours = 6 * 60 minutes * 60 seconds
