# 128T / Router EICAR AV Scheduled Testing Guide

## Overview

This guide explains how to install and use the **Svoto EICAR Outbound AV Test** as a **scheduled job** on a Linux host sitting behind your 128T router (or any router/firewall).

The tool:

- Installs a Python script that sends **EICAR test signatures** through your router to external test servers.
- Configures a **systemd service + timer** so the test:
  - Runs **once immediately** after install.
  - Then runs **automatically every 12 hours** with no user interaction.

Your router / firewall **should detect, log, and (ideally) block** these transmissions.

>  **Important:** Only use this on networks you own or are explicitly authorized to test.

---

## Components

### 1. `install_eicar_av_test.sh` – One-Shot Installer

- Creates the install directory:  
  `/usr/local/eicar-av-test/`
- Writes the main Python test script:  
  `/usr/local/eicar-av-test/eicar_router_test.py`
- Creates systemd units:

  - `/etc/systemd/system/eicar-av-test.service`
  - `/etc/systemd/system/eicar-av-test.timer`

- Reloads systemd and enables the timer:

  ```bash
  systemctl daemon-reload
  systemctl enable --now eicar-av-test.timer
  systemctl start eicar-av-test.service



# 128T Router EICAR Antivirus Testing Guide

## Overview
This guide helps you test the antivirus capabilities of your 128T router by sending EICAR test virus signatures through the router to external test servers. The router should detect and block these transmissions.

## Test Scripts

### 1. **eicar_128t_test.py** - Quick 128T Test
- Focused test for 128T routers
- Tests HTTP, HTTPS, and encoded transmissions
- Uses httpbin.org and postman-echo.com
- Takes about 1 minute

### 2. **eicar_outbound_test.py** - Comprehensive Outbound Test
- Tests 8 different transmission methods
- Includes socket-level testing
- DNS tunneling simulation
- WebSocket simulation
- Takes 3-5 minutes

### 3. **eicar_webhook_test.py** - Webhook Service Test
- Uses webhook testing services
- Tests multiple encoding methods
- Simulates cloud uploads
- Most detailed results

## Quick Start

```bash
# Run the quick 128T-specific test
python3 eicar_128t_test.py

# Or run the comprehensive test
python3 eicar_outbound_test.py
```

## How It Works

The scripts send the EICAR test string through your network:
```
X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*
```

This string is:
- **Completely harmless** - contains no actual malware
- **Industry standard** - recognized by all AV systems
- **Designed for testing** - specifically for validating AV detection

## What Gets Tested

1. **HTTP POST** - Plain text EICAR in HTTP body
2. **HTTPS POST** - Encrypted transmission (tests SSL inspection)
3. **File Upload** - Multipart form upload with EICAR
4. **Base64 Encoding** - Tests deep packet inspection
5. **Large Payloads** - EICAR embedded in larger data
6. **Various MIME Types** - Different content types
7. **Multiple HTTP Methods** - POST, PUT, PATCH
8. **Direct Socket** - Raw TCP transmission

## Expected Behavior

###  When AV is Working:
- Connections timeout or are reset
- HTTP requests return error codes
- Python shows connection errors
- Tests show "✓ Blocked" messages

###  When AV is NOT Working:
- HTTP requests return 200 OK
- Data successfully transmits
- Tests show "⚠ WARNING" messages
- No errors in transmission

## Checking 128T Router Logs

### 1. Via Conductor GUI
```
Navigate to:
- Monitor > Events > IDPS Events
- Look for "EICAR" or "Test" entries
- Check timestamp matches your test
```

### 2. Via CLI on Router
```bash
# SSH to your 128T router
ssh admin@<router-ip>

# Check antivirus service status
systemctl status 128t-anti-virus

# View real-time logs
sudo journalctl -u 128t-anti-virus -f

# Check log files
sudo ls -la /var/log/128t-anti-virus/
sudo tail -f /var/log/128t-anti-virus/anti-virus.log
```

### 3. Via PCLI
```bash
# Enter PCLI
pcli

