# 4D Attack Surface — Part 1


[4D](https://us.4d.com/) is a company that provides a platform for developing applications.
It can help with tasks like building websites, running a web server, managing an SQL server, and much more.

The technology is relatively unknown in the world of web frameworks. Together with [Enzo Cadoni](https://enzo-cadoni.fr/), we decided to explore it and examine the 4D attack surface for penetration testing.

This series is divided into two parts. The first part introduces the technology, explains its basic functionalities, and covers template injection vulnerabilities. The second part focuses on **[specific topic]** and can be found here: **link**.

## 4D Usages in the World
A quick search on Shodan reveals that 4D is used by [6.363 web servers](https://www.shodan.io/search?query=%22Server%3A&#43;4D%22) — according to the following filter `&#34;Server: 4D&#34;`. 

Despite this significant number, we have yet to come across detailed blog posts addressing its security. Which is why we believe this research could benefit the community by expanding our understanding of the technology.

During penetration testing, it could help improve the testing of websites. Additionally, for 4D, it emphasizes the need for greater caution regarding its functionalities, which could potentially be exploited by malicious actors.

---

## Creating a Simple Page
...

### Download the Trial Version
...

### Installing the Trial Version
...

### Creating a Project
...

### Creating a Form
...

#### Template Pages
...

#### Best Practices
...

---

## Template Exploits
...Introduction

### Probing for Vulnerabilities
...?

### Delivering Malicious Payloads
...Introduction

#### Simple Payloads
...4D commands (QUIT) ; XSS ; ?

#### Complex Payloads
...SystemWorkers

---

## End Notes
...

---

> Author: KenjiEndo  
> URL: http://localhost:1313/posts/4d-attack-surface/  

