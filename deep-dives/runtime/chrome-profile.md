# Chrome Data Analysis
## `/app/data/chrome_data/` — Chromium Profile Persistence

---

## Executive Summary

The `chrome_data/` directory serves as the persistent user data directory for Chromium instances launched by `browser_guard.py`. This profile stores cookies, browsing history, extension state, and cached data across browser sessions. This enables stateful browser automation.

---

## Directory Structure

```
chrome_data/
├── Default/                        # Default Chrome profile
│   ├── Cache/                      # HTTP response cache
│   ├── Code Cache/                 # V8 bytecode cache
│   ├── Cookies                     # SQLite cookies database
│   ├── Cookies-journal             # SQLite journal file
│   ├── DawnWebGPUCache/            # WebGPU shader cache
│   ├── Extension Scripts/          # Extension JavaScript
│   ├── Extension State/            # Extension storage
│   ├── Favicons                    # Favicon database
│   ├── GPUCache/                   # GPU shader cache
│   ├── History                     # Browsing history
│   ├── History-journal             # History journal
│   ├── Local Storage/              # DOM localStorage
│   ├── Login Data                  # Saved passwords
│   ├── Network Action Predictor    # Prefetch data
│   ├── Preferences                 # Chrome preferences JSON
│   ├── Safe Browsing Cookies       # Security cookies
│   ├── Secure Preferences          # Encrypted preferences
│   ├── Session Storage/            # DOM sessionStorage
│   ├── SharedDictionary/           # Compression dictionaries
│   ├── Shortcuts                   # URL shortcuts
│   ├── Site Characteristics Database/  # Site behavior data
│   ├── Top Sites                   # New tab page tiles
│   ├── Trust Tokens                # Privacy tokens
│   ├── Web Data                    # Autofill data
│   ├── WebStorage/                 # Storage quotas
│   └── shared_proto_db/            # Protocol buffer DB
├── first_party_sets.db             # First-party sets
├── GraphiteDawnCache/              # Graphite cache
├── Local State                     # Chrome state JSON
└── Variations                      # Field trial data
```

---

## Data Categories

**Authentication Data**

The `Cookies` file is an SQLite database containing session tokens and login state. This is high sensitivity data. The `Login Data` file is an encrypted SQLite database storing saved passwords. This is also high sensitivity. Chrome's encryption uses the OS keyring on Linux. The `Safe Browsing Cookies` file contains security-related cookies at medium sensitivity.

**Browsing History**

The `History` file is an SQLite database containing URL history and visit counts. This is medium sensitivity. The `Favicons` file stores favicon images. The `Shortcuts` file contains typed URL shortcuts. The `Top Sites` file stores new tab page tiles. These are all low sensitivity.

**Storage Data**

The `Local Storage/` directory holds persistent key-value storage through the localStorage API. This is medium sensitivity. The `Session Storage/` directory contains session-scoped storage. This is low sensitivity. The `WebStorage/` directory tracks storage quotas. The `IndexedDB/` directory holds structured client-side storage at medium sensitivity.

**Extension Data**

The `Extension Scripts/` directory contains extension JavaScript code. The `Extension State/` directory holds extension settings and data. The `../pdf-viewer/` directory contains the PDF.js extension state.

**Cache Data**

The `Cache/` directory stores HTTP response cache data. The `Code Cache/` directory holds V8 compiled bytecode. The `GPUCache/` directory contains GPU shader programs. The `DawnWebGPUCache/` directory holds WebGPU shader cache. The `GraphiteDawnCache/` directory contains Graphite rendering cache.

**Configuration Data**

The `Preferences` file is a JSON file containing user settings, extensions, and themes. The `Secure Preferences` file is an encrypted JSON file with sensitive settings. The `Local State` file is a JSON file containing Chrome state and profile info. The `Variations` file is a binary file with field trial configurations.

---

## Operational Role

**Persistence Across Sessions**

```python
# In browser_guard.py
launch_options = {
    "user_data_dir": "/app/data/chrome_data",
    # ...
}
```

This persistence provides several benefits. Cookies survive browser restarts. Login sessions remain active. Extension settings persist across sessions. Cache improves subsequent page loads. History enables navigation features.

**Launch Sequence**

First, `browser_guard.py` starts. It checks the X11 display. Then it launches Chromium with `--user-data-dir=/app/data/chrome_data`. Chrome reads the existing profile. Session state is restored.

---

## Security Analysis

**Sensitive Data Exposure**

Session cookies present high risk. Profile isolation per user mitigates this. Saved passwords are also high risk. OS-level encryption protects them. Browsing history is medium risk. Regular cleanup reduces exposure. Autofill data is medium risk. User consent controls this. Extension data is low risk. Extension permissions limit exposure.

**Multi-Tenant Considerations**

Shared profiles in multi-user environments expose data across users. The recommendation is creating isolated profiles per session:

```python
import tempfile
import shutil

# Create temp profile
profile_dir = tempfile.mkdtemp(prefix="chrome_profile_")

# Use isolated profile
launch_options["user_data_dir"] = profile_dir

# Cleanup after session
shutil.rmtree(profile_dir)
```

**Encryption Status**

The `Cookies` file is not encrypted. The `Login Data` file is encrypted using the OS keyring such as gnome-keyring or kwallet. The `Secure Preferences` file is encrypted with the same mechanism. The `History` file is not encrypted.

---

## Inter-Module Relationships

```
chrome_data/
    ├── browser_guard.py (reads/writes)
    │   └── Playwright/CDP
    ├── pdf-viewer/ (extension state)
    │   └── PDF.js settings
    └── logs/ (correlated with History)
```

---

## Maintenance

**Cleanup Operations**

Clear the cache with `rm -rf /app/data/chrome_data/Default/Cache/*`. Clear cookies to log out all sessions with `rm /app/data/chrome_data/Default/Cookies`. Clear history with `rm /app/data/chrome_data/Default/History`. For a full reset, use `rm -rf /app/data/chrome_data/*`.

**Size Monitoring**

The `Cache/` directory typically grows to 50 to 500 MB with linear growth based on usage. The `Code Cache/` directory stabilizes at 10 to 100 MB after warmup. The `History` file grows slowly to 1 to 10 MB. The `Cookies` file grows slowly to 100 KB to 1 MB.

---

## Troubleshooting

**Chrome won't start**: This is typically caused by a corrupted profile. The solution is to delete and recreate the profile.

**Extension not loading**: This indicates corrupt extension state. Clear the Extension State directory.

**Login lost**: This is expected behavior after clearing cookies.

**Slow startup**: This is usually caused by a large cache. Clear the Cache directory.

---

*Document Version: 1.0*
*Analysis Date: 2026-02-02*
