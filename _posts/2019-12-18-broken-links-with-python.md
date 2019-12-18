---
layout: post
title:  "testing with python scrapy"
date:   2019-12-18 11:27:00 +0200
categories: "testing"
---
# python web crawler using scrapy to check for broken links
**web crawlers are fascinating in terms or auditing a website, they are automated, fast and efficient**
 
in this article I will provide instructions to build a super simple out of the box web crawler using python and scrapy library to crawl through a given site and generate a .csv report with broken links 

## pre-requisites
for this article I will use [python3](https://realpython.com/installing-python/), so make sure that is installed.

## getting started
first, lets create a project folder and setup a [python environment](https://realpython.com/python-virtual-environments-a-primer/).
```bash
$ mkdir web-crawler && cd web-crawler
$ python3 -m venv venv
$ . venv/bin/activate
```

then we will install all our dependencies, in this case we just need scrapy:
```bash
$ pip install scrapy
```

now, we will create a script that will run the crawler. At this point I will sugges using a content editor (vscode, sublime, pyCharm, notepad++), but I will create the file using the terminal. 
``` bash
$ touch script.py
```

let's open the file and start scripting.

[!NOTE]
This is python be careful about indentation.

### imports
these are the list of modules that we will need from scrapy. 
```python
from scrapy.spiders import CrawlSpider, Rule
from scrapy.linkextractors import LinkExtractor
from scrapy.selector import Selector
from scrapy.item import Item, Field

```

### model
*Scrapy.CrawlSpider* require that we return an *Item* object, this will contain the data that will be reported.
```python
class MyItems(Item):
    referer =Field() # where the link is extracted
    response= Field() # url that was requested
    status = Field() # status code received
```

### crawlspider class
scrapy provides an *out of the box* web crawler called *CrawlSpider* that will crawl the given site based on the defined configuration.

```python
class MySpider(CrawlSpider):
    name = "test-crawler"
    target_domains = ["dev.to"] # list of domains that will be allowed to be crawled
    start_urls = ["https://dev.to/"] # list of starting urls for the crawler
    handle_httpstatus_list = [404,410,301,500] # only 200 by default. you can add more status to list

    # Throttle crawl speed to prevent hitting site too hard
    custom_settings = {
        'CONCURRENT_REQUESTS': 2, # only 2 requests at the same time
        'DOWNLOAD_DELAY': 0.5 # delay between requests
    }

    rules = [
        Rule(
            LinkExtractor( allow_domains=target_domains, deny=('patterToBeExcluded'), unique=('Yes')), 
            callback='parse_my_url', # method that will be called for each request
            follow=True),
        # crawl external links but don't follow them
        Rule(
            LinkExtractor( allow=(''),deny=("patterToBeExcluded"),unique=('Yes')),
            callback='parse_my_url',
            follow=False
        )
    ]
```

the rules explained above are the way the links will be extracted from each page, so:
1. the first rule says: extract all unique links under the target_domains and follow them, but exclude
those who contains *patterToBeExcluded*.
2. the second rule says: extract all unique links but do not follow them and exclude
those who contains *patterToBeExcluded*.

**why 2 rules?** in this case we want to make sure our site is not hitting external links for broken or 404 pages. for example:
- www.oursite.com -> www.google.com/this/does/not/exist

### the callback
this is the method that will be called for each link that gets requested. every item that will be returned will be added to the csv report. so here is where can filter out only what we need to report. 
```python
    def parse_my_url(self, response):
      # list of response codes that we want to include on the report, we know that 404
      report_if = [404] 
      if response.status in report_if: # if the response matches then creates a MyItem
          item = MyItems()
          item['referer'] = response.request.headers.get('Referer', None)
          item['status'] = response.status
          item['response']= response.url
          yield item
      yield None # if the response did not match return empty
```

knowing that 404 is the **not found page** code, everytime the web crawler hits a page and response 404 then a row will be added to the csv report. modify this list as requested.

### running the crawler
so running the crawler is really simple
```
$ scrapy runspider script.py -o report-file.csv
```

### look at the report
during the execution of the crawler the `report-file.csv` will be populated.

![test report](https://github.com/pjcalvo/pjcalvo.github.io/blob/master/resources/web-crawler-report.png?raw=true)

## more
please read more about the library on their official site, it is full of really useful information, how to deploy,
creating custom spiders, and much more.

[scrapy official documentation](https://docs.scrapy.org/en/latest/index.html)<br>
full code can be found [here](https://github.com/pjcalvo/pjcalvo.github.io/blob/master/samples/web-crawler/script.py)

## final notes
this is a relly simple way to crawl and find broken links, as I see it there is tons of room for adding custom features and custom checks.
please be aware that this is a super simple script, so don't ask for best practices, scalability or anything out of the scope of this post. this tool is meant to be a starting point so that you can build a customized script that will suite your needs properly.

[!NOTE]
Many many pages out there will block most of the crawlers unless they provide explicit rules, please look at `robots.txt` file
that typically leave under the base domain for rules and guidelines. [https://www.adobe.com/robots.txt](https://www.adobe.com/robots.txt)


cheers :) and happy crawling
