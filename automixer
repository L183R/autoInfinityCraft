import requests
from docx import Document
from concurrent.futures import ThreadPoolExecutor, as_completed
import json
import os
import time
import logging
import random
import csv

# Logging configuration
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

# API URL with the objects to mix
url = "https://neal.fun/api/infinite-craft/pair"

# JSON filename where the list of elements will be saved
json_filename = "elements.json"

# Filename where the results will be saved
docx_filename = "results.docx"

# CSV filename where the combinations will be saved
csv_filename = "combinations.csv"

# Load the list of elements from the JSON file, if it exists
if os.path.exists(json_filename):
    with open(json_filename, "r") as file:
        data = json.load(file)
else:
    data = {
        "elements": [
            {"text": "Water", "emoji": "💧", "discovered": False},
            {"text": "Fire", "emoji": "🔥", "discovered": False},
            {"text": "Wind", "emoji": "🌬️", "discovered": False},
            {"text": "Earth", "emoji": "🌍", "discovered": False},
        ],
        "darkMode": False
    }

elements = [element['text'] for element in data['elements']]

# Headers that emulate a browser
headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36",
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8",
    "Accept-Language": "es-419,es;q=0.9",
    "Accept-Encoding": "gzip, deflate, br",
    "Connection": "keep-alive",
    "Upgrade-Insecure-Requests": "1",
    "Referer": "https://neal.fun/infinite-craft/",
    "Sec-Fetch-Dest": "document",
    "Sec-Fetch-Mode": "navigate",
    "Sec-Fetch-Site": "same-origin",
    "Sec-Fetch-User": "?1"
}

# Create a new document
document = Document()

# Create the CSV file if it does not exist and write the headers
if not os.path.exists(csv_filename):
    with open(csv_filename, "w", newline="", encoding="utf-8") as csvfile:
        csvwriter = csv.writer(csvfile)
        csvwriter.writerow(["Element1", "Element2", "Result", "Discovered"])

# Load existing combinations from the CSV
combinations = set()
if os.path.exists(csv_filename):
    with open(csv_filename, "r", newline="", encoding="utf-8") as csvfile:
        csvreader = csv.reader(csvfile)
        next(csvreader)  # Skip the header row
        for row in csvreader:
            combinations.add((row[0], row[1]))

# Function to make a request and process the response
def make_random_request():
    first_element = random.choice(elements)
    second_element = random.choice(elements)
    
    # Check if the combination already exists
    if (first_element, second_element) in combinations or (second_element, first_element) in combinations:
        logging.info(f"Combination {first_element} + {second_element} already exists, skipping.")
        return None, None, None
    
    params = {
        "first": first_element,
        "second": second_element
    }
    try:
        response = requests.get(url, headers=headers, params=params, timeout=10)
        response.raise_for_status()  # Raise an exception for HTTP status codes 4xx/5xx
        return first_element, second_element, response
    except requests.RequestException as e:
        return first_element, second_element, None

# Function to handle the API response
def handle_response(first_element, second_element, response):
    if response and response.status_code == 200:
        data_response = response.json()
        emoji = data_response.get("emoji", "")
        result = data_response.get("result", "")
        formatted_response = f"{result}{emoji}"
        response_text = f"{first_element} + {second_element} = {formatted_response}"
        print(response_text)
        
        discovered = False
        if result and result not in elements:
            elements.append(result)  # Add the element if it is not in the list
            data['elements'].append({"text": result, "emoji": emoji, "discovered": True})
            discovered = True
            
            # Save the updated list in the JSON file
            with open(json_filename, "w") as file:
                json.dump(data, file)
        
        # Add the result to the document
        document.add_paragraph(response_text)
        
        # Save the document
        try:
            document.save(docx_filename)
        except PermissionError as e:
            logging.error(f"Could not save the document: {e}")

        # Add the combination to the CSV file
        try:
            with open(csv_filename, "a", newline="", encoding="utf-8") as csvfile:
                csvwriter = csv.writer(csvfile)
                csvwriter.writerow([first_element, second_element, formatted_response, discovered])
            
            # Add the combination to the set for future checks
            combinations.add((first_element, second_element))
        except PermissionError as e:
            logging.error(f"Could not save the combination to the CSV: {e}")
    elif response:
        error_text = f"Request error: {response.status_code}\n{response.text}"
        logging.error(error_text)
        
        # Add the error to the document
        document.add_paragraph(error_text)
        
        # Save the document
        try:
            document.save(docx_filename)
        except PermissionError as e:
            logging.error(f"Could not save the document: {e}")

# Parallel execution
while True:
    with ThreadPoolExecutor(max_workers=40) as executor:
        futures = []
        for _ in range(40):  # Make 5 random requests in parallel
            futures.append(executor.submit(make_random_request))
        
        for future in as_completed(futures):
            first_element, second_element, response = future.result()
            if first_element and second_element:
                handle_response(first_element, second_element, response)
    
    # Pause for 5 seconds before starting the next cycle
    time.sleep(1)
