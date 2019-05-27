# Yubkey macOS Setup

The first *problem* I encountered was putting the Yubikey in the wrong way -- it needs to be facing a certain way in order to be identified by the laptop.

## Logon Configuration

I've been working through [this link](https://support.yubico.com/support/solutions/articles/15000015045-macos-logon-tool-configuration-guide), but I am currently stuck; the login never asks for the Yubikey despite changing the command line-based configuration.

*Misc Notes*
* replace `users/username` with your `username`

## Git Signing

*Stuck with encryption key generation for [this tutorial](https://developers.yubico.com/PGP/Git_signing.html)*

# Credits

Thanks to the following people for instructions:

- Yubico's own documentation (referenced inline in the instructions where used)
- The [original version of this doc](https://github.com/liyanchang/yubikey-setup) by [David Chiang](https://github.com/liyanchang)
