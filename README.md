# codex_setup_101

# Codex CLI Installation and Configuration Tutorial

A complete step-by-step guide to installing, authenticating, and customizing OpenAI's Codex CLI on Linux.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Installation](#installation)
3. [Authentication](#authentication)
4. [Configuration](#configuration)
5. [Usage Examples](#usage-examples)
6. [Troubleshooting](#troubleshooting)
7. [Advanced Features](#advanced-features)

---

## Prerequisites

- Linux system (Ubuntu/Debian recommended)
- Internet connection
- Sudo privileges
- OpenAI API key (for authentication)

---

## Step 1: Installation

### 1.1 Install Node.js and npm

First, we need to install Node.js and npm to use the Codex CLI:

```bash
# Update package list
sudo apt update

# Install Node.js and npm
sudo apt install -y nodejs npm
```

**Alternative: Using Node Version Manager (nvm)**
If you prefer not to use sudo or want a specific Node.js version:

```bash
# Install nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Reload shell configuration
source ~/.bashrc

# Install and use latest LTS Node.js
nvm install --lts
nvm use --lts
```

### 1.2 Install Codex CLI

Install Codex CLI globally using npm:

```bash
# Install Codex CLI globally
sudo npm install -g @openai/codex
```

### 1.3 Verify Installation

Check that Codex is properly installed:

```bash
# Check version
codex --version

# View help
codex --help
```

**Expected Output:**
```
codex-cli 0.50.0
```

---

## Step 2: Authentication

### 2.1 Check Authentication Status

```bash
# Check current login status
codex login status
```

**Expected Output (before authentication):**
```
Not logged in
```

### 2.2 Authenticate with API Key

**Method 1: Direct API Key Input**
```bash
# Replace 'your-api-key-here' with your actual OpenAI API key
echo 'your-api-key-here' | codex login --with-api-key
```

**Method 2: From Environment Variable**
```bash
# If you have your API key in an environment variable
echo $OPENAI_API_KEY | codex login --with-api-key
```

**Method 3: From File**
```bash
# If you have your API key stored in a file
cat ~/.openai_api_key | codex login --with-api-key
```

### 2.3 Verify Authentication

```bash
# Check login status
codex login status
```

**Expected Output (after successful authentication):**
```
Logged in using an API key - sk-proj-***O90YA
```

---

## Step 3: Configuration

### 3.1 Create Configuration Directory

```bash
# Create Codex configuration directory
mkdir -p ~/.codex
```

### 3.2 Create Configuration File

Create a comprehensive configuration file at `~/.codex/config.toml`:

```bash
# Create the configuration file
cat > ~/.codex/config.toml << 'EOF'
# =============================================================================
# Codex CLI Configuration File
# =============================================================================
# This file allows you to customize Codex CLI behavior and create different
# profiles for various use cases. Save this file as ~/.codex/config.toml
#
# Usage Examples:
#   codex                           # Uses default settings below
#   codex --profile development     # Uses development profile
#   codex --profile production      # Uses production profile
#   codex --profile experimental    # Uses experimental profile
#   codex -c model="gpt-5-codex"    # Override specific setting
# =============================================================================

# =============================================================================
# MODEL CONFIGURATION
# =============================================================================
# Choose which AI model Codex should use by default
model = "gpt-5-codex"  # Default model: gpt-5-codex, gpt-5, gpt-4, etc.
# model_provider = "openai"  # Provider options: "openai" (default), "oss" (for Ollama)

# =============================================================================
# SANDBOX CONFIGURATION
# =============================================================================
# Controls how Codex can interact with your system
sandbox_mode = "workspace-write"  # Security levels:
                                  # - "read-only": Can only read files (safest)
                                  # - "workspace-write": Can read/write in current directory (recommended)
                                  # - "danger-full-access": Full system access (dangerous!)

# =============================================================================
# APPROVAL POLICY
# =============================================================================
# Controls when Codex asks for your permission before running commands
ask_for_approval = "on-failure"  # Options:
                                 # - "untrusted": Ask for approval on potentially dangerous commands
                                 # - "on-failure": Auto-run commands, ask only if they fail
                                 # - "on-request": Let Codex decide when to ask
                                 # - "never": Never ask (dangerous!)

# =============================================================================
# SHELL ENVIRONMENT
# =============================================================================
# Controls which environment variables Codex can access
[shell_environment_policy]
inherit = "all"  # Options:
                 # - "all": Inherit all environment variables
                 # - "none": Don't inherit any environment variables
                 # - ["VAR1", "VAR2"]: Inherit only specific variables

# =============================================================================
# SANDBOX PERMISSIONS
# =============================================================================
# Fine-grained control over what Codex can access
sandbox_permissions = [
    "disk-full-read-access",  # Can read any file on disk
    "disk-write-access",      # Can write files (respects sandbox_mode)
    "network-access"          # Can make network requests
]

# =============================================================================
# FEATURE FLAGS
# =============================================================================
# Enable/disable specific Codex features
[features]
# STABLE FEATURES (safe to use)
view_image_tool = true          # Allow Codex to view and analyze images
web_search_request = true       # Enable web search capabilities

# BETA FEATURES (may have bugs)
apply_patch_freeform = false    # Allow applying patches in freeform mode

# EXPERIMENTAL FEATURES (use with caution)
unified_exec = false            # Unified execution system (experimental)
streamable_shell = false        # Stream shell output in real-time (experimental)
rmcp_client = false             # Model Context Protocol client (experimental)
experimental_sandbox_command_assessment = false  # Advanced command safety (experimental)

# =============================================================================
# WEB SEARCH CONFIGURATION
# =============================================================================
[web_search]
enabled = false  # Enable/disable web search capability
                 # Can also be enabled with: codex --search

# =============================================================================
# LOGGING CONFIGURATION
# =============================================================================
[logging]
level = "info"  # Logging levels:
                # - "debug": Very verbose output (useful for troubleshooting)
                # - "info": Standard output (recommended)
                # - "warn": Only warnings and errors
                # - "error": Only errors

# =============================================================================
# SESSION CONFIGURATION
# =============================================================================
[session]
auto_save = true     # Automatically save session history
max_history = 100    # Maximum number of commands to keep in history

# =============================================================================
# CUSTOM PROFILES
# =============================================================================
# Create different configurations for different use cases
# Use with: codex --profile <profile-name>

# DEVELOPMENT PROFILE
# Good for coding and development work
[profiles.development]
model = "gpt-5-codex"                    # Use the best coding model
sandbox_mode = "workspace-write"         # Allow file modifications
ask_for_approval = "on-failure"          # Auto-run until failure
features.web_search_request = true       # Enable web search for documentation

# PRODUCTION PROFILE
# Safe settings for production environments
[profiles.production]
model = "gpt-5-codex"                    # Use reliable model
sandbox_mode = "read-only"               # Prevent accidental changes
ask_for_approval = "untrusted"           # Ask before running commands
features.web_search_request = false      # Disable web access

# EXPERIMENTAL PROFILE
# For testing new features (use with caution!)
[profiles.experimental]
model = "gpt-5-codex"                           # Use latest model
sandbox_mode = "danger-full-access"             # Full system access
ask_for_approval = "never"                      # No approval prompts
features.unified_exec = true                    # Enable experimental execution
features.streamable_shell = true                # Enable real-time output

# =============================================================================
# QUICK REFERENCE
# =============================================================================
# Command Line Overrides:
#   codex -c model="gpt-4"                    # Change model
#   codex -c sandbox_mode="read-only"         # Change sandbox mode
#   codex --enable web_search_request         # Enable feature
#   codex --disable view_image_tool           # Disable feature
#   codex --full-auto                         # Auto-run with workspace write
#   codex --search                            # Enable web search
#   codex --sandbox read-only                 # Set sandbox mode
#   codex --ask-for-approval untrusted        # Set approval policy
#
# Common Usage Patterns:
#   codex --profile development "help me code"     # Development work
#   codex --profile production "analyze this"      # Safe analysis
#   codex --full-auto "fix the bugs"               # Auto-fix with safety
#   codex --search "find React documentation"      # With web search
# =============================================================================
EOF
```

### 3.3 Verify Configuration

```bash
# Check that the configuration file was created
ls -la ~/.codex/config.toml

# View the first few lines to confirm
head -20 ~/.codex/config.toml
```

---

## Step 4: Usage Examples

### 4.1 Basic Usage

**Interactive Mode:**
```bash
# Start Codex in interactive mode
codex
```

**Direct Command:**
```bash
# Run Codex with a specific prompt
codex "help me understand this codebase"
```

**Non-interactive Execution:**
```bash
# Run Codex non-interactively
codex exec "fix the linting errors in this project"
```

### 4.2 Using Profiles

**Development Profile:**
```bash
# Use development profile for coding work
codex --profile development "help me write a Python function"
```

**Production Profile:**
```bash
# Use production profile for safe analysis
codex --profile production "analyze this code for security issues"
```

**Experimental Profile:**
```bash
# Use experimental profile for testing new features
codex --profile experimental "test the latest features"
```

### 4.3 Command Line Overrides

**Change Model:**
```bash
# Use a different model
codex --model gpt-4 "explain this code"
```

**Enable Web Search:**
```bash
# Enable web search capability
codex --search "find the latest React documentation"
```

**Full Auto Mode:**
```bash
# Auto-run commands with workspace write access
codex --full-auto "fix the bugs in this project"
```

**Custom Configuration:**
```bash
# Override specific settings
codex -c model="gpt-5-codex" -c sandbox_mode="read-only" "analyze this"
```

### 4.4 Feature Management

**Enable Features:**
```bash
# Enable specific features
codex --enable web_search_request --enable view_image_tool
```

**Disable Features:**
```bash
# Disable specific features
codex --disable view_image_tool
```

**Check Available Features:**
```bash
# List all available features
codex features list
```

---

## Step 5: Troubleshooting

### 5.1 Common Issues

**Issue: "npm: command not found"**
```bash
# Solution: Install Node.js and npm
sudo apt update && sudo apt install -y nodejs npm
```

**Issue: "Permission denied" when installing globally**
```bash
# Solution: Use sudo for global installation
sudo npm install -g @openai/codex
```

**Issue: "Not logged in"**
```bash
# Solution: Authenticate with your API key
echo 'your-api-key' | codex login --with-api-key
```

**Issue: "Device code request failed"**
```bash
# Solution: Use API key authentication instead
echo 'your-api-key' | codex login --with-api-key
```

### 5.2 Debugging

**Enable Debug Logging:**
```bash
# Run with debug logging
codex -c logging.level="debug" "your command"
```

**Check Configuration:**
```bash
# Verify configuration file exists and is readable
cat ~/.codex/config.toml
```

**Check Authentication:**
```bash
# Verify authentication status
codex login status
```

---

## Step 6: Advanced Features

### 6.1 Image Analysis

```bash
# Analyze an image
codex -i screenshot.png "explain what's in this image"
```

### 6.2 Working Directory

```bash
# Run Codex in a specific directory
codex -C /path/to/project "analyze this project"
```

### 6.3 Additional Directories

```bash
# Give Codex access to additional directories
codex --add-dir /path/to/other/dir "work with files in both directories"
```

### 6.4 Session Management

```bash
# Resume a previous session
codex resume

# Resume the most recent session
codex resume --last
```

### 6.5 Apply Changes

```bash
# Apply the latest diff produced by Codex
codex apply
```

---

## Step 7: Best Practices

### 7.1 Security

- **Start with read-only mode** for unfamiliar codebases
- **Use development profile** for coding work
- **Use production profile** for analysis
- **Never use experimental profile** in production

### 7.2 Performance

- **Use appropriate models** for your needs
- **Enable web search** only when needed
- **Use profiles** to quickly switch between configurations
- **Monitor resource usage** with experimental features

### 7.3 Workflow

- **Test commands** in a safe environment first
- **Use version control** before letting Codex make changes
- **Review changes** before applying them
- **Keep backups** of important files

---

## Conclusion

You now have a fully configured Codex CLI installation with:

✅ **Codex CLI installed** and authenticated  
✅ **Comprehensive configuration** with multiple profiles  
✅ **Security settings** appropriate for different use cases  
✅ **Feature flags** for experimental capabilities  
✅ **Documentation** for all configuration options  

### Quick Start Commands

```bash
# Start with development profile
codex --profile development

# Enable web search
codex --search "help me with React"

# Auto-fix with safety
codex --full-auto "fix the bugs"

# Safe analysis
codex --profile production "analyze this code"
```

### Next Steps

1. **Explore the codebase** with `codex "explain this project"`
2. **Fix issues** with `codex --full-auto "fix the linting errors"`
3. **Get help** with `codex --search "find documentation"`
4. **Experiment** with different profiles and features

Happy coding with Codex CLI! 🚀

---

## Additional Resources

- [Official Codex CLI Documentation](http://developers.openai.com/codex/cli/)
- [Codex CLI GitHub Repository](https://github.com/openai/codex)
- [OpenAI API Documentation](https://platform.openai.com/docs)
- [Node.js Documentation](https://nodejs.org/docs/)

---

*This tutorial was created based on a complete installation and configuration process on Ubuntu Linux.*
