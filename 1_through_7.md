# Bug Bounty Methodology

A structured workflow from recon to reporting.

---

## 1. Reconnaissance and Subdomain Enumeration

### 1.1 Passive Subdomain Enumeration
**Tools:** Subfinder, Amass, crt.sh, GitHub Search

```bash
# Subfinder
subfinder -d target.com -silent -all -recursive -o subfinder_subs.txt

# Amass (passive mode)
amass enum -passive -d target.com -o amass_passive_subs.txt

# crt.sh query
curl -s "https://crt.sh/?q=%25.target.com&output=json" | jq -r '.[].name_value' | sed 's/\*\.//g' | anew crtsh_subs.txt

# GitHub dorking
github-subdomains -d target.com -t YOUR_GITHUB_TOKEN -o github_subs.txt

# Combine results
cat *_subs.txt | sort -u | anew all_subs.txt
```

### 1.2 Active Subdomain Enumeration
**Tools:** MassDNS, Shuffledns, dnsx, SubBrute, FFuF

```bash
# MassDNS
massdns -r resolvers.txt -t A -o S -w massdns_results.txt wordlist.txt

# Shuffledns
shuffledns -d target.com -list all_subs.txt -r resolvers.txt -o active_subs.txt

# dnsx resolution
dnsx -l active_subs.txt -resp -o resolved_subs.txt

# SubBrute
python3 subbrute.py target.com -w wordlist.txt -o brute_force_subs.txt

# FFuF subdomain fuzzing
ffuf -u https://FUZZ.target.com -w wordlist.txt -t 50 -mc 200,403 -o ffuf_subs.txt
```

### 1.3 Handling Specific (Non-Wildcard) Targets
**Tools:** GAU, Waybackurls, Katana, Hakrawler

```bash
gau target.example.com | anew gau_results.txt
waybackurls target.example.com | anew wayback_results.txt
katana -u target.example.com -silent -jc -o katana_results.txt
echo "https://target.example.com" | hakrawler -depth 2 -plain -js -out hakrawler_results.txt
```

### 1.4 Additional Advanced Techniques
**Tools:** CloudEnum, AWSBucketDump, S3Scanner

```bash
# Reverse DNS
dnsx -ptr -l resolved_subs.txt -resp-only -o reverse_dns.txt

# ASN enumeration
amass intel -asn <ASN_NUMBER> -o asn_results.txt

# Cloud asset enumeration
cloud_enum -k target.com

# Validate live results
cat all_subs.txt | httpx -silent -title -o live_subdomains.txt
```

---

## 2. Discovery and Probing

### 2.1 HTTP Probing
**Tools:** httpx, httprobe

```bash
httpx -l resolved_subs.txt -p 80,443,8080,8443 -silent -title -sc -ip -o live_websites.txt

# Custom filtering
cat live_websites.txt | grep -i "login\|admin" | tee login_endpoints.txt
```

### 2.2 JavaScript Analysis
**Tools:** LinkFinder, subjs, JSFinder, GF

```bash
# JS extraction
cat live_websites.txt | waybackurls | grep "\.js" | anew js_files.txt

# LinkFinder analysis
python3 linkfinder.py -i js_files.txt -o js_endpoints.txt

# Sensitive pattern search
cat js_files.txt | gf aws-keys | tee aws_keys.txt
cat js_files.txt | gf urls | tee sensitive_urls.txt

# API key validation
curl -X GET "https://api.example.com/resource" -H "Authorization: Bearer <extracted_key>"
```

### 2.3 Advanced Google Dorking
**Tools:** GitDorker

```bash
python3 GitDorker.py -tf <github_token.txt> -q target.com -d dorks.txt -o git_dorks_output.txt
```

Example dorks:
```
site:*.example.com inurl:"*admin | login" | inurl:.php | .asp
site:*.example.com ext:env | ext:yaml | ext:ini
site:*.example.com inurl:"id_rsa.pub" | inurl:".pem"
```

### 2.4 URL Discovery
**Tools:** Katana, GoSpider, Hakrawler

```bash
katana -list live_websites.txt -jc -o katana_urls.txt
gospider -s "https://target.com" -d 2 -o gospider_output/
echo "https://target.com" | hakrawler -depth 3 -plain -out hakrawler_results.txt
```

### 2.5 Archive Enumeration
**Tools:** GAU, Waybackurls, ParamSpider

```bash
gau --subs target.com | anew archived_urls.txt
waybackurls target.com | anew wayback_urls.txt

# Parameter extraction
cat archived_urls.txt | grep "=" | anew parameters.txt
```

---

