# Information Disclosure

## What is Information Disclosure?

Information disclosure happens when a web application unintentionally reveals sensitive information that can help an attacker understand or compromise the system.

Sometimes this information looks harmless, but attackers can chain multiple small leaks together to perform more serious attacks.

---

# Common Sources of Information Disclosure

## 1. Files for Web Crawlers

### What are Web Crawlers?

Web crawlers (also called spiders or bots) automatically browse websites and collect information.

### How Crawlers Work

A crawler:

1. Starts with a list of URLs
2. Visits those pages
3. Reads content (text, links, images, etc.)
4. Follows links to discover new pages
5. Repeats the process

### Why Crawlers Are Used

* Search engines (Google, Bing) use crawlers to index websites
* Data collection and analytics
* Website monitoring
* SEO analysis

### Important Files

Web crawlers often rely on:

```txt
/robots.txt
/sitemap.xml
```

Attackers check these files because they may expose:

* Hidden endpoints
* Sensitive directories
* Administrative panels
* Backup locations

---

## 2. Directory Listing

### What is Directory Listing?

Directory listing occurs when a web server displays the contents of a folder because no default page exists (such as `index.html`).

### Example

Instead of blocking access to:

```txt
/uploads/
```

The server shows:

```txt
image.png
backup.zip
error.log
```

### Why is it Dangerous?

Attackers may discover:

* **Backup files** → Source code, credentials
* **Log files** → Sensitive information
* **Temporary/dev files** → Vulnerable code
* **Hidden directories** → Better reconnaissance

### Important Note

Directory listing itself is not always a vulnerability.

It becomes dangerous when sensitive files are exposed or access control is weak.

**Main Idea:**
Directory listing helps attackers quickly discover useful files and information.

---

## 3. Developer Comments

Developers sometimes forget to remove HTML comments before deployment.

Example:

```html
<!-- TODO: remove debug page -->
<!-- Admin Panel: /secret-admin -->
```

Always inspect the page source for comments.

---

## 4. Error Messages

Verbose error messages may reveal:

* Technology stack
* Framework versions
* Internal paths
* Database details
* Stack traces

### Stack Trace

A stack trace is an error report showing the path the application took before crashing.

This may expose sensitive implementation details.

---

# Labs

## Lab: Information Disclosure in Error Messages

### Recon

Checked:

* Developer comments
* `/robots.txt`
* `/sitemap.xml`

No useful information was found.

### Observation

While browsing products, I noticed each product had an ID parameter.

I tested:

* Invalid strings
* Large numbers

Instead of a normal error, the website revealed a **full stack trace**.

### Information Disclosed

The application revealed:

```txt
Apache Struts 2 2.3.31
```

### Impact

Framework disclosure can help attackers identify known vulnerabilities related to that version.

---

## Lab: Information Disclosure on Debug Page

### Recon

Checked:

```txt
/robots.txt
/sitemap.xml
```

No findings.

### Finding Developer Comments

Using Burp Suite:

**Target → Site Map → Right Click → Engagement Tools → Find Comments**

Found an HTML comment containing:

```txt
/cgi-bin/phpinfo.php
```

### Observation

The page exposed debugging information.

Under environment variables, a **secret key** was revealed.

### Impact

Debug pages may expose:

* API keys
* Secrets
* Environment variables
* Internal configuration

---

## Lab: Source Code Disclosure via Backup Files

### Why Source Code Disclosure Matters

Source code access helps attackers:

* Understand application behavior
* Find vulnerabilities
* Discover hardcoded credentials
* Identify sensitive logic

Examples of exposed secrets:

* API keys
* Database credentials
* Backend access tokens

### Important Note

Files like `.php` are usually rendered, not shown as source code.

However, backup files may expose the raw code.

Common backup file patterns:

```txt
index.php~
index.php.bak
index.php.old
```

### Lab Walkthrough

Checked:

```txt
/sitemap.xml
```

Nothing found.

Checked:

```txt
/robots.txt
```

Found:

```txt
/backup
```

Visited the directory.

Observed that **directory listing was enabled**.

A backup source code file was exposed.

Inside the database connection section, a hardcoded password was found.

### Impact

Exposed source code can reveal:

* Credentials
* Secrets
* Internal logic
* Security flaws

---

## Lab: Authentication Bypass via Information Disclosure

### TRACE Method

The `TRACE` HTTP method is used for diagnostic purposes.

If enabled, the server responds with the exact request it received.

Sometimes this leaks internal authentication headers.

### Lab Walkthrough

Visited the admin panel.

Response:

```txt
401 Unauthorized
```

Sent the request to **Repeater**.

Changed the request method to:

```http
TRACE
```

### Observation

The response revealed a header:

```http
X-Custom-IP-Authorization
```

This header determined whether the request came from:

```txt
127.0.0.1
```

Modified the header:

```http
X-Custom-IP-Authorization: 127.0.0.1
```

### Result

Successfully bypassed authentication.

---

## Lab: Information Disclosure in Version Control History

### What is Version Control?

Version control systems track changes made to files over time.

Example:

* Git

Developers create snapshots called **commits**.

This allows them to:

* Restore previous versions
* Compare changes
* Track mistakes
* See who changed what

### Lab Walkthrough

Browsed to:

```txt
/.git
```

This exposed Git version control data.

Downloaded the `.git` directory.

### Useful Commands

View commits:

```bash
git log
```

Compare changes:

```bash
git diff <commit1> <commit2>
```

### Observation

A hardcoded password had been removed and replaced with a variable.

Recovered the old password from Git history.

Used it to log in as administrator.

### Impact

Exposed Git history may leak:

* Deleted secrets
* Hardcoded credentials
* Sensitive code
* API keys

---