# Show IDPS events
show events type idps

# Show security events
show events category security

# Check signature version
show idps signatures
```

## Configuration Checklist

Ensure your 128T router has:

### Service Policy Configuration
```
service-policy <policy-name>
    name <policy-name>
    service <service-name>
    security idps-inline
        mode inline  # Not just 'monitor'
    exit
exit
```

### Security Settings
```
authority
    router <router-name>
        node <node-name>
            device-interface <interface-name>
                network-interface <network-name>
                    source-nat true  # If needed
                exit
            exit
        exit
    exit
exit
```

### IDP/IPS Configuration
```
authority
    idps-profile <profile-name>
        name <profile-name>
        provider fortinet  # or your provider
        mode inline
        ssl-decryption enabled  # For HTTPS inspection
    exit
exit
```

## Troubleshooting

### Issue: All tests pass through (not blocked)

1. **Check AV Service**
   ```bash
   systemctl status 128t-anti-virus
   systemctl restart 128t-anti-virus
   ```

2. **Update Signatures**
   ```bash
   # From PCLI
   request idps update-signatures
   show idps signatures
   ```

3. **Verify Mode**
   - Ensure mode is "inline" not "monitor"
   - Monitor mode only logs, doesn't block

4. **Check Session Type**
   - Session must allow payload inspection
   - Check service-class settings

5. **SSL Inspection**
   - Enable for HTTPS detection
   - Configure certificate authority

### Issue: Partial blocking

- **HTTP blocked but HTTPS passes**: Enable SSL decryption
- **Small payloads blocked, large pass**: Increase inspection buffer
- **Some encodings pass**: Enable deep packet inspection

### Issue: No logs generated

1. Check logging level:
   ```bash
   # From PCLI
   configure authority router <name> system log-level idps debug
   ```

2. Verify log categories are enabled:
   ```bash
   show events categories
   ```

## Best Practices

1. **Test Regularly**: Run tests after any configuration change
2. **Check Logs**: Always verify detection in logs, not just blocking
3. **Update Signatures**: Keep AV signatures current
4. **Test All Protocols**: HTTP, HTTPS, FTP if used
5. **Document Results**: Keep records of test results

## Understanding Results

### Excellent Protection (90-100% blocked)
- Router AV properly configured
- All major threat vectors covered
- Continue regular testing

### Good Protection (70-89% blocked)
- Most threats blocked
- Review failed tests
- Fine-tune configuration

### Needs Improvement (<70% blocked)
- Significant gaps in protection
- Review configuration thoroughly
- Consider professional assistance

## Support Resources

- **128T Documentation**: https://docs.128technology.com/
- **AV Configuration**: https://docs.128technology.com/docs/sec-config-antivirus/
- **IDPS Guide**: https://docs.128technology.com/docs/idps/
- **Community Forum**: https://community.128technology.com/

## Safety Reminder

 **Only test on networks you own or have permission to test**

These tests will:
- Generate security alerts
- Create log entries
- May notify security teams
- Could impact performance during testing

## Next Steps

After testing:
1. Review all blocked/allowed transmissions
2. Check router logs for detection details
3. Adjust configuration as needed
4. Document your security posture
5. Schedule regular retesting

---

**Remember**: The EICAR test string is harmless but designed to trigger AV systems. This is the industry-standard way to verify your antivirus is working correctly.

*   Result:
    
    *   **One immediate test run**
        
    *   **Automatic runs every 12 hours**
        

### 2\. `eicar_router_test.py` – Scheduled Outbound Test Script

*   Sends EICAR test traffic using multiple methods:
    
    *   HTTP, HTTPS, file upload, encoded payloads
        
    *   Raw socket, PUT, DNS-like patterns, WebSocket-like payloads
        
*   Prints a **summary report**:
    
    *   Which paths were **BLOCKED vs ALLOWED**
        
    *   Overall detection score
        
    *   Recommendations for improving coverage
        
*   Supports:
    
    *   `--no-prompt` (for non-interactive runs from systemd)
        
    *   `--interval` and `--runs` (optional self-loop mode)
        

* * *

## Quick Start

### 1\. Clone the Repository

`git clone https://github.com/<your-username>/<your-repo>.git cd <your-repo>`

