# GitHub Secrets Import Tool

A production-quality CLI tool that reads variables from a `.env` file and imports them as GitHub repository secrets with validation and conflict resolution.

## Features

- ✅ Parse `.env` files with proper handling of quotes and comments
- ✅ Import secrets to GitHub repository via GitHub API
- ✅ Validate GitHub token and repository access
- ✅ Detect existing secrets and offer conflict resolution options
- ✅ Dry-run mode to preview changes
- ✅ Comprehensive error handling and logging
- ✅ No additional dependencies beyond standard Unix tools
- ✅ Works on macOS and common Linux distributions

## Prerequisites

The following tools must be available on your system:

- `curl` - for GitHub API communication
- `jq` - for JSON processing
- `base64` - for encoding (standard on most systems)
- `openssl` - for encryption fallback (standard on most systems)
- `python3` with `cryptography` library (optional, for better encryption)

Most systems have these tools pre-installed. If `jq` is missing, install it:

**macOS:**
```bash
brew install jq
```

**Ubuntu/Debian:**
```bash
sudo apt-get install jq
```

**CentOS/RHEL:**
```bash
sudo yum install jq
```

## Installation

1. Clone or download this repository
2. Make the script executable:
   ```bash
   chmod +x github-secrets-import/github-secrets-import
   ```
3. Optionally, add to your PATH:
   ```bash
   export PATH="$PATH:/path/to/helper/github-secrets-import"
   ```

## Usage

### Basic Syntax

```bash
./github-secrets-import [OPTIONS] <.env-file> <owner/repo>
```

### Arguments

- `<.env-file>` - Path to the `.env` file containing variables to import
- `<owner/repo>` - GitHub repository in format `owner/repository`

### Options

| Option | Description |
|--------|-------------|
| `-h, --help` | Show help message |
| `-v, --version` | Show version information |
| `-t, --token <token>` | GitHub token (or set `GITHUB_TOKEN` environment variable) |
| `-f, --force` | Overwrite existing secrets without prompting |
| `-s, --skip` | Skip existing secrets without prompting |
| `-d, --dry-run` | Show what would be imported without making changes |

### Examples

**Basic usage:**
```bash
./github-secrets-import .env myuser/myrepo
```

**With explicit GitHub token:**
```bash
./github-secrets-import -t ghp_1234567890abcdef .env myuser/myrepo
```

**Dry run to preview changes:**
```bash
./github-secrets-import --dry-run .env myuser/myrepo
```

**Force overwrite existing secrets:**
```bash
./github-secrets-import --force .env myuser/myrepo
```

**Skip existing secrets automatically:**
```bash
./github-secrets-import --skip .env myuser/myrepo
```

## GitHub Token Setup

You need a GitHub Personal Access Token with appropriate permissions:

1. Go to GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)
2. Click "Generate new token (classic)"
3. Select the following scopes:
   - `repo` (Full control of private repositories)
   - `read:org` (Read org and team membership)
4. Copy the generated token

### Setting the Token

**Method 1: Environment Variable (Recommended)**
```bash
export GITHUB_TOKEN=ghp_your_token_here
./github-secrets-import .env myuser/myrepo
```

**Method 2: Command Line Option**
```bash
./github-secrets-import -t ghp_your_token_here .env myuser/myrepo
```

## .env File Format

The tool supports standard `.env` file format:

```bash
# Comments are ignored
API_KEY=abc123
DATABASE_URL="postgresql://user:pass@localhost/db"
SECRET_KEY='my-secret-key'
MULTI_WORD_VAR=value with spaces

# Empty lines are ignored

# Variables without values are skipped
EMPTY_VAR=
```

### Supported Features

- **Comments**: Lines starting with `#` are ignored
- **Quoted values**: Single and double quotes are supported and removed
- **Spaces**: Leading/trailing spaces in names and values are trimmed
- **Empty lines**: Ignored
- **Variable names**: Must start with letter or underscore, contain only alphanumeric characters and underscores

### Unsupported Features

- Variable expansion (e.g., `VAR2=$VAR1/suffix`)
- Multiline values
- Escape sequences in quoted strings

## Conflict Resolution

When a secret already exists in the repository, you have three options:

1. **Overwrite** - Replace the existing secret with the new value
2. **Skip** - Keep the existing secret, don't import the new value
3. **Interactive** - Prompt for each conflict (default behavior)

Use `--force` to automatically overwrite all existing secrets, or `--skip` to automatically skip all existing secrets.

## Security Considerations

- **Token Security**: Never commit your GitHub token to version control
- **Secret Values**: The tool encrypts secret values using the repository's public key before transmission
- **Logging**: Secret values are never logged or displayed in output
- **Temporary Files**: Any temporary files are securely cleaned up
- **Encryption**: Uses GitHub's recommended encryption method with fallback options

## Error Handling

The tool provides comprehensive error handling for common scenarios:

- **Invalid .env file**: Validates file existence, readability, and format
- **GitHub API errors**: Handles authentication, authorization, and network issues
- **Repository access**: Validates repository existence and permissions
- **Encryption failures**: Provides fallback encryption methods
- **Invalid token**: Validates token format and permissions

## Output and Logging

The tool provides clear, structured output:

```
[INFO] Starting GitHub secrets import process
[INFO] Parsing environment file: .env
[INFO] Found 5 environment variables to import
[INFO] Getting repository public key for encryption
[INFO] Getting existing repository secrets
[WARN] Secret 'API_KEY' already exists
[INFO] Overwriting existing secret: API_KEY
[SUCCESS] Secret created/updated: API_KEY
[INFO] Import process completed
[INFO] Successfully processed: 5 secrets
[INFO] Skipped: 0 secrets
[INFO] Errors: 0 secrets
```

## Troubleshooting

### Common Issues

**"Missing required dependencies"**
- Install missing tools (`curl`, `jq`, etc.)
- Ensure tools are in your PATH

**"Invalid GitHub token"**
- Check token format (should start with `ghp_`, `gho_`, etc.)
- Verify token has correct permissions
- Check token hasn't expired

**"Repository not found or no access"**
- Verify repository name format (`owner/repo`)
- Check token has access to the repository
- Ensure repository exists

**"Failed to encrypt secret"**
- Install Python `cryptography` library: `pip install cryptography`
- Ensure `openssl` is available for fallback encryption

**"Failed to parse .env file"**
- Check file format and encoding
- Verify file is readable
- Check for invalid variable names

### Debug Mode

For additional debugging, you can modify the script to enable verbose output:

```bash
# Add this line after the shebang for verbose output
set -x
```

## Contributing

1. Follow the existing code style and conventions
2. Add comprehensive error handling for new features
3. Update documentation for any changes
4. Test on both macOS and Linux systems

## License

MIT License - see repository root for full license text.

## Version History

- **v1.0.0** - Initial release with core functionality
  - .env file parsing
  - GitHub API integration
  - Conflict resolution
  - Comprehensive error handling
  - Dry-run mode
  - Production-quality logging