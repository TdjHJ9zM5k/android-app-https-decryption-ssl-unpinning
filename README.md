# Mobile App HTTPS Traffic Decryption via SSL/TLS Interception, Certificate Pinning Bypass, and System CA Injection

## Disclaimer
This guide is intended exclusively for lawful research, debugging, and development activities. The techniques described here must only be applied to applications owned by the user or those with explicit, documented permission for testing. All information is to be used strictly in non-production environments. Misuse of the material may breach laws or regulations, for which the user assumes full responsibility. The author disclaims liability for unauthorized or unethical application of these methods.

---

## Table of Contents
1. [Required Tools](#required-tools)  
2. [Step 1: Configuring Android Studio](#step-1-configuring-android-studio)  
3. [Step 2: Rooting the Emulator](#step-2-rooting-the-emulator)  
4. [Step 3: Configuring Charles Proxy](#step-3-configuring-charles-proxy)  
5. [Step 4: Trusting User Certificate](#step-4-trusting-user-certificate)  
6. [Step 5: Decrypted HTTPS Traffic](#step-5-decrypted-https-traffic)  
7. [Extra: SSL Unpinning](#extra-ssl-unpinning) 

---

## Required Tools
These tools are essential for completing the steps in this guide:  
- [Android Studio](https://developer.android.com/studio) – Integrated development environment for Android development.  
- [Google Platform Tools](https://developer.android.com/tools/releases/platform-tools) – Command-line tools for interfacing with Android devices.  
- [Charles Proxy](https://www.charlesproxy.com/) – A web debugging proxy application.  
- [rootAVD](https://github.com/newbit1/rootAVD) – Tool for rooting Android Virtual Devices.  
- [MagiskTrustUserCerts](https://github.com/NVISOsecurity/MagiskTrustUserCerts) – A Magisk module to trust user-installed certificates.  

Optional tools for advanced scenarios:  
- [android-unpinner](https://github.com/mitmproxy/android-unpinner) – For bypassing SSL pinning in apps.  
- [JustTrustMe](https://github.com/Fuzion24/JustTrustMe) – Xposed module for trusting user-installed certificates by default.  

---

## Step 1: Configuring Android Studio

### Setup Android Studio
1. Download and install [Android Studio](https://developer.android.com/studio).  
2. Launch the application and go to `Settings > Tools > Emulator`.  
3. Disable **"Launch in the Running Device tool window."**  

### Create a Virtual Device
1. Open the `Device Manager` tab.  
2. Configure and create a virtual device with the following specifications:  
   - Device: Pixel 7  
   - API Level: SDK 30  
   - Architecture: x86_64  
   - Google API version  
3. Launch the virtual device.  

---

## Step 2: Rooting the Emulator

### Prepare the Emulator for Rooting
1. Navigate to the `rootAVD` folder in your terminal.  
2. List all available AVDs:  
   ```sh
   ./rootAVD.sh ListAllAVDs
   ```  
3. Identify the corresponding command for your virtual device.  

### Root the Emulator
1. With the emulated device running, root the device with the identified command:  
   ```sh
   ./rootAVD.sh system-images/android-30/google_apis/x86_64/ramdisk.img
   ```  
2. Reboot the emulator. The Magisk app will automatically be installed.  

### Verify Root Access
1. Open the Magisk app on the emulator.  
2. Confirm that the device is rooted.  

---

## Step 3: Configuring Charles Proxy

### Set Up Charles Proxy
1. Download and install [Charles Proxy](https://www.charlesproxy.com/).  
2. Configure the port:  
   - Navigate to `Charles > Proxy > Proxy Settings`.  
   - Set the port to `8888`.  

### Configure Proxy on Emulator
1. On the emulator, go to `Settings > Advanced > Proxy`.  
2. Enter the local IP of your machine and port `8888`.  

### Install the Charles Certificate
1. Visit [http://chls.pro/ssl](http://chls.pro/ssl) on the emulator.  
2. Download and install the `.crt` certificate.  
   - For Android 11+, install the certificate by navigating to `Settings > Trusted credentials`.  
3. Navigate to `Settings > CA Certificate` and confirm the certificate is installed under the *User* tab.  

### Troubleshooting
- If the certificate download fails, set the proxy in Wi-Fi settings, enable airplane mode, and disable it again.  

---

## Step 4: Trusting User Certificate

### Install the Magisk Module
1. Copy the `AlwaysTrustUserCerts.zip` file to the emulator's internal storage:  
   ```sh
   adb push AlwaysTrustUserCerts.zip /sdcard/
   ```  
2. Open the Magisk app and go to `Modules > Install from storage`.  
3. Select the `.zip` file and install it.  
4. Reboot the emulator.  

### Verify System Trusted Credentials
1. Check that the Charles certificate is now listed under `Settings > CA Certificate` in the *System* tab.  

---

## Step 5: Decrypted HTTPS Traffic

### Intercept Traffic
1. Re-enable SSL proxying in Charles Proxy:  
   - Navigate to `Proxy > Start SSL Proxying`.  
2. Open the target app on the emulator.  
3. Observe decrypted HTTPS requests in the Charles interface.  

---

## Extra: SSL Unpinning

### For Android SDK 30+
The *AlwaysTrustUserCerts* Magisk module is compatible with Android up to SDK 30. With more recent Android versions, certificates cannot be installed as system-level certificates.  

Furthermore, some apps may implement SSL pinning, accepting only specific certificate fingerprints. To bypass this, you can perform SSL unpinning by modifying the APK to remove certificate pinning.  

### Using android-unpinner
1. Clone the android-unpinner repository:  
   ```sh
   git clone https://github.com/mitmproxy/android-unpinner.git
   ```  
2. Install dependencies and run the tool:  
   ```sh
   android-unpinner all {path-to-my-apk.apk}
   ```  
3. The modified APK will automatically be installed on the emulator.  

---