### 2\. Make the Installer Executable

`chmod +x install_eicar_av_test.sh`

### 3\. Run the Installer (One Time, as Root)

`sudo ./install_eicar_av_test.sh`

The installer will:

*   Create:
    
    `/usr/local/eicar-av-test/`
    
*   Install the script:
    
    `/usr/local/eicar-av-test/eicar_router_test.py`
    
*   Create systemd units:
    
    `/etc/systemd/system/eicar-av-test.service /etc/systemd/system/eicar-av-test.timer`
    
*   Run:
    
    `systemctl daemon-reload systemctl enable --now eicar-av-test.timer systemctl start eicar-av-test.service`
    

**Result:**

*   **One immediate test run** (via `eicar-av-test.service`)
    
*   **Automatic tests every 12 hours** (via `eicar-av-test.timer`)
    

* * *

## How It Works

The script sends the standard **EICAR antivirus test string** through your router:

`X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*`

This string is:

*   ✅ **Harmless** – no real malware
    
*   ✅ **Industry standard** – recognized by most AV engines
    
*   ✅ **Designed for testing** – purpose-built to trigger AV detection
    

The traffic is sent to safe testing endpoints (e.g. `httpbin.org`, `postman-echo.com`).  
Your **router/firewall AV / IDPS** should:

*   Detect the EICAR signature
    
*   Block or disrupt the connection (if configured inline)
    
*   Generate logs / alerts that you can review
    

* * *

## What Gets Tested (Per Run)

Each run of `eicar_router_test.py` tests multiple outbound vectors:

1.  **HTTP POST**
    
    *   EICAR in plain HTTP form-data to public test endpoints.
        
2.  **HTTP Multipart Upload**
    
    *   EICAR as a pseudo “uploaded file” using `multipart/form-data`.
        
3.  **HTTPS POST**
    
    *   EICAR in JSON over TLS, simulating encrypted API uploads.
        
    *   Tests whether **SSL inspection / decryption** is enabled and working.
        
4.  **Encoded Payloads**
    
    *   EICAR sent in different encodings:
        
        *   Base64
            
        *   Hex
            
        *   URL-encoded
            
        *   Double URL-encoded
            
    *   Tests **deep packet inspection** for encoded content.
        
5.  **Raw Socket Transmission**
    
    *   Manually crafted HTTP POST via TCP socket with EICAR in the body.
        
    *   Exercises lower-level inspection and generic TCP rules.
        
6.  **PUT / FTP-like Simulation**
    
    *   EICAR sent via HTTP PUT.
        
    *   Mimics file transfer APIs or FTP-over-HTTP behavior.
        
7.  **DNS-like Tunneling Simulation**
    
    *   EICAR hex data split into DNS label-sized chunks.
        
    *   Attempts DNS-style lookups with EICAR-encoded names.
        
    *   Tests detection of DNS-tunneling-like patterns.
        
8.  **WebSocket-like Simulation**
    
    *   EICAR-like payload inside a JSON message, sent with WebSocket-ish upgrade headers.
        
    *   Tests application-layer inspection beyond basic HTTP.
        

* * *

## How Often Does It Run?

The installer creates this timer:

`# /etc/systemd/system/eicar-av-test.timer [Unit] Description=Run EICAR AV test every 12 hours (Svoto lab tool)  [Timer] OnBootSec=5min OnUnitActiveSec=12h Unit=eicar-av-test.service Persistent=true  [Install] WantedBy=timers.target`

Meaning:

*   **First automatic run**: ~5 minutes after boot (`OnBootSec=5min`)
    
*   **Subsequent runs**: every **12 hours after the last test completes** (`OnUnitActiveSec=12h`)
    

### Check Next / Last Run

`systemctl list-timers eicar-av-test.timer`

Example:

