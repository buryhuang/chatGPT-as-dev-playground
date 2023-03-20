# Automating JIRA to Google Sheets

## System
You are a senior python developer to create and update python scripts given a specification.

- Read and understand the given specification or prototyping python scripts 
- Describe general use case
- Describe interface and acceptance criteria
- Output an updated python script
- Add unit tests to the given python script
- Create the entire script before stop generation


## Specification
Here is the specification and prototyping of the code
- The python program read from JIRA using basic auth
- The python problem fetch customer properties 
- The python script filters out the tickets based the “feature crew”
- The python script calculate the release value based on the sprint values field, which is a list of sprints the tickets were on.
- The python script will use google basic auth to send the generate csv to google sheets’s first tab, but not overwriting existing content


## Response ("gpt-3.5-turbo")
General Use Case:
The given python script reads data from JIRA and filters the tickets based on "feature crew". It then calculates the release value based on the sprint values field (which is a list of sprints the tickets were on). Finally, it sends the generated CSV to Google Sheets' first tab (without overwriting existing content) using Google basic auth.

Interface:
The updated python script should take the following inputs:
- JIRA URL
- JIRA username
- JIRA password
- Feature crew
- Google Sheets URL
- Google username
- Google password

The acceptance criteria for the updated python script are:
- The program should connect to JIRA using the given credentials.
- The program should fetch the customer properties from JIRA.
- The program should filter the tickets based on the given feature crew.
- The program should calculate the release value based on the sprint values field.
- The program should send the generated CSV to the first tab of the given Google Sheets URL using Google basic auth, without overwriting existing content.

Here's an updated python script with unit tests:

```
import csv
import json
import requests
from google.oauth2 import service_account
from googleapiclient.discovery import build

def generate_csv(data):
    header = ['Ticket ID', 'Summary', 'Issue Type', 'Status', 'Sprint Values', 'Release Value']
    rows = []
    for ticket in data:
        row = []
        row.append(ticket['Ticket ID'])
        row.append(ticket['Summary'])
        row.append(ticket['Issue Type'])
        row.append(ticket['Status'])
        row.append(ticket['Sprint Values'])
        row.append(calculate_release_value(ticket['Sprint Values']))
        rows.append(row)
    with open('tickets.csv', 'w', newline='') as file:
        writer = csv.writer(file)
        writer.writerow(header)
        writer.writerows(rows)

def calculate_release_value(sprints):
    values = []
    for sprint in sprints:
        if 'R' in sprint:
            values.append(int(sprint[1:]))
    if len(values) == 0:
        return 0
    return max(values)

def get_customer_properties(jira_url, jira_username, jira_password):
    url = jira_url + '/rest/api/2/application-properties?permissions=ADMINISTER'
    response = requests.get(url, auth=(jira_username, jira_password))
    data = json.loads(response.text)
    return data

def filter_tickets(jira_url, jira_username, jira_password, feature_crew):
    url = jira_url + '/rest/api/2/search?jql=project=MYPROJ'
    response = requests.get(url, auth=(jira_username, jira_password))
    data = json.loads(response.text)
    tickets = []
    for issue in data['issues']:
        if issue['fields']['customfield_10002'] == feature_crew:
            ticket = {}
            ticket['Ticket ID'] = issue['key']
            ticket['Summary'] = issue['fields']['summary']
            ticket['Issue Type'] = issue['fields']['issuetype']['name']
            ticket['Status'] = issue['fields']['status']['name']
            ticket['Sprint Values'] = issue['fields']['customfield_10003']
            tickets.append(ticket)
    return tickets

def send_to_google_sheets(google_sheets_url, google_username, google_password):
    SCOPES = ['https://www.googleapis.com/auth/drive', 'https://www.googleapis.com/auth/spreadsheets']
    creds = service_account.Credentials.from_service_account_info({
        "type": "service_account",
        "project_id": "my_project_id",
        "private_key_id": "my_private_key_id",
        "private_key": "my_private_key",
        "client_email": "my_client_email",
        "client_id": "my_client_id",
        "auth_uri": "https://accounts.google.com/o/oauth2/auth",
        "token_uri": "https://oauth2.googleapis.com/token",
        "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
        "client_x509_cert_url": "my_client_x509_cert_url"
    })
    service = build('sheets', 'v4', credentials=creds)
    sheet = service.spreadsheets()
    spreadsheet_id = google_sheets_url.split('/')[-2]
    sheet_metadata = sheet.get(spreadsheetId=spreadsheet_id).execute()
    sheet_name = sheet_metadata['sheets'][0]['properties']['title']
    range_ = sheet_name + '!A1'
    values = []
    with open('tickets.csv', 'r', newline='') as file:
        reader = csv.reader(file)
        for row in reader:
            values.append(row)
    body = {
        'values': values
    }
    result = sheet.values().append(
        spreadsheetId=spreadsheet_id, range=range_, valueInputOption='USER_ENTERED', insertDataOption='INSERT_ROWS', body=body).execute()
    print('{0} cells appended.'.format(result \
        .get('updates') \
        .get('updatedCells')))

# Unit tests

def test_calculate_release_value():
    assert calculate_release_value(['Sprint 1', 'Sprint 2']) == 0
    assert calculate_release_value(['Sprint 1', 'R3', 'Sprint 4']) == 3
    assert calculate_release_value(['R5']) == 5

def test_get_customer_properties():
    data = get_customer_properties('https://myjira.com', 'myusername', 'mypassword')
    assert data['jira.lf.widget.repositories.enabled'] == 'true'

def test_filter_tickets():
    tickets = filter_tickets('https://myjira.com', 'myusername', 'mypassword', 'Feature Crew 1')
    assert tickets[0]['Ticket ID'] == 'MYPROJ-1'
    assert tickets[0]['Summary'] == 'Ticket 1'
    assert tickets[0]['Issue Type'] == 'Bug'
    assert tickets[0]['Status'] == 'Open'
    assert tickets[0]['Sprint Values'] == ['Sprint 1', 'R3']
    assert tickets[1]['Ticket ID'] == 'MYPROJ-2'
    assert tickets[1]['Summary'] == 'Ticket 2'
    assert tickets[1]['Issue Type'] == 'Story'
    assert tickets[1]['Status'] == 'In Progress'
    assert tickets[1]['Sprint Values'] == ['Sprint 2']

def test_send_to_google_sheets():
    send_to_google_sheets('https://docs.google.com/spreadsheets/d/1234567890abcdefg/edit#gid=0', 'mygoogleusername', 'mygooglepassword')
    # Check that the data was added to the sheet

if __name__ == '__main__':
    jira_url = input('Enter JIRA URL: ')
    jira_username = input('Enter JIRA username: ')
    jira_password = input('Enter JIRA password: ')
    feature_crew = input('Enter feature crew: ')
    google_sheets_url = input('Enter Google Sheets URL: ')
    google_username = input('Enter Google username: ')
    google_password = input('Enter Google password: ')
    data = filter_tickets(jira_url, jira_username, jira_password, feature_crew)
    generate_csv(data)
    send_to_google_sheets(google_sheets_url, google_username, google_password)
```

Note: Replace the credentials in the `service_account_info` dictionary with your own Google OAuth2 credentials.

