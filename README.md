Scrapy
======

Using scrapy to scrape sites

Notes on spiders and spider creation in Scrapy:
It is highly recoommended to avoid scraping data on sites where there are user accounts like most e-commerce sites for example. It's much easier, faster and smarter to get root access to the server and/or direct access to the origin site's database.

Step 0: Learn Python
Scrapy is written in Python, and to create a spider you must (or really should) know Python. If you don't know Python, it's going to be a lot harder to figure out. That being said, there are many tutorials for both Scrapy and Python online.

Scrapy docs are here:
http://scrapy.org/doc/
http://doc.scrapy.org/en/latest/

Python documentation is here:
https://www.python.org/

Step 1: Create a Scrapy Project:
scrapy startproject {projectname}

for example, if you wanted to create a spider from the command line for questar, log into the server, go to the scrapy directory (currently at /home/airdisa/scrapy) and type:
scrapy startproject questar

... this will create some directories  with a structure like this:
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

Step 2: Create the Spider:
Assuming you're still in the 'scrapy' directory, to create a spider for questar, run the command: vi questar/questar/spiders/questar_spider.py which will open a new text file for you to start creating the spider. Now comes the hard part. To create a spider using Scrapy, you should know how to code in Python.

 - First, we need to import the libraries we'll be using for the spider. This will vary depending on the scrape requirements, but you must always import the projects item class (from questar.items import QuestarItem in our example):

-------------
from scrapy.selector import HtmlXPathSelector
from scrapy.contrib.linkextractors.sgml import SgmlLinkExtractor
from scrapy.contrib.spiders import CrawlSpider, Rule
from questar.items import QuestarItem

-------------

 - Next we create the spider class and pass it the scrapy object we want it to be based on; which in this case is the CrawlSpider:
        - http://doc.scrapy.org/en/latest/topics/spiders.html?highlight=crawlspider#crawlspider
 - Then we set up some initial values:
        - name: the name of the project. for our example, we are naming it 'questar'
        - allowed_comains: this prevents the spider from following offsite links. We don't want to crawl the entire internet.
        - start_urls: a list of urls to start from. Because we're scraping the entire site, we're starting at the very top.
        - rules: a list of one (or more) Rule objects. Each Rule defines a certain behaviour for crawling the site. Rules objects are described here - http://doc.scrapy.org/en/latest/topics/spiders.html?highlight=crawlspider#crawling-rules . If multiple rules match the same link, the first one will be used, according to the order they’re defined in this attribute.

-------------
class QuestarSpider(CrawlSpider):

        name = "questar"
        allowed_domains = ["questar.435digitalpreview.com"]
        start_urls = [
                "http://questar.435digitalpreview.com/"
        ]

        rules = (
                Rule (SgmlLinkExtractor(allow='/'), callback="parse_items", follow=True),
        )

-------------

You can output the results of your spider in several ways but for the purposes of this document we are scraping for WordPress items for WooCommerce. Wordpress imports and exports XML files, and we'll be looking at those for this tutorial.

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
                        item['post_title'] = titles.select('//div/h1[contains(@class, "product_title")]').extract()
                        item['post_link'] = response.url
                        item['publish_date'] = titles.select('//p/a/img[contains(@class, "wp-image")]').extract()
                        item['post_content'] = titles.select('//div[contains(@itemprop, "description")]/p/text()').extract()
                        item['author'] = titles.select('//a[contains(@rel,"tag")]').extract()
                        item['category'] = titles.select('//a[contains(@rel,"tag")]').extract()
                        #item[''] = titles.select().extract()
                        items.append(item)
                return(items)

Items:
http://doc.scrapy.org/en/latest/topics/items.html
"Item objects are simple containers used to collect the scraped data. They provide a dictionary-like API with a convenient syntax for declaring their available fields."

Items are declared in the items.py file (in questar/questar/items.py in our example version)  using the Field() syntax. Items are what is scraped with the scrapy spider and usually come in the form of python lists. The default items.py file has nothing in it and items must be added:

Example items.py:

-------------
# Define here the models for your scraped items
#
# See documentation in:
# http://doc.scrapy.org/en/latest/topics/items.html

from scrapy.item import Item, Field

class QuestarItem(scrapy.Item):
    post_title = scrapy.Field()
    post_link = scrapy.Field()
    publish_date = scrapy.Field()
    post_content = scrapy.Field()
    author = scrapy.Field()
    category = scrapy.Field()

-------------

The Scrapy documention: "Item objects are simple containers used to collect the scraped data. They provide a dictionary-like API with a convenient syntax for declaring their available fields."

Settings:
http://doc.scrapy.org/en/latest/topics/settings.html
Before running the spider it is important to check out the settings.py file (located in questar/questar/settings.py in our example). The settings file is important because it tells you where the pipeline is located (if you have one) and can be used to set up a number of other things.

The Item Pipeline:
http://doc.scrapy.org/en/latest/topics/item-pipeline.html
The pipeline is what you do to data after you've scraped it. This can be used to insert items into a database, for example.

NOTES:
http://doc.scrapy.org/en/0.16/topics/exporters.html#xmlitemexporter - this is important!
