# Lab 3: SSL Offload and Security

Skip Lab 3 and go directly to Lab 4 by running the consolidated commands at the end of this document.  
[Consolidated Commands for Lab 3](#consolidated-commands-for-lab-3)

## Create a Self-Signed Certificate and Key

```tmsh
create sys crypto key my-selfsigned-cert key-size 2048 gen-certificate country US city Reston state Virginia organization 'IT' common-name www.f5demo.com lifetime 365
```

## Creating SSL Client Profile

```tmsh
create ltm profile client-ssl my-clientssl-profile defaults-from clientssl cert my-selfsigned-cert key my-selfsigned-cert
```

or

```tmsh
create ltm profile client-ssl my_clientssl_profile defaults-from clientssl cert-key-chain  add { my-selfsigned-cert_0 { cert my-selfsigned-cert key my-selfsigned-cert } }
```

## Building our New Secure Virtual Server

```tmsh
create ltm virtual secure_vs { destination 10.1.10.105:443 ip-protocol tcp pool www_pool profiles add { my_clientssl_profile tcp } source-address-translation { type automap } }
```

## Securing web applications with the HTTP profile

```tmsh
create ltm profile http secure-my-website defaults-from http fallback-host https://www.f5.com fallback-status-codes add { 404 } response-headers-permitted add { Content-Type Set-Cookie Location } insert-xforwarded-for enabled 
```

Attach this HTTP profile to the secure virtual server

```tmsh
modify ltm virtual secure_vs profiles add { secure-my-website }
```

Now browse to https://www.f5demo.com and check the certificate. You will see a warning because the certificate is self-signed. Add the exception to your browser to continue.  

Then try to go to a non-existent page like https://www.f5demo.com/abc.html and you will see the fallback page from the HTTP profile.

## End of Lab 3

## Consolidated Commands for Lab 3

To skip lab 3 and go directly to lab 4, you can run the following commands in TMSH.

```tmsh
create sys crypto key my-selfsigned-cert key-size 2048 gen-certificate country US city Reston state Virginia organization 'IT' common-name www.f5demo.com lifetime 365
create ltm profile client-ssl my_clientssl_profile defaults-from clientssl cert-key-chain add { my-selfsigned-cert_0 { cert my-selfsigned-cert key my-selfsigned-cert } }
create ltm virtual secure_vs { destination 10.1.10.105:443 ip-protocol tcp pool www_pool profiles add { my_clientssl_profile tcp } source-address-translation { type automap } }
create ltm profile http secure-my-website defaults-from http fallback-host https://www.f5.com fallback-status-codes add { 404 } response-headers-permitted add { Content-Type Set-Cookie Location } insert-xforwarded-for enabled 
modify ltm virtual secure_vs profiles add { secure-my-website }
```

[NEXT - Lab 4: BIG-IP Policies and iRules](../Lab4/readme.md)
