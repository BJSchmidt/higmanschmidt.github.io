---
layout: post
title:  "Firefox 62 Nightlies: Improving DNS Privacy in Firefox"
date:   2018-06-06 11:56:07 -0600
categories:
---
Firefox recently introduced **DNS over HTTPS (DoH)** and **Trusted Recursive Resolver (TRR)** in nightly builds for Firefox 62.

**DoH** and **TRR** are intended to help mitigate these potential privacy and security concerns:
1. Untrustworthy DNS resolvers tracking your requests, or tampering with responses from DNS servers.
1. On-path routers tracking or tampering in the same way.
1. DNS servers tracking your DNS requests.

**DNS over HTTPs (DoH)** encrypts DNS requests and responses, protecting against on-path eavesdropping, tracking, and response tampering.

**Trusted Recursive Resolver (TRR)** allows Firefox to use a DNS resolver that's different from your machines network settings. You can use any recursive resolver that is compatible with DoH, but it should be a trusted resolver (one that won't sell users’ data or trick users with spoofed DNS).  Mozilla is partnering with Cloudflare (but not using the 1.1.1.1 address) as the initial default TRR, however it's possible to use another 3rd party TRR or run your own.

>Cloudflare is providing a recursive resolution service with a pro-user privacy policy. They have committed to throwing away all personally identifiable data after 24 hours, and to never pass that data along to third-parties. And there will be regular audits to ensure that data is being cleared as expected.

Additionally, Cloudflare will be doing [**QNAME minimization**](https://datatracker.ietf.org/doc/rfc7816/?include_text=1) where the DNS resolver no longer sends the full original QNAME (foo.bar.baz.example.com) to the upstream name server. Instead it will only include the label for the zone it's trying to resolve.

For example, let's assume the DNS Resolver is trying to find foo.bar.baz.example.com, and already knows that ns1.nic.example.com is authoritative for .example.com, but does not know a more specific authoritative name server.
1. It will send the query for just baz.example.com to ns1.nic.example.com which returns the authoritative name server for baz.example.com.
2. The resolver then sends a query for bar.baz.example.com to the nameserver for baz.example.com, and gets a response with the authoritative nameserver for bar.baz.example.com
3. Finally the resolver sends the query for foo.bar.baz.example.com to bar.baz.example.com's nameserver.
In doing this the full requested name (foo.bar.baz.example.com) is not exposed to intermediate name servers (bar.baz.example.com or baz.example.com)

Collectively **DNS over HTTPs (DoH)**, **Trusted Recursive Resolver (TRR)**, and **QNAME Minimization** are a step in the right direction, this does not fix DNS related data leaks entirely:
>After you do the DNS lookup to find the IP address, you still need to connect to the web server at that address. To do this, you send an initial request. This request includes a server name indication, which says which site on the server you want to connect to. **And this request is unencrypted.**
That means that your ISP can still figure out which sites you’re visiting, because it’s right there in the server name indication. Plus, **the routers that pass that initial request from your browser to the web server can see that info too.**


**So How do I enable it?**
DoH and TRR can be enabled in Firefox 62 or newer by going to **about:config**:
* Set **network.trr.mode** to **2**
    * Here's the possible network.trr.mode settings:
        * **0 - Off (default)**: Use standard native resolving only (don't use TRR at all)
        * **1 - Race**: Native vs. TRR. Do them both in parallel and go with the one that returns a result first.
        * **2 - First**: Use TRR first, and only if the name resolve fails use the native resolver as a fallback.
        * **3 - Only**: Only use TRR. Never use the native (after the initial setup).
        * **4 - Shadow**: Runs the TRR resolves in parallel with the native for timing and measurements but uses only the native resolver results.
        * **5 - Off by choice**: This is the same as 0 but marks it as done by choice and not done by default.
* Set **network.trr.uri** to your DoH Server:
    * Cloudflare’s is https://mozilla.cloudflare-dns.com/dns-query
    (but you can use any DoH compliant endpoint)
* The DNS Tab on **about:networking** will show which names were resolved using TRR via DoH.

**Links:**

[A cartoon intro to DNS over HTTPS](https://hacks.mozilla.org/2018/05/a-cartoon-intro-to-dns-over-https/)

[Improving DNS Privacy in Firefox](https://blog.nightly.mozilla.org/2018/06/01/improving-dns-privacy-in-firefox/)

[DNS Query Name Minimization to Improve Privacy](https://datatracker.ietf.org/doc/rfc7816/?include_text=1)

[TRR Preferences](https://gist.github.com/bagder/5e29101079e9ac78920ba2fc718aceec)

I'm not affiliated with Mozilla or Firefox, I just thought ~ would find this interesting.