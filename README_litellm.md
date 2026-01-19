# OpenCode with LiteLLM

OpenCode is an AI-powered code intelligence platform that can be configured to use LiteLLM as a proxy for various AI models, including local models via Ollama.

## Installation

### Option 1: CLI Installation

#### Linux (x64)

Download and install the OpenCode CLI:

```bash
# Download the latest release
wget https://github.com/ai-codespark/opencode/releases/latest/download/opencode-linux-x64.tar.gz

# Extract to /usr/local
sudo mkdir -p /usr/local/opencode
sudo tar -xzf opencode-linux-x64.tar.gz -C /usr/local/opencode

# Create symlink
sudo ln -s /usr/local/opencode/opencode /usr/local/bin/opencode

# Verify installation
opencode --version
```

#### Windows (x64)

Download and install the OpenCode CLI:

```powershell
# Download the latest release
Invoke-WebRequest -Uri "https://github.com/ai-codespark/opencode/releases/latest/download/opencode-windows-x64.zip" -OutFile "opencode-windows-x64.zip"

# Extract the archive
Expand-Archive -Path opencode-windows-x64.zip -DestinationPath "$env:LOCALAPPDATA\OpenCode"

# Add to PATH (run as Administrator)
[Environment]::SetEnvironmentVariable("Path", "$env:Path;$env:LOCALAPPDATA\OpenCode", "Machine")

# Verify installation (restart terminal first)
opencode --version
```

### Option 2: Desktop Application (Windows)

Download the desktop installer:

```powershell
# Download the latest release
Invoke-WebRequest -Uri "https://github.com/ai-codespark/opencode/releases/latest/download/opencode-desktop-windows-x64.exe" -OutFile "opencode-desktop-installer.exe"

# Run the installer
.\opencode-desktop-installer.exe
```

### Option 3: Docker

Pull and run the Docker image:

```bash
# Pull the latest image
docker pull ghcr.io/ai-codespark/opencode:latest

# Or pull a specific version
docker pull ghcr.io/ai-codespark/opencode:1.0.0

# Run the container
docker run -d \
  --name opencode \
  -p 3000:3000 \
  -v $(pwd)/opencode.jsonc:/app/opencode.jsonc \
  ghcr.io/ai-codespark/opencode:latest
```

## Configuration

Create an `opencode.jsonc` configuration file in your project root or home directory:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "model": "litellm/ollama-kimi-k2",
  "provider": {
    "litellm": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "LiteLLM",
      "options": {
        "baseURL": "https://litellm.example.com",
        "apiKey": "sk-1234"
      },
      "models": {
        "ollama-kimi-k2": {
          "name": "Ollama Kimi K2"
        }
      }
    }
  }
}
```

### Configuration Options

- **model**: The model identifier in the format `litellm/<model-name>`
- **provider.litellm.npm**: The NPM package for OpenAI-compatible SDK
- **provider.litellm.name**: Display name for the provider
- **provider.litellm.options.baseURL**: Your LiteLLM server URL
- **provider.litellm.options.apiKey**: API key for authentication
- **provider.litellm.models**: Dictionary of available models and their display names

### Using with Local Ollama

To use OpenCode with a local Ollama instance via LiteLLM:

1. Start your Ollama server
2. Start LiteLLM proxy pointing to Ollama:
   ```bash
   litellm --model ollama/kimi-k2 --api_base http://localhost:11434
   ```
3. Update your `opencode.jsonc` to point to the LiteLLM proxy:
   ```jsonc
   {
     "model": "litellm/ollama-kimi-k2",
     "provider": {
       "litellm": {
         "options": {
           "baseURL": "http://localhost:4000",
           "apiKey": "sk-local"
         }
       }
     }
   }
   ```

## Usage

### CLI Usage

```bash
# Start OpenCode server
opencode start

# Start with custom config
opencode start --config /path/to/opencode.jsonc

# Check version
opencode --version

# Get help
opencode --help
```

### Docker Usage

#### Using Docker Compose

Create a `docker-compose.yml`:

```yaml
version: '3.8'

services:
  opencode:
    image: ghcr.io/ai-codespark/opencode:latest
    container_name: opencode
    ports:
      - "3000:3000"
    volumes:
      - ./opencode.jsonc:/app/opencode.jsonc
      - ./workspace:/workspace
    environment:
      - OPENCODE_VERSION=latest
      - OPENCODE_CHANNEL=latest
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "opencode", "--version"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 5s
```

Start the service:

```bash
docker-compose up -d
```

#### Building Custom Docker Image

If you want to build your own image:

```bash
# Clone the repository
git clone https://github.com/ai-codespark/opencode.git
cd opencode

# Download the CLI tarball (or build it yourself)
# Place it in the root directory as opencode-linux-x64.tar.gz

# Build the Docker image
docker build -f Dockerfile_litellm \
  --build-arg OPENCODE_VERSION=1.0.0 \
  -t opencode:custom .

# Run your custom image
docker run -d \
  --name opencode-custom \
  -p 3000:3000 \
  -v $(pwd)/opencode.jsonc:/app/opencode.jsonc \
  opencode:custom
```

## Environment Variables

- `OPENCODE_VERSION`: Version of OpenCode (set automatically)
- `OPENCODE_CHANNEL`: Release channel (latest, stable, dev)
- `PATH`: Include OpenCode binary location

## Health Check

The Docker container includes a health check that runs every 30 seconds:

```bash
# Check container health
docker inspect --format='{{.State.Health.Status}}' opencode

# View health check logs
docker inspect --format='{{range .State.Health.Log}}{{.Output}}{{end}}' opencode
```

## Troubleshooting

### Connection Issues

If OpenCode cannot connect to LiteLLM:

1. Verify LiteLLM server is running: `curl http://localhost:4000/health`
2. Check firewall settings
3. Verify API key is correct
4. Check logs: `docker logs opencode` (for Docker) or check console output (for CLI)

### Model Not Available

If the model is not responding:

1. Verify the model is loaded in Ollama: `ollama list`
2. Check LiteLLM supports your model
3. Verify model name in `opencode.jsonc` matches LiteLLM configuration

### Docker Permission Issues

If you encounter permission errors:

```bash
# Run with current user
docker run -d \
  --name opencode \
  --user $(id -u):$(id -g) \
  -p 3000:3000 \
  -v $(pwd)/opencode.jsonc:/app/opencode.jsonc \
  ghcr.io/ai-codespark/opencode:latest
```

## Support

- GitHub Issues: https://github.com/ai-codespark/opencode/issues
- Email: support@ai-codespark.com
- Documentation: https://opencode.ai/docs

## License

See [LICENSE](LICENSE) file for details.
