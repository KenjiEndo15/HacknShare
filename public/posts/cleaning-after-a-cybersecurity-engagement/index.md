# Cleaning During Cybersecurity Engagement


During security engagements, there are instances when we may employ attack techniques that leave traces within a client’s infrastructure. This post outlines a non-exhaustive list of essential actions that responsible professionals should take to ensure proper cleanup after the engagement.

---

## An Important Reminder
While vulnerability exploitation may seem like a routine task during security assessments, it is important to remember our professional and legal responsibilities.

If a breach occurs due to a backdoor left behind during an engagement, the security testing firm could face legal repercussions.

Even though not entirely related, this concern was clearly [raised by a Redditor](https://www.reddit.com/r/cybersecurity/comments/12ugrtx/comment/jh72y8x/?utm_source=share&amp;utm_medium=web3x&amp;utm_name=web3xcss&amp;utm_term=1&amp;utm_content=share_button):

&gt; If you leave malware around, and that malware is used to breach them - they will sue your ass. If you open a port, and that&#39;s used to breach them - they will sue your ass. Gross negligence is a crime, and it may even become a criminal procedure. Your contract does not protect you from gross negligence.

&gt; This is why Pentesters must understand their tools at a deep level. If your tool does something and you don&#39;t know about it - you&#39;re liable. 

&gt; I had a junior who used some code he found on github - it was backdoored and exfiltated data about the victim machine. Thankfully, the domain it went to was offline - but we still had to inform the client, and it opened us up to legal action.

To date, I have not come across a real legal case, either personally or online. If you have any information on such cases or court rulings related to this, please feel free to share it with me: https://x.com/KenjiEndo15.

## The CISA Red Team case
The Cybersecurity and Infrastructure Security Agency (CISA) conducted a Red Team Assessment (RTA) for an organization and shared details about it in their [blog post](https://www.cisa.gov/news-events/cybersecurity-advisories/aa24-326a).

I found the following paragraph especially interesting:
&gt; Within this assessment, the red team (also referred to as ‘the team’) gained initial access through a web shell left from a third party’s previous security assessment.

There is limited information available on this topic, but it is crucial for us to pause and recognize the importance of cleaning up after ourselves.

---

## Best Practices
### Understanding Your Tools
...

### Writing a Tool
...

### Documentation
...

### Post-Pentest Validation
...

---

## Example of Leftovers
Below are examples of artifacts that could unintentionally be left behind in a client&#39;s environment. While my expertise lies in pentesting, these considerations are equally applicable to other disciplines, such as red team and purple team operations — and likely beyond.

### Web Shells
[We have seen the CISA](#the-cisa-red-team) which breached its client&#39;s website using a leftover web shell from a prior engagement. Web shells are very much used nowadays to maintain persistent access to compromised systems, execute commands remotely, and facilitate lateral movement within a network. What can be worse than leaving it as-is after an engagement? Leaving your house door wide open without surveillance is a perfect example of this situation.

### Active Directory Objects Attributes
This one is more precise, though it remains quite common. Some exploitation scenario requires the modification of AD (Active Directory) objects attributes.

Without going into details, we can mention the abuse of Kerberos&#39; Resource-Based Constrained Delegation (RBCD). The goal is to modify the `msDS-AllowedToActOnBehalfOfOtherIdentity` attribute of another object to compromise it. Impacket&#39;s [rbcd.py](https://github.com/fortra/impacket/blob/master/examples/rbcd.py) script includes a cleaning feature, as long as you remember to use it.

### Weak Passwords
Imagine you are tasked with creating a domain user in an Active Directory environment. What kind of password would you select for this account? Many might feel tempted to choose a weak password for the sake of convenience—but this approach can lead to significant risks.

Choosing a weak password, particularly for a highly privileged account, is strongly discouraged. Prioritizing convenience over security is never a worthwhile trade-off. A weak password could be discovered during the engagement, either by an attacker or even by the client themselves, which could undermine your professional credibility.

At the end of an engagement, it is standard practice to remove any accounts you created or, at the very least, advise the client to do so. However, for peace of mind and to uphold best practices, using a strong password from the start is always the safer choice.

### Temporary Files
There are many instances where we need to upload or store files containing sensitive data.

For example, in web penetration testing, a file upload functionality can present issues. Beyond web shells, attackers could achieve command execution through the file upload and retrieve the results directly from a webpage on the client’s website. Failing to remove the file afterward could allow anyone with sufficient access and privileges to retrieve the outcome of the exploitation.

When it comes to Windows exploits, there are several [ways to dump passwords](https://www.thehacker.recipes/ad/movement/credentials/dumping/). Some dumps may end up in the target computer&#39;s filesystem. Some dumps may end up in the target computer&#39;s file system. Copying the dump to your machine will leave a copy behind. I can imagine the intern stumbling upon an NTDS dump - enjoy the free credentials 🎉.

In certain cases, there are exploits targeting specific tools like KeePass. For example, [KeePwn](https://github.com/Orange-Cyberdefense/KeePwn) can generate a cleartext export of a user&#39;s password database. It goes without saying that this dump is extremely valuable to any malicious individuals attacking your client after your engagement.

---

> Author:   
> URL: http://localhost:1313/posts/cleaning-after-a-cybersecurity-engagement/  

