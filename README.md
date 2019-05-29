# Yubkey macOS Setup

*You bought a YubiKey - now what?*

The goal is to outline the steps to configure your YubiKey in a sane method
and to use it to maximize your security.

This guide is for users who are comfortable with the command line and various
technical jargon.

This is highly opinionated on how you should and should not use your YubiKey
but is organized well enough that you should be able to modify if you have a
need.

The instructions have been tested on macOS 10.14.5 (Mojave) with a
YubiKey Neo. While there are sections that are OS independent, most of the
tricky bits are macOS specific.

To perform these instructions, the YubiKey should be plugged into your
computer's USB port.

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
*this section heavily references [this page](https://developers.yubico.com/PGP/)*

1. Install [GnuPG](https://www.gnupg.org/) version 2.022 or later; there are a few other privacy tool installations of gpg documented [here](https://help.github.com/en/articles/generating-a-new-gpg-key), but use with Yubikey requires GnuPG specifically.

2. Check that the version of the Yubikey's OpenGPG module is 1.05 or later. To check this, run the following command in the Terminal after inserting your Yubikey:
```bash
$ gpg-connect-agent --hex "scd apdu 00 f1 00 00" /bye
D[0000]  01 00 05 90 00     
OK
```

If you have an existing key you want to import, the key must be a RSA 2048 bit key. You'll also need the admin PIN for the Yubikey.

> **NOTE**: the default PIN set is `123456` and the default admin PIN is `12345678` -- follow [directions here](https://developers.yubico.com/PGP/Card_edit.html) to change it.

3. Genersate a RSA 2048 bit key. Paste the following in the terminal after following step (1) above
```bash
$ gpg --full-generate-key

gpg (GnuPG) 2.0.22; Copyright (C) 2013 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
Your selection?
```
Select option `(1)` and choose the default of 2048 bits.

```bash
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048)

Requested keysize is 2048 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0)
```
Select an expiry date if you desire -- answer the next few questions truthfully.

```bash
Real name:
```
The actual name associated with this key.
```bash
Email address:
```
The email address associated with the key. You can leave the `Comment` empty or add a comment if you want. Select `O` for okay when prompted.

```bash
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
gpg: key 13AFCE85 marked as ultimately trusted
public and secret key created and signed.

gpg: checking the trustdb
gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
gpg: depth: 0  valid:   4  signed:   8  trust: 0-, 0q, 0n, 0m, 0f, 4u
gpg: depth: 1  valid:   8  signed:   2  trust: 3-, 0q, 0n, 5m, 0f, 0u
gpg: next trustdb check due at 2014-03-23
pub   2048R/13AFCE85 2014-03-07 [expires: 2014-06-15]
      Key fingerprint = 743A 2D58 688A 9E9E B4FC  493F 70D1 D7A8 13AF CE85
uid                  Foo Bar <foo@example.com>
sub   2048R/D7421CDF 2014-03-07 [expires: 2014-06-15]
```
Take note of the key's id -- `13AFCE85`

4. Add an authentication key. Enter the following command in the terminal:
```bash
$ gpg --expert --edit-key 13AFCE85

gpg (GnuPG) 2.0.22; Copyright (C) 2013 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Secret key is available.

pub  2048R/13AFCE85  created: 2014-03-07  expires: 2014-06-15  usage: SC
                     trust: ultimate      validity: ultimate
sub  2048R/D7421CDF  created: 2014-03-07  expires: 2014-06-15  usage: E
[ultimate] (1). Foo Bar <foo@example.com>

gpg> addkey

2048-bit RSA key, ID 13AFCE85, created 2014-03-07

Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) Elgamal (encrypt only)
   (6) RSA (encrypt only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
Your selection?
```
Select `8` to get another RSA key attached to the key generated in step (3).

```bash
Possible actions for a RSA key: Sign Encrypt Authenticate
Current allowed actions: Sign Encrypt

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection?
```
Select `A`, then `S`, then `E` to get a pure authentication key. Then enter `Q` to continue.
```bash
What keysize do you want? (2048)
```
Choose the 2048 bit key size again. Then choose the same expiry as the generated key and answer `y` if everything is correct.
```bash
Is this correct? (y/N) y
Really create? (y/N) y
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.

pub  2048R/13AFCE85  created: 2014-03-07  expires: 2014-06-15  usage: SC
                     trust: ultimate      validity: ultimate
sub  2048R/D7421CDF  created: 2014-03-07  expires: 2014-06-15  usage: E
sub  2048R/B4000C55  created: 2014-03-07  expires: 2014-06-15  usage: A
[ultimate] (1). Foo Bar <foo@example.com>

gpg> Save changes? (y/N) y
```

5. (OPTIONAL) You can create a backup of the key and store it in a secure place
```bash
$ gpg --export-secret-key --armor 13AFCE85
```

6. Import the key into the Yubikey.
```bash
$ gpg --edit-key 13AFCE85

gpg (GnuPG) 2.0.22; Copyright (C) 2013 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Secret key is available.

pub  2048R/13AFCE85  created: 2014-03-07  expires: 2014-06-15  usage: SC
                     trust: ultimate      validity: ultimate
sub  2048R/D7421CDF  created: 2014-03-07  expires: 2014-06-15  usage: E
sub  2048R/B4000C55  created: 2014-03-07  expires: 2014-06-15  usage: A
[ultimate] (1). Foo Bar <foo@example.com>

gpg> toggle

sec  2048R/13AFCE85  created: 2014-03-07  expires: 2014-06-15
ssb  2048R/D7421CDF  created: 2014-03-07  expires: never
ssb  2048R/B4000C55  created: 2014-03-07  expires: never
(1)  Foo Bar <foo@example.com>

gpg> keytocard
Really move the primary key? (y/N) y
Signature key ....: [none]
Encryption key....: [none]
Authentication key: [none]

Please select where to store the key:
   (1) Signature key
   (3) Authentication key
Your selection? 1
```
Here we've moved the primary key to the PGP Signature slot of the Yubikey. Next move the Encryption key:
```bash
gpg> key 1

sec  2048R/13AFCE85  created: 2014-03-07  expires: 2014-06-15
                     card-no: 0000 00000001
ssb* 2048R/D7421CDF  created: 2014-03-07  expires: never
ssb  2048R/B4000C55  created: 2014-03-07  expires: never
(1)  Foo Bar <foo@example.com>

gpg> keytocard
Signature key ....: 743A 2D58 688A 9E9E B4FC  493F 70D1 D7A8 13AF CE85
Encryption key....: [none]
Authentication key: [none]

Please select where to store the key:
   (2) Encryption key
Your selection? 2
```
Next, move the authentication key to the Yubikey:
```bash
gpg> key 1

sec  2048R/13AFCE85  created: 2014-03-07  expires: 2014-06-15
                     card-no: 0000 00000001
ssb  2048R/D7421CDF  created: 2014-03-07  expires: never
                     card-no: 0000 00000001
ssb  2048R/B4000C55  created: 2014-03-07  expires: never
(1)  Foo Bar <foo@example.com>

gpg> key 2

sec  2048R/13AFCE85  created: 2014-03-07  expires: 2014-06-15
                     card-no: 0000 00000001
ssb  2048R/D7421CDF  created: 2014-03-07  expires: never
                     card-no: 0000 00000001
ssb* 2048R/B4000C55  created: 2014-03-07  expires: never
(1)  Foo Bar <foo@example.com>

gpg> keytocard
Signature key ....: 743A 2D58 688A 9E9E B4FC  493F 70D1 D7A8 13AF CE85
Encryption key....: 8D17 89A0 5C2F B804 22E5  5C04 8A68 9CC0 D742 1CDF
Authentication key: [none]

Please select where to store the key:
   (3) Authentication key
Your selection? 3
```

After saving the keyring, your computer no longer contains the real secret key -- only a pointer indicating it's stored on the Yubikey smart card.

*If you have any trouble, refer to this [this official guide](https://developers.yubico.com/PGP/Importing_keys.html) for generating and importing the 2048 bit RSA key into the Yubikey*

7. Set the global signing key for git such that AABBCCDD is the GPG key ID (in our example, `13AFCE85`)
```bash
$ git config --global user.signingkey AABBCCDD
```

*Read more in the official Yubico guide for [git commit signing with Yubikey](https://developers.yubico.com/PGP/Git_signing.html)*

8. Add the GPG key ID to your github account by following the github guides [here](https://help.github.com/en/articles/generating-a-new-gpg-key) and, then, [here](https://help.github.com/en/articles/adding-a-new-gpg-key-to-your-github-account)

# Credits

Thanks to the following for instructions:

- Yubico's own documentation (referenced where used)
- The [original version of this doc](https://github.com/liyanchang/yubikey-setup) by [David Chiang](https://github.com/liyanchang)
