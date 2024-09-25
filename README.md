# Introduction-to-Web-Hacking

---
## Walking An Application

More often than not, automated security tools and scripts will miss many potential vulnerabilities and useful information.

Here is a short breakdown of the in-built browser tools you will use:
- **View Source:** Use your browser to view the human-readable source code of a website.
- **Inspector:** Learn how to inspect page elements and make changes to view usually blocked content.
- **Debugger:** Inspect and control the flow of a page's JavaScript
- **Network:** See all the network requests a page makes.

---
## Content Discovery

When we talk about content discovery, we're not talking about the obvious things we can see on a website; it's the things that aren't immediately presented to us and that weren't always intended for public access.

There are three main ways of discovering content on a website which we'll cover. **Manually**, **Automated** and **OSINT** (Open-Source Intelligence).

### 1. Manually (Robot.txt, Favicon, Site.xml, HTTP Headers)

#### Robot.txt

A `robots.txt` file is a simple text file that webmasters create to guide web crawlers (like Googlebot) on how to interact with their websites. It contains rules specifying which parts of a website should be crawled or indexed by search engines, and which parts should be ignored. 

The `robots.txt` file is placed in the root directory of a website (e.g., `example.com/robots.txt`) and follows the "Robots Exclusion Protocol."

Here’s a basic example of a `robots.txt` file:

```
User-agent: *
Disallow: /private/
Allow: /public/
```

In this example:
- `User-agent: *` means the rules apply to all web crawlers.
- `Disallow: /private/` instructs crawlers not to index the `/private/` directory.
- `Allow: /public/` allows crawlers to index the `/public/` directory.

##### Common Uses:
- Preventing indexing of sensitive or irrelevant pages (like login pages, search results).
- Controlling traffic to avoid overloading the server with too many crawling requests.

It's important to note that `robots.txt` is not a security feature. It only provides guidelines for well-behaved crawlers but does not prevent malicious actors from accessing restricted sections of a site.

#### Favicon

A **favicon** (short for "favorites icon") is a small, icon-sized image that represents a website. It typically appears in the browser's address bar, tab, bookmarks, and sometimes in browser history or search results.

Sometimes when frameworks are used to build a website, a favicon that is part of the installation gets leftover, and if the website developer doesn't replace this with a custom one, this can give us a clue on what framework is in use. OWASP host a database of common framework icons that you can use to check against the targets favicon https://wiki.owasp.org/index.php/OWASP_favicon_database. Once we know the framework stack, we can use external resources to discover more about it.

If you run the following command on the AttackBox, it will download the favicon and get its md5 hash value which you can then lookup on the
https://wiki.owasp.org/index.php/OWASP_favicon_database.


```
user@machine$ curl https://static-labs.tryhackme.cloud/sites/favicon/images/favicon.ico | md5sum
```

#### Site.xml

A sitemap.xml (commonly referred to as site.xml) is an XML file that provides a list of all the important URLs on a website, helping search engines like Google, Bing, and others to understand and index the site’s content more efficiently. It acts as a roadmap for search engine crawlers, showing them what pages exist, how frequently they are updated, and the relative importance of each page.

Take a look at the sitemap.xml file on the Acme IT Support website to see if there's any new content we haven't yet discovered: http://10.10.181.184/sitemap.xml (open this in the FireFox browser on the AttackBox).

