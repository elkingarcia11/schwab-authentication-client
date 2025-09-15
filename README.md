# Charles Schwab Authentication Module

A comprehensive Python module for handling Charles Schwab API authentication with OAuth 2.0 flow, automatic token management, and Google Cloud Storage integration.

## Features

- 🔐 **OAuth 2.0 Authentication**: Complete implementation of Schwab API OAuth flow
- 🔄 **Flexible Token Management**: Support for both fresh authentication and GCS-stored refresh tokens
- 💾 **Token Persistence**: Saves refresh tokens locally and to Google Cloud Storage
- ☁️ **Cloud Storage Integration**: Automatic token backup and retrieval from GCS
- 🛡️ **Robust Error Handling**: Comprehensive error handling for all authentication scenarios
- 🎯 **Easy Integration**: Simple class-based design for seamless integration into larger projects
- ⚡ **Dual Mode Operation**: Fresh authentication or GCS-based token refresh
- 🔧 **Modular Architecture**: Separate GCS module for cloud storage operations

## Prerequisites

- Python 3.8+
- Charles Schwab Developer Account
- Registered Schwab API Application
- Google Cloud Storage Account (for cloud token storage)

## Installation

1. **Clone the repository with submodules:**

   ```bash
   git clone --recursive <repository-url>
   cd charles-schwab-authentication-module
   ```

2. **Create and activate a virtual environment:**

   ```bash
   # Create virtual environment
   python -m venv venv

   # Activate virtual environment
   # On macOS/Linux:
   source venv/bin/activate

   # On Windows:
   venv\Scripts\activate
   ```

3. **Install required dependencies:**

   ```bash
   # Ensure pip is up to date
   pip install --upgrade pip

   # Install dependencies
   pip install -r requirements.txt
   ```

4. **Initialize submodules:**
   ```bash
   git submodule update --init --recursive
   ```

## Setup

### 1. Get Schwab API Credentials

