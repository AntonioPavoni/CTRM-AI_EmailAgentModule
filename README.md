# Email Agent with Mixtral, LangChain, and EasyOCR

This package provides a complete solution for processing emails with attachments using:
- Gmail API via LangChain's Gmail tools
- EasyOCR for document text extraction
- Hugging Face's Mixtral API for document understanding

## Features

- **Gmail API Integration**: Connect to Gmail using LangChain's Gmail tools
- **Document Extraction**: Download and process email attachments
- **OCR Processing**: Extract text from PDFs and images using EasyOCR
- **AI Analysis**: Process extracted text with Hugging Face's Mixtral API
- **Email Monitoring**: Continuous monitoring of unread emails

## Quick Start

### 1. Installation

1. Install the required Python packages:
   ```
   pip install -r requirements.txt
   ```

2. Install system dependencies for PDF processing:
   - Ubuntu: `sudo apt-get install poppler-utils`
   - macOS: `brew install poppler`
   - Windows: Download from http://blog.alivate.com.au/poppler-windows/

### 2. Configuration

1. Copy the template configuration file:
   ```
   cp email_agent/config/config.json.template email_agent/config/config.json
   ```

2. Edit `email_agent/config/config.json` to add your:
   - Gmail API credentials path
   - Hugging Face API token
   - Other settings as needed

### 3. Run the Example

```
python example.py
```

### 4. Run the Email Monitor

```
python monitor.py
```

## Setup Instructions

### Gmail API Setup

1. Go to the [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project
3. Enable the Gmail API:
   - Go to "APIs & Services" > "Library"
   - Search for "Gmail API" and enable it
4. Create OAuth credentials:
   - Go to "APIs & Services" > "Credentials"
   - Click "Create Credentials" > "OAuth client ID"
   - Select "Desktop app" as the application type
   - Download the credentials JSON file
5. Place the credentials file in the `email_agent/config` directory or update the path in your config.json

### Hugging Face API Token

1. Create or log in to your [Hugging Face account](https://huggingface.co/)
2. Go to your profile > Settings > Access Tokens
3. Create a new token with "read" access
4. Add the token to your config.json file

## Package Structure

```
email_agent/
├── config/                 # Configuration files
│   ├── __init__.py
│   ├── settings.py         # Settings management
│   └── config.json.template
├── src/                    # Source code
│   ├── __init__.py
│   └── email_agent.py      # Main EmailAgent class
├── docs/                   # Documentation
│   └── README.md
└── __init__.py             # Package initialization

example.py                  # Example usage script
monitor.py                  # Email monitoring script
requirements.txt            # Python dependencies
```

## Usage Examples

### Basic Usage

```python
from email_agent import EmailAgent, Settings

# Load settings from config.json
settings = Settings()

# Create the agent
agent = EmailAgent(settings=settings)

# Get unread emails
unread_emails = agent.get_unread_emails(limit=5)

# Process each email
for email in unread_emails:
    # Process with agent
    result = agent.process_email_with_agent(email["id"])
    
    # Print results
    print(f"Analysis: {result.get('agent_analysis', '')}")
```

### Continuous Monitoring

```python
from email_agent import EmailAgent, Settings
import time

# Load settings
settings = Settings()

# Create the agent
agent = EmailAgent(settings=settings)

# Run continuous monitoring
while True:
    # Get unread emails
    unread_emails = agent.get_unread_emails(limit=5)
    
    # Process each email
    for email in unread_emails:
        result = agent.process_email_with_agent(email["id"])
    
    # Wait for next check
    time.sleep(settings.check_interval)
```

## Customization

You can extend the `EmailAgent` class to add custom functionality:

- Implement your own notification system
- Add custom document processing logic
- Integrate with other services or APIs

## Troubleshooting

### Common Issues

1. **Authentication errors**: Make sure your Gmail API credentials are correct and have the necessary permissions
2. **Rate limiting**: Be mindful of API rate limits for both Gmail API and Hugging Face
3. **PDF processing errors**: Check that Poppler is installed correctly
4. **Hugging Face API errors**: Verify your API token and check the model availability

### Logging

The agent uses Python's logging module. To see detailed logs:

```python
import logging
logging.basicConfig(level=logging.DEBUG)
```
