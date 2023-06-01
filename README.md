# CSV-Scrapper
Outlook Domain Scrapper

import csv
import requests
from bs4 import BeautifulSoup
import pandas as pd
import logging

# Set up logging configuration
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

def search_company_website(company_name):
    try:
        search_query = f"{company_name} official website"
        url = f"https://www.google.com/search?q={search_query}"
        
        headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0;Win64) AppleWebkit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.93 Safari/537.36'}
        
        response = requests.get(url, headers=headers)
        response.raise_for_status()  # Check for any HTTP errors
        
        soup = BeautifulSoup(response.text, 'html.parser')
        result_div = soup.find('div', class_='yuRUbf')
        if result_div:
            link = result_div.a['href']
            return link
        else:
            return None
    except requests.exceptions.RequestException as e:
        logging.error(f"Error occurred while searching company website: {e}")
        return None

def check_outlook_domain(url):
    try:
        mxtoolbox_url = f"https://mxtoolbox.com/SuperTool.aspx?action=mx%3a{url}"
        
        headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0;Win64) AppleWebkit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.93 Safari/537.36'}
        
        response = requests.get(mxtoolbox_url, headers=headers)
        response.raise_for_status()  # Check for any HTTP errors
        
        soup = BeautifulSoup(response.text, 'html.parser')
        result_div = soup.find('div', class_='mt-results-panel')
        if result_div and 'Outlook' in result_div.text:
            return True
        else:
            return False
    except requests.exceptions.RequestException as e:
        logging.error(f"Error occurred while checking Outlook domain: {e}")
        return False

input_file = 'company_list.csv'
output_file = 'outlook_sites.csv'

company_websites = []

# Read the input CSV file
try:
    with open(input_file, 'r') as file:
        reader = csv.DictReader(file)
        for row in reader:
            company_name = row['Company Name']
            website = search_company_website(company_name)
            if website:
                is_outlook_domain = check_outlook_domain(website)
                if is_outlook_domain:
                    company_websites.append({'Company Name': company_name, 'URL': website})
            else:
                logging.warning(f"No website found for company: {company_name}")

    # Create a new CSV file with outlook sites
    df = pd.DataFrame(company_websites)
    df.to_csv(output_file, index=False)

    logging.info("Process completed successfully. Outlook sites saved to outlook_sites.csv.")
except FileNotFoundError:
    logging.error("Input file not found.")
except Exception as e:
    logging.error(f"An unexpected error occurred: {e}")
