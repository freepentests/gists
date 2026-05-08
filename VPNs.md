I know this article is quite poorly written, but I'll revise it later.

# Why VPNs Are Bad For Security

There's a common myth that VPNs are one of the greatest things you can use to increase your online security. However, the truth is that VPNs provide minimal to no advantages when it comes to security, and in fact, they can sometimes even be more harmful than good.

# Common arguments for why VPNs are good

## 1. VPNs encrypt your traffic

A lot of VPN supporters claim that VPNs provide protection against eavesdropping by encrypting your network traffic. However, this argument is invalid because almost every modern site on the internet uses HTTPs, which encrypts your traffic using almost unbreakable encryption algorithms such as RSA regardless of whether you're using a VPN or not.

## 2. Without VPNs, your ISP can see what sites you're visiting

Unlike HTTPs requests, DNS queries are sent in an unencrypted form, so a lot of VPN supporters claim that without using a VPN, your ISP will be able to see that you've at least visited a site even if they can't see the exact data that you sent.

However, the claim that you need a VPN to mask your DNS queries is invalid; the same effect can be achieved with DNS-over-HTTPS (DoH).
Put simply, DNS-over-HTTPS works by using the HTTPs protocol for making DNS queries.
For example, you could make this request to cloudflare's DoH servers:

```
curl -X GET 'https://cloudflare-dns.com/dns-query?name=discord.com&type=A'
  -H 'Accept: application/dns-json'
```

and the DoH server will respond with the DNS A records of discord.com in JSON form.
Now, all your ISP is seeing is a bunch of DNS queries to cloudflare-dns.com; not discord.com.

## 3. Hackers will be able to see your location

Another argument that is often given by VPN supporters is that without VPNs, hackers will be able to see your location. This point is invalid mainly because most residential ISPs have really poor location accuracy, so hackers will only be able to pinpoint you to your country or state, not your house address or street name.

If you're really paranoid and don't even want to reveal your country name to the sites you visit, then just use Tor. Tor routes your traffic through 3 decentralized nodes, each adding their own layer of encryption, which gives you much more security than a VPN would.

# Why VPNs can be bad for security

## 1. Shared IP addresses

Most VPN companies do not give you your own unique VPN server; that would cost too much. Instead, VPN companies rent hundreds of VPN servers which are shared amongst other users. When you're using a VPN, there are often hundreds of people using the same IP address as you at the same time.

Now, you might be asking yourself, "what's the problem with multiple users sharing IP addresses at the same time?"

The answer to that is that a lot of websites use your IP address to enforce security measures; for example, when you log in to a website from an unrecognized IP address, you might be required to enter a 2FA code.

Let's suppose I am a hacker and you use a VPN. I just found your email and password for Discord. I'm going to try to log in to your account with your email and password, but since I'm logging in from an unrecognized IP address, Discord is asking me to enter a 2FA code. Because you're using a VPN which shares IP addresses with different users, I can just change my IP to yours and then attempt to log in again, and this time I won't have to complete 2FA since the request is coming from a recognized IP address. This is a critical security flaw.

## 2. It gives you a more unique fingerprint

A lot of people say that VPNs can prevent you being tracked between different sites, but this idea is outdated; most sites nowadays use more sophisticated techniques such as browser and device fingerprinting, which are much more effective than IP-based tracking, as they takes many factors into account.

VPNs can actually make it easier for sites to single you out and track you; only around 23% of people on the internet use VPNs, and since you visited a tracking site with a VPN on, the site knows that you are one of those 23%, which eliminates a lot of possibilities.
