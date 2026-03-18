# Lab 4: BIG-IP Policies and iRules

Skip Lab 4 and go directly to Lab 5 by running the consolidated commands at the end of this document.  
[Consolidated Commands for Lab 4](#consolidated-commands-for-lab-4)

## Write an iRule to retrieve images when an HTTP request is received

```tmsh
create ltm pool image_pool monitor http members add { 10.1.20.14:80 }
```

Change the irule editor to nano

```tmsh
modify cli preference editor nano
```

Paste the irule from the snippet below inserting a new line after the `create rule retrieve_images {` line. Then save and exit the editor using these key combos; CNTRL+X, Enter, Y, Enter.

```tmsh
create rule retrieve_images {
# If the content is a jpeg or portable graphic (png) go to the image pool
when HTTP_REQUEST {
   if { ([HTTP::uri] ends_with "jpg") or ([HTTP::uri] ends_with "svg") }
   {
      pool image_pool
   }
}
}
```

Add this irule to the virtual server

```tmsh
modify ltm virtual secure_vs rules { retrieve_images }
```

Use a BIG-IP Policy to retrieve images from a different pool¶

Create a Draft Policy called access_images_policy

```tmsh
create ltm policy Drafts/access_image_pool strategy all-match controls add { forwarding } rules add { get_images { conditions add { 0 { http-uri extension values { jpg svg } ends-with request } } actions add { 0 { forward pool image_pool request } } } }
```

Publish the policy

```tmsh
publish ltm policy Drafts/access_image_pool 
```

Remove iRule we added earlier from the virtual server

```tmsh
modify ltm virtual secure_vs rules none
```

Add the new policy to the virtual server

```tmsh
modify ltm virtual secure_vs policies add { access_image_pool }
```

## End of Lab 4

## Consolidated Commands for Lab 4

To skip lab 4 and go directly to lab 5, you can run the following commands in TMSH.

```tmsh
create ltm pool image_pool monitor http members add { 10.1.20.14:80 }
create ltm policy Drafts/access_image_pool strategy all-match controls add { forwarding } rules add { get_images { conditions add { 0 { http-uri extension values { jpg svg } ends-with request } } actions add { 0 { forward pool image_pool request } } } }
publish ltm policy Drafts/access_image_pool 
modify ltm virtual secure_vs rules none
modify ltm virtual secure_vs policies add { access_image_pool }
```

[NEXT - Lab 5: Support and Troubleshooting](../Lab5/readme.md)
