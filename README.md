# GoogleScraper - Scraping search engines professionally

## New News - August 2018

This project is back to live after two years of abandonment. In the coming weeks, I will take some time to update all functionality to the most recent developments. This encompasses updating all Regexes and changes in search engine behavior. After a couple of weeks, you can expect this project to work again as documented here.

#### Goals

+ Remove PhantomJS support **[Done]**
+ Maybe remove Firefox support **[Done: Decision was to keep Firefox]**
+ Think about using https://github.com/GoogleChrome/puppeteer instead of selenium for Chrome.
+ Test all supported search engines

### Table of Contents

1. [Installation](#install)
2. [Quick Start](#quick)
3. [Asynchronous mode](#async)
4. [About](#about)
5. [Command line usage](#cli-usage)
6. [Contact](#contact)


<a name="install" />

## Installation

GoogleScraper is written in Python 3. You should install at least Python 3.6. The last major development was all done with Python 3.7. So when using Ubuntu 16.04 and Python 3.7 for instance, please install Python 3 from the official packages. I use the [Anaconda Python distribution](https://anaconda.org/anaconda/python), which does work very well for me.

Furthermore, you need to install the Chrome Browser and also the ChromeDriver for Selenium mode. Alternatively install the Firefox Browser and the geckodriver for Selenium Mode. See instructions below.

You can also install GoogleScraper comfortably with pip:

```
virtualenv --python python3 env
source env/bin/activate
pip install GoogleScraper
```

Right now (September 2018) this is discouraged. Please install from latest Github sources.

### Alternatively install directly from Github

Sometimes the newest and most awesome stuff is not available in the cheeseshop (That's how they call
https://pypi.python.org/pypi/pip). Therefore you maybe want to install GoogleScraper from the latest source that resides in this Github repository. You can do so like this:

```
virtualenv --python python3 env
source env/bin/activate
pip install git+git://github.com/NikolaiT/GoogleScraper/
```

Please note that some features and examples might not work as expected. I also don't guarantee that
the app even runs. I only guarantee (to a certain degree at least) that installing from pip will yield a
usable version.


### Chromedriver

Download the latest chromedriver from here: https://sites.google.com/a/chromium.org/chromedriver/downloads

Unzip the driver and save it somewhere and then update the `chromedriver_path` in the GoogleScraper configuration file `scrape_config.py` to the path where you saved the driver `chromedriver_path = 'Drivers/chromedriver'`


### Geckodriver

Download the latest geckodriver from here: https://github.com/mozilla/geckodriver/releases

Unzip the driver and save it somewhere and then update the `geckodriver_path` in the GoogleScraper configuration file `scrape_config.py` to the path where you saved the driver `geckodriver_path = 'Drivers/geckodriver'`

### Update the settings for selenium and firefox/chrome

Update the following settings in the GoogleScraper configuration file `scrape_config.py` to your values.

```
# chrome driver executable path
# get chrome drivers here: https://chromedriver.storage.googleapis.com/index.html?path=2.41/
chromedriver_path = 'Drivers/chromedriver'

# geckodriver executable path
# get gecko drivers here: https://github.com/mozilla/geckodriver/releases
geckodriver_path = 'Drivers/geckodriver'

# path to firefox binary
firefox_binary_path = '/home/nikolai/firefox/firefox'

# path to chromium browser binary
chrome_binary_path = '/usr/bin/chromium-browser'
```


<a name="quick" />

## Quick Start

Install as described above. Make sure that you have the selenium drivers for chrome/firefox if you want to use GoogleScraper in selenium mode.

See all options
```
GoogleScraper -h
```

Scrape the single keyword "apple" with http mode:
```
GoogleScraper -m http --keyword "apple" -v info
```

Scrape all keywords that are in the file `SearchData/5words` in selenium mode using chrome in headless mode:
```
GoogleScraper -m selenium --sel-browser chrome --browser-mode headless --keyword-file SearchData/5words -v info
```

Scrape all keywords that are in
+ keywords.txt
+ with http mode
+ using 5 threads
+ scrape in the search engines bing and yahoo
+ store the output in a JSON file
+ increase verbosity to the debug level
```
GoogleScraper -m http --keyword-file SearchData/some_words.txt --num-workers 5 --search-engines "bing,yahoo" --output-filename threaded-results.json -v debug
```

Do an image search for the keyword "K2 mountain" on google:

```
GoogleScraper -s "google" -q "K2 mountain" -t image -v info
```

<a name="async" />

## Asynchronous mode

This is probably the most awesome feature of GoogleScraper. You can scrape with thousands of requests per second if either

+ The search engine doesn't block you (Bing didn't block me when requesting **100 keywords / second**)
+ You have enough proxies

Example for Asynchronous mode:

Search the keywords in the keyword file [SearchData/marketing-models-brands.txt](SearchData/marketing-models-brands.txt) on bing and yahoo. By default asynchronous mode spawns 100 requests at the same time. This means around 100 requests per second (depends on the actual connection...).

```
GoogleScraper -s "bing,yahoo" --keyword-file SearchData/marketing-models-brands.txt -m http-async -v info -o marketing.json
```

The results (partial results, because there were too many keywords for one IP address) can be inspected [here](Output/marketing.json).


<a name="about" />

## What does GoogleScraper.py?

GoogleScraper parses Google search engine results (and many other search engines *_*) easily and in a fast way. It allows you to extract all found
links and their titles and descriptions programmatically which enables you to process scraped data further.

There are unlimited *usage scenarios*:

+ Quickly harvest masses of [google dorks][1].
+ Use it as a SEO tool.
+ Discover trends.
+ Compile lists of sites to feed your own database.
+ Many more use cases...
+ Quite easily extendable since the code is well documented

First of all you need to understand that GoogleScraper uses **two completely different scraping approaches**:
+ Scraping with low level http libraries such as `urllib.request` or `requests` modules. This simulates the http packets sent by real browsers.
+ Scrape by controlling a real browser with the selenium framework

Whereas the former approach was implemented first, the later approach looks much more promising in comparison, because
search engines have no easy way detecting it.

GoogleScraper is implemented with the following techniques/software:

+ Written in Python 3.7
+ Uses multithreading/asynchronous IO.
+ Supports parallel scraping with multiple IP addresses.
+ Provides proxy support using [socksipy][2] and built in browser proxies:
  * Socks5
  * Socks4
  * HttpProxy
+ Support for alternative search modes like news/image/video search.

### What search engines are suppported ?
Currently the following search engines are supported:
+ Google
+ Bing
+ Yahoo
+ Yandex
+ Baidu
+ Duckduckgo

### How does GoogleScraper maximize the amount of extracted information per IP address?

Scraping is a critical and highly complex subject. Google and other search engine giants have a strong inclination
to make the scrapers life as hard as possible. There are several ways for the search engine providers to detect that a robot is using
their search engine:

+ The User-Agent is not one of a browser.
+ The search params are not identical to the ones that browser used by a human sets:
  * Javascript generates challenges dynamically on the client side. This might include heuristics that try to detect human behaviour. Example: Only humans move their mouses and hover over the interesting search results.
+ Robots have a strict requests pattern (very fast requests, without a random time between the sent packets).
+ Dorks are heavily used
+ No pictures/ads/css/javascript are loaded (like a browser does normally) which in turn won't trigger certain javascript events

So the biggest hurdle to tackle is the javascript detection algorithms. I don't know what Google does in their javascript, but I will soon investigate it further and then decide if it's not better to change strategies and
switch to a **approach that scrapes by simulating browsers in a browserlike environment** that can execute javascript. The networking of each of these virtual browsers is proxified and manipulated such that it behaves like
a real physical user agent. I am pretty sure that it must be possible to handle 20 such browser sessions in a parallel way without stressing resources too much. The real problem is as always the lack of good proxies...

### How to overcome difficulties of low level (http) scraping?

As mentioned above, there are several drawbacks when scraping with `urllib.request` or `requests` modules and doing the networking on my own:

Browsers are ENORMOUSLY complex software systems. Chrome has around 8 millions line of code and firefox even 10 LOC. Huge companies invest a lot of money to push technology forward (HTML5, CSS3, new standards) and each browser
has a unique behaviour. Therefore it's almost impossible to simulate such a browser manually with HTTP requests.  This means Google has numerous ways to detect anomalies and inconsistencies in the browsing usage. Alone the
dynamic nature of Javascript makes it impossible to scrape undetected.

This cries for an alternative approach, that automates a **real** browser with Python. Best would be to control the Chrome browser since Google has the least incentives to restrict capabilities for their own native browser.
Hence I need a way to automate Chrome with Python and controlling several independent instances with different proxies set. Then the output of result grows linearly with the number of used proxies...

Some interesting technologies/software to do so:
+ [Selenium](https://pypi.python.org/pypi/selenium)
+ [Mechanize](http://wwwsearch.sourceforge.net/mechanize/)


<a name="cli-usage" />

## More detailed Explanation

Probably the best way to use GoogleScraper is to use it from the command line and fire a command such as
the following:
```
GoogleScraper --keyword-file /tmp/keywords --search-engine bing --num-pages-for-keyword 3 --scrape-method selenium
```

Here *sel* marks the scraping mode as 'selenium'. This means GoogleScraper.py scrapes with real browsers. This is pretty powerful, since
you can scrape long and a lot of sites (Google has a hard time blocking real browsers). The argument of the flag `--keyword-file` must be a file with keywords separated by
newlines. So: For every google query one line. Easy, isnt' it?

Furthermore, the option `--num-pages-for-keyword` means that GoogleScraper will fetch 3 consecutive pages for each keyword.

Example keyword-file:
```
keyword number one
how to become a good rapper
inurl:"index.php?sl=43"
filetype:.cfg
allintext:"You have a Mysql Error in your"
intitle:"admin config"
Best brothels in atlanta
```

After the scraping you'll automatically have a new sqlite3 database in the named `google_scraper.db` in the same directory. You can open and inspect the database with the command:
```
GoogleScraper --shell
```

It shouldn't be a problem to scrape **_10'000 keywords in 2 hours_**. If you are really crazy, set the maximal browsers in the config a little
bit higher (in the top of the script file).

If you want, you can specify the flag `--proxy-file`. As argument you need to pass a file with proxies in it and with the following format:

```
protocol proxyhost:proxyport username:password
(...)
```
Example:
```
socks5 127.0.0.1:1080 blabla:12345
socks4 77.66.55.44:9999 elite:js@fkVA3(Va3)
```

In case you want to use GoogleScraper.py in *http* mode (which means that raw http headers are sent), use it as follows:

```
GoogleScraper -m http -p 1 -n 25 -q "white light"
```

<a name="contact" />

## Contact

If you feel like contacting me, do so and send me a mail. You can find my contact information on my [blog][3].

[1]: http://www.webvivant.com/google-hacking.html "Google Dorks"
[2]: https://code.google.com/p/socksipy-branch/ "Socksipy Branch"
[3]: http://incolumitas.com/about/contact/ "Contact with author"
[4]: http://incolumitas.com/2013/01/06/googlesearch-a-rapid-python-class-to-get-search-results/
[5]: http://incolumitas.com/2014/11/12/scraping-and-extracting-links-from-any-major-search-engine-like-google-yandex-baidu-bing-and-duckduckgo/
