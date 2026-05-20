# Android HTTPS Inspection  
## Disabling SSL Pinning with Objection + Proxy

> Mobile Application Security Laboratory  
> Analyst: Omar Haouani  
> Tools: Objection 1.12.4, Frida 17.9.5  
> Proxy: Burp Suite / mitmproxy

---

# Overview

This laboratory demonstrates how to inspect HTTPS traffic from Android applications by bypassing SSL Pinning using **Objection**.

Objection is a Frida-based mobile instrumentation framework that simplifies SSL pinning bypass and dynamic analysis.

The lab covers:

- Installing and configuring Objection
- Preparing a rooted Android emulator
- Configuring interception proxies
- Disabling SSL pinning automatically
- Intercepting HTTPS traffic
- Troubleshooting common issues

---

# Objectives

- Install and configure Objection & Frida
- Prepare the Android testing environment
- Configure an interception proxy
- Disable SSL pinning with Objection
- Validate HTTPS interception
- Troubleshoot common errors

---

# Prerequisites

- Python 3.8+
- pip
- ADB (Android Platform Tools)
- Rooted Android device or emulator
- Frida & `frida-server` (matching versions)
- Interception proxy:
  - Burp Suite
  - mitmproxy

---

# Technical Environment

| Component | Specification |
|---|---|
| Objection | 1.12.4 |
| Frida Client | 17.9.5 |
| Frida Server | 17.9.5 |
| Android Device | Emulator (`emulator-5554`) |
| Architecture | `x86_64` |
| Proxy | Burp Suite / mitmproxy |

---

# Phase 1 — Installing & Verifying Objection

## Objection Help & Version

### Commands

```bash id="zcn06x"
objection --help
```

```bash id="m1mjlwm"
objection version
```

## Result

Objection `1.12.4` is successfully installed.

---

## Main Available Commands

| Command | Function |
|---|---|
| `start` | Start a session |
| `run` | Execute a single command |
| `patchapk` | Patch APK with `frida-gadget` |
| `signapk` | Sign an APK |
| `api` | Start the API server |

<img width="431" height="481" alt="1" src="https://github.com/user-attachments/assets/d8d6c7e7-a202-423d-845e-e152c45bb03d" />


---

# Phase 2 — Verifying the Frida Environment

---

## Verifying Frida Versions

### Commands

```bash id="l5r7v2"
frida --version
```

```bash id="mwx4if"
python -c "import frida; print(frida.__version__)"
```

## Results

- Frida CLI → `17.9.5`
- Frida Python → `17.9.5`

---

## Verifying CPU Architecture

### Command

```bash id="ndvmtx"
adb shell getprop ro.product.cpu.abi
```

## Result

```text id="8brbkl"
x86_64
```

---

## Verifying Android Processes

### Command

```bash id="mrg5l6"
frida-ps -U
```

## Observation

The Android emulator (`emulator-5554`) is correctly connected and Frida can interact with Android processes.

<img width="390" height="68" alt="2" src="https://github.com/user-attachments/assets/10887a59-8f8d-452c-a785-b3bd36a67761" />


---

# Phase 3 — Proxy Configuration & CA Certificate Installation

---

## Configuring Burp Suite / mitmproxy

1. Launch Burp Suite
2. Go to:
   - Proxy → Proxy Listeners
3. Add listener:
   - `0.0.0.0:8080`
4. On Android emulator:
   - Settings → Wi-Fi → Modify Network
5. Configure:
   - Manual Proxy
   - Host PC IP address
   - Port `8080`

---

## Installing the CA Certificate

1. In Burp Suite:
   - Proxy → Options → Import/Export CA Certificate
2. Export certificate
3. Push certificate to Android:

```bash id="2kl7k8"
adb push cacert.der /sdcard/
```

4. Install certificate:
   - Settings → Security → Install from Storage
5. Name the certificate and validate installation

---

# Phase 4 — Disabling SSL Pinning with Objection

---

## Launching Objection on the Target App

### Command

```bash id="18w5ff"
objection -g com.target.app start
```

---

## SSL Pinning Bypass Command

### Command

```bash id="c7olbd"
android sslpinning disable
```

## What This Command Hooks Automatically

- `SSLContext.init`
- `X509TrustManager`
- `TrustManagerImpl`
- `CertificatePinner` (OkHTTP)

---

## Complete Example

```bash id="9x7vhx"
objection -g com.pwnsec.firestorm start
```

```bash id="n4yujt"
android sslpinning disable
```

---

# Phase 5 — Validating HTTPS Interception

---

## Validation Steps

1. Observe HTTP/HTTPS traffic in Burp or mitmproxy
2. Verify that requests are decrypted
3. Use the application normally:
   - Login
   - Forms
   - API calls

---

## Expected Results

| Verification | Status |
|---|---|
| Proxy intercepts traffic | Success |
| HTTPS traffic decrypted | Success |
| Application works normally | Success |
| No SSL errors | Success |

---

# Troubleshooting (FAQ)

| Problem | Solution |
|---|---|
| Objection does not detect device | Verify `adb devices` & restart `frida-server` |
| `android sslpinning disable` fails | Verify app uses standard Java SSL classes |
| Proxy captures nothing | Verify proxy & CA certificate configuration |
| Frida version mismatch | Align client/server versions |
| Native pinning still active | Use custom Frida native hooks |

---

# Exploitation Workflow

```text id="4sjlww"
Objection + Frida Server
            ↓
android sslpinning disable
            ↓
Automatic Hooks:
- SSLContext.init
- X509TrustManager
- TrustManagerImpl
- CertificatePinner
            ↓
Proxy (Burp / mitmproxy)
            ↓
HTTPS Traffic Successfully Decrypted
```

<img width="312" height="47" alt="3" src="https://github.com/user-attachments/assets/15923578-a201-43d4-88b9-83cf795147b4" />


---

# Quick Checklist

- [x] Objection installed
- [x] Frida & `frida-server` aligned
- [x] Proxy configured correctly
- [x] CA certificate installed
- [x] Objection launched successfully
- [x] `android sslpinning disable` executed
- [x] HTTPS traffic visible inside proxy

---

# Conclusion

This laboratory demonstrated:

1. The simplicity of Objection for SSL pinning bypass
2. The effectiveness of `android sslpinning disable`
3. The importance of combining:
   - Proxy interception
   - Dynamic instrumentation
4. The necessity of understanding underlying SSL mechanisms

Objection provides a powerful and beginner-friendly way to inspect HTTPS traffic in Android applications.

---

# Resources

- Objection GitHub Repository
- Frida Documentation
- Burp Suite
- mitmproxy

---

# Author

**Omar Haouani**  
Mobile Application Security Laboratory  
