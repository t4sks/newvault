#web #BruteForce
instrumjent for URL/Domain/virtual hosts brutreforce - scan dir and files, subdomains and virtual hosts by wordlists.
main functions
`dir` - search dir/files on webservers
`dns` - search subdomains from wordlists
`vhosts` - check virtual hosts
Keys options 
```shell
gobuster dir -u URL -w WORDLIST [options]
```
Fast search and write in file
```shell
 gobuster dir -u http://example -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,php,json,zip -t 10
```
with check php or html
```shell
gobuster dir -u http://example.com -w /usr/share/wordlists/raft-small-words.txt -x php,html,txt -t 10 -s 200,403 -o res.txt
```
when site give redirects (301/301 http codes)
```shell
gobuster dir -u http://example.com -w wordlist.txt -s 200,301,302,403 -t 10
```
search subdomains
```shell
gobuster dns -d example.com -w /usr/share/wordlists/dns/subdomains-top1million-5000.txt -t 10 -o dns_res.txt
```
most useful wordlists 
```
/usr/share/wordlists/dirb/common.txt
/usr/share/wordlists/raft-large-directories.txt
Discovery/Web-Content
```
