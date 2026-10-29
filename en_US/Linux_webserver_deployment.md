# GNU/Linux Webserver deployment
In this post i will address how to reasonably make it safe to deploy a nginx webserver in a linux VPS, note that this will not include setting up a database, nor PHP or another wsgi applications.
This will be a basic web server, but reasonably secure, if you want to rent a VPS to make landing pages for clients and host several websites within the same host, you will be able to save money by hosting several static websites in the same box.
Note that every command will be prepended by # or $, # means you run as root(or with sudo) and $ as your regular user, but these are prompts, **do not copy them as part of the command line**.

## Provisioning your VPS
There are several cheap cloud VPS providers, if you can find a good one with a reasonably good price don't forget to look for discount coupons.
I can list a few ones that i know, but i can't vouch for their stability or good prices, if you want you can also host from home, but the investment on having a static public facing ip address, not being behind cgnat, a server, an ups for your server and networking equipment, also the power consumption, it is just not worth it.
Also for this example we will be using Debian, at the time of writing we are using Debian 13 but if you're reading this in the future use a newer version
Popular providers:
- [Linode/Akamai Cloud](https://www.linode.com/lp/refer/?r=f0c57e62c8c9a44008452111b93a8f8fc6fa4028): They have Shared CPU instances starting from $5 per month and a wide range of physical locations, with varying prices, personally i've tried and use this in production and it has been perfect.
- [Digital Ocean](https://www.digitalocean.com/): Never tried it, but they provide compute instances starting at $4 per month, it is quite popular though.
- [Vultr](https://www.vultr.com/): I've tried it, and it performed well, but i never used it in production, cancelling their services weren't nice.
- [Hetzner](https://www.hetzner.com/): Popular, and reliable, some GNU/Linux distros websites are hosted by it.
- [Oracle Cloud](https://www.oracle.com/cloud/):They have a free tier, but i've heard that they are a bit bad, the network interface isn't stable, and when i tried to use it my account got stuck in a limbo and nobody was able to help me, but hey you can try the free tier and try to get lucky.
WARNING: Be sure to pick a strong root password!!!

Now that you have your server, and we will use 198.51.100.1 as it's IPv4 and 2001:db8:1234:5678::1 as it IPv6, so it's time to SSH into it.
`$ ssh root@198.51.100.1`
Now ssh should ask for your root password, enter it and you will be dropped into a root shell at your vps.

## Updating the server
Now that we are in, the first thing to do is to be sure that we are running an updated system, so execute the following command
`# apt update && apt full-upgrade -y`
This will update the package list and will upgrade all the packages to the latest version, if you have kernel updates reboot your system now with the command `reboot` and connect again once the vps finishes booting. 

## Adding your regular user
The first thing we need to do is to create our regular user and add it to the sudoer group, for this example we will be using the user `remote` to differentiate, but it is convenient if you use your workstation username in your server as it will prevent you from having to type your username every time you connect to the ssh, so on the server type the following command
`# adduser remote`
Now it will prompt you for your password, pick a nice one that you can remember, keep in mind that the same password can be used to escalate to root with the command `sudo` so pick a strong one.
You can leave the other information empty, but now we need to add our new user to the sudoer group with the following command:
`# usermod -aG sudo remote`
This command is the usermod, it modifies a characteristic of an user, the parameter -aG appends ours to a group, `sudo`is the name of the group and `remote`is the name of the user.

## Installing the needed packages
Now we will install all the needed packages for our basic server, if you're planning to deploy for example a php+mysql/mariadb applications for example, you will have to add their package names here, or you can install them later, by the way i will be using vim as text editor, it may be challenging to newcomers, but if you rather use nano, feel free to omit the `vim` package in the next command.
So execute the following command:
`# sudo apt install tmux vim nginx wireguard`
Now wait for the installation of the packages to finish.

## (OPTIONAL) Generating your ssh keys
This step is only necessary if you don't have a keypair of ssh keys on your **local system**you will type the following commands **as your regular user**:
`$ mkdir -p ~/.ssh && chmod 700 ~/.ssh && ssh-keygen`
This will create a directory, make it readable and writable only for your user as a security measure, and will generate the keypair.

## Copying your pubkey to your the remote server
On your **local system** execute the following command:
`$ ssh-copy-id remote@198.51.100.1`
It will ask your remote user password, and then it will install your pubkey in your remote user home, just remember that `remote`is our example user, change it to the username you picked when you created the user.

## Securing the SSH server
Now it's time for a critical security measure, since passwords are mostly vulnerable to brute-forcing, we must use public key authentication so only with our key someone will be able to log into the system, also we must disable root login because the user is easily guessable since it is the standard super user name, and we must disable password authentication, so as **root** with your favorite text editor on your server open the file `/etc/ssh/sshd_config` and make the following changes:
- Locate the line with `PermitRootLogin yes` and change it to `PermitRootLogin no`.
- Locate the line with `#PubkeyAuthentication yes` and change it to `PubkeyAuthentication yes`.
- Locate the line with `PasswordAuthentication yes` and change it to `PasswordAuthentication no`.
Save the file and exit the text editor.

Now type the command `# systemctl restart sshd` and it will restart the ssh service so the canges you made in the config will take effect.


## Configure your Wireguard VPN tunnel
The wireguard setup is pretty easy first you will need to generate the server keypair, so connected to the server over ssh become root with `$ sudo -i`, then `# cd /etc/wireguard`, now we will generate the keypair with this command `# wg genkey | tee privatekey | wg pubkey > publickey`.
Also be sure to have a wireguard keypair on your machine(by using the same command above but you could place it in `~/.wireguard`
Now create a wireguard configuration file on the server called `wg0.conf` with your favorite text editor and paste the following
```
[Interface]
Address = 10.0.0.1/32
ListenPort = 51821
PrivateKey = replace with your server private key

[Peer]
PublicKey = replace with your computer PUBLIC key
AllowedIPs = 10.0.0.2/32
```
Be sure to add new peers for each machine you want to be able to access this server over wireguard.
After that execute the following commands to enable and start your wireguard interface on the server `# systemctl enable wg-quick@wg0 && systemctl start wg-quick@wg0.service`
After that on your machine, enable your wireguard connection and try pinging the server wireguard ip(in the case of this example it is 10.0.0.1), if the ping is successful then you're done setting wireguard, if ping fails you should troubleshoot it before 

## Configure the firewall
Now that we have connectivity through wireguard, we should be able to configure the firewall to make ssh only accessible through wireguard, this will make only your systems that were configured as peers on the wireguard to access ssh.
So `# cd /etc` to change your current directory to the /etc and now edit the file `nftables.conf`with your favorite text editor and paste the following contents, also be sure to edit the `eth0` to match your server network interface name(check with the command `$ ip link`
```
#!/usr/sbin/nft -f

flush ruleset


table inet filter {
        chain input {
                type filter hook input priority 0; policy drop;
                 # accept any localhost traffic
                iif lo accept
                # accept previously allowed traffic
                ct state established,related accept
                ct state invalid log prefix "nftables CHAIN=input STATE=invalid " counter drop
                # accept ssh only over wg0(wireguard) interface
                iifname wg0 tcp dport 22 ct state new counter accept
                #accept http incoming connections
                iifname eth0 tcp dport 80 ct state new counter accept
                #accept https incoming connections
                iifname eth0 tcp dport 443 ct state new counter accept
                #accept wireguard incoming connections
                iifname eth0 udp dport 51821 ct state new counter accept
                #accept http incoming connections from vpn
                iifname wg0 tcp dport 80 ct state new counter accept
                #accept icmp requests
                ip protocol icmp accept
                ip6 nexthdr icmpv6 accept
                #add log prefix
                log prefix "nftables CHAIN=input POLICY=drop "
        }
        chain output {
                type filter hook output priority 0;
        }
}

```
Now that you edited the file let's enable the firewall with the command `# systemctl enable nftables && systemctl start nftables`
## Configure nginx for static websites
Now that we have our server secured, we will make the configuration for our static website, for our example our site name will be `site` so execute `# cd /etc/nginx/sites-available/` and create your configuration file with the name of our site and paste the following content, read it carefully and replace with the relevant information where the comments says so.
```
server {
    listen 80;
    listen [::]:80;

    # Replace with the real domain(s)
    server_name example.com www.example.com;

    # Replace with the path to your site rootdir(that is accesible for www-data)
    root /var/www/your-site-root-dir;
    index index.html;
    charset utf-8;

    # Don't advertise the nginx version
    server_tokens off;

    # Send relative Location headers so redirects keep the scheme/host/port
    # the client used (works unchanged once SSL is added in front)
    absolute_redirect off;

    # Static site: no uploads, no request bodies
    client_max_body_size 1k;

    # Never list directory contents
    autoindex off;

    # Security headers (inherited by every location that has no add_header of its own)
    add_header Content-Security-Policy "default-src 'none'; style-src 'self'; img-src 'self'; base-uri 'none'; form-action 'none'; frame-ancestors 'none'" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "DENY" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Permissions-Policy "camera=(), microphone=(), geolocation=(), interest-cohort=()" always;
    add_header Cross-Origin-Opener-Policy "same-origin" always;

    # Only GET and HEAD make sense for a static site
    if ($request_method !~ ^(GET|HEAD)$) {
        return 405;
    }

    # Hide the .html extension: redirect requests that include it to the clean URL.
    # $request_uri is the original request, so nginx's internal rewrites
    # (index -> /index.html, try_files -> $uri.html) don't trigger these.
    if ($request_uri ~ "^/index(\.html)?(\?|$)") {
        return 301 /$is_args$args;
    }
    if ($request_uri ~ "^/(.+)\.html(\?|$)") {
        return 301 /$1$is_args$args;
    }

    # Custom error page, also used by the "return 404" rules below
    error_page 404 /404.html;

    location = /404.html {
        internal;
    }

    # Don't serve the error page with a 200 at its clean URL
    location = /404 {
        return 404;
    }

    # Home page: served by the index directive (index.html)
    location = / {
    }

    location / {
        # /articles -> articles.html. Directories are never tried ($uri/ is
        # left out), so subdirectory URLs like /images/ return 404.
        try_files $uri $uri.html =404;
    }

    # Block hidden files (.git, .env, ...) - the .well-known directory stays reachable
    location ~ /\.(?!well-known/) {
        return 404;
    }

    # Cache static assets in the browser (expires doesn't reset the add_header list)
    location ~* \.(css|svg|png|jpe?g|gif|webp|ico|woff2?)$ {
        try_files $uri =404;
        expires 7d;
        access_log off;
    }
}

```
Save the file and exit the text editor.

Now nginx only loads the configurations that are inside `/etc/nginx/sites-enabled/`, so to enable our site we need to create a symlink of our configuration file there, execute the following command:
`# ln -s /etc/nginx/sites-available/site /etc/nginx/sites-enabled/site`
This command is the ln, it creates links between files, the parameter -s makes it a symbolic link, the first path is the file we are linking to and the second is where the link will be created, just remember that `site` is our example name, change it to the name you picked for your configuration file. This way if you ever want to disable a site you just need to remove the symlink from `sites-enabled` and your configuration file will still be safe in `sites-available`.

Before applying the changes we must test the configuration, a typo in the configuration file can make nginx fail to start and take all your websites down, so execute the following command:
`# nginx -t`
If everything is fine you should see an output like this:
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
If the test fails, nginx will tell you the file and the line where the error is, so fix it and run `# nginx -t` again until the test is successful.

Now that the configuration is tested, reload nginx so it will start serving your site with the command `# systemctl reload nginx`, the reload applies the new configuration without dropping the connections of the websites that are already being served.

## Configure cloudflare protection
Now that our site is being served, we will put it behind Cloudflare. Cloudflare will act as a reverse proxy between the visitors and our server, so the visitors will only see Cloudflare IP addresses instead of our server IP, and it will also protect our server against DDoS attacks and cache our static files in their CDN, and the free plan is more than enough for static websites.
For this example we will keep using `example.com` as our domain, replace it with your real domain.

### Creating your Cloudflare account
Go to [Cloudflare dashboard](https://dash.cloudflare.com/sign-up) and create your account with your email and a strong password, after that verify your email address with the link they will send to you.
WARNING: Whoever has access to this account controls where your domain points to, so be sure to enable two-factor authentication in your profile settings!!!

### Adding your domain to Cloudflare
In the dashboard click to add a domain, type your domain name(only `example.com`, without `www` or `https://`) and continue, now pick the **Free** plan.
Cloudflare will scan the DNS records that your domain currently has and import them, review this list carefully, especially if you have email in this domain, because if the `MX` and `TXT` records are missing your email will stop working after the migration.

### Configuring the DNS records
Still in the DNS records page, delete any `A`, `AAAA` or `CNAME` records that point to an old hosting and add the following records pointing to our server:

| Type | Name | Content | Proxy status |
|------|------|---------|--------------|
| A | @ | 198.51.100.1 | Proxied |
| AAAA | @ | 2001:db8:1234:5678::1 | Proxied |
| CNAME | www | example.com | Proxied |

The `@` means the domain itself(`example.com`) and the **Proxied** status(the orange cloud) is what makes the traffic pass through Cloudflare, if a record is marked as **DNS only**(the grey cloud) Cloudflare will just answer with your server real IP address and you won't have any protection.
Keep in mind that Cloudflare only proxies HTTP and HTTPS traffic, so do not create a record like `vpn.example.com` to use as your wireguard endpoint, it would have to be **DNS only** and it would expose your server IP address to anyone, keep using the IP address directly in your wireguard configuration.

### Migrating the DNS to Cloudflare
After reviewing the records, Cloudflare will show you two nameservers, something like `ada.ns.cloudflare.com` and `bob.ns.cloudflare.com`, write them down because each account gets its own pair.
Now login to your domain registrar(the company where you bought your domain), if DNSSEC is enabled for your domain disable it first, otherwise your domain will stop resolving after the change, then find the nameservers settings of your domain, replace the current nameservers with the two from Cloudflare and save.
The change can take from a few minutes up to 24 hours to propagate, Cloudflare will send you an email when your domain is active, but you can also check it on your **local system** with the following command:
`$ dig NS example.com +short`
When the output shows the Cloudflare nameservers the migration is done, if you disabled DNSSEC you can enable it again in the Cloudflare dashboard under **DNS > Settings**, it will give you a DS record that you have to add in your registrar.

### Testing the Cloudflare proxy
Now that the domain is active, let's be sure that the traffic is going through Cloudflare, so on your **local system** execute the following command:
`$ curl -I http://example.com`
In the response headers you should see `server: cloudflare` and a `cf-ray` header, also `$ dig A example.com +short` should return Cloudflare IP addresses instead of 198.51.100.1, if this is the case your site is now protected by Cloudflare.
Since our server only listens on port 80, Cloudflare will connect to it over plain HTTP, the visitors will still be able to access your site with HTTPS because Cloudflare provides the certificate for them, you can check this in the dashboard under **SSL/TLS > Overview**, we will encrypt the connection between Cloudflare and our server later in the certbot section.

### (OPTIONAL) Only allowing Cloudflare IPs in the firewall
Right now anyone that knows your server IP address can still access the site directly and skip Cloudflare entirely, so we will change the firewall to only accept HTTP and HTTPS connections from the Cloudflare IP addresses.
The list of Cloudflare IP addresses is published at [https://www.cloudflare.com/ips-v4](https://www.cloudflare.com/ips-v4) and [https://www.cloudflare.com/ips-v6](https://www.cloudflare.com/ips-v6), the list below was taken from there at the time of writing, but it can change, so compare it with these links before using it.
As root edit the file `/etc/nftables.conf` with your favorite text editor, add the two sets right after the `table inet filter {` line:
```
        set cloudflare_v4 {
                type ipv4_addr
                flags interval
                elements = {
                        173.245.48.0/20, 103.21.244.0/22, 103.22.200.0/22,
                        103.31.4.0/22, 141.101.64.0/18, 108.162.192.0/18,
                        190.93.240.0/20, 188.114.96.0/20, 197.234.240.0/22,
                        198.41.128.0/17, 162.158.0.0/15, 104.16.0.0/13,
                        104.24.0.0/14, 172.64.0.0/13, 131.0.72.0/22
                }
        }

        set cloudflare_v6 {
                type ipv6_addr
                flags interval
                elements = {
                        2400:cb00::/32, 2606:4700::/32, 2803:f800::/32,
                        2405:b500::/32, 2405:8100::/32, 2a06:98c0::/29,
                        2c0f:f248::/32
                }
        }
```
Now replace the two rules that accept http and https incoming connections on `eth0` with the following rules:
```
                #accept http and https incoming connections only from cloudflare
                iifname eth0 ip saddr @cloudflare_v4 tcp dport { 80, 443 } ct state new counter accept
                iifname eth0 ip6 saddr @cloudflare_v6 tcp dport { 80, 443 } ct state new counter accept
```
Keep the rule that accepts http from `wg0`, this way you can still access the server directly through wireguard to troubleshoot the site.
Save the file and exit the text editor, before applying the changes check the file for errors with the following command:
`# nft -c -f /etc/nftables.conf`
The -c parameter only checks the file without applying it, if there is no output the file is fine, so apply it with `# systemctl restart nftables`.
Now try to access `http://198.51.100.1` from your browser with the wireguard disconnected, the connection should time out, while `http://example.com` should keep working through Cloudflare.
Just remember that if Cloudflare adds new IP addresses to their list and you don't update your firewall, visitors that reach these new Cloudflare servers won't be able to access your site, so check the list from time to time.



## Enabling HTTPS with certbot
Right now the connection between the visitor and Cloudflare is encrypted, but the connection between Cloudflare and our server is plain HTTP, so anyone in the path between them can read or modify our pages. To fix this we will get a free certificate from Let's Encrypt with certbot and make Cloudflare only talk to our server over HTTPS.
So install certbot and its nginx plugin with the following command:
`# apt install certbot python3-certbot-nginx`
Now we will request the certificate, replace the domains and the root directory with the ones from your nginx configuration file:
`# certbot --authenticator webroot --webroot-path /var/www/your-site-root-dir --installer nginx --redirect -d example.com -d www.example.com`
This command uses two plugins, the `webroot` authenticator proves to Let's Encrypt that you own the domain by placing a temporary file inside your site root directory(in `.well-known/acme-challenge/`, that's why our configuration doesn't block the `.well-known` directory), and the `nginx` installer edits your nginx configuration file to use the certificate. We use the `webroot` authenticator instead of the `nginx` one because it keeps working behind Cloudflare when the renewals happen over HTTPS, the parameter `--redirect` makes nginx redirect every HTTP request to HTTPS and each `-d` is a domain that will be included in the certificate.
Certbot will ask for your email address, used to warn you if something goes wrong with your certificate, and to accept the terms of service, after that it will issue the certificate and reload nginx.
If you open your configuration file at `/etc/nginx/sites-available/site` you will see that certbot added the `listen 443 ssl` and the certificate lines to our server block and moved the `listen 80` lines to a new server block that only redirects to HTTPS, all these lines end with the comment `# managed by Certbot`.

WARNING: Do the next step right away, while Cloudflare is still connecting over HTTP your site will be stuck in a redirect loop!!!

Now in the Cloudflare dashboard go to **SSL/TLS > Overview**, pick the **Custom** mode and select **Full (strict)**, this way Cloudflare will always connect to our server over HTTPS and will refuse to connect if our certificate isn't valid.
Let's test the certificate directly on the server, connected over ssh execute the following command:
`$ curl -I --resolve example.com:443:127.0.0.1 https://example.com`
The `--resolve` parameter makes curl connect to our own server instead of Cloudflare, and since curl checks the certificate, if you get a response with `HTTP/1.1 200 OK` the certificate is fine. Now open `https://example.com` in your browser and check that the site loads normally.
The Let's Encrypt certificates are only valid for 90 days, but the certbot package already installs a systemd timer that renews them automatically, to be sure that the renewal will work execute the following command:
`# certbot renew --dry-run`
If it says that the simulated renewal succeeded you don't need to worry about your certificate anymore.

## Logging the real visitor IPs
Since all the traffic now goes through Cloudflare, the nginx logs will only show Cloudflare IP addresses as the visitors, which makes the logs useless to find out who is accessing your site. Cloudflare sends the real visitor IP address in the `CF-Connecting-IP` header, so we will tell nginx to trust this header, but only when the request comes from a Cloudflare IP address, otherwise anyone could fake their IP by sending this header directly to our server.
As root create the file `/etc/nginx/conf.d/cloudflare.conf` with your favorite text editor and paste the following contents(the IP ranges are the same from the firewall section, so compare them with the Cloudflare lists before using them):
```
# Cloudflare IPv4 ranges
set_real_ip_from 173.245.48.0/20;
set_real_ip_from 103.21.244.0/22;
set_real_ip_from 103.22.200.0/22;
set_real_ip_from 103.31.4.0/22;
set_real_ip_from 141.101.64.0/18;
set_real_ip_from 108.162.192.0/18;
set_real_ip_from 190.93.240.0/20;
set_real_ip_from 188.114.96.0/20;
set_real_ip_from 197.234.240.0/22;
set_real_ip_from 198.41.128.0/17;
set_real_ip_from 162.158.0.0/15;
set_real_ip_from 104.16.0.0/13;
set_real_ip_from 104.24.0.0/14;
set_real_ip_from 172.64.0.0/13;
set_real_ip_from 131.0.72.0/22;

# Cloudflare IPv6 ranges
set_real_ip_from 2400:cb00::/32;
set_real_ip_from 2606:4700::/32;
set_real_ip_from 2803:f800::/32;
set_real_ip_from 2405:b500::/32;
set_real_ip_from 2405:8100::/32;
set_real_ip_from 2a06:98c0::/29;
set_real_ip_from 2c0f:f248::/32;

# Header that has the real visitor IP
real_ip_header CF-Connecting-IP;
```
Every file inside `/etc/nginx/conf.d/` is loaded for all the sites, so you don't need to change your site configuration file. Save the file, exit the text editor and as always test the configuration and reload nginx:
`# nginx -t && systemctl reload nginx`
Now access your site and watch the log with `# tail -f /var/log/nginx/access.log`, your requests should now show up with your own public IP address instead of a Cloudflare one, press Ctrl+C to stop watching the log.

## Updating the Cloudflare IP list automatically
Both the firewall and the real IP configuration depend on the Cloudflare IP list, and if Cloudflare adds new addresses and we forget to update our server, visitors may not be able to reach the site or they will show up with Cloudflare IPs in the logs. So we will make a script that downloads the lists, regenerates both configurations and run it every day with a systemd timer.
If you followed the optional firewall section, we first need to move the Cloudflare sets to their own file so the script can replace them, so execute `# mkdir -p /etc/nftables.d`, edit the file `/etc/nftables.conf`, delete the two sets(`cloudflare_v4` and `cloudflare_v6`) and in their place, right after the `table inet filter {` line, add this line:
```
        include "/etc/nftables.d/cloudflare.nft"
```
Save the file, the script will create `cloudflare.nft` when it runs, if you didn't follow the optional firewall section just skip this part and the script will only update nginx.
NOTE: Be sure to really delete the old sets, the `nft -c` check the script does won't warn you if you forget them, nftables just merges the two sets with the same name into one, so the old addresses will stay in your firewall even after Cloudflare removes them from the list. You can check it with `# nft list set inet filter cloudflare_v4` after the script runs, it should only show the addresses from the Cloudflare list.
Now make sure that curl is installed with `# apt install curl`, and as root create the file `/usr/local/sbin/update-cloudflare-ips` with your favorite text editor and paste the following contents:
```
#!/bin/sh
# Downloads the Cloudflare IP lists and updates the nginx real IP
# configuration and the nftables sets (when nftables.conf includes them)
set -eu

NGINX_CONF=/etc/nginx/conf.d/cloudflare.conf
NFT_SETS=/etc/nftables.d/cloudflare.nft

TMP=$(mktemp -d)
trap 'rm -rf "$TMP"' EXIT

curl -fsS --max-time 30 -o "$TMP/v4" https://www.cloudflare.com/ips-v4
curl -fsS --max-time 30 -o "$TMP/v6" https://www.cloudflare.com/ips-v6

# Stop if a list is empty or has anything that isn't an IP range
for list in v4 v6; do
    if [ ! -s "$TMP/$list" ] || grep -qvE '^[0-9a-f.:]+/[0-9]+$' "$TMP/$list"; then
        echo "Invalid Cloudflare $list list, nothing was changed" >&2
        exit 1
    fi
done

# Generate the nginx real IP configuration
{
    echo "# Generated by update-cloudflare-ips, do not edit"
    awk '{ print "set_real_ip_from " $1 ";" }' "$TMP/v4" "$TMP/v6"
    echo "real_ip_header CF-Connecting-IP;"
} > "$TMP/nginx.conf"

if ! cmp -s "$TMP/nginx.conf" "$NGINX_CONF"; then
    cp "$NGINX_CONF" "$TMP/nginx.conf.old" 2>/dev/null || true
    cp "$TMP/nginx.conf" "$NGINX_CONF"
    if nginx -t -q; then
        systemctl reload nginx
        echo "nginx real IP configuration updated"
    else
        # Restore the previous configuration if the new one is broken
        if [ -f "$TMP/nginx.conf.old" ]; then
            cp "$TMP/nginx.conf.old" "$NGINX_CONF"
        else
            rm -f "$NGINX_CONF"
        fi
        echo "nginx test failed, previous configuration restored" >&2
        exit 1
    fi
fi

# Only touch the firewall if nftables.conf includes the Cloudflare sets
if grep -q "$NFT_SETS" /etc/nftables.conf; then
    {
        echo "# Generated by update-cloudflare-ips, do not edit"
        echo "set cloudflare_v4 {"
        echo "        type ipv4_addr"
        echo "        flags interval"
        echo "        elements = { $(paste -sd, "$TMP/v4") }"
        echo "}"
        echo "set cloudflare_v6 {"
        echo "        type ipv6_addr"
        echo "        flags interval"
        echo "        elements = { $(paste -sd, "$TMP/v6") }"
        echo "}"
    } > "$TMP/cloudflare.nft"

    if ! cmp -s "$TMP/cloudflare.nft" "$NFT_SETS"; then
        cp "$NFT_SETS" "$TMP/cloudflare.nft.old" 2>/dev/null || true
        cp "$TMP/cloudflare.nft" "$NFT_SETS"
        if nft -c -f /etc/nftables.conf; then
            systemctl reload nftables
            echo "nftables Cloudflare sets updated"
        else
            if [ -f "$TMP/cloudflare.nft.old" ]; then
                cp "$TMP/cloudflare.nft.old" "$NFT_SETS"
            else
                rm -f "$NFT_SETS"
            fi
            echo "nftables check failed, previous sets restored" >&2
            exit 1
        fi
    fi
fi
```
The script only changes a file when the list is different from what we already have, and it always tests the new configuration before reloading, if the test fails it puts the previous file back, so a bad download will never take your site or your firewall down.
Save the file and make it executable with the following command:
`# chmod 700 /usr/local/sbin/update-cloudflare-ips`
Now we need to create the systemd service that runs the script, as root create the file `/etc/systemd/system/update-cloudflare-ips.service` and paste the following contents:
```
[Unit]
Description=Update the Cloudflare IP ranges for nginx and nftables
Wants=network-online.target
After=network-online.target

[Service]
Type=oneshot
ExecStart=/usr/local/sbin/update-cloudflare-ips
```
And the timer that will start this service every day, create the file `/etc/systemd/system/update-cloudflare-ips.timer` and paste the following contents:
```
[Unit]
Description=Update the Cloudflare IP ranges daily

[Timer]
OnCalendar=daily
RandomizedDelaySec=1h
Persistent=true

[Install]
WantedBy=timers.target
```
The `RandomizedDelaySec` adds a random delay of up to one hour so the servers don't all download the list at the same time, and `Persistent=true` makes the timer run at boot if the server was off when it should have run.
Now let's run the script once to be sure it works, and then enable the timer with the following commands:
`# systemctl daemon-reload && systemctl start update-cloudflare-ips.service`
`# journalctl -u update-cloudflare-ips.service`
The journal will show if the configurations were updated, or the error if something went wrong, if you followed the optional firewall section check that the sets were loaded with `# nft list set inet filter cloudflare_v4`, then enable the timer:
`# systemctl enable --now update-cloudflare-ips.timer`
You can check when it will run next with `# systemctl list-timers update-cloudflare-ips.timer`.
