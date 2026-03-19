# Lab 5: Suppport and Troubleshooting

Skip Lab 4 and go directly to Lab 5 - there are no prerequisites in this lab for Lab 5.  
[SKIP to Lab 6: Device Service Clusters (DSC)](../Lab6/readme.md)

## Troubleshooting using TCPdump or Curl

To test turn off AutoMap on the virtual server's source address translation.

```tmsh
modify ltm virtual www_vs source-address-translation { type none }
```

Then run a TCP dump on the client side to capture traffic.

```tmsh
tcpdump –i client_vlan host –X –s128 10.1.10.100 and port 80
```

Run a TCP dump on the server side to capture traffic.

```tmsh
tcpdump –i server_vlan host –X –s128 10.1.20.13 and port 80
```

We see the communication between the client and the server, but no return traffic from the server to the client. This is because the virtual server's source address translation is turned off, so the server is trying to send the response back to the client directly, but it cannot reach it.

Then run a curl command to the one of the backend servers to test the connection.
The -I option shows the HTTP response headers.

```tmsh
curl -I 10.1.20.13 
```

The output shows a 200 OK response, so the server is communicating with the BIG-IP.
The likely cause is that Server trying to reach the client directly.

Return the virtual server's source address translation back to AutoMap.

```tmsh
modify ltm virtual www_vs source-address-translation { type automap } 
```

## End of Lab 5

[NEXT - Lab 6: Device Service Clusters (DSC)](../Lab6/readme.md)