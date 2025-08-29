
# Deprecated

During Elon Musk took over Twitter (current company name is X), the API changed a lot, hence maybe this script cannot correctly use the former free API. This script is just for record, and it's not feasible anymore.



## Twitter Trends

##### 需求

2023-05-07 要求单条推特的多个cve不要;分割放入，而应一条一条

鹏哥说开发需要cve是一条一条的，而非'cve-xxxx;cve-xxx'的存储，这样会导致数据冗余。我改为可以重复插入推特id、推特text，但是cve不一样 的这种。

##### 结论

通过调研，我们在注册twitter的开发者平台后，直接可以免费、直接使用twitter standard v1.1。使用`search/tweets`接口搜索`cve-`的关键字，利用正则匹配cve，实时搜索到当前要变火热的cve趋势。[参考链接:search/tweets接口文档](https://developer.twitter.com/en/docs/twitter-api/v1/tweets/search/api-reference/get-search-tweets)

该接口近乎实时从twitter中获取数据，比如我们搜`CVE`关键字，可以搜到近5min-2h别人发推的相关内容。

速率限制：`GET search/tweets`，`user`为每15分钟180个请求，`app`为每15分钟450个请求。[参考链接:速率限制](https://developer.twitter.com/en/docs/twitter-api/v1/rate-limits)

`GET statuses/show`每15分钟900个请求，[该接口文档](https://developer.twitter.com/en/docs/twitter-api/v1/tweets/post-and-engage/api-reference/get-statuses-show-id)

##### 调研时twitter相关的东西

![img.png](img/img.png)

![img_1.png](img/img_1.png)

##### 编码设计

TwitterAPI [git地址](https://github.com/geduldig/TwitterAPI) 库是7年前推特官方的产物，很多接口获取的数据格式都变了。选定此库是因为他简单好用，但是在此基础上我们需要对其修改。

我们围绕官方文档[推特API search-tweets接口文档](https://developer.twitter.com/en/docs/twitter-api/v1/tweets/search/api-reference/get-search-tweets)修改TwitterAPI。

以两个小时为间隔，由于该接口速率为`180reqs/15min`，所以两个小时总共可以发起1440个请求，每个请求可以获取100条推特数据，所以总共可以获取144000条推特数据。

在请求时，如果在关键字搜索推文中搜到推文，但是推文中没有遇到cve关键字，且推文是被截断的（含有`…`字符），则使用`GET statuses/show`接口查询对应的推特id，获取未截断的推文数据。

search-tweets API可选项：

1. 限定位置搜索：指定经纬度及半径内用户
2. 限定语言搜索

##### 单条推文内容示例

参考[推特API search-tweets接口文档](https://developer.twitter.com/en/docs/twitter-api/v1/tweets/search/api-reference/get-search-tweets)

比对截断情况和非截断推文内容，如下

```
statuses/show:获取full_text
{'created_at': 'Fri May 05 22:12:22 +0000 2023', 'id': 1654610012457648129, 'id_str': '1654610012457648129', 'full_text': 'More actors are exploiting unpatched CVE-2023-27350 in print management software Papercut since we last reported on Lace Tempest. Microsoft has now observed Iranian state-sponsored threat actors Mint Sandstorm (PHOSPHORUS) &amp; Mango Sandstorm (MERCURY) exploiting CVE-2023-27350.', 'truncated': False, 'display_text_range': [0, 281], 'entities': {'hashtags': [], 'symbols': [], 'user_mentions': [], 'urls': []}, 'source': '<a href="https://prod1.sprinklr.com" rel="nofollow">Sprinklr Publishing</a>', 'in_reply_to_status_id': None, 'in_reply_to_status_id_str': None, 'in_reply_to_user_id': None, 'in_reply_to_user_id_str': None, 'in_reply_to_screen_name': None, 'user': {'id': 217462908, 'id_str': '217462908', 'name': 'Microsoft Threat Intelligence', 'screen_name': 'MsftSecIntel', 'location': 'Redmond, WA', 'description': "We are Microsoft's global network of security experts. Follow for security research and threat intelligence.", 'url': 'https://t.co/Qx50p5zXgk', 'entities': {'url': {'urls': [{'url': 'https://t.co/Qx50p5zXgk', 'expanded_url': 'https://aka.ms/threatintelblog', 'display_url': 'aka.ms/threatintelblog', 'indices': [0, 23]}]}, 'description': {'urls': []}}, 'protected': False, 'followers_count': 166318, 'friends_count': 1053, 'listed_count': 2433, 'created_at': 'Fri Nov 19 16:03:07 +0000 2010', 'favourites_count': 2044, 'utc_offset': None, 'time_zone': None, 'geo_enabled': False, 'verified': False, 'statuses_count': 4870, 'lang': None, 'contributors_enabled': False, 'is_translator': False, 'is_translation_enabled': False, 'profile_background_color': '0074C6', 'profile_background_image_url': 'http://abs.twimg.com/images/themes/theme14/bg.gif', 'profile_background_image_url_https': 'https://abs.twimg.com/images/themes/theme14/bg.gif', 'profile_background_tile': True, 'profile_image_url': 'http://pbs.twimg.com/profile_images/1268200269277351936/a2naHzbe_normal.png', 'profile_image_url_https': 'https://pbs.twimg.com/profile_images/1268200269277351936/a2naHzbe_normal.png', 'profile_banner_url': 'https://pbs.twimg.com/profile_banners/217462908/1681507015', 'profile_link_color': '0078D4', 'profile_sidebar_border_color': 'FFFFFF', 'profile_sidebar_fill_color': 'DDEEF6', 'profile_text_color': '333333', 'profile_use_background_image': False, 'has_extended_profile': False, 'default_profile': False, 'default_profile_image': False, 'following': False, 'follow_request_sent': False, 'notifications': False, 'translator_type': 'none', 'withheld_in_countries': []}, 'geo': None, 'coordinates': None, 'place': None, 'contributors': None, 'is_quote_status': False, 'retweet_count': 67, 'favorite_count': 141, 'favorited': False, 'retweeted': False, 'lang': 'en'}

search/tweets:存在截断情况
{'created_at': 'Fri May 05 22:12:22 +0000 2023', 'id': 1654610012457648129, 'id_str': '1654610012457648129', 'text': 'More actors are exploiting unpatched CVE-2023-27350 in print management software Papercut since we last reported on… https://t.co/LiQ2WCN9oe', 'truncated': True, 'entities': {'hashtags': [], 'symbols': [], 'user_mentions': [], 'urls': [{'url': 'https://t.co/LiQ2WCN9oe', 'expanded_url': 'https://twitter.com/i/web/status/1654610012457648129', 'display_url': 'twitter.com/i/web/status/1…', 'indices': [117, 140]}]}, 'source': '<a href="https://prod1.sprinklr.com" rel="nofollow">Sprinklr Publishing</a>', 'in_reply_to_status_id': None, 'in_reply_to_status_id_str': None, 'in_reply_to_user_id': None, 'in_reply_to_user_id_str': None, 'in_reply_to_screen_name': None, 'user': {'id': 217462908, 'id_str': '217462908', 'name': 'Microsoft Threat Intelligence', 'screen_name': 'MsftSecIntel', 'location': 'Redmond, WA', 'description': "We are Microsoft's global network of security experts. Follow for security research and threat intelligence.", 'url': 'https://t.co/Qx50p5zXgk', 'entities': {'url': {'urls': [{'url': 'https://t.co/Qx50p5zXgk', 'expanded_url': 'https://aka.ms/threatintelblog', 'display_url': 'aka.ms/threatintelblog', 'indices': [0, 23]}]}, 'description': {'urls': []}}, 'protected': False, 'followers_count': 166318, 'friends_count': 1053, 'listed_count': 2433, 'created_at': 'Fri Nov 19 16:03:07 +0000 2010', 'favourites_count': 2044, 'utc_offset': None, 'time_zone': None, 'geo_enabled': False, 'verified': False, 'statuses_count': 4870, 'lang': None, 'contributors_enabled': False, 'is_translator': False, 'is_translation_enabled': False, 'profile_background_color': '0074C6', 'profile_background_image_url': 'http://abs.twimg.com/images/themes/theme14/bg.gif', 'profile_background_image_url_https': 'https://abs.twimg.com/images/themes/theme14/bg.gif', 'profile_background_tile': True, 'profile_image_url': 'http://pbs.twimg.com/profile_images/1268200269277351936/a2naHzbe_normal.png', 'profile_image_url_https': 'https://pbs.twimg.com/profile_images/1268200269277351936/a2naHzbe_normal.png', 'profile_banner_url': 'https://pbs.twimg.com/profile_banners/217462908/1681507015', 'profile_link_color': '0078D4', 'profile_sidebar_border_color': 'FFFFFF', 'profile_sidebar_fill_color': 'DDEEF6', 'profile_text_color': '333333', 'profile_use_background_image': False, 'has_extended_profile': False, 'default_profile': False, 'default_profile_image': False, 'following': False, 'follow_request_sent': False, 'notifications': False, 'translator_type': 'none', 'withheld_in_countries': []}, 'geo': None, 'coordinates': None, 'place': None, 'contributors': None, 'is_quote_status': False, 'retweet_count': 67, 'favorite_count': 141, 'favorited': False, 'retweeted': False, 'lang': 'en'}
```

##### 表单设计

存储字段：
```
twitter_id
cve_id
text
created_at
username
cve
```

### 测试数据

以“2小时”为时间段爬行，本时间段处理的推特是上个“2个小时”段人们发出的推特。
如果“有效”大于上个2小时处理的推特数，则是存在单推特含多个CVE的原因，使得有效数量大于处理推特的数量。

5.7 处理
```
22:52--5.8 00:53的程序   爬上个两小时处理推特数量:214（有效：73）
```

5.8 处理
```
00:53--2:53的程序  	爬上个两小时处理推特数量:85（有效:52）
2:53--4:53的程序	爬上个两小时处理推特数量:34（有效:52）
4:53--6:53的程序	爬上个两小时处理推特数量:25（有效:31）
6:53--8:54的程序	爬上个两小时处理推特数量:36（有效:30）
8:53--10:54的程序	爬上个两小时处理推特数量:53（有效:26）
10:53--12:54的程序	爬上个两小时处理推特数量:72（有效:30）

```
