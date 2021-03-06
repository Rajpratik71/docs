# Register a new PoP of type Amazon

The table below provides an explaination about the different parameters used in the PoP JSON file. 
**Please make sure you have read the [prerequisites][amazon-driver] before registering an Amazon PoP.**

| Params         | Meaning                                                                                                                                                                                | Mandatory |
|----------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------:|
| name           | The name of the Point of Presence, important for later on linking it into your VNFD. **_Please note that the name chosen must be unique inside the project and will be used to refer to this particular PoP instance_.**                                                                                                                                                         |       yes |
| authUrl        | NA |       yes |
| vpcName        | Name of your VPC (check the amazon console)           |       yes |
| accesKey       | Your key that you can find under your user account settings |       yes |
| secretKey      | The secret key associated with the key that you can find under your user account settings |       yes |
| keyPair        | Name of the Key Pair available in your amazon cloud |       no |
| securityGroups | Keep default, open baton will create one         |        yes |
| type           | amazon |       yes |
| region         | Code region (example for EU (Ireland) add eu-west-1, etc.)                  |        yes |


## Examples 

```javascript
{
  "name":"amazon-pop",
  "authUrl":"http://amazon.com",
  "vpcName":"<assigned-vpc-name>",
  "accessKey":"<access-key>",
  "secretKey":"<secret-key>",
  "keyPair":"<key-pair-name>",
  "securityGroups": [
    "<security-group-name>"
  ],
  "type":"amazon",
  "region": "<region-where-vpc-is-created>"
}


```

[amazon-driver]: amazon-driver.md
