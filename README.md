# frisky

Instruments to assist in binary application reversing and augmentation, geared towards walled gardens like iOS. Most, if not all, recently tested on iOS 11.1.2 and macOS 10.12.6.

![iOS](assets/ios-inj-small.jpg)

##### [frida-url-interceptor.js](frida-url-interceptor.js) - Intercepts all URLs of an iOS/macOS application, allowing you to trace and alter/intercept all network traffic, including https, per app before encryption and after decryption:
  - iOS: open app of interest first, e.g. Safari
  - macOS: `frida -U -n Safari -l frida-url-interceptor.js`
  - It will also display a popup window upon the first request in the iOS app to confirm code interception
  
##### [ldid / ldid2](https://github.com/samyk/ldid) - When building recent iOS jailbreaks dependent on SHA256 signatures, `ldid2` is required. This repo will allow you to easily compile `ldid` and `ldid2` for signing and modifying an iOS binary's entitlements, and thus jailbreaking a device.
  - macOS: `ldid{2} -e MobileSafari` # to dump MobileSafari's entitlements
  - macOS: `ldid{2} -S cat` # to sign cat

##### Extract shared libraries used by apps not directly available on iOS filesystem for static analysis:
  - Grab the patched [dyld-210.2.3-patched](dyld-210.2.3-patched) (included in this repo) and run the custom [dsc_extractor](dyld-210.2.3-patched/launch-cache/dsc_extractor) (you may need to compile from the xcodeproject) to dump iOS' `/System/Library/Caches/com.apple.dyld/dyld_shared_cache_arm*` into individual dylibs:
  - macOS: `mkdir -p dylibs && dyld-210.2.3-patched/launch-cache/dsc_extractor /path/to/copied/dyld_shared_cache_arm* dylibs`

##### Discover and modify library/framework function call arguments and return codes via [Frida](https://www.frida.re/):
  - iOS: open app of interest first, e.g. Twitter
  - macOS: `frida-trace -U -i "*tls*" Twitter` # hook all calls matching */tls/i* for the Twitter app
  - macOS: Now `__handlers__/libcoretls.dylib/tls_private_key_create.js` will be generated:
    - `onEnter`'s `args[2]` is first argument to the function
      - Extract string from first argument: `Memory.readUtf8String(args[2])` or `ObjC.Object(args[2]))`
    - `onLeave`'s `retval` is the return value
      - Print out retval: `log(retval.toInt32())`
      - Adjust retval: `retval.replace(0)`

##### Sniff network traffic from (non-jailbroken/jailbroken) iOS device from your mac:
  - macOS: ```system_profiler SPUSBDataType|perl -n0e'`rvictl -s $1`if/iP(?:hone|ad):.*?Serial Number: (\S+)/s';sudo tcpdump -i rvi0```
  - standard tcpdump options/filters apply

##### Decrypt IPA (iOS apps)/Frameworks for static analysis via [dumpdecrypted.dylib](https://github.com/conradev/dumpdecrypted):
  - iOS: `su mobile && mkdir -p ~/tmp && cd ~/tmp && DYLD_INSERT_LIBRARIES=/usr/lib/dumpdecrypted.dylib /var/containers/Bundle/Application/*/AppName.app/AppName`

##### View system logs on iOS live using [deviceconsole](https://github.com/rpetrich/deviceconsole):
  - macOS: `deviceconsole`
  - macOS: `unbuffer deviceconsole | grep something` # keeps pretty colors
  		- requires `expect`, can be installed via `sudo port install expect` or `brew install expect`

##### Electra: allow jailbroken Tweaks to appear in Settings:
  - iOS: `mv /Library/TweakInject /Library/TweakInject.bak && ln -s /Library/MobileSubstrate/DynamicLibraries /Library/TweakInject && killall -HUP SpringBoard`

## Contact

Shaped by [@SamyKamkar](https://twitter.com/samykamkar) / [https://samy.pl](https://samy.pl)
