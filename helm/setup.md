
---

# Helm Installation Guide

This guide outlines the steps to install Helm, the Kubernetes package manager, on a Linux system using the official installation script.

---

## Prerequisites
- A Linux-based system with `curl` installed.
- Root or sudo privileges (if required for installation).
- Access to the internet to download the Helm installation script.

---

## Installation Steps

### 1. Download the Helm Installation Script
Retrieve the Helm installation script from the official Helm repository.

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
```

**Command explanation**:
- `-fsSL`: Options for `curl` to fail silently (`-f`), show errors (`-s`), follow redirects (`-L`), and save the output (`-o`).
- Saves the script as `get_helm.sh`.

### 2. Set Execute Permissions
Make the downloaded script executable.

```bash
chmod 700 get_helm.sh
```

**Command explanation**:
- `chmod 700`: Grants read, write, and execute permissions to the owner only, ensuring the script is secure.

### 3. Run the Installation Script
Execute the script to install Helm.

```bash
./get_helm.sh
```

**Script behavior**:
- The script downloads the latest stable version of Helm, verifies its integrity, and installs it to `/usr/local/bin/helm` (or another system-appropriate location).
- It may prompt for sudo privileges if required.

---

## Verification
After installation, verify that Helm is installed correctly.

```bash
helm version
```

This should display the installed Helm version (e.g., `version.BuildInfo{Version:"v3.x.x", ...}`).

---

## Troubleshooting Tips
- **curl fails**: Ensure internet connectivity and that `curl` is installed (`sudo apt install curl` on Ubuntu/Debian).
- **Permission denied**: Verify the script has execute permissions (`ls -l get_helm.sh`) and run with appropriate privileges.
- **Helm not found**: Confirm the installation path (`/usr/local/bin/helm`) is in your `PATH` (`echo $PATH`) and the script ran successfully.
- **Script errors**: Check for error messages during execution and ensure the system meets Helm’s requirements (e.g., compatible OS).

---

## Notes
- The script installs Helm 3, the latest major version at the time of the script’s release. For specific versions or alternative installation methods, refer to the [official Helm documentation](https://helm.sh/docs/intro/install/).
- Clean up the script after installation if desired:

```bash
rm get_helm.sh
```

---

This documentation provides a clear and complete guide for installing Helm using the provided commands. For advanced Helm usage or configuration, consult the official Helm website.

--- 

