# subdomain-recon-
**Web subdomain reconnaissance** (or *subdomain recon*) is the process of identifying subdomains associated with a target domain. It's a crucial step in penetration testing, bug bounty hunting, or general cybersecurity reconnaissance, as subdomains can expose development environments, internal tools, or forgotten services.

Here’s a practical breakdown of how to perform subdomain reconnaissance:

---

### **1. Passive Enumeration (Stealthy, No Direct Interaction)**

These tools and methods collect data from public sources:

#### Tools & Techniques:

* **[crt.sh](https://crt.sh/)** – Search for subdomains in public SSL certificate logs:

  ```
  site:crt.sh "example.com"
  ```
* **Amass (Passive Mode)**

  ```bash
  amass enum -passive -d example.com
  ```
* **Sublist3r**

  ```bash
  sublist3r -d example.com
  ```
* **Assetfinder**

  ```bash
  assetfinder --subs-only example.com
  ```
* **GitHub & GitLab Search**
  Sometimes hardcoded URLs and endpoints reveal subdomains.

---

###  **2. Active Enumeration (More Aggressive)**

These methods interact with the target or use brute-forcing.

#### Tools & Techniques:

* **Amass (Active Mode)**

  ```bash
  amass enum -active -d example.com
  ```
* **Subfinder**

  ```bash
  subfinder -d example.com
  ```
* **DNSMap or Fierce** – Brute-force subdomain names.
* **dnsx** (from ProjectDiscovery)

  ```bash
  subfinder -d example.com | dnsx -silent
  ```
* **MassDNS + Wordlist**

  ```bash
  massdns -r resolvers.txt -t A -o S -w results.txt wordlist.txt
  ```

---

###  **3. Tools for Wordlists and Automation**

* **Wordlists**: Use lists like `dns-Jhaddix.txt`, `subdomains-top1million-5000.txt` (from SecLists)
* **Automation Frameworks**:

  * **ProjectDiscovery’s Chaos dataset**: Public bug bounty domains.
  * **ReconFTW**
  * **OWASP Amass**
  * **OneForAll**

---

###  **4. Validate and Resolve Subdomains**

Not all found subdomains are live. Tools like:

* `httpx` – Check if subdomains are alive (HTTP/S):

  ```bash
  cat subdomains.txt | httpx -silent
  ```
* `dnsx` – Validate DNS resolution:

  ```bash
  cat subdomains.txt | dnsx -silent
  ```

---

If you want to perform **subdomain reconnaissance with status code checking** (to see which subdomains are live and what their HTTP response status is), you can combine tools like `subfinder` (for subdomain discovery) and `httpx` (for checking HTTP status codes).

---

###  **One-Liner for Subdomain Recon + Status Codes**

Here’s a powerful command-line one-liner using `subfinder` and `httpx`:

```bash
subfinder -d example.com -silent | httpx -status-code -title -silent
```

####  Output Example:

```
https://blog.example.com [200] [Welcome to Our Blog]
http://dev.example.com [403] [Forbidden]
https://api.example.com [301] [Moved Permanently]
```

---

###  **Installation (if needed)**

#### Install `subfinder`:

```bash
GO111MODULE=on go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
```

#### Install `httpx`:

```bash
GO111MODULE=on go install -v github.com/projectdiscovery/httpx/cmd/httpx@latest
```

Make sure `GOPATH` is in your system `$PATH` so you can run them directly.

---

###  **Optional Enhancements**

* Use `-ports 80,443,8080` to test multiple common ports.
* Use `-tech-detect` to fingerprint technologies.
* Output to a file:

  ```bash
  subfinder -d example.com -silent | httpx -status-code -title -silent -o live_subdomains.txt
  ```
Great! Here's a **Bash script** that performs subdomain reconnaissance with `subfinder` and `httpx`, checks HTTP response status codes, page titles, and saves everything to a timestamped output file.

---

###  **Bash Script: `subdomain_recon.sh`**

```bash
#!/bin/bash

# Usage: ./subdomain_recon.sh example.com

if [ -z "$1" ]; then
    echo "Usage: $0 <domain>"
    exit 1
fi

DOMAIN=$1
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
OUTPUT_DIR="recon_results_$DOMAIN"
mkdir -p $OUTPUT_DIR

echo "[*] Running subfinder on $DOMAIN..."
subfinder -d "$DOMAIN" -silent -o "$OUTPUT_DIR/subdomains.txt"

echo "[*] Probing for live subdomains with status codes and titles..."
httpx -silent -status-code -title -no-color -threads 50 -o "$OUTPUT_DIR/live_subdomains_$TIMESTAMP.txt" < "$OUTPUT_DIR/subdomains.txt"

echo "[+] Recon complete. Results saved to: $OUTPUT_DIR/live_subdomains_$TIMESTAMP.txt"
```

---

### **Requirements**

Make sure the following tools are installed:

```bash
go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
go install -v github.com/projectdiscovery/httpx/cmd/httpx@latest
```

Ensure your `GOPATH/bin` (typically `~/go/bin`) is in your `$PATH`:

```bash
export PATH=$PATH:$(go env GOPATH)/bin
```

---

###  **How to Run**

1. Save the script as `subdomain_recon.sh`
2. Make it executable:

   ```bash
   chmod +x subdomain_recon.sh

   ```
   
3. Run it:

   ```bash
   ./subdomain_recon.sh example.com
   ```

---





