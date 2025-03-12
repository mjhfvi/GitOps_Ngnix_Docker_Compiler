# NGINX Windows Build Project

## Overview

This project provides scripts and configurations to build a fully working 64-bit NGINX for Windows environments. It uses Docker to create a reproducible build environment and outputs a Windows-compatible NGINX executable that runs on Windows Nano Server.

The main advantages of this approach:

- **Fully isolated build environment** - All required tools and dependencies are installed within the container, requiring no additional installations on the host machine
- **Optimized image size** - Final image uses Windows Nano Server (~200MB) instead of Windows Server Core (~6GB)
- **Complete 64-bit support** - Builds native 64-bit NGINX executable for modern Windows environments

## Features

- Automated NGINX compilation for Windows environments
- Docker-based build process for reproducibility
- Support for custom NGINX modules (SSL, etc.)
- Uses Visual Studio 2017 Build Tools for compilation
- Compatible with Windows Server Core for building and Nano Server for runtime
- No host machine dependencies required beyond Docker

## Prerequisites

To use this project, you'll need:

- Docker Desktop for Windows
- Git

## Quick Start

1. Clone this repository:

   ```
   git clone https://github.com/yourusername/nginx-windows-build.git
   cd nginx-windows-build
   ```

2. Build the Docker image:

   ```
   docker build -t nginx-windows-builder .
   ```

3. Run the container to get the compiled NGINX executable:
   ```
   docker run -d --name nginx-windows nginx-windows-builder
   docker cp nginx-windows:C:/nginx/nginx.exe ./
   ```

## Customization

### NGINX Version

You can specify a different NGINX version during build:

```
docker build --build-arg NGINX_BRANCH_VERSION=1.25 -t nginx-windows-builder .
```

### Build Configuration

The build configuration is defined in `compile-nginx.sh`. You can modify this file to:

- Enable/disable specific NGINX modules
- Change paths and configuration options
- Adjust compiler flags

## Project Structure

- `dockerfile` - Docker build script for creating the build environment
- `compile-nginx.sh` - The main build script that configures and initiates NGINX compilation
- `makefile.msvc` - Modified makefile for building OpenSSL with MSVC
- `msvc/` - Contains modified MSVC compiler configuration files

## Build Process

The build process:

1. Sets up a Windows Server Core container as the build environment
2. Installs build dependencies (Visual Studio 2017 Build Tools, Perl, NASM, etc.)
3. Downloads and extracts required libraries (PCRE2, zlib, OpenSSL)
4. Configures the build environment for MSVC
5. Compiles NGINX with the specified modules
6. Creates a minimal Windows Nano Server image (~200MB) with the compiled binary

## Technical Details

- Uses Visual Studio 2017 Build Tools for compilation
- Fully 64-bit build process and output executable
- Two-stage Docker build:
  - First stage: Windows Server Core for compilation
  - Second stage: Windows Nano Server for the final image
- All build tools and dependencies are contained within the Docker container
- No need to install any frameworks or SDKs on the host machine
- default windows docker image 'mcr.microsoft.com/windows/servercore:ltsc2019'

## Troubleshooting

### Common Issues

- **Build fails with OpenSSL errors**: Make sure you have the correct NASM version installed.
- **Compilation errors with MSVC**: Verify Visual Studio Build Tools installation in the container.
- **Missing DLLs**: The Nano Server image may need additional DLLs for certain modules.

## Sources

This project was created with information and inspiration from the following sources:

- [Official NGINX documentation on building for Windows](https://nginx.org/en/docs/howto_build_on_win32.html)
- [AMEFS tutorial on building NGINX for Windows](https://amefs.net/en/archives/1935.html?amp)
- [YouTube tutorial on NGINX Windows builds](https://www.youtube.com/watch?v=M-cj-p4rZtU)

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Acknowledgements

- [NGINX Project](https://nginx.org/)
- [OpenSSL Project](https://www.openssl.org/)
- [PCRE2 Project](https://github.com/PCRE2Project/pcre2)
