# 1. Reconnaissance and Subdomain Enumeration
1.1 Passive Subdomain Enumeration
🛠️Tools: Subfinder, Amass, CRTSH, Github-Search
Subfinder
subfinder -d target.com -silent -all -recursive -o subfinder_subs.txt
Amass (Passive Mode)
amass enum -passive -d target.com -o amass_passive_subs.txt
CRT.sh Query
curl -s "https://crt.sh/?q=%25.target.com&output=json" | jq -r '.[].name_value' | sed 's/\*\.//g' | anew crtsh_subs.txt
Github Dorking
github-subdomains -d target.com -t YOUR_GITHUB_TOKEN -o github_subs.txt
Results Combination
cat *_subs.txt | sort -u | anew all_subs.txt
1.2 Active Subdomain Enumeration
🛠️Tools: MassDNS, Shuffledns, DNSX, SubBrute, FFuF
MassDNS
massdns -r resolvers.txt -t A -o S -w massdns_results.txt wordlist.txt
Shuffledns
shuffledns -d target.com -list all_subs.txt -r resolvers.txt -o active_subs.txt
DNSX Resolution
dnsx -l active_subs.txt -resp -o resolved_subs.txt
SubBrute
python3 subbrute.py target.com -w wordlist.txt -o brute_force_subs.txt
FFuF Subdomain
ffuf -u https://FUZZ.target.com -w wordlist.txt -t 50 -mc 200,403 -o ffuf_subs.txt
1.3 Handling Specific (Non-Wildcard) Targets
🛠️Tools: GAU, Waybackurls, Katana, Hakrawler
GAU
gau target.example.com | anew gau_results.txt
Waybackurls
waybackurls target.example.com | anew wayback_results.txt
Katana
katana -u target.example.com -silent -jc -o katana_results.txt
Hakrawler
echo "https://target.example.com" | hakrawler -depth 2 -plain -js -out hakrawler_results.txt
Additional Advanced Techniques
🛠️Tools: CloudEnum, AWSBucketDump, S3Scanner
Reverse DNS
dnsx -ptr -l resolved_subs.txt -resp-only -o reverse_dns.txt
ASN Enumeration
amass intel -asn <ASN_NUMBER> -o asn_results.txt
Cloud Asset Enumeration
cloud_enum -k target.com
Results Validation
cat all_subs.txt | httpx -silent -title -o live_subdomains.txt

# 2. Discovery and Probing
2.1 HTTP Probing
🛠️Tools: httpx, httprobe
HTTPX Probing
httpx -l resolved_subs.txt -p 80,443,8080,8443 -silent -title -sc -ip -o live_websites.txt
Custom Filtering
cat live_websites.txt | grep -i "login\|admin" | tee login_endpoints.txt
2.2 JavaScript Analysis
🛠️Tools: LinkFinder, subjs, JSFinder, GF
JS Extraction
cat live_websites.txt | waybackurls | grep "\.js" | anew js_files.txt
LinkFinder Analysis
python3 linkfinder.py -i js_files.txt -o js_endpoints.txt
Sensitive Pattern Search
cat js_files.txt | gf aws-keys | tee aws_keys.txt
cat js_files.txt | gf urls | tee sensitive_urls.txt
API Key Validation
curl -X GET "https://api.example.com/resource" -H "Authorization: Bearer <extracted_key>"
2.3 Advanced Google Dorking
🛠️Tools: GitDorker
Automated Dorking
python3 GitDorker.py -tf <github_token.txt> -q target.com -d dorks.txt -o git_dorks_output.txt
Admin/Login Files
site:*.example.com inurl:"*admin | login" | inurl:.php | .asp
Config Files
site:*.example.com ext:env | ext:yaml | ext:ini
Public Keys
site:*.example.com inurl:"id_rsa.pub" | inurl:".pem"
2.4 URL Discovery
🛠️Tools: Katana, Gospider, Hakrawler
Katana Crawling
katana -list live_websites.txt -jc -o katana_urls.txt
Gospider
gospider -s "https://target.com" -d 2 -o gospider_output/
Hakrawler
echo "https://target.com" | hakrawler -depth 3 -plain -out hakrawler_results.txt
2.5 Archive Enumeration
🛠️Tools: GAU, Waybackurls, ParamSpider
Archive URL Collection
gau --subs target.com | anew archived_urls.txt
waybackurls target.com | anew wayback_urls.txt
Parameter Extraction
cat archived_urls.txt | grep "=" | anew parameters.txt

