# Deployer

This site is deploy on GitHub.

# 魔改

Modify `util.js`

```javascript
// 获取自定义播放列表
  getCustomPlayList: function () {
    if (!window.location.pathname.startsWith("/music/")) {
      return;
    }
    const urlParams = new URLSearchParams(window.location.search);
    const userId = "7032963191";
    const userServer = "netease";
    const anMusicPageMeting = document.getElementById("anMusic-page-meting");
    if (urlParams.get("id") && urlParams.get("server")) {
      const id = urlParams.get("id");
      const server = urlParams.get("server");
      anMusicPageMeting.innerHTML = `<meting-js id="${id}" server=${server} type="playlist" type="playlist" mutex="true" preload="auto" theme="var(--anzhiyu-main)" order="list" list-max-height="calc(100vh - 169px)!important"></meting-js>`;
    } else {
      anMusicPageMeting.innerHTML = `<meting-js id="${userId}" server="${userServer}" type="playlist" mutex="true" preload="auto" theme="var(--anzhiyu-main)" order="list" list-max-height="calc(100vh - 169px)!important"></meting-js>`;
    }
    anzhiyu.changeMusicBg(false);
  },
```

music.styl 添加以下

```stylus
// 音樂界面隱藏一些音樂按鈕
#anMusic-page
  #anMusicRefreshBtn, #anMusicBtnGetSong, #anMusicSwitching
    display none
```

並且把 aplayer-icon-loop、aplayer-icon-order、aplayer-volume-wrap 設為不可見

```stylus
&.aplayer-icon-loop, &.aplayer-icon-order
  display none
  margin-right 15px
```

```stylus
.aplayer-volume-wrap
  display none
  .aplayer-volume-bar-wrap
    bottom: 0;
    right: -5px;
```

highlight.styl 修改如下

```stylus
&.expand-done
  i.anzhiyufont.anzhiyu-icon-angle-double-down
    transform: rotate(180deg)
    transition: all 0s, background 0.3s
```

sidebar.styl 新增以下

```stylus
.fa, .fa-solid, .fab, .far, .fas
  line-height inherit !important // 或者您想要设置的其他值
```

code.css 修改如下（背景色和字體顏色在 yml 改）

```css
#article-container code {
    /* padding: 0.2rem 0.4rem; */
    border-radius: 4px;
    /* margin: 0 4px; */
    line-height: 2;
    /* color: var(--anzhiyu-theme); 更改行內代碼樣式 */
    /* box-shadow: var(--anzhiyu-shadow-border); */
}
```

en.yml 修改如下

```yaml
sticky: Pinned
```

card_webinfo.pug 修改如下

```pug
if theme.aside.card_webinfo.enable
  .card-webinfo
    .item-headline
      i.anzhiyufont.anzhiyu-icon-chart-line
      span= _p('aside.card_webinfo.headline')
    .webinfo
      if theme.aside.card_webinfo.post_count
        .webinfo-item
          .webinfo-item-title
            i.fa-solid.fa-file-lines
            .item-name= _p('aside.card_webinfo.article_name') + " :"
          .item-count= site.posts.length
      if theme.runtimeshow.enable
        .webinfo-item
          .webinfo-item-title
            i.fa-solid.fa-stopwatch
            .item-name= _p('aside.card_webinfo.runtime.name') + " :"
          .item-count#runtimeshow(data-publishDate=date_xml(theme.runtimeshow.publish_date))
            i.anzhiyufont.anzhiyu-icon-spinner.anzhiyu-spin
      if theme.wordcount.enable && theme.wordcount.total_wordcount
        .webinfo-item
          .webinfo-item-title
            i.fa-solid.fa-font
            .item-name=_p('aside.card_webinfo.site_wordcount') + " :"
          .item-count=totalcount(site)
      if theme.busuanzi.site_uv
        .webinfo-item
          .webinfo-item-title
            i.fa-solid.fa-universal-access
            .item-name= _p('aside.card_webinfo.site_uv_name') + " :"
          .item-count#busuanzi_value_site_uv
            i.anzhiyufont.anzhiyu-icon-spinner.anzhiyu-spin
      if theme.busuanzi.site_pv
        .webinfo-item
          .webinfo-item-title
            i.fa-solid.fa-square-poll-vertical
            .item-name= _p('aside.card_webinfo.site_pv_name') + " :"
          .item-count#busuanzi_value_site_pv
            i.anzhiyufont.anzhiyu-icon-spinner.anzhiyu-spin
      if theme.aside.card_webinfo.last_push_date
        .webinfo-item
          .webinfo-item-title
            i.fa-solid.fa-hourglass-start
            .item-name= _p('aside.card_webinfo.last_push_date.name') + " :"
          .item-count#last-push-date(data-lastPushDate=date_xml(Date.now()))
            i.anzhiyufont.anzhiyu-icon-spinner.anzhiyu-spin
```

