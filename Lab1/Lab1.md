# lab 1: Basics:

## Credentials
  ### Passwords:
f5UDFrocks!

### Understanding TMSH
The Traffic Management Shell (tmsh) is F5's command-line interface (CLI) for managing and configuring their BIG-IP systems. 
TMSH can be used to automate system administration tasks.
<br>
<br>[TMSH Reference](https://clouddocs.f5.com/cli/tmsh-reference/latest/)
###
When SSH'ing or opening a Web Shell session.
<br>Enter the TMSH by typing: `tmsh`


### Create Vlans
```
create /net vlan client_vlan interfaces add { 1.1 }
create /net vlan server_vlan interfaces add { 1.2 }
```

### Create Self IPs
```
create net self server_ip { address 10.1.20.245/24 vlan server_vlan }
create net self client_ip { address 10.1.10.245/24 vlan client_vlan }
```

### Create default gateway
```
create net route def_gw { network 0.0.0.0/0.0.0.0 gw 10.1.10.1 }
```

### Create Pool
```
create ltm pool /Common/www_pool members add { /Common/10.1.20.11:80  } monitor http
modify ltm pool /Common/www_pool members add { /Common/10.1.20.12:80 /Common/10.1.20.13:80 }
```

### Create Virtual Server
```
create ltm virtual www_vs destination 10.1.10.100:80 snat automap pool www_pool
```

<br>Test website on Jumpbox ###

### Save an Archive
```
save sys ucs lab2_the_basics_net_pool_vs
```

## Extra Credit
While in tmsh
```
run /util bash
bigtop
```
or `bigtop -n` which shows numeric port numbers.

### End of Lab 1



