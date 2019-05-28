# Yubkey macOS Setup

The first *problem* I encountered was putting the Yubikey in the wrong way -- it needs to be facing a certain way in order to be identified by the laptop.

* [Login 2FA with Yubikey](#2FA)
* [Github Commit Signing](#git)

## Mac Login Configuration <a name = "2FA"></a>

The macOS Login Tool enables two-factor authentication using HMAC-SHA1 challenge-response feature of the Yubikey.

1. Download and install the [macOS Logon Tool](https://www.yubico.com/products/services-software/download/computer-logon-tools/)

2. Download and install the [Yubikey Manager](https://www.yubico.com/products/services-software/download/yubikey-manager/)

3. Configure Yubikey with Yubikey Manager

* open Yubikey Manager
* insert YubiKey to an available USB port on the Mac (make sure that it is facing the right way -- this is an easy mistake)
* click `Applications`, then `OTP`
* select `challenge-response` and click `next`
* click `generate` to generate a new secret
* (optional) click the `require touch` option to require a metal touch to the metal contact on the Yubikey to apprve challenge-response actions
* click `finish`

4. Associate Yubikeys with Account

* open terminal
* insert Yubikey
* run `ykpamcfg -2` (touch the Yubikey if you clicked `require touch` in step (3))

5. Test Configuration

* Open `System Preferences`
* Click `Security & Privacy`
* Click on the `General` tab
* Check the `Require password` box and set to `immediately`

Now we can configure 2FA for the screensaver.

* open `Terminal`
* run `sudo nano /etc/pam.d/screensaver`
* when prompted, type password and press `Enter`
* add the line below above the `account required pam_opendirectory.so` line:

```bash
auth required /usr/local/lib/security/pam_yubico.so mode=challenge-response
```

*Make sure that the actual location corresponse to the challenge-response key configured in Step 3*

Press `Ctrl+x`, `Y`, and then `Enter` to save the file.

Press `Command+Ctrl+Q` to lock the Mac -- you should only be able to login with the Yubikey inserted and the password provided.

6. Enable Configuration

* open Terminal
* run `sudo nano /etc/pam.d/authorization`
* when prompted, type the password and press `Enter`
* add the line below the `account required pam_opendirectory.so` line,

```bash
auth required /usr/local/lib/security/pam_yubico.so mode=challenge-response
```

* press `Ctrl+X`, `Y`, and then `Enter` to save the file


*Read this official guide for trouble shooting and more information: [macOS Logon Tool Configuration Guide](https://support.yubico.com/support/solutions/articles/15000015045-macos-logon-tool-configuration-guide)*

## Signing Git Commits <a name = "git"></a>

1. Download and install [the GPG command line tools](https://help.github.com/en/articles/generating-a-new-gpg-key) for your OS.

*I keep getting stuck with generating encryption keys for this; will give it another go again soon...in the mean time, here are a few links that might help someone else figure it out before me...*

To generate and import keys (make sure both keys are 2048 bit keys):
* [Importing Keys](https://developers.yubico.com/PGP/Importing_keys.html)
* [Generating Keys](https://help.github.com/en/articles/generating-a-new-gpg-key)

*Some blog post that makes it seem easy...*
* [Yubikey Signed Commits](https://www.engineerbetter.com/blog/yubikey-signed-commits/)

*Read this official guide for [git commit signing with Yubikey](https://developers.yubico.com/PGP/Git_signing.html)*

# Credits

Thanks to the following people for instructions:

- Yubico's own documentation (referenced inline in the instructions where used)
- The [original version of this doc](https://github.com/liyanchang/yubikey-setup) by [David Chiang](https://github.com/liyanchang)
