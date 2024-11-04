# 0x00 - Disclaimers
I'm not a certified expert, these are just some stuff that i gathered with my ~20 years of fooling around, and i'm writing this on good faith and i hope that it will be helpful to someone.

# 0x01 - Security x Privacy​

While those are two different concepts, they overlap, some takes to the extreme of and make themselves a "ghost", a little bit of privacy can really help your security, but privacy is really not the focus of this post.
With that in mind we need to define our goal, i don't mind sharing mine and i think yours might be somewhat similar; my ultimate goal is to preserve my well being, which includes making a potential threat actor being unable to have a meaningful negative impact on my quality of life or health; that is a quite vague and they overlap with physical security.

# 0x02 - Security through obscurity​

This concept is the major pitfall that we often fall, but this is often exaggerated, storing your passwords in a safe manner does not constitute security through obscurity, but changing your SSH server listen port could be considered a security through obscurity method, since by itself it doesn't really make things more secure.

# 0x03 - Know your adversary​

Who are you trying to protect against? that is the question that you need to think about, because that is imperative to know what measures would be sufficient, it is easier to deal with a script-kiddie watching youtube videos on how to use kali linux tools than dealing with intelligence agencies and israeli spyware development companies.

# 0x04 - Preparing a good foundation for security​

Ever heard of "your security is as good as the weakest link", if your foundation is Windows 10 or 11, then you are up to a bad start, the magic of FOSS is the capability of auditing, but i will tell you a secret so please don't tell anyone alright? i don't read every single line of code that gets compiled and executed on my system.
So it is really important for you to have an operating system that runs software from a source that you trust, personally, i trust the openSUSE team, also the Debian team, so openSUSE and Debian are often my pick for operating systems, but beware that is just an example, other 'mainstream' distros have a good record of security, but security incidents can happen, which was the case [when Linux Mint got attacked](https://arstechnica.com/information-technology/2016/02/linux-mint-hit-by-malware-infection-on-its-website-and-forum-after-hack-attack/) , [or that time that Debian servers got hacked](https://www.linuxinsider.com/story/hackers-compromise-debian-linux-project-servers-58264.html), or even [this incident with Red Hat](https://www.zdnet.com/article/red-hats-ceph-and-inktank-code-repositories-were-cracked/).

# 0x05 - Password Strength and 2 Factor Authentication​

Please don't use obvious passwords like names, birthdays, or any default passwords like your router default web administration credentials.
Also storing them safely is advised, my recommendation is to have a strong password memorized as a master password to a password manager like KeePassXC or BitWarden.
Beware that commercial services that promise to store your password are dangerous, if they get attacked, it can be catastrophic for you.
If you need to use your passwords on multiple devices you can store your KeePassXC database on a flash drive(if you want to be extra secure you can even use full disk encryption on that drive)
Also as a good measure always have a updated backup of that database.
Another important aspect is enabling 2FA whenever possible, even with hardware keys as a Yubi Key, But Google Authenticator App can be useful.

# 0x06 - Hashes and Signatures, they are one of your best friends​

Let's assume that you don't trust your operating system anymore and thinks that it may be tampered because you didn't checked its hashes and signatures and you want a fresh start, so you download a new iso image, and flash it into your usb drive and you're ready to boot the installer, but slow down there buckaroo, we need to verify that what you downloaded is what you actually intended to download and wasn't tampered with, usually distros give specific instructions on how to verify your download for example on [Linux Mint's install instruction guide](https://linuxmint-installation-guide.readthedocs.io/en/latest/verify.html) there is a really well explained section on how to verify, Debian also provides a [guide on how to verify your ISO](https://www.debian.org/CD/verify), same can be said for [arch linux on it's wiki](https://wiki.archlinux.org/title/Installation_guide); so i guess you get the point right now.
Now that you're all verified and set, let's install the system and that's where the next point is important.

# 0x07 - Secure Boot​

Some distros makes deployment on platforms with Secure Boot enabled really easy, from my daily experience, Debian and openSUSE are examples that makes installing on a secureboot enabled system a straight forward and requires no user intervention, arch linux on the other hand will [require user intervention](https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface/Secure_Boot).
Secure boot allows only signed bootloaders, kernels and kernel modules to be executed, which can be really handy to avoid tampering on your system.
While secure boot has it's flaws, the upsides are bigger than the downsides, and some issues can be mitigated with replacing the Platform Keys, but that poses a risk of bricking your system, so i won't go deep on this subject.
Also be sure to set a password to access your UEFI Firmware configuration utility, otherwise someone can disable Secure Boot you will not notice, rendering you vulnerable.

# 0x08 - Full Disk Encryption(FDE)​

This might seem unnecessary, but trust me on this one, it is really important, that's where the cyber meets physical security, most easy to install distros at some time during the installation will ask you if you want to enable full encryption, even arch with it's install utility called arch install, openSUSE even goes a step further and allows you to encrypt your boot partition, however manually setting up LUKS is pretty universal.
If you don't feel like entering a long password a lot of times you can use a TPM module with some additional setup.
Beware that the biggest protection from your data being accessed that FDE provides is when your system is powered down, so if your going to leave your device unnatended for a while, it is nice to turn it off if you're in place that your physical security can't guarantee that your device will not be accessible to someone else.

# 0x09 - Firewalling​

Cool name huh? while by default no mainstream linux distribution comes with an application firewall, having one network firewall enabled can be really nice, some frontends like ufw and gufw makes it really simple.
Make sure that your incoming connection policy is set to DROP everything not allowed by a rule.
Firewalls are such a versatile tools that you can do so many cool stuff, you can even go the extra paranoid mode and make your default policy of outgoing connections to be dropped and only make rules allowing the critical ports like TCP 80(HTTP), TCP 443(HTTPS), UDP 53(DNS), but beware of the implications of your rules, some applications rely on some other ports.
The golden rule is to know what you gonna need.

# 0x0A - Securing your Web Browser​

This is really important, websites can be seen not only as content delivery mechanisms, but also as code delivery mechanisms, it is really important for you to keep your web browser updated and really well configured.
Personally i use Mozilla Firefox and have my profile and settings well backed up, but if you're starting from scratch [FFProfile](https://ffprofile.com/) can be really useful to assist you creating your own profile.
Some addons that i really recommend are uBlock Origin, Decentraleyes, ClearURL, CanvasBlocker(on it's stealthy preset) and Privacy Badger.
Be sure to configure your browser to use HTTPS only(or even the addon HTTPS Everywhere, you may get some warnings on websites that doesn't implemented SSL but it is great to warn you when you're stepping on some potentially insecure website, so avoid inserting any personal data)
You may or may not want to use a User Agent Switcher, but it is important to know it's limitations, and beware that even with a custom user agent, websites can identify the true core of your browser.
A good place to test is [DeviceInfo website](deviceinfo.me).
If you're into more tweaking and more granular control you may want to check out the addon NoScript.
Another thing to consider is that is a specific spot that privacy can conflict with security, these addons will aid you with security, but because it deviates from the configuration and addons used by the bulk of internet users, it will make your browser stand out.
Also it is important to mention that other browsers like Chromium, Ungoogled Chromium, Brave and Vivaldi exists and they can also be setup for improved security.
Short update: Most Chromium based browsers are not suffering from the deprecation of manivest V2, and manifest V3 breaks the most powerful adblockers, try using Brave if you need a a chromium based browser, or stick to Firefox.

# 0x0B - X11/Xorg vs Wayland​

This one will probably spark some discussions, Wayland being the new kid in the school can cause some issues, but it's protocol have way more security considerations in mind.
Since X has been around for a good while, most applications supports it; but on the other hand on X it is easy for some malicious application to monitor your screen, your keyboard input and mouse position.
With NVIDIA GPU Wayland can be a bit iffy(but it's getting better) but that's pretty much it, on Wayland you can use XWayland to still use applications made for X.
So if you can it is better to be in Wayland session.

# 0x0C - Sandboxing​

Following that rule that anything should have access only to stuff that is needed, sandboxing an application can help to achieve this goal, there are two famous alternatives for sandboxing bubblewrap(used in Flatpak) and Firejail, Firejail is really nice, comes with built-in profiles, but you still need to do some manual configuration, so you might feel more appealing to use flatpaks and customize the privileges with flatseal, you can even have some flatpaks, and use firejail for non flatpak applications.
Remember when i said on the firewalling section that mainstream linux distributions doesn't come with an application firewall? with sandboxing you can mitigate this issue, you can deny network access to certain applications.
Just beware of the restrictions of the sandboxes, they might get in the way and makes it look like a bug.

# 0x0D - Mandatory Access Control(SELinux/AppArmor)​

What if a trusted application get exploited and an attacker gets to inject arbitrary code into it?
M.A.C can help mitigate the damage, by limiting what the application has access to, so check if your system has these set and are being enforced.
Since a misconfigured M.A.C system can cause all kinds of problems i strongly advise a good reading and understanding beforing enforcing it.
[Arch wiki](https://wiki.archlinux.org/title/AppArmor) and [Debian wiki](https://wiki.debian.org/AppArmor) both have nice articles on AppArmor, and [RedHat goes into details on M.A.C and SELinux](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/using_selinux/getting-started-with-selinux_using-selinux), [openSUSE also have a nice article on setting up SELinux](https://en.opensuse.org/SDB:SELinux), just beware that [SELinux is available for Arch Linux](https://wiki.archlinux.org/title/SELinux) but it is not officially supported.

# 0x0E - Tweaking sysctl.conf ​

Some settings in the kernel can be tuned for a little bit more protection, but they may interfere with some functionality.
Personally i use just some man in the middle prevention and disable stuff that i don't need. a bit more can be read on the following links

- https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/kernel_administration_guide/working_with_sysctl_and_kernel_tunables
- https://www.cyberciti.biz/faq/linux-kernel-etcsysctl-conf-security-hardening/ 
- https://geektnt.com/sysctl-conf-hardening.html
- https://linoxide.com/how-tos/linux-server-protection/
- https://github.com/klaver/sysctl/blob/master/sysctl.conf

# 0x0F - Common sense/Software sources​

On my experience this is the major source of issues, not only regarding security.
Please avoid as much as possible using any third party repositories, and when you have to use it, for example openSUSE doesn't ship with proprietary multimedia codecs and you need to use packman repository for those, but this is kind of an exception to the rule because it is a trustworthy project that really improves openSUSE usability, another example of trustworthy 3rd party source is Wine repository, altough being able to run Windows executables, also opens the door for running Windows malware, and by default Wine expose a good part of your system that will allow data exfiltration.
Unless you know what you're doing refrain from running random binaries or AppImages acquired from untrusted sources, the same applies for scripts, being them shell scripts, perl scripts, python scripts.
python's Pypi, and Node.js npm packages are often targetted and claimed some victims, so be careful!
Also avoid piping stuff to bash.

# 0x10 - Patch Patch Patch​

Keep your system updated, this will not help much with 0day exploits but some of those are responsably disclosed to the developers before getting public, so perhaps one update could bring you a patch to an exploit disclosed to the developers that will be made public tomorrow.
This can be a bit trickier on servers, but for your personal computer that shouldn't bring too many issues, unless you're using Arch linux, in that case before updating give the forums and their news a good read before updating.

# 0x11 - (Encrypted) Backups​

This is really important, while some filesystems like btrfs may allow you to have snapshots and certain RAID levels can tolerate some hardware fault, those are not substitutes for real backups, if you have a NAS, it is really convenient to have a backup there, but you should also consider to have a totally offline backup, at home it is really nice to have an external HDD, but having the equipment for backups are as good as your backup schedule.
Some people might be happy with daily backups, some people with weekly backups and some people with monthly backups.
For your really critical stuff, you may consider compressing them on a tarball and them using gpg to encrypt and keep a copy of somewhere else, like your own VPS or even those mainstream cloud storage providers.
Just don't upload your private keys there

# 0x12 - Know the limitations of Proxies and "VPN" and beware of TOR​

While there's plenty of youtubers shilling VPN and VPN companies with ads that promises you a secret tunnel that will make you anonymous and super secure, in reality those are mostly marketing exaggerations.
Sure a tunnel will make your ISP know that you're only connecting to the tunnel, and the destination will know that something is coming from the tunnel, the VPN provider knows your traffic and they can collect logs(including those that claims that they dont collect logs by default, but usually a subpoena is enough to make them start collecting logs on someone and delivering to the justice), and your payments.
"Free" VPN are sketchy, if you're lucky your bandwidth will be limited, and you will have a data cap that the VPN provider will use to entice you to use their paid version, if you're not lucky well they can be spying on you, or even worse, they will be using your computer as a residential proxy.
Now TOR is really nice, but you have to be careful, i will not go too much into details, but it is important to know that all Entry and Exit nodes of Tor are publicly known, and correlation attacks are really popular.