# 3. Advanced Enumeration Techniques
3.1 Parameter Discovery
🛠️Tools: Arjun, ParamSpider, FFuF
Arjun Parameter Discovery
arjun -u "https://target.example.com" -m GET,POST --stable -o params.json
ParamSpider Web Parameters
python3 paramspider.py --domain target.com --exclude woff,css,js --output paramspider_output.txt
FFuF Parameter Bruteforce
ffuf -u https://target.com/page.php?FUZZ=test -w /usr/share/wordlists/params.txt -o parameter_results.txt
3.2 Cloud Asset Enumeration
🛠️Tools: CloudEnum, AWSBucketDump, S3Scanner
Cloud Bucket Enumeration
cloud_enum -k target.com -b buckets.txt -o cloud_enum_results.txt
S3 Bucket Access Test
aws s3 ls s3://<bucket_name> --no-sign-request
S3 Bucket Content Dump
python3 AWSBucketDump.py -b target-bucket -o dumped_data/
3.3 Content Discovery
🛠️Tools: Feroxbuster, FFuF, Dirsearch
Feroxbuster
feroxbuster -u https://target.com -w /usr/share/wordlists/common.txt -r -t 20 -o recursive_results.txt
Dirsearch
dirsearch -u https://target.com -w /usr/share/wordlists/content_discovery.txt -e php,html,js,json -x 404 -o dirsearch_results.txt
FFuF Recursive
ffuf -u https://target.com/FUZZ -w /usr/share/wordlists/content_discovery.txt -mc 200,403 -recursion -recursion-depth 3 -o ffuf_results.txt
3.4 API Enumeration
🛠️Tools: Kiterunner, Postman, Burp Suite
Kiterunner
kr scan https://api.target.com -w /usr/share/kiterunner/routes-large.kite -o api_routes.txt
3.5 ASN Mapping
🛠️Tools: Amass, Shodan, Censys
ASN Lookup
amass intel -asn <ASN_Number> -o asn_ips.txt
Shodan Enumeration
shodan search "net:<ip_range>" --fields ip_str,port --limit 100
Censys Asset Search
censys search "autonomous_system.asn:<ASN_Number>" -o censys_assets.txt

# 4. Vulnerability Testing
4.1 High-Priority Vulnerabilities
🐞CSRF Testing
cat live_websites.txt | gf csrf | tee csrf_endpoints.txt
🐞LFI Testing
cat live_websites.txt | gf lfi | qsreplace "/etc/passwd" | xargs -I@ curl -s @ | grep "root:x:" > lfi_results.txt
🐞RCE Testing
curl -X POST -F "file=@exploit.php" https://target.com/upload
🐞SQLi Testing
ghauri -u "https://target.com?id=1" --dbs --batch
🐞Sensitive Data Search
cat js_files.txt | grep -Ei "key|token|auth|password" > sensitive_data.txt
🐞Open Redirect Test
cat urls.txt | grep "=http" | qsreplace "https://evil.com" | xargs -I@ curl -I -s @ | grep "evil.com"

# 5. The "Two-Eye" Approach 👀
First Eye: Focus on testing every gathered subdomain, endpoint, or parameter for common vulnerabilities.
Second Eye: Identify “interesting” findings like exposed credentials, forgotten subdomains, or admin panels.
Actionable Steps:
If a vulnerability is identified, create a proof of concept (POC) and test its impact.
If no vulnerabilities are found, pivot to deeper testing on unique subdomains or endpoints.

# 6. Proof of Concept (POC) Creation
🎥Video POC
Demonstrate vulnerabilities in action using screen recording tools like Greenshot or OBS Studio.
📸Screenshot POC
Capture clear screenshots with annotations to explain each step.
🛠️Tool: Greenshot.

# 7. Reporting
📝Report Structure
Executive Summary
Target Scope
Testing Timeline
Key Findings Summary
Risk Ratings
Technical Details
Vulnerability Title
Severity Rating
Affected Components
Technical Description
Steps to Reproduce
Impact Analysis
Supporting Evidence (POC)
Remediation
Detailed Recommendations
Mitigation Steps
Additional Security Controls
References & Resources
Supporting Materials
Video Demonstrations
Screenshots & Annotations
HTTP Request/Response Logs
Code Snippets
Timeline of Discovery
Best Practices
Write clear, concise descriptions
Include detailed reproduction steps
Provide actionable remediation advice
Support findings with evidence
Use professional formatting
Highlight business impact
Include verification steps
Report Format
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

