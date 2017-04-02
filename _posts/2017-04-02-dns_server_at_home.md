---
layout: post
title:  "DNS server at home"
date:   2017-04-02 22:21:00 +0200
author: damien
image_header: network.jpg
tags: [raspberry pi, home_server, DNS]
---


At the beginning of the year, I finally received my first [Raspberry Pi][pi-home] ! I have got a lot of ideas which can use it like a [magic mirror][magic-mirror]. But for this first one, I will simply use it a my home server with a PiDrive storage extension.

Once I got Raspbian installed, I started to look into setting up the DNS server, the end goal is to have domain names for almost all devices on my internal network (e.g. instead of 192.168.1.14, I can use mydevice.home.lan).

## DNS software

After some googling, I have found [dnsmasq][app-dnsmasq] which is a lightweight alternative to [bind][app-bind]. Here is a list of the functionalities is decided to use:
- **DNS for static IPs** - Define domain names for devices with static IPs on your network
- **DNS forwarding and cache** - Continue to use a default DNS server (in my case my ISP's DNS server) and caching it's response to increase performances

I could also use the DHCP feature of dnsmasq but I am satisfied by the one of my ISP router so I will continue to use it.

## Picking an internal domain

The recommendation is to use a domain which I already own. But at the time I am writing this post I don't own one.
I have searched for a TLD not used by the ICANN ([here is the list][tld-list]) and decided to use the `.lan` TLD.

The full domain of my home will now be `home.lan`.

## Installation and configuration

Since Raspbian is a debian based OS, the installation is simple :

{% highlight bash %}
sudo apt-get install dnsmasq
{% endhighlight %}

For my setup, the configuration happens in the following files:
- **/etc/dnsmasq.conf** - dnsmasq specific configuration
- **/etc/hosts** - Hostnames for the static IPs

#### /etc/dnsmasq.conf

This configures how the DNS server should behave. I will only display the changes I have done in this file.

**Speeding up DNS requests**
{% highlight ini %}
# won't forward unqualified names (e.g. myserver)
domain-needed

# won't forward some non-routed addresses
bogus-priv

# won't forward requests for your intranet subdomain
local=/lan.mydomain.com/
{% endhighlight %}

**Specify root domain of my intranet**
{% highlight ini %}
# append the domain (below) to all hosts in the hosts file
expand-hosts

# appended to DHCP hosts and, if above option specified, to hosts from static IPs
domain=lan.mydomain.com
{% endhighlight %}

#### /etc/hosts

This file will contains the mapping between the static IPs and the domain name.
I have extended the original file with the following lines :

{% highlight bash %}
192.168.0.1 router
192.168.0.100 printer
192.168.0.150 pi
{% endhighlight %}

*Note:* since I enabled the `expand-hosts` options I do not need to write the full domain name in the `hosts` file (e.g. `router` will become `router.home.lan` ).

[pi-home]: https://www.raspberrypi.org
[magic-mirror]: https://magicmirror.builders/
[pidrive]: http://wdlabs.wd.com/category/wd-pidrive/
[app-dnsmasq]: http://www.thekelleys.org.uk/dnsmasq/doc.html
[app-bind]: https://www.isc.org/downloads/bind/
[tld-list]: http://data.iana.org/TLD/tlds-alpha-by-domain.txt
