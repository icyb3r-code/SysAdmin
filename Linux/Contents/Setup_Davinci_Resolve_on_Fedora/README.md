# Setup DaVinci Resolve on Fedora

## Download Davinci 

you can download DaVinci Resolve from [here](https://sw.blackmagicdesign.com/DaVinciResolve/v21.0.2/DaVinci_Resolve_21.0.2_Linux.zip?Key-Pair-Id=APKAJTKA3ZJMJRQITVEA&Signature=psqlnovbm6SfgFKgZsLVdTu13swLS8vdV5Ulo9KGRmVhjr71ZzbogLpFXKBWbUw+l49wLsKOCtuaxee1UcXDopCRXIglFXHDs9uB6KxfG/Afh8tAQ45VZ3l2xdyvriL+9LPkoKZWyCvBHWxnFuk6dkCmLx8xUTPyd67BEEuWjK+E42V8YkT1ORGEpw4MkwS70XtNQ7zrHrq5TfyaiFP7XiV2VqDqh0MIRqZQeQpC0bX1PkVE94rtfLCLf0guv0eAOaM3AGrTVQgOUM7dhSAvgmwQFAvaV5mufKp1W5KChBPCWicm3ejkrMsrpY2niVd2MfTIxRWGWWiDmchSBETFoQ==&Expires=1784461644)

## Install DaVinci Resolve 

```bash
sudo SKIP_PACKAGE_CHECK=1 ./DaVinci_Resolve_21.0.2_Linux.run -i
```

## Remove/Backup Old Library 

```bash
# 1. Switch into the app's library directory
cd /opt/resolve/libs

# 2. Create a backup storage drawer for the conflicting binaries
sudo mkdir -p _disabled_libs

# 3. Safely move the outdated structures out of Resolve's direct fallback visibility
sudo mv libglib-2.0.so* _disabled_libs/
sudo mv libgio-2.0.so* _disabled_libs/
sudo mv libgmodule-2.0.so* _disabled_libs/
sudo mv libgobject-2.0.so* _disabled_libs/

 ls -l _disabled_libs
lrwxrwxrwx@    - root 19 Jul 12:57  libgio-2.0.so -> libgio-2.0.so.0
lrwxrwxrwx@    - root 19 Jul 12:57  libgio-2.0.so.0 -> libgio-2.0.so.0.6800.4
.rwxr-xr-x@ 2.5M root 19 Jul 12:57  libgio-2.0.so.0.6800.4
lrwxrwxrwx@    - root 19 Jul 12:57  libglib-2.0.so -> libglib-2.0.so.0
lrwxrwxrwx@    - root 19 Jul 12:57  libglib-2.0.so.0 -> libglib-2.0.so.0.6800.4
.rwxr-xr-x@ 1.8M root 19 Jul 12:57  libglib-2.0.so.0.6800.4
lrwxrwxrwx@    - root 19 Jul 12:57  libgmodule-2.0.so -> libgmodule-2.0.so.0
lrwxrwxrwx@    - root 19 Jul 12:57  libgmodule-2.0.so.0 -> libgmodule-2.0.so.0.6800.4
.rwxr-xr-x@  19k root 19 Jul 12:57  libgmodule-2.0.so.0.6800.4
lrwxrwxrwx@    - root 19 Jul 12:57  libgobject-2.0.so -> libgobject-2.0.so.0
lrwxrwxrwx@    - root 19 Jul 12:57  libgobject-2.0.so.0 -> libgobject-2.0.so.0.6800.4
.rwxr-xr-x@ 479k root 19 Jul 12:57  libgobject-2.0.so.0.6800.4

```
## Change Startup Env parameters


![](attachments/Pasted%20image%2020260719130218.png)

## Fix GPU for Nvidia

Go to preferences and select Memory and GPU make sure the settings are similar to below:

![](attachments/Pasted%20image%2020260719130355.png)
