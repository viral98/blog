---
layout: post
title: Signing your electron app!
---
![image](https://cdn.rawgit.com/electron/electron/f083380c3838242ae19cd1d8cb42c466b90c183a//docs/images/gatekeeper.png)

## Pre-requisites

### Certificates

Your auto update doesn’t necessarily require certificates, but it's honestly worthless to incorporate auto-updates without this - Especially in MacOS, you won't be able to distribute your app at all, to begin with, leading to a situation where you build the app and auto-update your own app. Buy the certificates for the respective platforms. I’ll be assuming you have the certificates handy!

### Some CI platform for releases

From a PR merger -> updating the desktop app at the client’s end there are a whole lot of things that need to happen in order to get this module working correctly. You can opt to do all of this manually, but having a CI makes life a lot easier.

## Getting Started

Code Signing for someone having no experience with actual development and handling fancy infrastructure handling tools is DAUNTING. However, it can be streamlined fairly easily. Your auto-updates essentially rely on two major modules

However, we need this, period. Code signing is a non-negotiable thing that you should set up early on in the development phases. For windows, it requires you to buy a DigiCert based certificate (there are other fancier & more pricey methods but for most startups, this is the sweet spot for ROI). Things over on the apple side of things are pretty straightforward. Just make a project in your developer account!

### For Mac

The steps will be -

1. Go to your CI tool's project, and under project settings -> env variables, add APPLEID , APPLEIDPASS & APPLE_ID_PROVIDER (you will get these in the apple developer console)

2. Next, add the following script to be called at build time

```javascript
require("dotenv").config();
const { notarize } = require("electron-notarize");
const { appId } = require("./build-env");
exports.default = async function notarizing(context) {
  const { electronPlatformName, appOutDir } = context;
  if (electronPlatformName !== "darwin") {
    return;
  }
  const appName = context.packager.appInfo.productFilename;
  return await notarize({
    appBundleId: appId,
    appPath: `${appOutDir}/${appName}.app`,
    appleId: process.env.APPLEID,
    appleIdPassword: process.env.APPLEIDPASS,
    ascProvider: process.env.APPLE_ID_PROVIDER,
  });
};
```

This uses electron notarize as you might have guessed in order to provide the abstractions for the underlying notarization infrastructure

You will also need to add your p12 certificate (can be exported from the dev console)
Once you upload it to ci, fetch your certificate password and in the YOUR_CI_TOOL'S config.yml in your repo, add this command

```shell
- run:
          name: Setup Certificates
          no_output_timeout: 45m
          command: |
            echo $CERTIFICATE_OSX_P12 | base64 --decode > certificate.p12
            security create-keychain -p YOUR_CI_TOOL build.keychain
            security default-keychain -s build.keychain
            security unlock-keychain -p YOUR_CI_TOOL build.keychain
            security import certificate.p12 -k build.keychain -P $CERTIFICATE_P12_PASS -T /usr/bin/codesign;
            security set-key-partition-list -S apple-tool:,apple: -s -k YOUR_CI_TOOL build.keychain
            rm -fr *.p12
```

Now all you got to do is configure this script to be called once your build is done

Assuming you are using electron-builder, then in the afterSign property, add afterSign: shouldSign ? "script/notarize.js" : undefined  where should sign is essentially a variable indicating that your env isnt local and you are running on your CI tool (you can check that by const runningOnCI= process.env.CI_TOOL === "true";)
This would come under the build configuration file for your CI tool

### Windows

Getting the correct certificate for this is hard, a certifcate for windows will require a good chunk of paperwork and is generally a long drawn process.
Quoting from <https://blog.dcpos.ch/how-to-make-your-electron-app-sexy>
"To get a Windows signing certificate, we recommend Digicert. The documentation for Windows app signing is surprisingly bad. If you go with the wrong vendor, they'll ask you to mail them notarized paperwork. That makes it a slow and annoying process to get the cert. Digicert is easier: they just send you a password via Certified Mail, you go to the post office, show your ID to pick it up, and bam, you get your signing certificate."

The large difference in the certificate types/authorities is why I am not specifying particular steps for this one, however feel free to ping me! I'd be more than happy to help you out!

## Closing notes

I cannot emphasize this enough, use a CI tool. Your certificates need to be shoved somewhere safe and developers shouldn't directly interact with them as far as possible. Circle, for example, doesn't allow reading the env variables once they are set to other developers.
Also, getting code signing up and running might not sound fun/business need at the start of the project but getting this module right, takes time especially for windows, it's better to start early on.