1. Visit the [Charles Schwab Developer Portal](https://developer.schwab.com/)
2. Create a developer account
3. Register a new application
4. Note your App Key and App Secret
5. Set your redirect URI to `https://127.0.0.1`

### 2. Set up Google Cloud Storage

1. Create a Google Cloud Storage bucket for token storage
2. Set up a service account with Storage Object Admin permissions
3. Download the service account key (JSON format)
4. Configure the `GOOGLE_APPLICATION_CREDENTIALS` environment variable

### 3. Configure Environment Variables

Create a `.env` file with your credentials:

```env
SCHWAB_APP_KEY=your_schwab_app_key
SCHWAB_APP_SECRET=your_schwab_app_secret
GCS_BUCKET_NAME=your_gcs_bucket_name
GOOGLE_APPLICATION_CREDENTIALS=/path/to/your/service-account-key.json
```

## Usage

### Method 1: Fresh Authentication (Interactive)

Run the script to authenticate locally and upload refresh token to GCS:

```bash
# Authenticate locally and upload refresh token to GCS
python schwab_auth.py --authenticate
# or simply (default behavior)
python schwab_auth.py
```

This will:

1. Prompt you to visit the authorization URL
2. Ask you to paste the redirect URL after authorization
3. Generate fresh access and refresh tokens
4. Display the access token for use in your applications
5. **Automatically upload the refresh token to Google Cloud Storage**

### Method 2: GCS-Based Token Refresh (Automated)

Get a new access token using the refresh token stored in Google Cloud Storage:

```bash
python schwab_auth.py --get-access-token
```

This will:

1. Download the refresh token from Google Cloud Storage (if not present locally)
2. Use the refresh token to get a new access token
3. Display the access token without requiring user interaction
4. Perfect for automated scripts and services

### Expected Output

When you run the program, you'll see:

```
=== GETTING FRESH TOKENS ===
Getting new Schwab tokens...

1. Visit this URL: https://api.schwabapi.com/v1/oauth/authorize?client_id=...
2. Log in and authorize the app
3. You'll be redirected to a URL that looks like:
   https://127.0.0.1/?code=LONG_CODE_HERE&session=...
4. Copy the ENTIRE redirect URL

Paste the full redirect URL here: [YOU PASTE THE URL HERE]
Extracted code: ABC123...
Tokens obtained successfully!
Access Token: eyJ0eXAiOiJKV1QiLCJ...
Refresh Token: eyJ0eXAiOiJKV1QiLCJ...
Refresh token saved to schwab_refresh_token.txt
File schwab_refresh_token.txt uploaded to schwab_refresh_token.txt.
✅ Tokens obtained and uploaded to GCS successfully!
```

## API Reference

### SchwabAuth Class

#### `__init__()`

Initializes the authenticator with credentials from environment variables.

**Environment Variables Required:**

- `SCHWAB_APP_KEY`: Your Schwab application key
- `SCHWAB_APP_SECRET`: Your Schwab application secret
- `GCS_BUCKET_NAME`: Your Google Cloud Storage bucket name (optional)
- `GOOGLE_APPLICATION_CREDENTIALS`: Path to your GCS service account key file (optional)

#### `get_valid_access_token(use_gcs_refresh_token=False)`

Main method to get a valid access token.

**Parameters:**

- `use_gcs_refresh_token` (bool): If True, attempts to use GCS-stored refresh token first

**Returns:**

- `str`: Valid access token, or None if authentication fails

**Example:**

```python
auth = SchwabAuth()

# Fresh authentication (interactive)
access_token = auth.get_valid_access_token()

# Use GCS refresh token (automated)
access_token = auth.get_valid_access_token(use_gcs_refresh_token=True)
```

#### `automated_token_management()`

Handles the complete fresh token generation workflow.

**Returns:**

- `str`: Refresh token, or None if authentication fails

**Example:**

```python
auth = SchwabAuth()
refresh_token = auth.automated_token_management()
```

#### `refresh_access_token(refresh_token, app_key, app_secret)`

Refreshes access token using refresh token.

**Parameters:**

- `refresh_token` (str): The refresh token to use
- `app_key` (str): Your Schwab application key
- `app_secret` (str): Your Schwab application secret

**Returns:**

- `str`: New access token, or None if refresh fails

#### `save_refresh_token(refresh_token)`

Saves refresh token to local file.

**Parameters:**

- `refresh_token` (str): The refresh token to save

#### `load_refresh_token()`

Loads refresh token from local file.

**Returns:**

- `str`: Refresh token if found, None otherwise

#### `upload_refresh_token_to_gcs()`

Uploads the local refresh token file to Google Cloud Storage.

#### `download_refresh_token_from_gcs()`

Downloads refresh token from Google Cloud Storage to local file.

**Returns:**

- `str`: Refresh token if download successful, None otherwise

## Integration Examples

### Basic Integration

```python
from schwab_auth import SchwabAuth

# Initialize the authenticator
auth = SchwabAuth()

# Get access token (automatically uses GCS if available)
access_token = auth.get_valid_access_token(use_gcs_refresh_token=True)

# Use the token for API calls
headers = {
    'Authorization': f'Bearer {access_token}',
    'Content-Type': 'application/json'
}

# Make API requests
import requests
response = requests.get('https://api.schwabapi.com/v1/accounts', headers=headers)
```

### Integration with Streaming Client

```python
from schwab_auth import SchwabAuth

# In your streaming client
auth = SchwabAuth()
access_token = auth.get_valid_access_token(use_gcs_refresh_token=True)

# Use token for WebSocket authentication
streaming_client.login_with_token(access_token)
```

### Automated Script Integration

```python
from schwab_auth import SchwabAuth
import time

def get_schwab_data():
    auth = SchwabAuth()

    # Try GCS first, fall back to fresh auth if needed
    access_token = auth.get_valid_access_token(use_gcs_refresh_token=True)

    if not access_token:
        print("GCS token failed, getting fresh authentication...")
        access_token = auth.get_valid_access_token(use_gcs_refresh_token=False)

    return access_token

# Use in automated scripts
access_token = get_schwab_data()
```

## File Structure

```
charles-schwab-authentication-module/
├── schwab_auth.py                    # Main authentication class
├── schwab_refresh_token.txt          # Stored refresh token (auto-generated)
├── .env                              # Environment variables (you create this)
├── requirements.txt                  # Python dependencies
├── README.md                         # This file
├── .gitmodules                       # Git submodule configuration
└── gcs-python-module/                # Google Cloud Storage module
    ├── gcs_client.py                 # GCS client class
    ├── requirements.txt              # GCS module dependencies
    ├── README.md                     # GCS module documentation
    └── tests/                        # GCS module tests
        ├── integration_test.py
        └── README.md
```

## Environment Variables

| Variable                         | Description                               | Required | Default |
| -------------------------------- | ----------------------------------------- | -------- | ------- |
| `SCHWAB_APP_KEY`                 | Your Schwab application key               | Yes      | -       |
| `SCHWAB_APP_SECRET`              | Your Schwab application secret            | Yes      | -       |
| `GCS_BUCKET_NAME`                | Your Google Cloud Storage bucket name     | No       | -       |
| `GOOGLE_APPLICATION_CREDENTIALS` | Path to your GCS service account key file | No       | -       |

## Error Handling

The module includes comprehensive error handling for:

- Missing environment variables
- API request failures
- Invalid authorization codes
- Token refresh failures
- File I/O errors
- Google Cloud Storage operation failures
- Network connectivity issues
- Authentication flow interruptions

## Security Notes

- **Never commit your `.env` file** to version control
- **Keep your App Key and App Secret secure**
- **The refresh token file contains sensitive data** - protect it accordingly
- **Use HTTPS in production environments**
- **Store your Google Cloud service account key securely**
- **Consider using Google Cloud Secret Manager** for production deployments
- **Rotate credentials regularly** for enhanced security

## Troubleshooting

### Common Issues

1. **"No valid refresh token available"**

   - **Solution**: Run the program again to get fresh tokens
   - **Check**: Ensure GCS bucket exists and is accessible

2. **"Error getting tokens"**

   - **Check**: Your App Key and App Secret are correct
   - **Verify**: Redirect URI matches your app configuration
   - **Ensure**: Authorization code is complete and unmodified

3. **"Failed to get access token"**

   - **Try**: Running the program again to get fresh tokens
   - **Check**: Internet connection and API credentials
   - **Verify**: Refresh token hasn't expired

4. **Google Cloud Storage Errors**

   - **Verify**: Service account has correct permissions
   - **Check**: Bucket name is correct and exists
   - **Ensure**: Service account key file path is correct
   - **Test**: GCS connectivity and bucket access

5. **Virtual Environment Issues**

   - **Ensure**: Virtual environment is activated before installing dependencies
   - **Fix**: Use `venv\Scripts\activate` on Windows instead of `source venv/bin/activate`
   - **Reinstall**: `pip install -r requirements.txt` if packages aren't found

6. **Git Submodule Issues**
   - **Initialize**: `git submodule init && git submodule update`
   - **Recursive**: `git submodule update --init --recursive`
   - **Verify**: `gcs-python-module/` directory exists and contains `gcs_client.py`

### Debug Mode

For debugging, you can add print statements or modify the error messages in the class methods:

```python
# Enable debug output
import logging
logging.basicConfig(level=logging.DEBUG)

# Or add debug prints
auth = SchwabAuth()
print(f"App Key: {auth.APP_KEY[:10]}...")
print(f"GCS Bucket: {auth.GCS_BUCKET_NAME}")
```

## Google Cloud Storage Module

The project includes a comprehensive Google Cloud Storage module (`gcs-python-module/`) that provides:

- **Service Account Authentication**: Secure authentication using service account keys
- **Comprehensive Operations**: Upload, download, list, delete, and manage files and buckets
- **Error Handling**: Robust error handling with detailed logging
- **Type Hints**: Full type annotation support for better development experience
- **Testing**: Integration tests for all GCS operations

See the [gcs-python-module/README.md](gcs-python-module/README.md) for detailed documentation on the GCS client functionality.

## Testing

Run tests with pytest:

```bash
pytest tests/
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Disclaimer

This tool is for educational and development purposes. Always follow Charles Schwab's API terms of service and rate limiting guidelines. The tool is not intended for production trading without proper testing and validation.

## Support

For issues related to:

- **Schwab API**: Contact Charles Schwab Developer Support
- **Google Cloud Storage**: Check the [GCS module documentation](gcs-python-module/README.md)
- **This Tool**: Open an issue in this repository
- **Integration**: Check the parent project documentation

## Changelog

### v2.1.0

- **Enhanced GCS Integration**: Improved token management with fallback options
- **Better Error Handling**: More comprehensive error scenarios and recovery
- **Updated Documentation**: Comprehensive README with integration examples
- **Python 3.8+ Support**: Updated minimum Python version requirement

### v2.0.0

- **Added Google Cloud Storage Integration**: Automatic token upload to GCS
- **Modular Architecture**: Separated GCS functionality into dedicated module
- **Enhanced Security**: Cloud-based token backup and storage
- **Comprehensive GCS Client**: Full GCS operations support
- **Updated Dependencies**: Added Google Cloud Storage requirements

### v1.0.0

- **Initial Release**: Class-based architecture
- **Fresh Token Generation**: Always gets fresh tokens on every run
- **Token Persistence**: Saves refresh tokens for reference
- **Comprehensive Error Handling**: Robust error handling for API calls
