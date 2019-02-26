# Twitter爬虫：scrapy+redis
## 项目地址
## 环境准备
### 安装scrapy框架
```
sudo pip install scrapy
```
### 安装redis
鸟的话直接apt
```
sudo apt-get install redis-server
```
Mac的话直接brew
```
brew install redis-server
```
### 安装python的redis模块
```
sudo pip install redis-python
```
### 终端代理
爬推特使用http代理，所以需要在你的代理软件设置http
一种是，直接在你的```~/.bashrc```或者```~/.zshrc```添加
```
export http_proxy=http://127.0.0.1:1087
export https_proxy=$http_proxy
```
1087为你的http代理监听端口，
另一种是，使用proxychains进行终端代理，鸟已经自带，详细访问了社区
## 简单介绍一下scrapy
### scrapy是python的一个爬虫框架，主要包括八个部分
```
  引擎(Scrapy)
  调度器(Scheduler)
  下载器(Downloader)
  爬虫(Spiders)
  项目管道(Pipeline)
  下载器中间件(Downloader Middlewares)
  爬虫中间件(Spider Middlewares)
  调度中间件(Scheduler Middewares)
```
### scrapy运行过程
首先，scrapy从调度器中取出一个URL用于抓取，然后，scrapy把URL封装成一个Request请求传给downloader
downloader把资源下载下来，封装成Response，爬虫解析Response，解析到Item,交给管道进行处理，若解析出的是URL,就直接把URL交给调度器等待抓取
### 大概操作
生成爬虫项目
```
scrapy startproject demo
```
生成爬虫
```
cd demo
scrapy genspider demos demos.com
```
目录结构大概为：
demo
  +---- scrapy.cfg  这个文件主要存项目的配置信息
  |
  +---- items.py    这个文件用于存储数据的模板
  |
  +---- pipelines    这个文件主要写一些结构化的数据持久化相关的东西
  |
  +---- settings.py 爬虫的配置文件，这才是爬虫相关的配置文件，不要 混淆
  |
  +---- spiders      这个是爬虫目录，里边主要是爬虫文件，编写爬虫规则之类的

这里大概介绍一下scrapy，想详细了解的可以去看一下python3网络爬虫开发实战，貌似是 崔庆才大佬的，就介绍到这里
## 爬虫代码目录结构
推特爬虫
  +---twitterspider                                         
  |     |
  |     +---spiders
  |     |       |
  |     |       +--- __init__.py
  |     |       |
  |     |       +--- twitter_spider.py
  |     +---- __init__.py
  |     |
  |     +---- Bloomfilter.py
  |     |
  |     +---- items.py
  |     |
  |     +---- middleware.py
  |     |
  |     +---- pipelines.py
  |     |
  |     +---- settings.py
  |     |
  |     +---- spdierman.py
  |     |
  |     +---- user_agents.py
  +--- check_proxy.py           
  |     
  +--- dump.rdb                                
  |     
  +--- spidergo.py                   
  |     
  +---scrapy.cfg
  |     
  +---README.MD

### 使用方法
这里有个```check_proxy.py```,检测终端代理，
```
python check_proxy.py
```
看到如下提示，则终端代理成功
```
第一道检测闸放行
第二道检测闸检测中...
200
放行,代理可用！
···
爬虫之前先要把redis启起来,直接
```
redis-server
```
然后直接运行```spidergo.py```开启爬虫
```
python spidergo.py
```
```
➜  推特爬虫 git:(master) ✗ ✗ sudo python spidergo.py
Password:
2019-02-26 18:21:08 [scrapy.utils.log] INFO: Scrapy 1.6.0 started (bot: twitterspider)
2019-02-26 18:21:08 [scrapy.utils.log] INFO: Versions: lxml 4.3.1.0, libxml2 2.9.9, cssselect 1.0.3, parsel 1.5.1, w3lib 1.20.0, Twisted 18.9.0, Python 2.7.10 (default, Aug 17 2018, 19:45:58) - [GCC 4.2.1 Compatible Apple LLVM 10.0.0 (clang-1000.0.42)], pyOpenSSL 19.0.0 (OpenSSL 1.1.1a  20 Nov 2018), cryptography 2.5, Platform Darwin-18.2.0-x86_64-i386-64bit
2019-02-26 18:21:08 [scrapy.crawler] INFO: Overridden settings: {'NEWSPIDER_MODULE': 'twitterspider.spiders', 'SPIDER_MODULES': ['twitterspider.spiders'], 'DUPEFILTER_CLASS': 'scrapy.dupefilters.BaseDupeFilter', 'DOWNLOAD_DELAY': 2, 'BOT_NAME': 'twitterspider'}
2019-02-26 18:21:08 [scrapy.extensions.telnet] INFO: Telnet Password: fada391dc8e59611
2019-02-26 18:21:08 [scrapy.middleware] INFO: Enabled extensions:
['scrapy.extensions.memusage.MemoryUsage',
 'scrapy.extensions.logstats.LogStats',
 'scrapy.extensions.telnet.TelnetConsole',
 'scrapy.extensions.corestats.CoreStats']