![image](https://github.com/user-attachments/assets/404044a0-6e21-46d3-890f-9fcfea393d2f)

#### HTTP Headers

When you send a request to a web server, it gives back some HTTP headers that can spill some tea 🫖! These headers might show info like the webserver software and even the programming language it's running. For example, in one case, the server is running **NGINX version 1.18.0** and **PHP version 7.4.3**. With this knowledge, you could totally look up if those versions have any known vulnerabilities and see if there’s something juicy to exploit! 😈

You can check out these headers by running a `curl` command with the `-v` (verbose) option. That’ll make the headers spill all their secrets for you to analyze.

### 2. OSINT (Google Dorking, Wappalyzer, Wayback Machine, GitHub, S3 Buckets)

#### Google Dorking

Google hacking / Dorking utilizes Google's advanced search engine features, which allow you to pick out custom content. You can, for instance, pick out results from a certain domain name using the site: filter, for example (site:tryhackme.com) you can then match this up with certain search terms, say, for example, the word admin (**site**:tryhackme.com admin) this then would only return results from the tryhackme.com website which contain the word admin in its content. You can combine multiple filters as well. Here is an example of more filters you can use:

| Filter    | Example             | Description                                              |
|-----------|---------------------|----------------------------------------------------------|
| `site`    | `site:tryhackme.com` | Returns results only from the specified website address   |
| `inurl`   | `inurl:admin`        | Returns results that have the specified word in the URL   |
| `filetype`| `filetype:pdf`       | Returns results which are a particular file extension     |
| `intitle` | `intitle:admin`      | Returns results that contain the specified word in the title |

#### Wappalyzer

Wappalyzer (https://www.wappalyzer.com/) is an online tool and browser extension that helps identify what technologies a website uses, such as frameworks, Content Management Systems (CMS), payment processors and much more, and it can even find version numbers as well.

#### Wayback Machine

The Wayback Machine (https://archive.org/web/) is a historical archive of websites that dates back to the late 90s. You can search a domain name, and it will show you all the times the service scraped the web page and saved the contents. This service can help uncover old pages that may still be active on the current website.

#### Github

To understand GitHub, you first need to understand Git. Git is a **version control system** that tracks changes to files in a project. Working in a team is easier because you can see what each team member is editing and what changes they made to files. When users have finished making their changes, they commit them with a message and then push them back to a central location (repository) for the other users to then pull those changes to their local machines. 

GitHub is a hosted version of Git on the internet. Repositories can either be set to public or private and have various access controls. You can use GitHub's search feature to look for company names or website names to try and locate repositories belonging to your target. Once discovered, you may have access to source code, passwords or other content that you hadn't yet found.

#### S3 Buckets

S3 Buckets are a storage service provided by Amazon AWS, allowing people to save files and even static website content in the cloud accessible over HTTP and HTTPS. The owner of the files can set access permissions to either make files public, private and even writable. Sometimes these access permissions are incorrectly set and inadvertently allow access to files that shouldn't be available to the public. 

The format of the S3 buckets is http(s)://{name}.s3.amazonaws.com where {name} is decided by the owner, such as tryhackme-assets.s3.amazonaws.com. S3 buckets can be discovered in many ways, such as finding the URLs in the website's page source, GitHub repositories, or even automating the process. One common automation method is by using the company name followed by common terms such as {name}-assets, {name}-www, {name}-public, {name}-private, etc.

---
## Subdomain Enumeration

Subdomain enumeration is the process of finding valid subdomains for a domain, but why do we do this? We do this to expand our attack surface to try and discover more potential points of vulnerability.

We will explore three different subdomain enumeration methods: **Brute Force**, **OSINT** (Open-Source Intelligence) and **Virtual Host**.

### OSINT (SSL/TLS Certificates, Search Engines, 

#### OSINT - SSL/TLS Certificates

We can use this service to our advantage to discover subdomains belonging to a domain, sites like https://crt.sh and https://ui.ctsearch.entrust.com/ui/ctsearchui offer a searchable database of certificates that shows current and historical results.

#### OSINT - Search Engines

Search engines contain trillions of links to more than a billion websites, which can be an excellent resource for finding new subdomains. Using advanced search methods on websites like Google, such as the site: filter, can narrow the search results. For example, `site:*.domain.com -site:www.domain.com` would only contain results leading to the domain name domain.com but exclude any links to www.domain.com; therefore, it shows us only subdomain names belonging to domain.com.






