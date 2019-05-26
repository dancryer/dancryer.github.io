---
layout: post
title: "Dynamic DNS with CloudFlare"
permalink: /2013/03/dynamic-dns-with-cloudflare
---

I have a Raspberry Pi at home that I use as a simple web server for a few bits and pieces, and I wanted to make it web accessible. I couldn't rely on my ISP to provide a static IP address (it is, at best, usually static.)

So, I decided to put together a really simple PHP script to update CloudFlare every X minutes with my current IP. The script is as follows:
<pre>&lt;?php

$request = array();
$request['a'] = 'rec_edit';
$request['tkn'] = 'YOUR CLOUDFLARE API TOKEN';
$request['z'] = 'example.com';
$request['email'] = 'YOUR CLOUDFLARE EMAIL';
$request['id'] = 'DNS RECORD ID FOR home.example.com';
$request['type'] = 'A';
$request['name'] = 'home'; // Change this if you want the record to be something else. 
$request['content'] = file_get_contents('http://www.dancryer.com/ip.php?t='.time());
$request['service_mode'] = '0';
$request['ttl'] = '120';

$response = @json_decode(file_get_contents('https://www.cloudflare.com/api_json.html?' . http_build_query($request)), true);

if(!$response || $response['result'] != 'success')
{
print 'Failed to update DNS. :-(';
var_dump($response);
}</pre>
I then run that script via Cron and viola, home-brew dynamic DNS.
