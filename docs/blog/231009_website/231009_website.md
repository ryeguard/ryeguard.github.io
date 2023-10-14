# Setting up a website and email with minimal cost using Hostup.se, Github Pages and Apple iCloud+

In this post, we will register our own domain using [Hostup.se](https://hostup.se/en/). For free hosting of our website, we will use [Github Pages](https://pages.github.com).
Although these instructions are written for Hostup.se, they should be applicable to any domain registrar that allows you to configure DNS records. We will also set up custom email addresses using [Apple iCloud+](https://www.apple.com/icloud/). Usually, this would require a paid email hosting service, but Apple iCloud+ allows you to create a custom email domain as part of the subscription.

## Table of Contents

- [Registering a Domain](#registering-a-domain)
- [Setting up Github Pages](#setting-up-github-pages)
  - [Creating a Repository](#creating-a-repository)
  - [Configuring a Custom Domain](#configuring-a-custom-domain)
- [Setting up Hostup.se](#setting-up-hostupse)
    - [DNS Management](#dns-management)
        - [A Records](#a-records)
        - [CNAME Record](#cname-record)
        - [ALIAS Record](#alias-record)
        - [CAA Record](#caa-record)
        - [DNSSEC](#dnssec)
    - [Registrar Lock](#registrar-lock)
- [Email Addresses with Apple iCloud+](#email-addresses-with-apple-icloud)
- [Conclusion](#conclusion)

## Registering a Domain

At the time of writing, [Hostup.se](https://hostup.se/en/) had the best renewal price for a `.se` domain. I bought `rygard.se` for 108 SEK with yearly renewal at 110 SEK. Purchasing the domain was simply done by searching for the domain to check availability and then adding it to the cart.

During setup, there is no need to set nameservers or add hosting.

After completing the purchase, the domain is registered and ready to be configured.

> ℹ️ The client information you provide during the purchase will not be made public in the Whois record via [ICANN Lookup](https://lookup.icann.org) or [IIS.SE](https://www.iis.se/en/). For `.se` and `.nu` domains, no client information is made public by Hostup. This is the Whois record for rygard.se:
> ```
> state:            active
> domain:           rygard.se
> holder:           (not shown)
> created:          2023-10-08
> modified:         2023-10-08
> expires:          2024-10-08
> nserver:          ns1.hostup.se 129.151.210.240
> nserver:          ns2.hostup.se 46.227.67.107
> nserver:          ns3.hostup.se 54.37.205.181
> nserver:          ns4.hostup.se 198.251.84.202
> dnssec:           unsigned delegation
> registry-lock:    locked
> status:           serverUpdateProhibited
> status:           serverDeleteProhibited
> status:           serverTransferProhibited
> registrar:        Hostup AB
> ```

## Setting up Github Pages

Github Pages is a free service that allows you to host a static website from a Github repository. The website is hosted at `https://<username>.github.io/` and can be configured to use a custom domain, which is what we will do. All documentation on Github Pages can be found [here](https://docs.github.com/pages). Please navigate via this page if any of the following links are broken.

### Creating a Repository

Create a new, public repository named `<username>.github.io` where `<username>` is your Github username. This is a special repository name that Github Pages will automatically use to host a website. Detailed instructions can be found [here](https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-github-pages-site). A simple `README.md` file in the repository root is enough for the website to be hosted. Allow a few minutes for the website to be built and deployed.

> ⚠️ Since this repository is public, anyone can see the source code of your website. Do not push any sensitive information to this repository.

### Configuring a Custom Domain

Detailed instructions can be found [here](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site). In short, navigate to the repository's Settings and click the Pages tab. Under Custom domain, enter your domain name, like `www.<your-domain>.se` and click Save. This will create a file named `CNAME` in the repository root. The file should contain only the domain name, without any protocol or trailing slash.

The Github specific configuration is now done. The next step is to configure the DNS records for the domain.

## Setting up Hostup.se

### DNS Management

Navigate and login to the Hostup.se control panel. There are two ways to navigate to the DNS management page.
- Either, click DNS under Services and click your domain,
- _OR_, navigate to your domain and click DNS management.

#### A Records

An A record is used to point a domain to an IP address. In this case, we want to point the `www` domain to Github Pages. Detailed instructions can be found [here](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site).

Create four A records in the Hostup.se control panel with the following values:

| Name* | Content (IP address)** |
| ----- | ---------------------- |
| `www` | `185.199.108.153` |
| `www` | `185.199.109.153` |
| `www` | `185.199.110.153` |
| `www` | `185.199.111.153` |

> ℹ️ \
> \* Hostup.se appends `.<your-domain>.se` to the Name field. This is why we only write `www` in the Name field. The resulting record will be `www.<your-domain>.se`.
>
> \*\* Double check these IP addresses in the official documentation. They may change.

#### CNAME Record

A CNAME record is used to point a subdomain to another domain. In this case, we want to point `www.<your-domain>.se` to the Github Pages domain.

Create a CNAME record with the following values:

| Name | Content (Hostname) |
| ---- | ------------------ |
| `www.<your-domain>.se` | `<username>.github.io` |

Where, again, `<your-domain>` is your domain name and `<username>` is your Github username.

#### ALIAS Record

An ALIAS record is used to point a domain to another domain. In this case, we want to point the root domain to the `www` subdomain.

Create an ALIAS record with the following values:

| Name | Content |
| ---- | ------- |
| (leave empty) | `www.<your-domain>.se` |

Where, again, `<your-domain>` is your domain name.

> ℹ️ The Name field is left empty because we want to point the root domain to the `www` subdomain.

#### CAA Record

A CAA record is used to specify which certificate authorities are allowed to issue certificates for your domain. This is a security measure to prevent unauthorized certificates from being issued. This will also allow you to use [Let's Encrypt](https://letsencrypt.org) to issue free SSL certificates for your domain. The result is that your website will be served over HTTPS and users will not be presented with a security warning when visiting your website.

Create a CAA record with the following values:

| Name | Content |
| ---- | ------- |
| `www.<your-domain>.se>` | `0 issue "letsencrypt.org"` |

#### DNSSEC

DNSSEC is a security measure that allows you to verify that the DNS records you receive are authentic. This is done by signing the DNS records with a private key and publishing the public key in the DNS records. The public key is then used to verify the authenticity of the DNS records.

DNSSEC should be enabled by default for Hostup.se domains. However, it is a good idea to verify that it is enabled. This is done by clicking the DNSSEC button in the Hostup.se DNS management control panel.

### Registrar Lock

A registrar lock is a security measure that prevents unauthorized transfers of your domain to another registrar. This is done by locking the domain at the registry level. This means that the domain cannot be transferred to another registrar without first unlocking the domain.

After completing all other configuration and verifying that the website is working, enable the registrar lock. Navigate to your domain in the Hostup.se control panel and click Registrar lock. Click Enable registrar lock.

## Email Addresses with Apple iCloud+

Apple iCloud+ allows you to create a custom email domain as part of the subscription. Detailed instructions can be found [here](https://support.apple.com/en-us/HT212514). In short, log in to [iCloud](https://www.icloud.com/), go to iCloud Setting, click iCloud+ Features and Custom Email Domain. Follow the instructions, which will ask you to add two MX records, two TXT records and one CNAME record. This is done in the Hostup.se DNS management control panel, just like the other DNS records. Here is a table of the records to add, with some example values, but please follow the instructions provided by Apple.

| Type | Host* | Priority | Example value** |
| ---- | ---- | -------- | ----- |
| MX | (leave empty) | `10` | `mx01.mail.icloud.com.` |
| MX | (leave empty) | `10` | `mx02.mail.icloud.com.` |
| TXT | (leave empty) | N/A | `apple-domain=<some-id>` |
| TXT ("SPF") | (leave empty) | N/A | `"v=spf1 include:icloud.com ~all"` |
| CNAME | `sig1._domainkey` | N/A | `sig1.dkim.<your-domain>.se.at.icloudmailadmin.com.` |

> ℹ️ \
> \* Apple's instructions say to set host as `@` for the MX and TXT records, at Hostup.se this is equivalent to leaving the Host field empty.
> \*\* Use the values provided in the Apple instructions.

## Conclusion

Congratulations! You have now set up a website with minimal cost using Hostup.se and Github Pages. The total cost is 108 SEK for the domain and 0 SEK for the hosting. The domain is registered for one year and can be renewed for 110 SEK. The hosting is free and will remain free as long as Github Pages is free.

[Back to top](#table-of-contents)
