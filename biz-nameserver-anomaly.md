## DROPTHISHOST Name Server anomaly


> Thousands of domain names at risk of adversarial takeovers

Recently, while auditing our DNS inventory datasets, we observed that several domains in our dataset were responding with `SERVFAIL` status and had their nameservers set to `dropthishost-.*.biz` domains. This behavior was interesting, and we had several questions, yet to be answered:

-   Why do these domains have these weird nameservers?
-   From where these `dropthishost-.*.biz` nameservers are coming?

Following our quest, we reached some interesting findings, which we will be presenting in this article.

---

## Dataset

For this research, we used a dataset comprising around **two hundred twenty-six million** (226,101,519 to be precise) domain names, sourced from the zone files of 1070 top-level domains (TLDs). The zone files contain a list of all the registered domain names within a particular TLD, along with their corresponding DNS records.

The dataset included a wide variety of TLDs, with the top four being **.com, .net, .org, .info**. Specifically, the breakdown of TLDs in the dataset is as follows:


![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fcb85441c-7a68-49a5-acff-e6aa41b6c5d2_967x582.webp)


---

## Interesting behavior

We observed that several domains in our dataset were responding with `SERVFAIL` status and had their nameservers set to `dropthishost-.*.biz` domains.


![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fb41c6151-b613-4635-b33e-1f48b40af7f7_1748x346.png)


On checking the availability of these `dropthishost-*.biz` domains, we observed that these domains were available for purchase

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F67a3fae3-5394-4da5-b775-c5753f992ce1_2880x1800.png)


This confirms our hypothesis that the domains which are responding with `SERVFAIL` status and had their nameservers set to `dropthishost-.*.biz` domains are at risk of adversarial takeovers.

After traversing our datasets, we were able to discover 5474 such domains:


![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F40cf383e-ead8-44ad-84cf-00e90ac9f2b9_1590x432.png)


---

## Root Cause

While digging into this issue, we stumbled upon this research paper:  “[Unresolved Issues: Prevalence, Persistence, and Perils  
of Lame Delegations](https://www.sysnet.ucsd.edu/~voelker/pubs/lame-imc20.pdf)“. This solved the mystery behind these weird `dropthis` nameservers.

For decades, registrars have used an undocumented practice to clean up expired nameserver domains, a practice developed in response to a situation created by requirements of the EPP specification. A registrar cannot delete the record for a nameserver domain that expires if there are other records (e.g., domains) in the same TLD that refer to a host object for that domain. However, by crafting a nameserver hostname in another TLD, and updating the host object record to use this “sacrificial” nameserver hostname instead — in effect updating the NS record of all domains referring to the original nameserver to use the new sacrificial nameserver host in a different superordinate domain — the registrar can then garbage collect the original expired nameserver object. Domains pointing to the sacrificial nameserver become lame delegated, but domain owners can always change the NS records to use a valid nameserver again if they choose. Anecdotally, it appears registrars chose `.biz` because it was a new gTLD at the time, leaving these domains at the risk of adversarial takeovers.

----
