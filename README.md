Scrapy
======

##Using scrapy to scrape sites

#Notes on spiders and spider creation in Scrapy:
It is highly recoommended to avoid scraping data on sites where there are user accounts like most e-commerce sites for example. It's much easier, faster and smarter to get root access to the server and/or direct access to the origin site's database.


#Step 0: Learn Python
Scrapy is written in Python, and to create a spider you must (or really should) know Python. If you don't know Python, it's going to be a lot harder to figure out. That being said, there are many tutorials for both Scrapy and Python online. 
Scrapy docs are here:
(http://scrapy.org/doc/)
(http://doc.scrapy.org/en/latest/)

Python documentation is here:
(https://www.python.org/)

#Step 1: Create a Scrapy Project:
scrapy startproject {projectname}

For example, if you wanted to create a spider from the command line for questar, log into the server, go to the scrapy directory (currently at /home/airdisa/scrapy) and type:
scrapy startproject questar

... this will create some directories  with a structure like this:

```
questar/
    scrapy.cfg
    questar/
        __init__.py
        items.py
        pipelines.py
        settings.py
        spiders/
            __init__.py
	 ... etc
```

#IMPORTANT: 
Most things can be tested using the scrapy shell (http://doc.scrapy.org/en/latest/topics/shell.html). Learn about the scrapy shell, because chances are you'll be using it a lot to test ideas and debug code.


#Step 2: Create the Spider:
(http://doc.scrapy.org/en/latest/topics/spiders.html)
Assuming you're still in the 'scrapy' directory, to create a spider for questar, run the command: vi questar/questar/spiders/questar_spider.py which will open a new text file for you to start creating the spider. Now comes the hard part. To create a spider using Scrapy, you should know how to code in Python. 


First, we need to import the libraries we'll be using for the spider. This will vary depending on the scrape requirements, but you must always import the projects item class, from questar.items import QuestarItem in our example. The line 'from questar.items import QuestarItem' imports the QuestarItem class from the questar/questar/items.py file, explained in more detail later on in this document, which initializes the Field() objects for scraped elements. The other libraries/classes imported will depend on what you want to do, but you will almost always import the CrawlSpider class (shown in the line 'from scrapy.contrib.spiders import CrawlSpider, Rule' below). For more info on link extractors, start here: (http://doc.scrapy.org/en/latest/topics/link-extractors.html) . 


```
from scrapy.selector import HtmlXPathSelector
from scrapy.contrib.linkextractors.sgml import SgmlLinkExtractor
from scrapy.contrib.spiders import CrawlSpider, Rule
from questar.items import QuestarItem
```

- Next we create the spider class and pass it the scrapy object we want it to be based on; which in this case is the CrawlSpider:
  - (http://doc.scrapy.org/en/latest/topics/spiders.html?highlight=crawlspider#crawlspider)
- Then we set up some initial values:
  - name: the name of the project. for our example, we are naming it 'questar'
  - allowed_comains: this prevents the spider from following offsite links. We don't want to crawl the entire internet.
  - start_urls: a list of urls to start from. Because we're scraping the entire site, we're starting at the very top.
  - rules: a list of one (or more) Rule objects. Each Rule defines a certain behaviour for crawling the site. Rules objects are described here - (http://doc.scrapy.org/en/latest/topics/spiders.html?highlight=crawlspider#crawling-rules) . If multiple rules match the same link, the first one will be used, according to the order they’re defined in this attribute.


```
class QuestarSpider(CrawlSpider):

	name = "questar"
        allowed_domains = ["questar.435digitalpreview.com"]
        start_urls = [
                "http://questar.435digitalpreview.com/"
        ]

        rules = (
                Rule (SgmlLinkExtractor(allow='/'), callback="parse_items", follow=True),
        )
```

You can output the results of your spider in several ways but for the purposes of this document we are scraping for WordPress items for WooCommerce. The next thing we need to do is loop through pages and extract items. In our example, we're using XPath to get elements. Below is our first example for getting product posts from Questar. 

```
from scrapy.selector import HtmlXPathSelector
from scrapy.contrib.linkextractors.sgml import SgmlLinkExtractor
from scrapy.contrib.spiders import CrawlSpider, Rule
from questar.items import QuestarItem

class QuestarSpider(CrawlSpider):
        name = "questar"
        allowed_domains = ["questar.435digitalpreview.com"]
        start_urls = [
                "http://questar.435digitalpreview.com/"
        ]

        rules = (
                Rule (SgmlLinkExtractor(allow='/'), callback="parse_items", follow=True),
        )

        def parse_items(self, response):
                hxs = HtmlXPathSelector(response)
                titles = hxs.select('//title')
                items = []
                for titles in titles:
                        item = QuestarItem()
                        item['page_title'] = titles.select('/html/head/title/text()').extract()
                        item['post_title'] = titles.select('//div/h1[contains(@class, "product_title")]/text()').extract()
                        item['post_link'] = response.url
                        item['price'] = titles.select('//p/span[contains(@class, "amount")]/text()').extract()
                        item['publish_date'] = titles.select('//p/a/img[contains(@class, "wp-image")]').extract()
                        item['post_content'] = titles.select('//div[contains(@itemprop, "description")]/p/text()').extract()
                        item['description_tab'] = titles.select('//*[@id="tab-description"]/p/text()').extract()
                        item['author'] = titles.select('//a[contains(@rel,"tag")]').extract()
                        item['category'] = titles.select('//span[contains(@class, "posted_in")]/text()').extract()
                        item['tags'] = titles.select('//a[contains(@rel,"tag")]/text()').extract()
                        #item[''] = titles.select().extract()
                        items.append(item)
                return(items)
```

To run the spider above type 'scrapy crawl questar' in the command line from the questar project directory. The results for each URL should look something like this:

```
2014-07-16 09:18:35-0500 [questar] DEBUG: Crawled (200) <GET http://questar.435digitalpreview.com/shop/download/jamie-olivers-food-escapes-download-2/> (referer: http://questar.435digitalpreview.com/shop/dvds/jamie-olivers-food-escapes/)
2014-07-16 09:18:35-0500 [questar] DEBUG: Scraped from <200 http://questar.435digitalpreview.com/shop/download/jamie-olivers-food-escapes-download-2/>
	{'author': [u'<a href="http://questar.435digitalpreview.com/product-category/download/" rel="tag">Download</a>',
	            u'<a href="http://questar.435digitalpreview.com/product-tag/cooking/" rel="tag">Cooking</a>',
	            u'<a href="http://questar.435digitalpreview.com/product-tag/television-series/" rel="tag">Television series</a>',
	            u'<a href="http://questar.435digitalpreview.com/product-tag/travel/" rel="tag">Travel</a>'],
	 'category': [u'Category: ', u'.'],
	 'description_tab': [u'DVD Contains: All 6 episodes',
	                     u'\nDVD Package: 6 discs in a DVD collector\u2019s pack',
	                     u'\nRuntime: 4 hours',
	                     u'Produced By: Freemantle Media Enterprises',
	                     u'\nStarring: Jamie Oliver',
	                     u'\nAs seen on the Cooking Channel.'],
	 'page_title': [u"Jamie Oliver's Food Escapes Download | Questar EntertainmentQuestar Entertainment"],
	 'post_content': [u"For most of us a short break means museums, galleries or beaches but for Jamie Oliver it's all about food and he reckons you can learn a lot more about a place from how they cook and eat. Jamie proves that travel and food are the real color of life! \u2026\u2026\u2026\u2026\u2026..One of the world's most beloved television personalities, chef Jamie Oliver is inspiring an interest in all things culinary in a new generation. In Jamie Oliver's Food Escapes, he hits the road on an exciting expedition through the world's cultures and cuisines. Heading off the tourist track, Jamie finds authentic ingredients and extraordinary people while preparing delicious recipes. The television series that originally aired on the Cooking Channel, Jamie Oliver's Food Escapes visits Marrakesh, Athens, Andalucia, the French Pyrenees, Venice, and Stockholm to reveal some great foodie destinations and bring you lots of delicious recipes that will inspire you to get straight into the kitchen."],
	 'post_link': 'http://questar.435digitalpreview.com/shop/download/jamie-olivers-food-escapes-download-2/',
	 'post_title': [u'Jamie Oliver\u2019s Food Escapes Download'],
	 'price': [u'$2.49'],
	 'publish_date': [],
	 'tags': [u'Download', u'Cooking', u'Television series', u'Travel']}
```

... as you can see, the some things look correct and some don't; the output of the 'author' field for example. This is part of the process to get to the correct information. In the case of questar there is no author or publish date shown for each product post, so those fields can probably be removed from our items.py field list and the questar_spider.py spider and other xpaths could be added.

#Items:
(http://doc.scrapy.org/en/latest/topics/items.html)
Item objects are simple containers used to collect the scraped data. They provide a dictionary-like API with a convenient syntax for declaring their available fields.

Items are declared in the items.py file (in questar/questar/items.py in our example version)  using the Field() syntax. Items are what is scraped with the scrapy spider and usually come in the form of python lists. The default items.py file has nothing in it and items must be added:

Example items.py:

```
# Define here the models for your scraped items
#
# See documentation in:
# http://doc.scrapy.org/en/latest/topics/items.html

from scrapy.item import Item, Field

class QuestarItem(scrapy.Item):
    page_title = Field()
    post_title = Field()
    post_link = Field()
    price = Field()
    publish_date = Field()
    post_content = Field()
    description_tab = Field()
    author = Field()
    category = Field()
    tags = Field()
```

The Scrapy documention: "Item objects are simple containers used to collect the scraped data. They provide a dictionary-like API with a convenient syntax for declaring their available fields."

#Settings:
(http://doc.scrapy.org/en/latest/topics/settings.html)
Before running the spider it is important to check out the settings.py file (located in questar/questar/settings.py in our example). The settings file is important because it tells you where the pipeline is located (if you have one) and can be used to set up a number of other things. One of the most important settings to add before running your spider is the `DOWNLOAD DELAY` which causes a delay between each request. A good default is 2 seconds - DOWNLOAD_DELAY = 2, so now our settings.py file looks something like this:

```
# Scrapy settings for questar project
#
# For simplicity, this file contains only the most important settings by
# default. All the other settings are documented here:
#
#     http://doc.scrapy.org/en/latest/topics/settings.html
#

BOT_NAME = 'questar'

SPIDER_MODULES = ['questar.spiders']
NEWSPIDER_MODULE = 'questar.spiders'
DOWNLOAD_DELAY = 2

# Crawl responsibly by identifying yourself (and your website) on the user-agent
#USER_AGENT = 'questar (+http://www.yourdomain.com)'
```

#NOTE: 
You can export to XML by running the command: 'scrapy crawl questar -o items.xml -t xml' but the formatting of the XNL will likely have to be updated using the Scrapy Item Pipeline.

#The Item Pipeline:
(http://doc.scrapy.org/en/latest/topics/item-pipeline.html)
The pipeline is what you do to data after you've scraped it. This can be used to insert items into a database, for example.

#NOTES:
(http://doc.scrapy.org/en/0.16/topics/exporters.html#xmlitemexporter) - this is important!