`NEXT                        LEFT          LAST                        PASSED       UNIT                 ACTIVATES Mon 2025-12-08 04:00:00     11h left      Sun 2025-12-07 16:00:10     1h ago      eicar-av-test.timer  eicar-av-test.service`

*   **LAST** → last test execution time
    
*   **NEXT** → next scheduled run
    

* * *

## Viewing Test Results (Linux Host)

All script output is captured in the **systemd journal**.

### Show Latest Run Output

`sudo journalctl -u eicar-av-test.service -n 100 --no-pager`

### Follow Logs Live

`sudo journalctl -u eicar-av-test.service -f`

You’ll see:

*   Per-test statuses, e.g.:
    
    `HTTP POST                    : BLOCKED Multipart Upload             : BLOCKED HTTPS                        : ALLOWED Encoded                      : ALLOWED ...`
    
*   Overall score:
    
    `Overall Score: 5/8 transmissions blocked`
    
*   Recommendations:
    
    *   Enable / tune HTTP content inspection
        
    *   Enable SSL inspection for HTTPS
        
    *   Ensure DPI is applied to encoded traffic
        
    *   Review raw-socket / DNS inspection
        

* * *

## Manual Script Usage (Ad-Hoc Tests)

Even with systemd scheduling, you may want to run the test manually.

### Script Location

Installed here:

`/usr/local/eicar-av-test/eicar_router_test.py`

### Run Once, Interactive

`python3 /usr/local/eicar-av-test/eicar_router_test.py`

You’ll see a prompt:

`Press Enter to begin outbound EICAR transmission tests...`

### Run Once, Non-Interactive

`python3 /usr/local/eicar-av-test/eicar_router_test.py --no-prompt`

* * *

## Self-Looping Mode (Without Systemd)

If you prefer the script to manage its own schedule (e.g. in a lab container), it supports an internal loop.

### Example: Every 10 Minutes, Forever

`python3 /usr/local/eicar-av-test/eicar_router_test.py --interval 600 --no-prompt`

### Example: Every Hour, 5 Runs, Then Exit

`python3 /usr/local/eicar-av-test/eicar_router_test.py --interval 01:00:00 --runs 5 --no-prompt`

`--interval` accepts:

*   Seconds (e.g. `600`), or
    
*   `HH:MM:SS` format (e.g. `01:00:00`)
    

* * *

## Customizing the Systemd Interval

To change the **12-hour** default:

1.  Edit the timer:
    
    `sudo nano /etc/systemd/system/eicar-av-test.timer`
    
2.  Update this line:
    
    `OnUnitActiveSec=12h`
    
    Examples:
    
    *   Every 6 hours: `OnUnitActiveSec=6h`
        
    *   Every 1 hour: `OnUnitActiveSec=1h`
        
    *   Every 30 minutes: `OnUnitActiveSec=30min`
        
3.  Reload and restart the timer:
    
    `sudo systemctl daemon-reload sudo systemctl restart eicar-av-test.timer`
    
4.  Confirm:
    
    `systemctl list-timers eicar-av-test.timer`
    

* * *

## Managing the Service & Timer

### Check Service Status

`systemctl status eicar-av-test.service`

### Check Timer Status

`systemctl status eicar-av-test.timer`

### Manually Trigger a Test Run

`sudo systemctl start eicar-av-test.service`

### Stop Automatic Runs (Keep Files)

`sudo systemctl disable --now eicar-av-test.timer`

### Re-enable Automatic Runs

`sudo systemctl enable --now eicar-av-test.timer`

* * *

## Checking 128T Router / Security Logs

Once the Linux host sends EICAR traffic, verify that your **128T router** (or other security device) detects it.

### 1\. Via 128T Conductor GUI

*   Navigate to:
    
    *   **Monitor → Events → IDPS Events**
        
    *   Filter for “`EICAR`” or “`Test`”
        
    *   Match timestamps with the test run time from the Linux host
        

### 2\. Via 128T Router CLI