2019-02-26 18:21:08 [scrapy.middleware] INFO: Enabled downloader middlewares:
['twitterspider.middleware.ProxyMiddleware',
 'twitterspider.middleware.UserAgentMiddleware',
 'twitterspider.middleware.CheckMiddleware',
 'scrapy.downloadermiddlewares.httpauth.HttpAuthMiddleware',
 'scrapy.downloadermiddlewares.downloadtimeout.DownloadTimeoutMiddleware',
 'scrapy.downloadermiddlewares.defaultheaders.DefaultHeadersMiddleware',
 'scrapy.downloadermiddlewares.useragent.UserAgentMiddleware',
 'scrapy.downloadermiddlewares.retry.RetryMiddleware',
 'scrapy.downloadermiddlewares.redirect.MetaRefreshMiddleware',
 'scrapy.downloadermiddlewares.httpcompression.HttpCompressionMiddleware',
 'scrapy.downloadermiddlewares.redirect.RedirectMiddleware',
 'scrapy.downloadermiddlewares.cookies.CookiesMiddleware',
 'scrapy.downloadermiddlewares.httpproxy.HttpProxyMiddleware',
 'scrapy.downloadermiddlewares.stats.DownloaderStats']
2019-02-26 18:21:08 [scrapy.middleware] INFO: Enabled spider middlewares:
['scrapy.spidermiddlewares.httperror.HttpErrorMiddleware',
 'scrapy.spidermiddlewares.offsite.OffsiteMiddleware',
 'scrapy.spidermiddlewares.referer.RefererMiddleware',
 'scrapy.spidermiddlewares.urllength.UrlLengthMiddleware',
 'scrapy.spidermiddlewares.depth.DepthMiddleware']
2019-02-26 18:21:08 [scrapy.middleware] INFO: Enabled item pipelines:
[]
2019-02-26 18:21:08 [scrapy.core.engine] INFO: Spider opened
2019-02-26 18:21:08 [scrapy.extensions.logstats] INFO: Crawled 0 pages (at 0 pages/min), scraped 0 items (at 0 items/min)
2019-02-26 18:21:08 [scrapy.extensions.telnet] INFO: Telnet console listening on 127.0.0.1:6023
爬虫初始化中...
URL:https://twitter.com/i/profiles/show/realdonaldtrump/timeline/tweets?include_available_features=1&include_entities=1&reset_error_state=false
**************ProxyMiddleware no pass************127.0.0.1:1087
获取请求头
Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2227.1 Safari/537.36
https://twitter.com/i/profiles/show/realdonaldtrump/timeline/tweets?include_available_features=1&include_entities=1&reset_error_state=false
爬虫运行正常
2019-02-26 18:21:09 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://twitter.com/i/profiles/show/realdonaldtrump/timeline/tweets?include_available_features=1&include_entities=1&reset_error_state=false> (referer: https://www.google.com)
1100211499350450176已存在,进行去重过滤
1100211495223218176已存在,进行去重过滤
1100184874466664448已存在,进行去重过滤
1100127553203798016已存在,进行去重过滤
1100126391729774592已存在,进行去重过滤
1100110416812724224已存在,进行去重过滤
1100110413209858048已存在,进行去重过滤
1100110409736978432已存在,进行去重过滤
1100105732450443265已存在,进行去重过滤
1100050798015401984已存在,进行去重过滤
1100048114382266374已存在,进行去重过滤
1100017237228949506已存在,进行去重过滤
1100015815515082752已存在,进行去重过滤
1100012620441100290已存在,进行去重过滤
1099722424059420673已存在,进行去重过滤
1099837054400253953已存在,进行去重过滤
1100002139282309121已存在,进行去重过滤
1100000030319169537已存在,进行去重过滤
1098948459405721600已存在,进行去重过滤
1099995611305254912已存在,进行去重过滤
下一页等待中...
URL:https://twitter.com/i/profiles/show/realdonaldtrump/timeline/tweets?include_available_features=1&include_entities=1&max_position=1099995611305254912&reset_error_state=false
**************ProxyMiddleware no pass************127.0.0.1:1087
获取请求头
```
