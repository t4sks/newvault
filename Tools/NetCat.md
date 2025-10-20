tool what help to listen ports and do reverse shell from another PC
syntax:
``` shell
  nc -lvnp port_number
  ``` 
1. `-l` is used to tell what this will be a listener
2. `-v` used to see more information from console
3. `-n` say dont use dns and resolve hostname
4. `-p` say what well be used port identification
Mark: use port before 1024 because firewall can block ur connection, if u use port number more than 1024 use <mark style="background: #D00C0CA6;">sudo</mark>
if u want to use ***BIND SHELL***  Netcat syntax
- `nc Target-ip chosen-port`

