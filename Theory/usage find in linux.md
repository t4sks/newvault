[[Linux]]
 
in linux find so powerful instrument to find file or something else
examples of command:
<mark style="background: #D00C0CA6;">find . -name flag1.txt</mark> - find file in current catalog
<mark style="background: #D00C0CA6;">find /home -name flag1.txt</mark> - find file flag.txt in /home catalog
<mark style="background: #D00C0CA6;">find / -type d -name config</mark> - find from / catalog, -type - only dir, name config - name must be config
<mark style="background: #D00C0CA6;">find / -type file -perm 0777</mark> - find in / files with max open privileges write execute read
<mark style="background: #D00C0CA6;">find / -perm a=x</mark> - find executable files in /
<mark style="background: #D00C0CA6;">find /home -user frank</mark> - find only in /home files from frank
<mark style="background: #D00C0CA6;">find / -mtime 10</mark> - files what was changed 10 days ago
<mark style="background: #D00C0CA6;">find / -atime 10</mark> - files what read or execute 10 days ago
<mark style="background: #D00C0CA6;">find / -cmin -60</mark> - files what was changed from last 60 minutes
<mark style="background: #D00C0CA6;">find / -amin -60</mark> - files what was open o read from last 60 minutes
<mark style="background: #D00C0CA6;">find / -size 50m</mark> - files with size 50 mbait, m - megabait, k - kilobaites, g - gigabaites
script for fast find flag if u root
```shell
find / -type f -name "flag7.txt" -exec cat {} \; 2>/dev/null
```