## 3. Advanced Enumeration Techniques

### 3.1 Parameter Discovery
**Tools:** Arjun, ParamSpider, FFuF

```bash
arjun -u "https://target.example.com" -m GET,POST --stable -o params.json
python3 paramspider.py --domain target.com --exclude woff,css,js --output paramspider_output.txt
ffuf -u https://target.com/page.php?FUZZ=test -w /usr/share/wordlists/params.txt -o parameter_results.txt
```

### 3.2 Cloud Asset Enumeration
**Tools:** CloudEnum, AWSBucketDump, S3Scanner

```bash
cloud_enum -k target.com -b buckets.txt -o cloud_enum_results.txt
aws s3 ls s3://<bucket_name> --no-sign-request
python3 AWSBucketDump.py -b target-bucket -o dumped_data/
```

### 3.3 Content Discovery
**Tools:** Feroxbuster, FFuF, Dirsearch

```bash
feroxbuster -u https://target.com -w /usr/share/wordlists/common.txt -r -t 20 -o recursive_results.txt
dirsearch -u https://target.com -w /usr/share/wordlists/content_discovery.txt -e php,html,js,json -x 404 -o dirsearch_results.txt
ffuf -u https://target.com/FUZZ -w /usr/share/wordlists/content_discovery.txt -mc 200,403 -recursion -recursion-depth 3 -o ffuf_results.txt
```

### 3.4 API Enumeration
**Tools:** Kiterunner, Postman, Burp Suite

```bash
kr scan https://api.target.com -w /usr/share/kiterunner/routes-large.kite -o api_routes.txt
```

### 3.5 ASN Mapping
**Tools:** Amass, Shodan, Censys

```bash
amass intel -asn <ASN_Number> -o asn_ips.txt
shodan search "net:<ip_range>" --fields ip_str,port --limit 100
censys search "autonomous_system.asn:<ASN_Number>" -o censys_assets.txt
```

---

## 4. Vulnerability Testing

High-priority vulnerability checks to run against discovered assets:

| Vuln | Command |
|---|---|
| CSRF | `cat live_websites.txt \| gf csrf \| tee csrf_endpoints.txt` |
| LFI | `cat live_websites.txt \| gf lfi \| qsreplace "/etc/passwd" \| xargs -I@ curl -s @ \| grep "root:x:" > lfi_results.txt` |
| RCE (upload test) | `curl -X POST -F "file=@exploit.php" https://target.com/upload` |
| SQLi | `ghauri -u "https://target.com?id=1" --dbs --batch` |
| Sensitive data in JS | `cat js_files.txt \| grep -Ei "key\|token\|auth\|password" > sensitive_data.txt` |
| Open Redirect | `cat urls.txt \| grep "=http" \| qsreplace "https://evil.com" \| xargs -I@ curl -I -s @ \| grep "evil.com"` |

---

## 5. The "Two-Eye" Approach 👀

- **First Eye:** Test every gathered subdomain, endpoint, or parameter for common vulnerabilities.
- **Second Eye:** Look for "interesting" findings — exposed credentials, forgotten subdomains, admin panels.

**Actionable steps:**
- If a vulnerability is found → build a PoC and assess impact.
- If nothing is found → pivot to deeper testing on unique subdomains/endpoints.

---

## 6. Proof of Concept (PoC) Creation

- **Video PoC:** Screen-record the exploit in action (OBS Studio).
- **Screenshot PoC:** Annotated screenshots explaining each step (Greenshot).

---

## 7. Reporting

### Report Structure
1. **Executive Summary** — target scope, testing timeline, key findings, risk ratings
2. **Technical Details** — title, severity, affected components, description, reproduction steps, impact, evidence
3. **Remediation** — detailed recommendations, mitigation steps, additional controls
4. **References & Resources** — supporting materials, videos, screenshots, HTTP logs, code snippets, discovery timeline

### Best Practices
- Write clear, concise descriptions
- Include detailed reproduction steps
- Provide actionable remediation advice
- Support findings with evidence
- Use professional formatting
- Highlight business impact
- Include verification steps

### Report Template

```markdown
# Vulnerability Report: [Title]

## Overview
- Severity: [Critical/High/Medium/Low]
- CVSS Score: [Score]
- Affected Component: [Component]

## Description
[Detailed technical description]

## Steps to Reproduce
1. [Step 1]
2. [Step 2]
3. [Step n...]

## Impact
[Business and technical impact]

## Proof of Concept
[Screenshots, videos, code]

## Recommendations
[Detailed fix recommendations]

## References
[CVE, CWE, related resources]
