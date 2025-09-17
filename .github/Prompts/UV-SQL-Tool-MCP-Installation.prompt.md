# UV SQL Tool MCP Server Installation Guide

## Quick Setup Instructions

### 1. Install UV Python Package Manager
```powershell
# Install UV using PowerShell
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"

# Add UV to PATH for current session
$env:Path += ";$HOME\.local\bin"

# Verify installation
uv --version
```

### 2. Configure MCP Server
Create or update `.vscode/mcp.json` in your workspace:

```json
{
  "servers": {
    "uv-sql-tool": {
      "command": "uvx",
      "args": [
        "--from",
        "git+https://github.com/varuns-sunrise/uvsqltool.git",
        "uv-sql-server"
      ],
      "env": {
        "SQL_SERVER": "your-server-name",
        "SQL_DATABASE": "your-database",
        "SQL_USERNAME": "your-username",
        "SQL_PASSWORD": "your-password",
        "SQL_DRIVER": "ODBC Driver 17 for SQL Server",
        "SQL_PORT": "",
        "SQL_TRUSTED_CONNECTION": "false",
        "SQL_ENCRYPT": "true",
        "SQL_TRUST_SERVER_CERTIFICATE": "false",
        "SQL_CONNECTION_TIMEOUT": "30",
        "SQL_COMMAND_TIMEOUT": "30"
      }
    }
  }
}
```

### 3. Environment Configuration
Update the environment variables in your `mcp.json` with your actual SQL Server details:
- `SQL_SERVER`: Your SQL Server instance name or IP
- `SQL_DATABASE`: Target database name
- `SQL_USERNAME`: Your SQL Server username
- `SQL_PASSWORD`: Your SQL Server password

## Common Issues & Solutions

### "Access Denied" Error (Windows)
**Problem**: `error: Failed to spawn: uv-sql-server. Caused by: Access is denied. (os error 5)`

**Solutions**:
1. **Add Windows Defender Exclusion** (Recommended):
   - Open Windows Security
   - Go to Virus & threat protection
   - Click "Manage settings" under Virus & threat protection settings
   - Add an exclusion for folder: `C:\Users\[your-username]\.local\bin\`

2. **Run VS Code as Administrator**:
   - Close VS Code
   - Right-click VS Code icon
   - Select "Run as administrator"

### "Package Not Found" Error
**Problem**: `No solution found when resolving tool dependencies`

**Solution**: Ensure your `mcp.json` includes the `--from` argument pointing to the correct GitHub repository.

### Server Connection Issues
**Problem**: Server exits before responding to initialize request

**Solutions**:
1. Verify your SQL Server connection details
2. Test SQL Server connectivity separately
3. Check firewall settings
4. Ensure SQL Server allows remote connections

## Testing Installation

### Verify UV Installation
```powershell
uv --version
```

### Test Server Command (Optional)
```powershell
uvx --from "git+https://github.com/varuns-sunrise/uvsqltool.git" uv-sql-server --help
```

## Prerequisites
- Windows 10/11
- PowerShell 5.1 or later
- SQL Server access with valid credentials
- ODBC Driver 17 for SQL Server
- VS Code with MCP extension

## Next Steps
1. Complete the installation steps above
2. Update your SQL Server credentials in `mcp.json`
3. Restart VS Code
4. Test the MCP server connection

## Support
If you encounter issues:
1. Check the VS Code output panel for detailed error messages
2. Verify your SQL Server connection independently
3. Ensure all prerequisites are installed
4. Try running VS Code as administrator if permission issues persist