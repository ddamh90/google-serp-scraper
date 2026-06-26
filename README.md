# Google SERP API Python 完整指南：怎么抓取搜索结果？用哪个工具最稳？请求参数怎么配置？数据如何解析？（含 ScraperAPI 套餐对比与代码实战）

想用 Python 抓 Google 搜索结果，结果被封 IP 了？还是 CAPTCHA 一直过不去？这篇文章就聊这个事。

从最基础的"为什么直接 requests 抓 Google 行不通"，到"怎么用 Google SERP API 配合 Python 三行代码拿到结构化数据"，全部说清楚。

---

## 为什么直接用 requests 抓 Google 不行

很多人第一次做 SERP 采集，都走同一条路：打开 Python，写个 `requests.get('https://www.google.com/search?q=xxx')`，然后发现返回的是 CAPTCHA 页面，或者干脆 403。

Google 的反爬机制不是摆设。它会检测你的 User-Agent、请求频率、IP 地址、甚至 TLS 指纹。你的服务器 IP 一旦被标记，后续请求基本废掉。就算你用代理池，维护起来也是个无底洞——代理失效、轮换逻辑、异常重试，一套下来你写的是爬虫还是运维工具都分不清了。

这就是为什么现在做 Google SERP 数据采集，大家普遍转向 **Google SERP API**：把反爬、代理、CAPTCHA 这些脏活全交给第三方，自己只管发请求、拿 JSON。

---

## 什么是 Google SERP API

Google SERP API（Search Engine Results Page API）本质上是一个中间层服务。你把搜索关键词发过去，它帮你访问 Google、处理反爬逻辑、解析页面结构，然后把干净的 JSON 数据还给你。

你不用管代理、不用管 CAPTCHA、不用写解析代码，专注业务逻辑就行。

用途很广：SEO 关键词排名监控、竞品内容分析、广告投放情报收集、AI 训练数据采集、学术研究……只要你需要批量、稳定地获取 Google 搜索数据，SERP API 都是比自建爬虫靠谱得多的选择。

---

## ScraperAPI 的 Google SERP API：Python 开发者怎么用

市面上做 SERP API 的不少，但对 Python 开发者来说，**ScraperAPI** 是一个值得认真看的选项。它不仅提供了专门的 Google SERP 结构化数据端点，文档也相当清晰，配合 Python 的 `requests` 库用起来几乎没有学习成本。

