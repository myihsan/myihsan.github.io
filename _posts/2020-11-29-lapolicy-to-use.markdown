---
layout: post
title: LAPolicy to use for authentication
---

[Logging a User with Face ID or Touch ID](https://developer.apple.com/documentation/localauthentication/logging_a_user_into_your_app_with_face_id_or_touch_id) is simple, all we need to do is to include the `NSFaceIDUsageDescription` key in Info.plist file and choose a LAPolicy to evaluate. But which one should we choose?

The document above uses `deviceOwnerAuthentication`, which will fallback to the device's passcode when biometrics (Face ID or Touch ID) fails or is unavailable and indicates that we can use `deviceOwnerAuthenticationWithBiometrics` to avoid the fallback.

So we may consider using `deviceOwnerAuthenticationWithBiometrics` in some cases required higher security. But it's not a good idea for security.

Firstly, we cannot change our biometrics by purpose, so once our biometric is compromised, it may not secure anymore.

Secondly, `deviceOwnerAuthenticationWithBiometrics` has almost the same security level as `deviceOwnerAuthentication`. Indeed `deviceOwnerAuthentication` will fallback to the passcode, making it look less secure, but we can change the biometric settings by the passcode from Settings.

For the devices using Face ID, we cannot prevent other people from using the passcode to change our biometric settings, but we can notice the change. The setting button has three states:

1. Set Up Face ID
2. Set Up an Alternate Appearance
3. Reset Face ID

To remove the alternate appearance, we have to reset Face ID. So once the state has been changed or cannot unlock by our face, we can notice that our biometric settings have changed.

However, for the devices using Touch ID, we can add a new fingerprint to pass the biometric verification and delete it later using the passcode. The only way to notice biometric settings change is to set up all five (max) fingerprints at first and check if the five fingerprints are still working.

## Conclusions

Using `deviceOwnerAuthentication` should be a better choice for authentication. And it may cause a positive side effect to let users keep their passcode safely.

If there are some reasons to use `deviceOwnerAuthenticationWithBiometrics`, consider providing a setting for the users who want to fallback to the passcode when biometrics fails or is unavailable, since we have to wear a mask every day nowadays.
