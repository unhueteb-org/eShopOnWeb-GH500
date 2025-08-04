# CodeQL CLI Installation Guide

This guide provides step-by-step instructions for installing the CodeQL CLI in a development environment.

## Prerequisites

- Linux environment (tested on Debian GNU/Linux 12 bookworm)
- `wget`, `unzip`, and `git` available on the system
- `sudo` access for installation

## Installation Steps

### 1. Download CodeQL CLI

First, download the latest CodeQL CLI for Linux from GitHub releases:

```bash
cd /tmp
wget https://github.com/github/codeql-cli-binaries/releases/latest/download/codeql-linux64.zip
```

### 2. Extract the Archive

Extract the downloaded zip file:

```bash
unzip -q codeql-linux64.zip
```

### 3. Install to System Location

Move CodeQL to a permanent location and make it executable:

```bash
sudo mv /tmp/codeql /opt/codeql
sudo chmod +x /opt/codeql/codeql
```

### 4. Configure PATH

Add CodeQL to your PATH for easy access:

```bash
# Add to current session
export PATH="/opt/codeql:$PATH"

# Add to bashrc for future sessions
echo 'export PATH="/opt/codeql:$PATH"' >> ~/.bashrc
```

### 5. Verify Installation

Test that CodeQL CLI is working correctly:

```bash
codeql --version
```

Expected output:
```
CodeQL command-line toolchain release 2.22.2.
Copyright (C) 2019-2025 GitHub, Inc.
Unpacked in: /opt/codeql
   Analysis results depend critically on separately distributed query and
   extractor modules. To list modules that are visible to the toolchain,
   use 'codeql resolve packs' and 'codeql resolve languages'.
```

## Optional: Install CodeQL Standard Library

For running standard security queries, you may want to clone the CodeQL standard library:

```bash
cd /opt
sudo git clone https://github.com/github/codeql.git codeql-library
```

## Basic Usage

### Check Available Languages

```bash
codeql resolve languages
```

### Check Available Query Packs

```bash
codeql resolve packs --show-hidden-packs
```

### Create a Database

To analyze your code, first create a CodeQL database:

```bash
# For C# projects
codeql database create my-database --language=csharp --command="dotnet build"

# For JavaScript/TypeScript projects
codeql database create my-database --language=javascript

# For multiple languages
codeql database create my-database --language=csharp,javascript
```

### Run Queries

Run security queries against your database:

```bash
# Run all security queries
codeql database analyze my-database --format=csv --output=results.csv

# Run specific query suite
codeql database analyze my-database codeql/csharp-queries:codeql-suites/csharp-security-extended.qls --format=csv --output=results.csv
```

## DEMO FOR REPO

### Database Creation Process

**Source Code Extraction**: CodeQL analyzed the build process (`dotnet build eShopOnWeb.sln`) to understand how your code is compiled

**Code Parsing**: It parsed all C# files in your solution (8,697 lines of code across multiple projects)

**Relationship Mapping**: CodeQL created a relational database that represents your code's structure, including:
- Classes, methods, and variables
- Control flow (how execution moves through your code)
- Data flow (how data moves between variables)
- Call graphs (which methods call which other methods)
- Type hierarchies and inheritance relationships

### Commands Used

1. Create the CodeQL database:
```bash
codeql database create codeql-analysis/eshop-solution-db --language=csharp --command="dotnet build eShopOnWeb.sln" 
```

2. Run security analysis on the database:
```bash
codeql database analyze /workspaces/eShopOnWeb-GH500/codeql-analysis/eshop-solution-db codeql/csharp-queries --format=sarif-latest --output=codeql-analysis/security-results.sarif
```

### Analysis Process

**Query Execution**: CodeQL runs a comprehensive set of security queries against the database to identify potential vulnerabilities and code quality issues.

**Security Checks Include**:
- **SQL Injection**: Detects where user input might reach database queries without proper sanitization
- **Cross-Site Scripting (XSS)**: Identifies untrusted data flowing to web outputs
- **Authentication/Authorization**: Finds missing security checks or improper access controls
- **Injection Vulnerabilities**: Discovers command injection, LDAP injection, and other injection flaws
- **Resource Leaks**: Identifies unclosed database connections, files, or other resources
- **Null Reference Issues**: Detects potential null pointer exceptions

**Output Format**: Results are saved in SARIF (Static Analysis Results Interchange Format) which can be:
- Viewed in VS Code with the SARIF extension
- Integrated into CI/CD pipelines
- Uploaded to GitHub Security tab for pull request annotations

## Troubleshooting

### Common Issues

1. **Permission denied**: Ensure CodeQL binary is executable with `chmod +x`
2. **Command not found**: Verify that `/opt/codeql` is in your PATH
3. **Database creation fails**: Ensure the build command works independently

### Useful Commands

- `codeql --help`: Get help information
- `codeql version`: Show version information
- `codeql resolve languages`: List supported languages
- `codeql resolve packs`: List available query packs

## Resources

- [CodeQL Documentation](https://codeql.github.com/)
- [CodeQL CLI Reference](https://codeql.github.com/docs/codeql-cli/)
- [CodeQL Queries Repository](https://github.com/github/codeql)