👉 [免费注册 ScraperAPI，获取 5000 次免费 API 调用](https://www.scraperapi.com/?fp_ref=coupons)

### 核心端点

ScraperAPI 的 Google SERP 端点地址是：


https://api.scraperapi.com/structured/google/search


调用方式是标准的 GET 请求，把参数放进 `params` 字典里就行。

### 最简单的 Python 示例

python
import requests

payload = {
    'api_key': 'YOUR_API_KEY',
    'query': 'python web scraping',
    'country_code': 'us',
    'tld': 'com'
}

r = requests.get('https://api.scraperapi.com/structured/google/search', params=payload)
print(r.text)


四行有效代码，返回结果是结构化 JSON，包含自然搜索结果、广告、相关问题等字段，直接可用。

---

## 参数详解：怎么控制搜索行为

光知道怎么调用还不够，实际项目里你需要控制搜索的国家、语言、页码等细节。ScraperAPI 的 Google SERP API 支持以下关键参数：

| 参数 | 说明 | 示例值 |
|------|------|--------|
| `api_key` | 必填，你的 API Key | 注册后从控制台获取 |
| `query` | 必填，搜索关键词 | `python web scraping` |
| `tld` | Google 域名后缀，默认 `com` | `co.uk`、`de`、`co.jp` |
| `country_code` | 地理位置代码，用于 IP 定位 | `us`、`gb`、`au` |
| `output` | 输出格式，默认 JSON | `json` 或 `csv` |
| `num` | 返回结果数量 | `10`、`20` |
| `start` | 分页偏移，`start=10` 即第二页 | `0`、`10`、`20` |
| `hl` | 界面语言 | `de`、`fr`、`ja` |
| `gl` | 结果地理偏向 | `DE`、`FR` |
| `tbs` | 时间范围过滤 | `tbs=d`（过去一天）|

一个稍微完整点的例子，抓德国 Google 的德语结果：

python
import requests

payload = {
    'api_key': 'YOUR_API_KEY',
    'query': 'python web scraping',
    'tld': 'de',
    'country_code': 'de',
    'hl': 'de',
    'num': '20'
}

r = requests.get('https://api.scraperapi.com/structured/google/search', params=payload)
data = r.json()

for result in data.get('organic_results', []):
    print(result['position'], result['title'], result['link'])


---

## 异步批量抓取：一次提交多个查询

如果你要批量抓大量关键词，同步一个个发请求会很慢。ScraperAPI 提供了异步接口，把任务丢进去，等结果回来就行。

异步端点地址是：


https://async.scraperapi.com/structured/google/search


批量提交示例：

python
import requests

url = "https://async.scraperapi.com/structured/google/search"
headers = {"Content-Type": "application/json"}

data = {
    "apiKey": "YOUR_API_KEY",
    "queries": ["python tutorial", "web scraping tools", "SEO rank tracking"],
    "country_code": "us",
    "tld": "com"
}

response = requests.post(url, json=data, headers=headers)
print(response.text)


提交之后你会拿到任务 ID，后续轮询或者配置 webhook 拿结果。对于每天需要跑几千个关键词的项目，这个方式能省掉大量等待时间。

---

## 实战案例：用 Python 监控关键词排名

下面是一个完整的小脚本，输入关键词列表，输出每个词的前 10 名自然搜索结果，适合做 SEO 监控或竞品追踪：

python
import requests
import json

API_KEY = "YOUR_API_KEY"
BASE_URL = "https://api.scraperapi.com/structured/google/search"

keywords = [
    "python web scraping",
    "best serp api",
    "google search results api"
]

def get_serp(query, country="us"):
    payload = {
        "api_key": API_KEY,
        "query": query,
        "country_code": country,
        "num": "10"
    }
    r = requests.get(BASE_URL, params=payload)
    return r.json()

for kw in keywords:
    print(f"\n=== {kw} ===")
    results = get_serp(kw)
    for item in results.get("organic_results", [])[:5]:
        print(f"#{item['position']} {item['title']}")
        print(f"   {item['link']}")


跑起来之后你就有一个简单的排名追踪工具了。真实项目里可以把结果写入数据库、定时跑、对比历史变化趋势。

---

## 返回的 JSON 数据结构长什么样

ScraperAPI 的 Google SERP 接口返回标准化 JSON，主要字段包括：

- `organic_results`：自然搜索结果列表，每条包含 `position`（排名）、`title`（标题）、`link`（URL）、`snippet`（摘要）
- `ads`：付费广告结果
- `related_questions`：People Also Ask 问题框
- `related_searches`：相关搜索词
- `pagination`：分页信息，包含总页数和下一页 URL

这些字段直接拿来用，不需要写任何 HTML 解析代码。

---

## ScraperAPI 套餐对比：选哪个合适

ScraperAPI 提供 7 天免费试用，注册就送 5000 次 API 调用，不需要绑定信用卡。正式使用有以下套餐：

| 套餐 | 月付价格 | 年付价格（省10%）| API Credits | 并发线程 | 地理定位 | 推荐人群 |
|------|---------|----------------|-------------|---------|---------|---------|
| Hobby | $49/月 | $44.10/月 | 100,000 | 20 | 美国 & 欧盟 | 个人项目、小规模测试 |
| Startup | $149/月 | $134.10/月 | 1,000,000 | 50 | 美国 & 欧盟 | 轻量工作流、小团队 |
| Business | $299/月 | $269.10/月 | 3,000,000 | 100 | 全球 | 生产环境、中等规模 |
| Scaling | $475/月 | $427.50/月 | 5,000,000 | 200 | 全球 | 规模化采集，支持超额付费 |
| Professional | $975/月 | $877.50/月 | 10,500,000 | 300 | 全球 | 高频大批量，优先支持 |
| Advanced | $1,975/月 | $1,777.50/月 | 21,500,000 | 500 | 全球 | 多源持续数据管道 |
| Enterprise | 定制 | 定制 | 2200万+ | 500+ | 全球 | 大型企业，专属支持 |

几个选套餐的参考点：

- Google 搜索结果页每次抓取消耗 25 个 credits（Google 属于难度较高的站点）
- 100,000 credits ÷ 25 = 约 4000 次 Google SERP 请求，适合 Hobby 套餐做测试
- 需要全球 IP 地理定位（比如抓各国本地化排名）从 Business 套餐起才支持
- Scaling 及以上支持超量按需付费（Pay as you go），不会因为超配额就停掉

👉 [查看 ScraperAPI 完整套餐与价格](https://www.scraperapi.com/pricing/?fp_ref=coupons)

---

## 关于 Credits 消耗：Google 为什么比普通页面贵

很多人刚用的时候会问：为什么抓 Google 消耗的 credits 比普通网页多？

ScraperAPI 按页面难度计费。普通页面 1 credit，Amazon 5 credits，而 Google 和 Bing 的 SERP 页面需要 **25 credits** 一次，因为 Google 的反爬机制最复杂，ScraperAPI 需要调用专门的绕过逻辑和高质量代理。

如果你不确定某个 URL 的实际 credit 消耗，登录控制台后可以用 Domain Cost Estimator 工具预估，或者在请求里设置 `max_cost` 参数来限制单次上限。

---

## 常见问题

**Q：抓 Google 会有法律风险吗？**

抓 Google 的 *公开搜索结果* 本身属于对公开可访问数据的采集，这在学术研究、SEO 工具、市场分析等领域是普遍实践。ScraperAPI 本身也明确说明支持公开数据的合规采集。但要注意不要绕过 Google 的账号体系、不要抓取需要登录才能看到的内容。

**Q：用 ScraperAPI 会比自己维护代理池便宜吗？**

算上服务器成本、代理费用、工程师维护时间，大多数场景下用现成 API 服务比自建要划算。特别是当你的业务不是专门做基础设施的时候，这部分精力花在核心功能上的回报更高。

**Q：ScraperAPI 是否支持 Google 的其他垂类结果？**

支持，ScraperAPI 有专门的端点处理 Google News（新闻）、Google Jobs（招聘）、Google Shopping（购物）、Google Maps（地图）等垂类 SERP，每个都有对应的结构化 JSON 输出。

**Q：注册之后多久可以开始用？**

注册完成后立刻可以用。控制台会直接显示你的 API Key，复制进代码就能跑。

---

## 写在最后

用 Python 调 Google SERP API，核心逻辑其实挺简单：一个 GET 请求 + 几个参数，返回结构化 JSON，然后你爱怎么用怎么用。

麻烦的是 Google 这边的反爬机制，以及你自己维护代理池、处理失败重试的成本。ScraperAPI 把这些事情包掉了，你专注写业务代码就行。

对于刚入门或者项目规模不大的开发者，Hobby 或 Startup 套餐的性价比很高。需要全球地理定位或者更大并发量的，从 Business 套餐起会更顺手。

👉 [免费注册 ScraperAPI，立即获取 5000 次免费 API 调用](https://www.scraperapi.com/?fp_ref=coupons)
