# FAQ[](https://docs.bareos.org/Appendix/FAQ.html#faq)

## General Questions[](https://docs.bareos.org/Appendix/FAQ.html#general-questions)

### How do I report a security issue?[](https://docs.bareos.org/Appendix/FAQ.html#how-do-i-report-a-security-issue)

The process for handling security-related problems is described in our GitHub [security policy](https://github.com/bareos/bareos/security/policy).



## Upgrade of Bareos 19.2 with installed Python plugin on Debian[](https://docs.bareos.org/Appendix/FAQ.html#upgrade-of-bareos-19-2-with-installed-python-plugin-on-debian)

On Debian platform, if you update from Bareos <=19.2  to Bareos >= 20 with any Python plugin installed, you will face some  difficulties with **apt upgrade**. This is due to the renaming of Python plugin packages into python2  packages and introducing the python3 packages. apt/dpkg is not able to  handle this situation alone.

Error upgrading from 19.2 with apt upgrade[](https://docs.bareos.org/Appendix/FAQ.html#id1)

```
apt upgrade bareos bareos-director bareos-filedaemon
Reading package lists... Done
Building dependency tree
Reading state information... Done
Calculating upgrade... Done
Some packages could not be installed. This may mean that you have
requested an impossible situation or if you are using the unstable
distribution that some required packages have not yet been created
or been moved out of Incoming.
The following information may help to resolve the situation:

The following packages have unmet dependencies:
   bareos-director-python-plugin : Depends: bareos-common (= 19.2.12-2) but 21.1.2-1 is to be installed
   bareos-filedaemon-python-plugin : Depends: bareos-common (= 19.2.12-2) but 21.1.2-1 is to be installed
E: Broken packages
```

In this case, it is advised to use **apt** with full-upgrade and directly move to new recommended python3 plugin.

for full upgrade to python3[](https://docs.bareos.org/Appendix/FAQ.html#id2)

```
apt full-upgrade bareos-director-python3-plugin bareos-filedaemon-python3-plugin
```



## Bareos 18.2.5 FAQ[](https://docs.bareos.org/Appendix/FAQ.html#bareos-18-2-5-faq)

### What is the important feature introduced in Bareos 18.2?[](https://docs.bareos.org/Appendix/FAQ.html#what-is-the-important-feature-introduced-in-bareos-18-2)

1. A new network protocol was introduced where TLS is immediately used.

> - When no certificates are configured, the network connection will still be encrypted using TLS-PSK.
> - When certificates are configured, Bareos will configure both TLS-PSK and TLS with certificates at the same time, so that the TLS protocol will choose which one to use.

### How to update from Bareos 17.2?[](https://docs.bareos.org/Appendix/FAQ.html#how-to-update-from-bareos-17-2)

To update from Bareos 17.2, as always all core components need to be updated as they need to be of the same Bareos version (Bareos Console, Bareos Director, Bareos Storage Daemon).

### How can I see what encryption is being used?[](https://docs.bareos.org/Appendix/FAQ.html#how-can-i-see-what-encryption-is-being-used)

Whenever a connection is established, the used cipher is logged and will be shown in the job log and messages output:

console output[](https://docs.bareos.org/Appendix/FAQ.html#id3)

```
Connecting to Director localhost:9101
 Encryption: ECDHE-PSK-CHACHA20-POLY1305
```

job log[](https://docs.bareos.org/Appendix/FAQ.html#id4)

```
[...] JobId 1: Connected Storage daemon at bareos:9103, encryption: ECDHE-PSK-CHACHA20-POLY1305
```

### What should I do when I get “TLS negotiation failed”?[](https://docs.bareos.org/Appendix/FAQ.html#what-should-i-do-when-i-get-tls-negotiation-failed)

Bareos components use TLS-PSK as default. When the TLS negotiation fails then most likely identity or password do not match. Doublecheck the component name and password in the respective configuration to match each other.

### How does the compatibility with old clients work?[](https://docs.bareos.org/Appendix/FAQ.html#how-does-the-compatibility-with-old-clients-work)

The Bareos Director always connects to clients using the new immediate TLS protocol.  If that fails, it will fall back to the old protocol and try to connect again.

When the connection is successful, the director will store which protocol needs to be used with the client and use this protocol the next time this client will be connected.  Whenever the configuration is reloaded, the protocol information will be cleared and the probing will be done again when the next connection to this client is done.

probing the client protocol[](https://docs.bareos.org/Appendix/FAQ.html#id5)

```
[...] JobId 1: Probing... (result will be saved until config reload)
[...] JobId 1: Connected Client: bareos-fd at localhost:9102, encryption: ECDHE-PSK-CHACHA20-POLY1305
[...] JobId 1:    Handshake: Immediate TLS
```

### Does Bareos support TLS 1.3?[](https://docs.bareos.org/Appendix/FAQ.html#does-bareos-support-tls-1-3)

Yes. If Bareos is compiled with OpenSSL 1.1.1, it will automatically use TLS 1.3 where possible.

### Are old Bareos clients still working?[](https://docs.bareos.org/Appendix/FAQ.html#are-old-bareos-clients-still-working)

Bareos clients < 18.2 will still work, and the old protocol will be used. This was mostly tested with Bareos 17.2 clients.

### Can I use a new Bareos 18.2 client with my Bareos 17.2 system?[](https://docs.bareos.org/Appendix/FAQ.html#can-i-use-a-new-bareos-18-2-client-with-my-bareos-17-2-system)

Yes, it is possible to use a Bareos 18.2 client, but some changes need to be done in the configuration.

It is possible to use the Bareos 18.2 client with a Bareos 17.2 Server. However, the new immediate TLS Protocol and TLS-PSK are not usable, as the server components do not support it. This also means that it is **not** possible to use TLS with certificates in this setup. The communication will be unencrypted using the old protocol.

As in Bareos 18.2, the default value of **TLS Enable** was changed to **yes** to automatically use TLS-PSK, and the meaning of **TLS Require** also was altered so that it enforces the new protocol, these settings need to be changed.

In order to make Bareos 18.2 clients work with a Bareos 17.2 server, the following changes need to be done:

- **On all Bareos 18.2 clients**, the directive **TLS Enable** in the file `/etc/bareos/bareos-fd.d/director/bareos-dir.conf` needs to be set to **no**. If the directive **TLS Require** is set, it also needs to be set to **no** in the same file. This is enough for standard clients which do not have any special setup for the connections, and also for clients that are configured to use **client initiated connections**.
- For **clients that use the passive mode**, also the clients’ setting in the Bareos 17.2 director in file `/etc/bareos/bareos-dir.d/client/passive-fd.conf` needs to to be altered so that both directives **TLS Enable** and **TLS Require** are set to **no**.