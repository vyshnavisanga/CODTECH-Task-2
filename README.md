# CODTECH-Task-2
#Vulnerability Scanning Tool Code
import nmap
import requests
from bs4 import BeautifulSoup

def scan_ports(target_ip):
    """
    Scan for open ports on the target IP address.

    Parameters:
    target_ip (str): The IP address to scan.

    Returns:
    list: A list of open ports.
    """
    nm = nmap.PortScanner()
    nm.scan(target_ip, '1-1024')
    open_ports = [port for port in nm[target_ip]['tcp'] if nm[target_ip]['tcp'][port]['state'] == 'open']
    return open_ports

def check_software_versions(target_url):
    """
    Check for outdated software versions on the target URL.

    Parameters:
    target_url (str): The URL to check.

    Returns:
    dict: A dictionary of software versions.
    """
    response = requests.get(target_url)
    soup = BeautifulSoup(response.text, 'html.parser')
    software_versions = {}
    for tag in soup.find_all('meta'):
        if 'generator' in tag.attrs:
            software_versions['CMS'] = tag.attrs['generator']
        elif 'powered-by' in tag.attrs:
            software_versions['Web Server'] = tag.attrs['powered-by']
    return software_versions

def check_misconfigurations(target_url):
    """
    Check for misconfigurations on the target URL.

    Parameters:
    target_url (str): The URL to check.

    Returns:
    list: A list of misconfigurations.
    """
    response = requests.get(target_url)
    soup = BeautifulSoup(response.text, 'html.parser')
    misconfigurations = []
    if soup.find('title') == 'Index of /':
        misconfigurations.append('Directory listing enabled')
    if soup.find('meta', attrs={'name': 'robots'}) == 'index, follow':
        misconfigurations.append('Search engine indexing enabled')
    return misconfigurations

def main():
    target_ip = input("Enter the target IP address: ")
    target_url = input("Enter the target URL: ")

    open_ports = scan_ports(target_ip)
    print("Open Ports:")
    for port in open_ports:
        print(port)

    software_versions = check_software_versions(target_url)
    print("\nSoftware Versions:")
    for software, version in software_versions.items():
        print(f"{software}: {version}")

    misconfigurations = check_misconfigurations(target_url)
    print("\nMisconfigurations:")
    for misconfiguration in misconfigurations:
        print(misconfiguration)

if __name__ == "__main__":
    main()
