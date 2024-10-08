# Tricky Store

A trick of keystore. **Android 10 or above is required**

Shamiko (or similar) is also required for the global props changes and root hiding it provides.

## Usage

1. Flash this module and reboot.  
2. For more than DEVICE integrity, put an unrevoked hardware keybox.xml at `/data/adb/tricky_store/custom_keybox.xml` (Optional).  
3. Customize target packages at `/data/adb/tricky_store/target.txt` (Optional).  
4. Enjoy!  

**All configuration files will take effect immediately.**

## Global mode
If the `global_mode` file exists in the `/data/adb/tricky_store` directory, the application will be effective for all apps without needing to create the target.txt file.

## Tee broken mode
If the `tee_broken_mode` file exists in the `/data/adb/tricky_store` directory, all package names in the `target.txt` file will be added to generatePackages, regardless of whether the package name ends with `!`.

## keybox.xml

format:

```xml
<?xml version="1.0"?>
<AndroidAttestation>
    <NumberOfKeyboxes>1</NumberOfKeyboxes>
    <Keybox DeviceID="...">
        <Key algorithm="ecdsa|rsa">
            <PrivateKey format="pem">
-----BEGIN EC PRIVATE KEY-----
...
-----END EC PRIVATE KEY-----
            </PrivateKey>
            <CertificateChain>
                <NumberOfCertificates>...</NumberOfCertificates>
                    <Certificate format="pem">
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----
                    </Certificate>
                ... more certificates
            </CertificateChain>
        </Key>...
    </Keybox>
</AndroidAttestation>
```

## Build Vars Spoofing

> **Zygisk (or Zygisk Next) is needed for this feature to work.**

If you still do not pass you can try enabling/disabling Build variable spoofing by creating/deleting the file `/data/adb/tricky_store/spoof_build_vars`.

Tricky Store will automatically generate example config props inside `/data/adb/tricky_store/spoof_build_vars` once created, on next reboot, then you may manually edit your spoof config.

Here is an example of a spoof config:

```
MANUFACTURER=Google
MODEL=Pixel 8 Pro
FINGERPRINT=google/husky_beta/husky:15/AP31.240617.009/12094726:user/release-keys
BRAND=google
PRODUCT=husky_beta
DEVICE=husky
RELEASE=15
ID=AP31.240617.009
INCREMENTAL=12094726
TYPE=user
TAGS=release-keys
SECURITY_PATCH=2024-07-05
```

For Magisk users: if you don't need this feature and zygisk is disabled, please remove or rename the
folder `/data/adb/modules/tricky_store/zygisk` manually.

## Support TEE broken devices

Tricky Store will hack the leaf certificate by default. On TEE broken devices, this will not work because we can't retrieve the leaf certificate from TEE. You can add a `!` after a package name to enable generate certificate support for this package.

For example:

```
# target.txt
# use leaf certificate hacking mode for KeyAttestation App
io.github.vvb2060.keyattestation
# use certificate generating mode for gms
com.google.android.gms!
```
## Custom ROMs support

If you are using a custom ROM and it passes Play Integrity (BASIC & DEVICE) by default, there is a good chance that this module won't work for you as your ROM is probably blocking Key Attestation.
To see if your ROM is compatible, look in the `android_frameworks_base` repo of your ROM and search for `PixelPropsUtils` or `setProps`.

To fix this issue, search for `engineGetCertificateChain` in that repo and see if there's some block of code that throws an exception if some condition that checks if it's related to key attestation (e.g. `PixelPropsUtils.getIsKeyAttest()` or `isCallerSafetyNet()`) is filled. You can delete this block of code and build your ROM yourself, or submit a commit to the maintainer of your ROM to add, for example, a system property to enable/disable this blocking. See [this commit](https://github.com/PixelBuildsROM/android_frameworks_base/commit/378ae3b7034d441fd0455639dc1a4b29b2876798) for reference.

In some custom ROMs you can disable the spoof, just set to false "persist.sys.pixelprops.pi" prop:

```
setprop persist.sys.pixelprops.pi false
```

If it doesn't work:

```
resetprop persist.sys.pixelprops.pi false
```

## TODO

- Support App Attest Key.
- Support automatic selection mode.

PR is welcomed.

## Acknowledgement

- [PlayIntegrityFix](https://github.com/chiteroman/PlayIntegrityFix)
- [FrameworkPatch](https://github.com/chiteroman/FrameworkPatch)
- [BootloaderSpoofer](https://github.com/chiteroman/BootloaderSpoofer)
- [KeystoreInjection](https://github.com/aviraxp/Zygisk-KeystoreInjection)
- [LSPosed](https://github.com/LSPosed/LSPosed)