`# SSH to your 128T router ssh admin@<router-ip>  # Check antivirus service status systemctl status 128t-anti-virus  # View real-time antivirus logs sudo journalctl -u 128t-anti-virus -f  # Check log files directly sudo ls -la /var/log/128t-anti-virus/ sudo tail -f /var/log/128t-anti-virus/anti-virus.log`

### 3\. Via 128T PCLI

`# Enter PCLI pcli  # Show IDPS events show events type idps  # Show security events show events category security  # Check signature version show idps signatures`

* * *

## Expected Behavior

### ✅ When AV / IDPS is Working Properly

*   Some / all test transmissions:
    
    *   Time out
        
    *   Receive non-200 HTTP status codes
        
    *   Trigger connection errors in the script
        
*   Script output shows many **`BLOCKED`** results.
    
*   128T / firewall logs contain EICAR-related events.
    

### ⚠️ When AV / IDPS is NOT Working Well

*   Many transmissions succeed with `200 OK`.
    
*   Script shows **`ALLOWED`** on key tests (HTTP / HTTPS / Encoded).
    
*   Logs show little or no detection activity.
    

* * *

## Uninstall / Cleanup

To fully remove the tool and schedule:

1.  Stop and disable the timer:
    
    `sudo systemctl disable --now eicar-av-test.timer`
    
2.  Remove systemd units:
    
    `sudo rm -f /etc/systemd/system/eicar-av-test.service sudo rm -f /etc/systemd/system/eicar-av-test.timer`
    
3.  Reload systemd:
    
    `sudo systemctl daemon-reload`
    
4.  Remove install directory:
    
    `sudo rm -rf /usr/local/eicar-av-test`
    

* * *

## Best Practices

1.  **Test regularly** – especially after config changes or upgrades.
    
2.  **Review logs**, not just whether the script says “BLOCKED”.
    
3.  **Keep AV signatures up to date** on your router / firewall.
    
4.  **Test HTTP and HTTPS**, plus any other critical protocols in your environment.
    
5.  **Document results** for audits and internal security reviews.
    

* * *

## Credits & References

*   Created and maintained by **Steve Voto (Svoto)**.
    
*   Uses the industry-standard **EICAR antivirus test string**.
    

