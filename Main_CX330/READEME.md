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

# hexo-butterfly-envelope

`main.js`中把中文改為英文

# 魔改資源

-   [文章主色调(插件)](https://www.naokuo.top/p/fb2f8d77.html)
-   [为主页文章卡片添加擦亮动画效果](https://blog.kouseki.cn/posts/dda6.html)
