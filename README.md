## Bypass-Android-SSL-Pinning

First pull the android application using adb

`
$ adb pull ./pathto/test.apk
`

Decompile the application using apktool

`
$ apktool d test.apk
`

Now edit the `network_security_config.xml` file in `/base/res/xml/network_security_config.xml`

network_security_config.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <domain-config>
        <domain includeSubdomains="true">api.testingdomain.com</domain>
        <domain includeSubdomains="true">staging-api.testingdomain.com</domain>
        <pin-set>
            <pin digest="SHA-256">YyKlrbMBaYz9z5JY/8ZhpoXhUJY9ZZUycEPcDoU7w2s=</pin>
            <pin digest="SHA-256">AoqlvZFWR5AIpe/sDL0AvjqxtHCydEHF0WdTRitLKCY=</pin>
        </pin-set>
    </domain-config>
</network-security-config>
```
Now remove `<ping-set>...</pin-set>` and add the `trust-anchors`tag and make it look like below

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <domain-config>
        <domain includeSubdomains="true">api.remembear.com</domain>
        <domain includeSubdomains="true">staging-api.remembear.com</domain>
        <trust-anchors>
            <certificates src="user" />
            <certificates src="system" />
        </trust-anchors>
    </domain-config>
</network-security-config>
```
now cd to application root directory and build the application again using apktool

`
$ apktool b ./
`

apktool will save the new modified apk in `dist` directory, Now use the keytool to generate the private key 

`
$ keytool -genkey -v -keystore my-release-key.keystore -alias alias_name -keyalg RSA -keysize 2048 -validity 10000
`

Now sign the modified application with the generated private key using jarsigner

`
$ jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore my-release-key.keystore test.apk alias_name
`

Now uninstall the old application from the device and install the new application

`
$ adb install test.apk
`

Done! now you can intercept the request with burp
