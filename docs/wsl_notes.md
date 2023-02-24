# connect to github
Needed to stop goofing up DNS to get git commands to work on WSL2 Ubuntu on Windows 11
Used the resolv.conf directions from [superuser stackexchange]([url](https://superuser.com/questions/1691097/wsl2-cannot-access-the-internet-on-windows-11)) 

# Allow the port on the host
Need to open port on the host; in admin terminal on Windows OS machine, enable the port:
```
netsh advfirewall firewall add rule name= "open port 50000" dir=in action=allow protocol=TCP localport=50000
ok.
```
# Connect the host OS to the WSL2 instance
Need to connect the ports between WSL2 IP to the host Windows IP, per the WSL2 networking notes linked above:
```
netsh interface portproxy add v4tov4 listenport=50000 listenaddress=0.0.0.0 connectport=50000 connectaddress=<WSL2 IP Address>
```
Where the `<WSL2 IP Address>` is found on the WSL2 OS via the command `~$ ip addr | grep eth0` per the WSL2 networking notes linked above.
   
Further, need to open port for WSL2 "out" to let WSL2 in VS code to connect through Win 11 OS to network:
```
netsh advfirewall firewall add rule name= "open port 50000" dir=in action=allow protocol=TCP localport=50000
```
