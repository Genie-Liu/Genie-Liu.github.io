---
date: '2025-02-05T21:22:12+08:00'
draft: false
title: 'Tips for Your Privacy'
---

Here are some tips, based on my experience, for safeguarding your privacy

## Password Manager

Never reuse passwords. But how can you efficiently manage numerous passwords? The solution is a password manager.

I personally use Bitwarden, and here’s why:

- Open Source: This is crucial—open source ensures transparency, meaning there are no hidden secrets behind the scenes.
- Multiple Access Options: You can use Bitwarden through browser extensions or install it on your phone.
- Online Capability: While storing passwords online might seem less secure, it offers a balance between convenience and security. Moreover, Bitwarden allows you to self-host, adding an extra layer of control.
- Ease of Backup: I frequently export my passwords, encrypt them with Cryptomator, and store the encrypted files in my cloud storage for safekeeping.

## Two-Factor Authentication (2FA)

2FA or MFA (Multi-Factor Authentication) is essential to safeguard your accounts. Even if your password is compromised, 2FA adds a second layer of security, preventing unauthorized access.

It is especially critical to enable 2FA for your password manager. There have been cases where people suspected unauthorized access to their password manager, which could compromise all their accounts. This would be a nightmare scenario—so make sure you enable 2FA for your most sensitive accounts.

I use Bitwarden Authenticator for the same reasons I trust Bitwarden as a password manager. I also export my 2FA secrets and back them up securely with encryption. Avoid using SMS for 2FA since it’s not encrypted and can be vulnerable to attacks. By the way if your website provide passkey, just use it.

## Browser & Search Engine

For browsers, I recommend LibreWolf. It offers excellent privacy protection out of the box, eliminating the need for extensive configuration. It’s comparable to a hardened Firefox setup like Arkenfox.  You can check out [my user experience with LibreWolf](https://blog.csdn.net/Mophistoliu/article/details/141475605)

As for search engines, I set DuckDuckGo as the default.

If you encounter website compatibility issues with LibreWolf, Brave serves as a reliable backup browser.

## End-to-End Encrypted Messengers

Messaging apps often handle sensitive and personal data. At a minimum, you should use an app that offers end-to-end encryption (E2EE).

However, the choice of app doesn’t depend solely on you—it also depends on those you communicate with. I personally recommend Signal for its strong privacy features.

Since most E2EE apps are blocked, iMessage could be a last resort for Apple users.

## Encrypting Cloud Storage

Most cloud storage providers do not guarantee privacy protection. To secure sensitive data, encrypt it before uploading. I use Cryptomator for this purpose, ensuring my private data remains safe. You can read more about [my experience using cryptomator](https://zhuanlan.zhihu.com/p/713148956)

## Email Encryption with GPG

Emails are inherently unencrypted, meaning your messages are accessible to your provider. Setting up your own email server can be labor-intensive, but encryption is an easier alternative.

For instance, GPG encryption can secure your emails, much like how Snowden communicated with journalists. I use GPG to generate encryption keys and import them into Thunderbird, which automates the encryption process. I remember Thunderbird also provide a OpenPGP Key Manager, you can just use it.

Keep in mind that both parties need to use encryption for it to work. Educate your contacts about its simplicity and importance.

## Privacy Setting

Develop the habit of reviewing privacy settings whenever you use a new app. Even apps renowned for protecting privacy often have settings that need to be configured for optimal security.
