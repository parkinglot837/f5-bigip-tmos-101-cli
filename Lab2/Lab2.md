# Lab 2: Lab, Monitor, & Persistence 
### Ratio Load Balancing
Change Load Balancing
```
tmsh modify /ltm pool /Common/www_pool load-balancing-mode ratio-member
tmsh modify /ltm pool /Common/www_pool members modify { 10.1.20.11:80 {ratio 3 } }
```
Check the website and Statistics

### Priority Groups
Change LB back to Round Robin
```
tmsh modify /ltm pool /Common/www_pool load-balancing-mode round-robin
```
Add .11 and .12 to priority-group of 2
```
tmsh modify /ltm pool /Common/www_pool members modify { 10.1.20.11:80 10.1.20.12:80 {priority-group 2 } }
tmsh modify /ltm pool /Common/www_pool min-active-members 2
```
Check the website and Statistics
<br>Now Disable .11 pool member
```
tmsh modify /ltm node 10.1.20.11 session user-disabled
```
Check the website and Statistics
<br>
<br>Re-enable .11 pool member
```
tmsh modify /ltm node 10.1.20.11 session user-enabled
```
Change the Priority Group Activation back to Disabled
```
tmsh modify /ltm pool /Common/www_pool min-active-members 0
```

## Monitor Lab
```
tmsh modify /ltm default-node-monitor rule icmp
```

## Content Monitors
Create a content monitor called www_test
```
tmsh create /ltm monitor http www_test send "GET /index.php HTTP/1.0\r\n\r\n" recv "200 OK" 
```
Change the monitor for the Pool from 'http' to 'www_test'
```
tmsh modify ltm pool www_pool monitor www_test
```

### Check the Pool members

Unassign monitor and enable Reverse on the www_test
```
tmsh modify ltm pool www_pool monitor none 
modify /ltm monitor http www_test reverse enabled 
tmsh modify ltm pool www_pool monitor www_test
```

### Disable Reverse on the www_test
```
tmsh modify ltm pool www_pool monitor none 
modify /ltm monitor http www_test reverse disabled 
tmsh modify ltm pool www_pool monitor www_test
```

### Check the website, VS, & Pool and Stats

## Persistence Lab
### Persistence using Source Address
```
tmsh create /ltm persistence source-addr my-src-persist defaults-from source_addr timeout 60
tmsh modify /ltm virtual www_vs persist replace-all-with { my-src-persist }
```

### enable Persistence records
```
tmsh modify sys db ui.statistics.modulestatistics.localtraffic.persistencerecords value true
```


### Persistence from Cookie
```
tmsh create /ltm persistence cookie my_cookie_insert defaults-from cookie 
```

### Enable http client profile for www_vs
```
tmsh modify /ltm virtual www_vs profiles replace-all-with { http } 
tmsh modify /ltm virtual www_vs persist replace-all-with { my_cookie_insert }
```

### Save an Archive
```
tmsh save sys ucs lab3_lb_monitor_and_persist
```

### End of Lab 2