nav.styl 修改如下

```stylus
.back-home-button
  display none
  width 35px
  height 35px
  padding 0 !important
  align-items center
  justify-content center
  margin-right 4px
  transition 0.3s
  border-radius 8px
  @media (min-width: 770px)
    display flex
```

main.js 修改如下

```javascript
if (willChangeMode === "dark") {
    activateDarkMode();
    // GLOBAL_CONFIG.Snackbar !== undefined &&
    //     anzhiyu.snackbarShow(GLOBAL_CONFIG.Snackbar.day_to_night);
} else {
    activateLightMode();
    // GLOBAL_CONFIG.Snackbar !== undefined &&
    //     anzhiyu.snackbarShow(GLOBAL_CONFIG.Snackbar.night_to_day);
}
```

sidebar.styl 修改如下

```stylus
#aside-content #card-toc span.toc-number {
  display: inline;
}
```

index.styl 新增如下

```stylus
#catalog-list
    overflow hidden
    scrollbar-width none /* Firefox 特有 */

    &::-webkit-scrollbar
        display none /* 完全隱藏滾動條 */
```

aside.styl 修改如下

```stylus
.toc-content
      overflow-y: scroll
      overflow-y: overlay
      margin: 0 -24px
      max-height: calc(100vh - 200px)
      width: calc(100% + 48px)
```

\_config.yml 新增以下

```yml
search:
    path: search.xml
    field: post
    content: true # 文章内容
    template: source/_data/search.xml # 自定义模板
```

新增 source/\_data/search.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<search>
  {% if posts %}
  {% for post in posts.toArray() %}
    {% if post.indexing == undefined or post.indexing %}
    <entry>
      <title>{{ post.title }}</title>
      <link href="{{ (url + post.path) | uriencode }}"/>
      <url>{{ (url + post.path) | uriencode }}</url>
      <cover>{{ post.cover | uriencode }}</cover>
      <date>{{ post.date }}</date>
      {% if content %}
        <content type="html"><![CDATA[{{ post.content | noControlChars | safe }}]]></content>
      {% endif %}
      {% if post.categories and post.categories.length>0 %}
      <categories>
          {% for cate in post.categories.toArray() %}
          <category> {{ cate.name }} </category>
          {% endfor %}
      </categories>
      {% endif %}
      {% if post.tags and post.tags.length>0 %}
        <tags>
            {% for tag in post.tags.toArray() %}
            <tag> {{ tag.name }} </tag>
            {% endfor %}
        </tags>
      {% endif %}
    </entry>
    {% endif %}
    {% endfor %}
  {% endif %}
  {% if pages %}
    {% for page in pages.toArray() %}
    {% if post.indexing == undefined or post.indexing %}
    <entry>
      <title>{{ page.title }}</title>
      <link href="{{ (url + page.path) | uriencode }}"/>
      <url>{{ (url + page.path) | uriencode }}</url>
      <cover>{{ post.cover | uriencode }}</cover>
      <date>{{ post.date }}</date>
      {% if content %}
        <content type="html"><![CDATA[{{ page.content | noControlChars | safe }}]]></content>
      {% endif %}
    </entry>
    {% endif %}
    {% endfor %}
  {% endif %}
</search>
```

local-search.js 修改如下 (cover 的部分)

```javascript
let cover =
    item.querySelector("cover") && item.querySelector("cover").textContent;
let srcArr = [];
if (arr) {
    for (let i = 0; i < arr.length; i++) {
        let src = arr[i].match(srcReg);
        // 獲取圖片地址
        if (!src[1].indexOf("http")) srcArr.push(src[1]);
    }
}

