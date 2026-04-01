# Lab 6: Device Service Clusters (DSC)

## bigip01.f5demo.com configs

On bigip01.f5demo.com save the configuration and create a ha_vlan

```tmsh
save sys ucs lab6_pre_dsc
create /net vlan ha_vlan interfaces add { 1.3 }
create net self ha_ip { address 10.1.30.245/24 vlan ha_vlan }
modify /net self ha_ip allow-service default
```

## bigip02.f5demo.com configs

On bigip02.f5demo.com create all the same vlans; and then unique self IPs.

```tmsh
create /net vlan client_vlan interfaces add { 1.1 }
create /net vlan server_vlan interfaces add { 1.2 }
create /net vlan ha_vlan interfaces add { 1.3 }
create net self server_ip { address 10.1.20.246/24 vlan server_vlan }
create net self client_ip { address 10.1.10.246/24 vlan client_vlan }
create net self ha_ip { address 10.1.30.246/24 vlan ha_vlan }
modify /net self ha_ip allow-service default
create net route def_gw { network 0.0.0.0/0.0.0.0 gw 10.1.10.1 }
```

## Create Device Certificates and Keys

In the bash tmp directory, copy over the contents from the **f5devcert.cnf** file given below to bigip01 and bigip02. Use the CAT command to create the file. For bigip02, ensure you comment out and uncomment the lines related to the **commonName, SAN and IP** to match. TIP: use nano or vi to edit the file once on the device.

Review the contents of the certificate configuration file.

[f5devcert.cnf](../Lab6/f5devcert.cnf)  

Then generate new self-signed certificates and keys on bigip01 and bigip02. Run each command separately to ensure the command completes successfully.

```bash
cd /tmp
cat > f5devcert.cnf
```

Paste the contents into the Webshell and then CTRL+D to save the file.

Generate new self-signed certificates and keys on bigip01 and bigip02. Run each command separately to ensure the command completes successfully.

```bash
openssl req -newkey rsa:2048 -nodes -keyout /tmp/server.key -x509 -sha256 -days 30 -config f5devcert.cnf -out /tmp/server.crt

openssl x509 -noout -text -in server.crt
```

If the certificate is correct, copy the certificate and key to /config/httpd/conf/ssl.key and /config/httpd/conf/ssl.crt on both bigip01 and bigip02.

```bash
yes | cp /tmp/server.crt /config/httpd/conf/ssl.crt
yes | cp /tmp/server.key /config/httpd/conf/ssl.key
```

Then restart the httpd service on both bigip01 and bigip02.

```bash
bigstart restart httpd
```

## Reset Device Trust and Configure Device Service Cluster (DSC)

These steps must be done via TMUI. - On bigip01 and bigip02, "Reset Device Trust" under Device Management > Device Trust.  

Back on the Web Shell for bigip01, run the following commands to set the ConfigSync IP, Networkfailover IP and enable Mirror using the ha_ip self IPs.

```tmsh
modify cm device bigip01.f5demo.com configsync-ip 10.1.30.245
modify cm device bigip01.f5demo.com unicast-address { { ip 10.1.30.245 port 1026 } }
modify cm device bigip01.f5demo.com mirror-ip 10.1.30.245
```

Run these commands on bigip02.

```tmsh
modify cm device bigip02.f5demo.com configsync-ip 10.1.30.246
modify cm device bigip02.f5demo.com unicast-address { { ip 10.1.30.246 port 1026 } }
modify cm device bigip02.f5demo.com mirror-ip 10.1.30.246
```

These steps must be done via TMUI. - Under Device Management > Device Trust > Device Trust Members, add bigip02.f5demo.com as a peer device with IP, Username and Password. Then click "Retrieve Device Information" and "Save Device Trust". You should see bigip02.f5demo.com as a peer device with a status of "Active".

Then create a device group called my-device-group of type sync-failover and network-failover, and add both bigip01.f5demo.com and bigip02.f5demo.com to the group.
Then run a config sync to the device group.

```tmsh
create cm device-group my-device-group type sync-failover network-failover enabled 
modify cm device-group my-device-group devices add { bigip01.f5demo.com bigip02.f5demo.com }
run cm config-sync to-group my-device-group 
```

Change the Persistence profile on the www_vs virtual server to use source address persistence and enable mirroring.

```tmsh
modify ltm persistence source-addr my-src-persist mirror enabled 
modify /ltm virtual www_vs persist replace-all-with { my-src-persist }

run cm config-sync to-group my-device-group
```

In browswer goto http://www.f5demo.com.  

Then run the following command to force a failover.

```tmsh
run sys failover standby traffic-group traffic-group-1 
```

Then check the Traffic Group status to confirm the failover.

```tmsh
show cm traffic-group traffic-group-1 
```

Then check the website again to confirm it is still working after the failover.

## End of Lab 6

[NEXT - Lab 7: Common Configuration Items](../Lab7/readme.md)