More on EICAR:  
[https://www.eicar.org/download-anti-malware-testfile/](https://www.eicar.org/download-anti-malware-testfile/)

> ⚠️ **Reminder:** Only run these tests on networks where you have proper authorization.  
> They will generate security events and logs and may notify security teams.

##### You said:

I need the whole thing in github markdown as one file not different text dfiles

###### ChatGPT said:

Thought for 10s

``# Svoto EICAR Router AV Scheduled Testing Guide  ## Overview  This project installs a **scheduled EICAR outbound antivirus test** on a Linux system that sits behind your **128T router** or any router/firewall. It uses the standard **EICAR test string** to verify:  - Your router/firewall **antivirus (AV)** is inspecting outbound traffic   - **IDPS / security logging** is capturing and recording events   - Encrypted or encoded paths (HTTPS, Base64, DNS-like, WebSocket-like) are being inspected (if configured)  The setup uses:  - A one-shot installer: `install_eicar_av_test.sh` - A Python test script: `eicar_router_test.py` - A **systemd service + timer** to run the test:   - Once **immediately** after install   - Then **automatically every 12 hours** (by default)  > ⚠️ **Use only on networks you own or are explicitly authorized to test.**  ---  ## Safety & Legal Disclaimer  This tool uses the **EICAR standard antivirus test string**, not real malware. The EICAR string is designed to be:  - ✅ Harmless   - ✅ Recognized by most AV engines   - ✅ Intended specifically for testing AV detection    However:  > This tool is provided **“AS IS”**, without warranty of any kind, express or implied.   > Use at your own risk.   > The author (**Steve Voto / Svoto**) assumes **no responsibility** for: > - Outages or downtime   > - Data loss or corruption   > - Security incidents or misconfigurations   > - Policy, compliance, or legal violations   > resulting from use or misuse of this tool.  Do **not** run this on networks where such testing is prohibited by policy, contract, or law.  ---  ## Components  ### 1. `install_eicar_av_test.sh` – One-Shot Installer  This script:  - Creates the install directory:    ```text   /usr/local/eicar-av-test/``

*   Writes the main Python script:
    
    `/usr/local/eicar-av-test/eicar_router_test.py`
    
*   Creates the systemd unit files:
    
    `/etc/systemd/system/eicar-av-test.service /etc/systemd/system/eicar-av-test.timer`
    
*   Reloads systemd and enables the timer:
    
    `systemctl daemon-reload systemctl enable --now eicar-av-test.timer systemctl start eicar-av-test.service`
    

**Result:**

*   A **single immediate test run** via `eicar-av-test.service`
    
*   Recurring tests **every 12 hours** via `eicar-av-test.timer`
    

### 2\. `eicar_router_test.py` – Outbound Test Script

This script:

*   Sends the EICAR string using multiple methods:
    
    *   HTTP POST
        
    *   HTTP multipart upload
        
    *   HTTPS JSON POST
        
    *   Encoded payloads (Base64, Hex, URL-encoded, double-encoded)
        
    *   Raw socket HTTP
        
    *   PUT (FTP-like simulation)
        
    *   DNS-like tunneling patterns
        
    *   WebSocket-like JSON messages
        
*   Prints a **detailed summary** of:
    
    *   Which tests were **BLOCKED** vs **ALLOWED**
        
    *   Overall detection score
        
    *   Recommendations for AV / IDPS configuration
        
*   Supports:
    
    *   `--no-prompt` for non-interactive runs (used by systemd)
        
    *   `--interval` and `--runs` for self-looping test mode (optional)
        

* * *

## Installation & Setup

### 1\. Clone the Repository

`git clone https://github.com/<your-username>/<your-repo>.git cd <your-repo>`

You should see at least:

*   `install_eicar_av_test.sh`
    

### 2\. Make the Installer Executable

`chmod +x install_eicar_av_test.sh`

### 3\. Run the Installer (One Time, as Root)

`sudo ./install_eicar_av_test.sh`

The installer will:

1.  Create:
    
    `/usr/local/eicar-av-test/`
    
2.  Install the Python test script:
    
    `/usr/local/eicar-av-test/eicar_router_test.py`
    
3.  Create systemd unit files:
    
    `/etc/systemd/system/eicar-av-test.service /etc/systemd/system/eicar-av-test.timer`
    
4.  Run:
    
    `systemctl daemon-reload systemctl enable --now eicar-av-test.timer systemctl start eicar-av-test.service`
    

**Outcome:**

*   One **immediate test run** happens right after install.
    
*   Future tests run **automatically every 12 hours**.
    

* * *

## EICAR Test String

The script uses the standard EICAR test string:

`X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*`

This is:

*   **Harmless** – no actual malicious code
    
*   **Standardized** – recognized test signature
    
*   **Designed for AV validation** – the de facto way to test AV engines
    

The script sends this string to safe testing endpoints (e.g., `httpbin.org`, `postman-echo.com`). Your router/firewall AV/IDPS should detect, block, and/or log this activity.

* * *

## What Gets Tested (Per Run)

Each scheduled run of `eicar_router_test.py` attempts multiple outbound EICAR patterns:

1.  **HTTP POST**
    
    *   EICAR in plain HTTP form-data to public HTTP endpoints.
        
    *   Validates basic HTTP content inspection.
        
2.  **HTTP Multipart Upload**
    
    *   EICAR as an uploaded file (`multipart/form-data`).
        
    *   Simulates web file uploads.
        
3.  **HTTPS POST**
    
    *   EICAR in JSON over TLS.
        
    *   Tests whether **SSL inspection / TLS decryption** is functioning.
        
4.  **Encoded Payloads**
    
    *   Sends EICAR in:
        
        *   Base64
            
        *   Hex
            
        *   URL-encoded
            
        *   Double URL-encoded
            
    *   Tests **deep packet inspection (DPI)** capabilities.
        
5.  **Raw Socket HTTP**
    
    *   Crafts HTTP POST requests directly over TCP sockets with EICAR in the body.
        
    *   Tests detection even when traffic is not generated by a high-level HTTP client library.
        
6.  **PUT / FTP-like Simulation**
    
    *   Sends EICAR via HTTP PUT.
        
    *   Mimics file transfer style uploads or API-based file transfers.
        
7.  **DNS-like Tunneling Simulation**
    
    *   Encodes EICAR in hex.
        
    *   Splits data into DNS label-sized chunks.
        
    *   Attempts DNS-style lookups with encoded labels.
        
    *   Tests detection of suspicious DNS patterns / tunneling.
        
8.  **WebSocket-like Simulation**
    
    *   Sends JSON containing EICAR-like payloads with WebSocket-style upgrade headers.
        
    *   Tests application-level inspection beyond simple HTTP.
        

**Goal:** Reveal which outbound paths your AV/IDPS is inspecting and which might be slipping through due to missing SSL decryption, incomplete DPI, or weak DNS/Socket inspection.

* * *

## How Often Does It Run?

The installer creates this systemd timer:

`# /etc/systemd/system/eicar-av-test.timer [Unit] Description=Run EICAR AV test every 12 hours (Svoto lab tool)  [Timer] OnBootSec=5min OnUnitActiveSec=12h Unit=eicar-av-test.service Persistent=true  [Install] WantedBy=timers.target`

This means:

*   **First automatic run**  
    Occurs about **5 minutes after boot** (`OnBootSec=5min`).
    
*   **Subsequent runs**  
    Occur **12 hours after the last run completes** (`OnUnitActiveSec=12h`).
    

### Checking Last / Next Run

Use:

`systemctl list-timers eicar-av-test.timer`

Example:

`NEXT                        LEFT          LAST                        PASSED       UNIT                 ACTIVATES Mon 2025-12-08 04:00:00     11h left      Sun 2025-12-07 16:00:10     1h ago      eicar-av-test.timer  eicar-av-test.service`

*   **LAST** → time of previous test run
    
*   **NEXT** → time of next scheduled test
    

* * *

## Viewing Test Results (Linux Host)

All output from `eicar_router_test.py` is captured by **systemd journal** under the service unit:

### Show Latest Run Output

`sudo journalctl -u eicar-av-test.service -n 100 --no-pager`

### Follow Logs in Real Time

`sudo journalctl -u eicar-av-test.service -f`

You’ll see:

*   Individual test results:
    
    `HTTP POST                    : BLOCKED Multipart Upload             : BLOCKED HTTPS                        : ALLOWED Encoded                      : ALLOWED Socket                       : BLOCKED File Transfer                : BLOCKED DNS Tunnel                   : BLOCKED WebSocket                    : ALLOWED`
    
*   An overall detection score:
    
    `Overall Score: 5/8 transmissions blocked`
    
*   Recommendations such as:
    
    *   Enable HTTP content inspection
        
    *   Enable SSL inspection for HTTPS
        
    *   Ensure DPI covers encoded content
        
    *   Review raw socket and DNS security policies
        

* * *

## Manual Script Usage (Ad-Hoc Testing)

Even with systemd scheduling, you can run the script manually when needed.

### Script Location

Installed at:

`/usr/local/eicar-av-test/eicar_router_test.py`

### Run Once (Interactive)

`python3 /usr/local/eicar-av-test/eicar_router_test.py`

You’ll be prompted:

`Press Enter to begin outbound EICAR transmission tests...`

### Run Once (Non-Interactive)

`python3 /usr/local/eicar-av-test/eicar_router_test.py --no-prompt`

* * *

## Self-Looping Mode (Without Systemd)

If you prefer not to use systemd, the script can loop itself using `--interval` and `--runs`.

### Example: Every 10 Minutes, Forever

`python3 /usr/local/eicar-av-test/eicar_router_test.py --interval 600 --no-prompt`

### Example: Every Hour, 5 Times, Then Exit

`python3 /usr/local/eicar-av-test/eicar_router_test.py --interval 01:00:00 --runs 5 --no-prompt`

`--interval` accepts:

*   A number of seconds (e.g. `600`), or
    
*   `HH:MM:SS` format (e.g. `01:00:00`)
    

* * *

## Customizing the Systemd Interval

To adjust how often the test is run by systemd:

1.  Edit the timer file:
    
    `sudo nano /etc/systemd/system/eicar-av-test.timer`
    
2.  Change:
    
    `OnUnitActiveSec=12h`
    
    Examples:
    
    *   Every 6 hours: `OnUnitActiveSec=6h`
        
    *   Every 1 hour: `OnUnitActiveSec=1h`
        
    *   Every 30 minutes: `OnUnitActiveSec=30min`
        
3.  Apply changes:
    
    `sudo systemctl daemon-reload sudo systemctl restart eicar-av-test.timer`
    
4.  Confirm:
    
    `systemctl list-timers eicar-av-test.timer`
    

* * *

## Managing the Service & Timer

### Check Service Status

`systemctl status eicar-av-test.service`

### Check Timer Status

`systemctl status eicar-av-test.timer`

### Manually Trigger a Test Run

`sudo systemctl start eicar-av-test.service`

### Stop Automatic Runs (Keep Files Installed)

`sudo systemctl disable --now eicar-av-test.timer`

### Re-Enable Automatic Runs

`sudo systemctl enable --now eicar-av-test.timer`

* * *

## Checking 128T Router & Security Logs

After each test run, verify that your **128T router** (or equivalent security device) detects the EICAR traffic.

### 1\. Conductor GUI (128T)

*   Navigate to:
    
    *   **Monitor → Events → IDPS Events**
        
    *   Filter for `"EICAR"` or `"Test"`
        
    *   Cross-check timestamps against the test run time on the Linux host
        

### 2\. On the 128T Router (Linux / CLI)

`# SSH to your 128T router ssh admin@<router-ip>  # Check antivirus service status systemctl status 128t-anti-virus  # Monitor antivirus logs in real time sudo journalctl -u 128t-anti-virus -f  # Check log files directly sudo ls -la /var/log/128t-anti-virus/ sudo tail -f /var/log/128t-anti-virus/anti-virus.log`

### 3\. PCLI on 128T

`# Enter PCLI pcli  # Show IDPS events show events type idps  # Show security-related events show events category security  # Check signature version show idps signatures`

* * *

## Expected Behavior

### ✅ When Protection is Working

*   Many or most test transmissions:
    
    *   Are blocked, reset, or error out
        
    *   Show connection or HTTP errors
        
*   Script output shows many `BLOCKED` results.
    
*   128T / firewall logs contain clear EICAR / test entries.
    

### ⚠️ When Protection Needs Work

*   Many tests succeed with `200 OK`.
    
*   Script shows several `ALLOWED` results for key vectors (HTTP, HTTPS, Encoded).
    
*   Logs show little or no detection activity for EICAR.
    

* * *

## Uninstall / Cleanup

To fully remove the tool and its scheduled test:

1.  Stop and disable the timer:
    
    `sudo systemctl disable --now eicar-av-test.timer`
    
2.  Remove systemd units:
    
    `sudo rm -f /etc/systemd/system/eicar-av-test.service sudo rm -f /etc/systemd/system/eicar-av-test.timer`
    
3.  Reload systemd:
    
    `sudo systemctl daemon-reload`
    
4.  Remove install directory and script:
    
    `sudo rm -rf /usr/local/eicar-av-test`
    

* * *

## Best Practices

1.  **Test regularly** – after policy changes, upgrades, or signature updates.
    
2.  **Verify logs**, not just blocking behavior.
    
3.  **Keep AV/IDPS signatures updated** on your router/firewall.
    
4.  **Test both HTTP and HTTPS**, and any other critical protocols your network uses.
    
5.  **Document your results** – screenshots, logs, and summaries are useful for audits and security reviews.