return {
    title: item.querySelector("title").textContent,
    content: content,
    url: item.querySelector("url").textContent,
    tags: tagsArr,
    // oneImage: srcArr && srcArr[0],
    oneImage: cover,
};
```

local-search.js 修改如下 (會有 injection 的部分)

```javascript
str += '<div class="local-search__hit-item">';
if (oneImage) {
    str += `<div class="search-left"><img src=${oneImage} data-fancybox='gallery'>`;
} else {
    str += '<div class="search-left" style="width:0">';
}
```

local_search.css 替換為

```css
#local-search .search-dialog .local-search__hit-item:before {
    content: none !important;
}

#local-search .search-dialog .local-search__hit-item {
    padding: 10px !important;
    border-radius: 8px;
    border: var(--style-border);
    margin-bottom: 10px;
    display: flex;
    justify-content: space-between;
    align-items: center;
}

#local-search .search-dialog .local-search__hit-item .search-left {
    width: 35%;
    padding: 1rem;
}

#local-search .search-dialog .local-search__hit-item .search-right {
    width: 60%;
}

#local-search .search-dialog .local-search__hit-item .search-left img {
    object-fit: cover;
    width: 100%;
    background: var(--anzhiyu-secondbg);
    border-radius: 8px;
    transition: all 0.6s ease 0s;
}

#local-search .search-dialog .local-search__hit-item .search-left:hover img {
    filter: brightness(0.82) !important;
    transform: scale(1.06) !important;
    transition: 0.3s ease-in-out;
}

#local-search .search-dialog .local-search__hit-item .search-result {
    cursor: pointer;
}

#local-search .tag-list {
    padding: 4px 8px;
    border-radius: 8px;
    margin-right: 0.5rem;
    margin-top: 0.5rem;
    border: var(--style-border);
    cursor: pointer;
}

#local-search .search-dialog .local-search__hit-item .tag-list {
    display: inline;
}

#local-search .search-dialog .local-search__hit-item .tag-list:hover {
    background: var(--anzhiyu-main);
    color: var(--anzhiyu-white);
}

#local-search .search-dialog .search-right .search-result-tags {
    display: flex;
    flex-wrap: wrap;
    justify-content: flex-start;
}

@media screen and (max-width: 768px) {
    #local-search .search-dialog .local-search__hit-item .search-right {
        width: 100%;
        padding: 0.8rem;
    }

    #local-search .search-dialog .local-search__hit-item .search-left {
        display: none;
    }
}
```

themes/my-anzhiyu/source/css/\_search/index.styl 修改如下

```diff
- .search-dialog
-   position: fixed
-   top: 5rem
-   left: 50%
-   z-index: 1001
-   display: none
-   margin-left: -18.75rem;
-   padding: 1.25rem;
-   width: 37.5rem;
-   border-radius: 8px
-   background: var(--search-bg)
-   border-radius: 12px;
-   box-shadow: var(--anzhiyu-shadow-lightblack);
-   background: var(--anzhiyu-card-bg);
-   border: var(--style-border);
-   transition: .3s;
+ .search-dialog
+   position: fixed
+   top: 5rem
+   left: 0
+   right: 0
+   z-index: 1001
+   display: none
+   // margin-left: -18.75rem;
+   margin: auto;
+   padding: 1.25rem;
+   max-width: 70rem;
+   width: 100%;
+   border-radius: 8px
+   background: var(--search-bg)
+   border-radius: 12px;
+   box-shadow: var(--anzhiyu-shadow-lightblack);
+   background: var(--anzhiyu-card-bg);
+   border: var(--style-border);
+   transition: .3s;
```

# hexo-butterfly-envelope

`main.js`中把中文改為英文

# 魔改資源

-   [文章主色调(插件)](https://www.naokuo.top/p/fb2f8d77.html)
-   [为主页文章卡片添加擦亮动画效果](https://blog.kouseki.cn/posts/dda6.html)
-   [重构记录 - 4](https://meuicat.com/blog/42/)
-   [好看的昼夜切换按钮](https://www.naokuo.top/p/1c3b759a.html)
