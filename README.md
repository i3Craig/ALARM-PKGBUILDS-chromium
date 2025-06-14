# ALARM-PKGBUILDS-chromium
PKGBUILD customized for building chromium on Arch Linux ARM (AARCH64).
This is designed to be used on Arch Linux ARM to build chromium, as no new builds have been provided for chromium in some time.
Note that to build this package, more than 4GB of RAM is recommended. But, 8GB allow the compile to run faster, as more files can be cached in RAM.
This build will take about 24 hours on an RK3588 CPU.


If you prefer not to use some "raondom" PKGBUILD you found on github, you can apply the diff below to the official Arch Linux `chromium` PKGBUILD.
Save the diff to a file (for example `patch.diff`), then run the following in the directory with the PKGBUILD: `patch -Np1 <patch.diff`
```diff
diff --git a/PKGBUILD b/PKGBUILD
index 1b6b1ba..e4ca81b 100644
--- a/PKGBUILD
+++ b/PKGBUILD
@@ -10,7 +10,7 @@ _launcher_ver=8
 _manual_clone=0
 _system_clang=1
 pkgdesc="A web browser built for speed, simplicity, and security"
-arch=('x86_64')
+arch=('aarch64')
 url="https://www.chromium.org/Home"
 license=('BSD-3-Clause')
 depends=('gtk3' 'nss' 'alsa-lib' 'xdg-utils' 'libxss' 'libcups' 'libgcrypt'
@@ -33,6 +33,7 @@ source=(https://commondatastorage.googleapis.com/chromium-browser-official/chrom
         chromium-136-drop-nodejs-ver-check.patch
         compiler-rt-adjust-paths.patch
         increase-fortify-level.patch
+        disable-clang-warning-suppression-flag.patch
         use-oauth2-client-switches-as-default.patch)
 sha256sums=('f5f051a30c732b21ce9957cdd7fe0a083623e19078a15ee20d49b27a5cb857e6'
             '213e50f48b67feb4441078d50b0fd431df34323be15be97c55302d3fdac4483a'
@@ -41,6 +42,7 @@ sha256sums=('f5f051a30c732b21ce9957cdd7fe0a083623e19078a15ee20d49b27a5cb857e6'
             '32f0080282fc0b2795a342bf17fcb3db4028c5d02619c7e304222230ba99d5fe'
             'cc8a71a312e9314743c289b7b8fddcc80350a31445d335f726bb2e68edf916d1'
             'd634d2ce1fc63da7ac41f432b1e84c59b7cceabf19d510848a7cff40c8025342'
+            'd6f3914c6adadaf061e7e2b1430c96d32b0cad05244b5cfaf58cf5344006a169'
             'e6da901e4d0860058dc2f90c6bbcdc38a0cf4b0a69122000f62204f24fa7e374')
 
 if (( _manual_clone )); then
@@ -97,6 +99,12 @@ prepare() {
   sed -i 's/OFFICIAL_BUILD/GOOGLE_CHROME_BUILD/' \
     tools/generate_shim_headers/generate_shim_headers.py
 
+  # ALARM patches.
+  CFLAGS="${CFLAGS/_FORTIFY_SOURCE=3/_FORTIFY_SOURCE=2}" && CXXFLAGS="$CFLAGS"
+
+  # Allow build to set march and options on AArch64 (crc, crypto)
+  [[ $CARCH == "aarch64" ]] && CFLAGS=`echo $CFLAGS | sed -e 's/-march=armv8-a//'` && CXXFLAGS="$CFLAGS"
+
   # https://crbug.com/893950
   sed -i -e 's/\<xmlMalloc\>/malloc/' -e 's/\<xmlFree\>/free/' \
          -e '1i #include <cstdlib>' \
@@ -122,6 +130,9 @@ prepare() {
   # Increase _FORTIFY_SOURCE level to match Arch's default flags
   patch -Np1 -i ../increase-fortify-level.patch
 
+  # Disable usage of --warning-suppression-mappings flag which needs clang 20
+  patch -Np1 -i ../disable-clang-warning-suppression-flag.patch
+
   # Fixes for building with libstdc++ instead of libc++
 
   # Link to system tools required by the build

```
