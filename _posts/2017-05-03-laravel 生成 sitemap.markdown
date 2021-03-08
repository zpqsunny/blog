---
title: Laravel 生成 sitemap
date: 2017-05-03 22:41:16 +0800
categories: [Laravel]
tags: [sitemap]
---
### 什么是 Sitemap?

Sitemap 可方便管理员通知搜索引擎他们网站上有哪些可供抓取的网页。最简单的 Sitepmap 形式，就是 XML 文件，在其中列出网站中的网址以及关于每个网址的其他元数据（上次更新的时间、更改的频率以及相对于网站上其他网址的重要程度为何等），以便搜索引擎可以更加智能地抓取网站。
网络抓取工具通常会通过网站内部和其他网站上的链接查找网页。Sitemap 会提供此数据以便允许支持 Sitemap 的抓取工具抓取 Sitemap 提供的所有网址，并了解使用相关元数据的网址。使用 Sitemap 协议并不能保证网页会包含在搜索引擎中，但可向网络抓取工具提供一些提示以便它们更有效地抓取网站。
### Sitemaps XML 格式

本文件说明 Sitemap 通讯协定的 XML 配置。
Sitemap 通讯协定格式是由 XML 标记所组成。Sitemap 中的所有资料值都必须是实体逸出。档案本身必须使用 UTF-8 编码。
Sitemap 必须：

- 以起始 <urlset> 标记做为开头，并以结束 </urlset> 标记做为结尾。
- 指定 <urlset> 内的名称领域 (通讯协定标准)。
- 让每个 URL 中包含一个<url> 项目做为母层 XML 标记。
- 在每个 <url> 母层标记包含一个 <loc> 子层项目。

其他所有标记均为选用的。是否支援这些选用的标记须视搜寻引擎而定。详细资讯请参阅各搜寻引擎的说明文件。
另外，Sitemap 中所有的 URL 必须来自单一主机，例如 www.example.com 或 store.example.com。

#### XML Sitemap 范例

下列范例显示仅包含一个 URL 并使用所有选用标记的 Sitemap。选用标记以斜体表示。

```xml
<?xml version="1.0" encoding="UTF-8"?>

<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">

   <url>

      <loc>http://www.example.com/</loc>

      <lastmod>2005-01-01</lastmod>

      <changefreq>monthly</changefreq>

      <priority>0.8</priority>

   </url>

</urlset>
```
#### XML 标记定义

可用的 XML 标记如下所述。

| 屬性 |   | 说明  |
| ------------ | ------------ | ------------ |
| ```<urlset>``` | 必须  | 压缩档案与参照最新的通讯协定标准。  |
| ```<url>```  |  必须 |  每个 URL 项目的母层标记。剩馀的标记是此标记的子阶层。 |
| ```<loc>```  | 必须  | 网页的 URL。这个 URL 必须以通讯协定开头 (例如 http) 并以尾端的斜线结束 (如果您的网页伺服器有此需求)。此值必须少于 2,048 个字元。  |
| ```<lastmod>```  |  可选 |  档案的最后修改日期。此日期应该採用 W3C 日期时间格式。必要的话，此格式允许您略过时间的部分，并使用 YYYY-MM-DD。请注意，此标记与伺服器可传回的 If-Modified-Since (304) 标头不同，且搜寻引擎可能会以不同的方式使用这两种来源的资讯。 |
| ```<changefreq>``` | 可选  | 网页可能变更的频率。此值会提供一般的资讯给搜寻引擎，而且与它们检索网页的频率不完全相关。有效值如下：always, hourly,daily,weekly,monthly,yearly,never "always" 值应该用来描述会随著每次存取而变更的文件。"never" 值应该用来描述已封存的 URL。请注意，此标记的值会当做提示而非指令。即使搜寻引擎的搜寻器在做决定时可能会参考此资讯，但它们检索标记为 "hourly" 网页的频率相较之下会偏低，而它们检索标记为 "yearly" 网页的频率则可能较高。搜寻器可能会定期检索标记为 "never" 的网页，以便处理这些网页的临时变更。|
| ```<priority>```  | 可选 | 此 URL 的优先顺序是相对于您网站上的其他 URL。有效值的范围为 0.0 到 1.0。此值不会影响您的网页与其他网站网页的比较，而只是让搜寻引擎知道您认为哪些网页对检索器来说最为重要。网页的预设优先顺序为 0.5。请注意，您指定给网页的优先顺序，并不会影响您的 URL 在搜寻引擎之结果网页上的排名。搜寻引擎可能会使用此资讯在同一个网站上的 URL 间做选择，因此您可以使用此标记来提高最重要的网页出现在搜寻索引中的可能性。同时请注意，将您网站中所有的 URL 都指定为高优先顺序并没有任何助益。因为优先顺序是相对性的，只会用来在您网站上的 URL 之间做选取。 |

> 出自于[sitemaps.org](https://www.sitemaps.org/index.html)

简单的说sitemap 就是网站的地图，它包含网站的所有网址，我们只要按照sitemapxml 格式进行编写，然后让搜索引擎进行抓取，从而纳入索引。


需要索引的文章地址：
http://www.example.com/post/1
http://www.example.com/post/2
```xml
<?xml version="1.0" encoding="UTF-8"?>

<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">

   <url>

      <loc>http://www.example.com/</loc>

      <lastmod>2005-01-01</lastmod>

      <changefreq>monthly</changefreq>

      <priority>0.8</priority>

   </url>

   <url>

      <loc>http://www.example.com/post/1</loc>

      <lastmod>2004-12-23</lastmod>

      <changefreq>weekly</changefreq>

   </url>
   <url>

      <loc>http://www.example.com/post/2</loc>

      <lastmod>2004-12-23</lastmod>

      <changefreq>weekly</changefreq>

   </url>

</urlset>
```
laravel 中生成sitemap 很简单，只要把需要索引的URL 按上面格式进行输出就行。
定义路由
```php
Route::get('sitemap','CommonController@getSitemap');   //获取sitemap

//控制器方法
public function getSitemap()
    {
        $posts = Post::select(['id','updated_at'])->get();

        return response()->view('sitemap',compact('posts'))
            ->header('Content-Type','application/xml');
    }
```
我这边用的是laravel 模板引擎渲染后输出，你也可以直接在Controller字符串拼接。header参数必须加上的，不然输出方式是以text/html进行输出。

模板文件
```php
<urlset
        xmlns="http://www.sitemaps.org/schemas/sitemap/0.9"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.sitemaps.org/schemas/sitemap/0.9
       http://www.sitemaps.org/schemas/sitemap/0.9/sitemap.xsd">

    <url>

        <loc>{{URL::to('/')}}</loc>

        <lastmod>{{date('Y-m-d',time())}}</lastmod>

        <changefreq>daily</changefreq>

        <priority>0.8</priority>

    </url>
    @foreach($posts as $post)

    <url>
        <loc>{{URL::to('post/'.$post->id)}}</loc>

        <lastmod>{{$post->updated_at->format('Y-m-d')}}</lastmod>

        <changefreq>weekly</changefreq>

    </url>
    @endforeach

</urlset>
```
