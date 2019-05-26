---
layout: post
title: "Handling robots.txt files in PHP based bots"
permalink: /2013/03/handling-robotstxt-in-php-bots
---

Modern web applications are rarely the stand-alone silos they once were. Web applications and sites alike pull data from countless other web sites as part of their normal operation -- This can be as a direct result of a user action (i.e. posting a link to a piece of content,) or a web crawling process that runs in the background.

Unfortunately, too many of these bots / spiders / crawlers fail to respect the basic protocols that robots are supposed to follow. Popular sites get hammered with hundreds of requests a second from the the same services and the vast majority of bots completely ignore robots.txt rules.

I've recently been working on a system that needed to honour these rules, so I thought I'd put together a quick guide.
### <strong></strong>Rule #1 - Give your bot a name
This might sound silly, but you need to give your bot a name in order to give it a user agent string of its own. So let's say it is called DanBot, you'd want to put together a user agent string like the following:
<pre>Mozilla/5.0 (compatible; DanBot/1.0; +http://www.dancryer.com/about-danbot)</pre>
This breaks down as **Mozilla/5.0 compatible**,** **which is a bit of a hang-back to an earlier time, but something that most browsers, crawlers, etc. still include. Then, **DanBot/1.0**, your bot's name and version number - allowing site admins to know who is visiting them and for you to know which version of your bot it was, should someone complain. Finally, you have a URL - This should link to a page that explains what your bot is, what it does, why it is crawling someone's site, and how to stop it.

### Rule #2 - Rate limit your requests

The worst thing a bot can do is fail to rate limit itself. You need to ensure that your bot only sends a reasonable number of requests to a given site in a given period. Now, how you determine what is reasonable is up to you, but some options include:


* Simply limiting yourself to X requests per second. Depending on your needs, that may even be less than one request per second.

* Calculate a request per second limit based on the response time of the site, size of response, etc. This is a little more complicated, but well worth doing if you're doing a significant amount of web crawling.

* Respect the crawl-delay parameter in the site's robots.txt file - This is covered in Rule #3.

Most importantly for this section, just remember that rate limiting should be done at a domain level. If you implement per-URL rate limiting on a site with a million pages, that's still up to a million requests per second for the site!

### Rule #3 - Respect robots.txt

Respecting robots.txt is vital for any well-meaning, well-behaved, web crawler. Robots.txt allows a site owner to define what they do not want any given bot to see, and how often they'd like a given bot to access their site. Some example rules for DanBot might be as follows:

```
User-Agent: DanBot
Crawl-Delay: 5
Disallow: *.zip
Disallow: /admin
```

What that would tell DanBot is that the site only wants us to visit once every five seconds, it must not crawl any URL that ends in .zip, and that it must not crawl any URL that starts with /admin.  These robots.txt files are not too complicated to handle. Here's a sample class to help you get started:
```php
<?php

class Robots
{
    const ROBOTS_TXT_LIFETIME = "-1 Day";
    protected $site;
    protected $robotsTxt;
    protected $robotsTxtLastUpdated;
    protected $lastCrawled;

    public function __construct($site)
    {
        $this-&gt;site = $site;
    }

    /**
     * Checks whether it is OK to crawl a given URI at this time.
     * @param   $uri                  string  URL to check, e.g. /my/page
     * @param   $ignoreCrawlDelays    bool    Ignore crawl delays - Used to check whether or not a page can be crawled at any time.
     * @return  bool
     */
    public function isOkToCrawl($uri, $ignoreCrawlDelays = false)
    {
        // Check that our robots.txt is sufficiently up to date:
        $botUserAgent               = $this-&gt;_getBotUserAgent();
        $lastRobotsUpdateThreshold  = new \DateTime(self::ROBOTS_TXT_LIFETIME);

        if(empty($this-&gt;robotsTxtLastUpdated) || $this-&gt;robotsTxtLastUpdated &lt; $lastRobotsUpdateThreshold) {             $this-&gt;robotsTxt            = $this-&gt;_parseRobotsTxt($this-&gt;_fetchRobotsTxt());
            $this-&gt;robotsTxtLastUpdated = new \DateTime();
        }

        // No rules? Free for all:
        if(!count($this-&gt;robotsTxt)) {
            return true;
        }

        foreach($this-&gt;robotsTxt as $userAgent =&gt; $botRules)
        {
            // No disallows and no crawl delay? Ignore this ruleset.
            if((empty($botRules['disallow']) || !count($botRules['disallow'])) &amp;&amp; empty($botRules['crawl-delay'])) {
                continue;
            }

            // If the user agent matches ours, or is a catch-all, then process the rules:
            if($userAgent == '*' || preg_match('/' . $userAgent . '/i', $botUserAgent)) {
                foreach($botRules['disallow'] as $rule) {
                    $disallow = $rule;
                    $disallow = preg_quote($disallow, '/');
                    $disallow = (substr($disallow, -1) != '*' &amp;&amp; substr($disallow, -1) != '$') ? $disallow . '*' : $disallow;
                    $disallow = str_replace(array('\*', '\$'), array('*', '$'), $disallow);
                    $disallow = str_replace('*', '(.*)?', $disallow);

                    if(preg_match('/^' . $disallow . '/i', $uri)) {
                        return false;
                    }
                }

                // Process crawl delay rules (unless we are ignoring them):
                if(!$ignoreCrawlDelays &amp;&amp; !empty($botRules['crawl-delay'])) {
                    $lastCrawlThreshold = new \DateTime('-' . $botRules['crawl-delay'] . ' SECOND');

                    if(!empty($this-&gt;lastCrawled) &amp;&amp; $this-&gt;lastCrawled &gt; $lastCrawlThreshold) {
                        return false;
                    }

                }
            }
        }

        return true;
    }

    /**
     * Gets our crawler user agent.
     * @return string
     */
    protected function _getBotUserAgent()
    {
        return 'Mozilla/5.0 (compatible; YourBotUserAgent/1.0; +http://www.yoursite.com/about-our-crawler)';
    }

    /**
     * Fetches the contents of a the site's robots.txt:
     * @return string
     * @throws \RuntimeException
     */
    protected function _fetchRobotsTxt()
    {
        return file_get_contents($this-&gt;site . '/robots.txt');
    }

    /**
     * Parses the robots.txt file content into a rules array.
     * @param $rules string
     * @return array
     */
    protected function _parseRobotsTxt($rules)
    {
        $rules      = explode("\n", str_replace("\r", "", $rules));
        $outRules   = array();

        $lastUserAgent = '*';
        foreach($rules as $rule)
        {
            if(trim($rule) == '') {
                continue;
            }

            if(strpos($rule, ':') === false) {
                continue;
            }

            $key = strtolower(trim(substr($rule, 0, strpos($rule, ':'))));
            $val = trim(substr($rule, strpos($rule, ':') + 1));

            if($key == 'user-agent') {
                $lastUserAgent = $val;
            }

            if(!isset($outRules[$lastUserAgent])) {
                $outRules[$lastUserAgent] = array();
            }

            if($key == 'disallow') {
                $outRules[$lastUserAgent]['disallow'][] = $val;
            }

            if($key == 'crawl-delay') {
                $outRules[$lastUserAgent]['crawl-delay'] = (float)$val;
            }
        }

        // Empty 'Disallow' means, effectively, allow all - so clear other rules.
        foreach($outRules as $ua =&gt; &amp;$userAgent)
        {
            if(!isset($userAgent['disallow'])){
                continue;
            }

            foreach($userAgent['disallow'] as $rule) {
                if($rule == '') {
                    $userAgent['disallow'] = array();
                    break;
                }
            }
        }

        return $outRules;
    }
}
```

**Note: **The above code snippet is for example purposes and should be used as a guide only.

To break the above code down:

* **isOkToCrawl()** is the public interface to this class, we first check when we last accessed robots.txt and update it if necessary, then we use the rules to check whether the given URL is disallowed, and finally we check whether we're allowed to crawl yet, based on the crawl-delay setting.

* **_getBotUserAgent()** would provide the code to get your bot user agent from a config file or similar.

* **_fetchRobotsTxt()** would be your application-specific code to fetch the contents of a URL, in this case we're just using a really simple file_get_contents() request.

* **_parseRobotsTxt()** takes the contents of a robots.txt file and breaks it down into a set of rules for a set of user agents.

I hope this article helps provide a foundation for implementing a respectful and well behaved web crawler in your PHP applications. If you have any questions, please feel free to ask!