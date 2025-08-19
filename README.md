# Improvised multistage Docker image for the Open Social distribution

- [Open Social Distribution Github repo](https://github.com/goalgorilla/open_social)
- [Open Social Distribution issue queue](https://www.drupal.org/project/issues/social).
- [Open Social development repo] (https://github.com/goalgorilla/drupal_social)
- [Docker Hub](https://hub.docker.com/r/goalgorilla/open_social_docker/)
- [Composer template](https://github.com/goalgorilla/social_template)

## 🚀 Optimized Multi-Stage Architecture

This repository now features an **optimized multi-stage Dockerfile** that provides significant improvements in image size, security, and build efficiency.

### Architecture Overview

The Dockerfile uses a 5-stage architecture:

```
Base Stage (800MB)
├── Essential runtime packages
├── Core PHP extensions
└── Composer installation

Dependencies Stage
├── Cached Composer dependencies
└── Optimized autoloader

Production Stage (1.59GB) ← Default
├── Minimal runtime only
├── No development tools
└── Optimized for security

Development Stage (4.8GB)
├── Full development environment
├── Xdebug configuration
├── Development tools
└── All development dependencies

CI Stage (4.2GB)
├── Testing-focused environment
├── Testing tools (behat, phpunit)
├── No Xdebug
└── All testing dependencies
```

### Size Improvements

| Environment | Original | Optimized | Reduction |
|-------------|----------|-----------|-----------|
| **Production** | 6.08GB | 1.59GB | **74%** |
| **Development** | 6.08GB | 4.8GB | **20%** |
| **CI** | ~6GB | 4.2GB | **30%** |

## 🏷️ Available Tags

- `latest`: Production image (minimal, secure)
- `production`: Production image (minimal, secure)
- `development`: Development image with Xdebug and dev tools
- `ci`: CI image with testing tools
- `cron`: Cron image for scheduled tasks

## 🛠️ Building Images

### Prerequisites

1. **Enable BuildKit** (for multi-platform builds):
```bash
export DOCKER_BUILDKIT=1
```

2. **Create buildx builder** (for multi-platform builds):
```bash
docker buildx create --use
```

### Basic Build Commands

#### Production Image (Default)
```bash
# Single platform
docker build -f drupal10/php8.3-v2/Dockerfile -t open-social:prod drupal10/php8.3-v2/

# Multi-platform
docker buildx build -f drupal10/php8.3-v2/Dockerfile --platform linux/amd64,linux/arm64 -t open-social:prod drupal10/php8.3-v2/
```

#### Development Image
```bash
# Single platform
docker build -f drupal10/php8.3-v2/Dockerfile --target development -t open-social:dev drupal10/php8.3-v2/

# Multi-platform
docker buildx build -f drupal10/php8.3-v2/Dockerfile --target development --platform linux/amd64,linux/arm64 -t open-social:dev drupal10/php8.3-v2/
```

#### CI Image
```bash
# Single platform
docker build -f drupal10/php8.3-v2/Dockerfile --target ci -t open-social:ci drupal10/php8.3-v2/

# Multi-platform
docker buildx build -f drupal10/php8.3-v2/Dockerfile --target ci --platform linux/amd64,linux/arm64 -t open-social:ci drupal10/php8.3-v2/
```

### Using the Build Script

Navigate to the specific version directory and use the provided build script:

```bash
cd drupal10/php8.3-v2/
chmod +x build.sh

# Build for current platform
./build.sh dev
./build.sh prod
./build.sh ci

# Build for multiple platforms
./build.sh multi-dev
./build.sh multi-prod
./build.sh multi-ci

# Build and push to registry
./build.sh push-dev
./build.sh push-prod
./build.sh push-ci

# Build all environments for all platforms
./build.sh all
```

## 🚀 Pushing Images

### Manual Push

#### Single Platform
```bash
# Tag the image
docker tag open-social:prod organization:latest

# Push to registry
docker push organization:latest
```

#### Multi-Platform Push
```bash
# Build and push in one command
docker buildx build -f drupal10/php8.3-v2/Dockerfile \
  --platform linux/amd64,linux/arm64 \
  --push \
  -t organization:latest \
  drupal10/php8.3-v2/
```

### Version-Specific Tags

```bash
# Production
docker buildx build -f drupal10/php8.3-v2/Dockerfile \
  --platform linux/amd64,linux/arm64 \
  --push \
  -t organization:10.3.11-php8.3-prod \
  drupal10/php8.3-v2/

# Development
docker buildx build -f drupal10/php8.3-v2/Dockerfile \
  --target development \
  --platform linux/amd64,linux/arm64 \
  --push \
  -t organization:10.3.11-php8.3-dev \
  drupal10/php8.3-v2/

# CI
docker buildx build -f drupal10/php8.3-v2/Dockerfile \
  --target ci \
  --platform linux/amd64,linux/arm64 \
  --push \
  -t organization:10.3.11-php8.3-ci \
  drupal10/php8.3-v2/
```

## 🔧 Environment-Specific Configurations

### Production Environment
- **Size**: ~1.59GB
- **Features**: Minimal runtime, optimized for performance
- **Security**: No development tools, minimal attack surface
- **Use Case**: Production deployments, staging environments

### Development Environment
- **Size**: ~4.8GB
- **Features**: Full development tools, Xdebug, all dependencies
- **Tools**: git, curl, wget, nano, vim, 7zip
- **Use Case**: Local development, debugging

### CI Environment
- **Size**: ~4.2GB
- **Features**: Testing tools, no Xdebug
- **Tools**: behat, phpunit, git, curl, wget
- **Use Case**: Continuous Integration, automated testing

## 🏗️ Architecture Benefits

### Security
- **Production isolation**: No development tools in production
- **Minimal attack surface**: Only essential components
- **Environment separation**: Clear boundaries between environments

### Performance
- **Faster deployments**: 74% smaller production images
- **Reduced storage costs**: Smaller images in registries
- **Better caching**: Optimized layer caching

### Maintainability
- **Modular architecture**: Easy to modify specific components
- **Clear separation**: Runtime vs build-time dependencies
- **Consistent builds**: Reproducible across platforms

## 📋 Platform Support

### Supported Architectures
- `linux/amd64`: Intel/AMD processors
- `linux/arm64`: Apple Silicon (M1/M2), ARM processors

### Platform-Specific Optimizations
- Native performance on target platforms
- Optimized base images for each architecture
- Consistent behavior across different systems

## 🔍 Troubleshooting

### Build Issues
```bash
# Check buildx builder status
docker buildx ls

# Inspect builder
docker buildx inspect

# Create new builder if needed
docker buildx create --name mybuilder --use
```

### Platform-Specific Issues
```bash
# Build for specific platform only
docker buildx build --platform linux/amd64 -t open-social:amd64 .

# Check image architecture
docker inspect open-social:latest | grep Architecture
```

### Performance Optimization
```bash
# Use build cache
docker buildx build --cache-from type=registry,ref=goalgorilla/open_social_docker:cache .

# Parallel builds
docker buildx build --parallel .
```

## 📚 Additional Resources

- [Docker Multi-Stage Builds](https://docs.docker.com/develop/dev-best-practices/multistage-build/)
- [Docker Buildx](https://docs.docker.com/engine/reference/commandline/buildx/)
- [Open Social Documentation](https://www.drupal.org/project/social)
- [Docker Hub Repository](https://hub.docker.com/r/goalgorilla/open_social_docker/)

---

**Note**: This repository now uses optimized multi-stage builds for improved performance, security, and maintainability. The original single-stage Dockerfiles are preserved for reference but are no longer the primary build method.
