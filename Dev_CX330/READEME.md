# Deployer

This site is deploy on Vercel.

# Music

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
