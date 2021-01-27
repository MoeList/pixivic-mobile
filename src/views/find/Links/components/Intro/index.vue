<template>
  <div class="intro">
    <div class="infos">
      <img :src="require('@/assets/images/girl.gif')"><h2>用前の须知</h2>
      <p>以下是一些值得仔细阅览的使用说明</p>
    </div>
    <div class="version">
      <h4>关于Pixivic</h4>
      <ul>
        <li>出生于2018-10-01，是一位兴趣使然的工具酱。<br>
          她精致（ui统一），敏捷（前后分离&纯js）。<br>
      </ul>
      <h4>感谢</h4>
      <ul>
        <li>Pixivic的开发与运维离不开</li> <br>
        <a v-for="(item, index) in people" :key="index" :href="item.href">
          <img :src="item.src" alt="">
        </a>
      </ul>
      <h4>新增功能</h4>
      <ul>
        <li>用户系统 画作收藏 画作评论 画师关注 搜索条件 以图搜图 画师新作 spotlight等 </li>
      </ul>
      <h4>浏览器的使用</h4>
      <ul>
        <li> 诸如chrome(谷歌浏览器)，夸克浏览器，qq浏览器等的主流浏览器</li>
      </ul>
      <h4>搜索跳转的兼容性</h4>
      <ul>
        <li> 首页直接搜索跳转的兼容性不佳，若在首页跳转后的搜索结果并不是你想要的，在该页面重新键入搜索词回车即可。</li>
      </ul>
      <h4>关于日排行</h4>
      <ul>
        <li> 默认最新一期为当前日期前天的排行数据，点击首页排行日历图标可进行日期的选定。中间文字可进行模式选择。</li>
      </ul>
      <h4>图片的浏览与保存</h4>
      <ul>
        <li> 详情页中，点击图片可查看原图，长按保存</li>
      </ul>
      <h4>阅读完毕</h4>
      <ul>
        <li> <a href="https://m.pixivic.com">点击此行返回主页，觉得站点好用的话请收藏噢(`・ω・´) </a></li>
      </ul>
    </div>
    <div class="reserved">
      ©2019-2020 m.pixivic.com All Right Reserved. 桂ICP备17012945号-3
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      people: [
        { href: 'https://www.oysterqaq.com/', src: require('@/assets/images/OysterQAQ.jpg') },
        { href: 'https://github.com/kimliy', src: require('@/assets/images/kimliy.jpg') },
        { href: 'https://www.ariussss.com/', src: require('@/assets/images/Arius.jpg') },
        { href: 'https://dongzheyu.github.io/', src: require('@/assets/images/tiger.jpg') },
        { href: 'https://github.com/goblinM', src: require('@/assets/images/mmx.jpg') }
      ]
    };
  }
};
</script>

<style lang="stylus" scoped>
.infos
  background-color #f8f9fa
  box-sizing border-box
  color #454d5d
  padding 10px 10px 10px 10px
  img
    width 80px
    float left
    padding-right 10px
.version
  padding 10px 40px 10px 40px
  overflow hidden
  h4, ul, li
    color #454d5d
  ul
    margin 0 0 10px 5px
    a img
      border-radius 5px
      box-shadow 0 0 10px rgba(0, 0, 0, .3)
      height 40px
      display inline
      margin-left 5px
  h2
    margin 0
    font-size 2.4em
    position relative
    color #454d5d
    padding-top 13px
  h4
    font-size 1.2em
    position relative
    &::before
      content ''
      width 14px
      height 14px
      background #b9eee5
      display inline-block
      vertical-align middle
      margin-top -4px
      margin-right 11px
      border-radius 100%
      border 3px solid #fff
      box-shadow 0 0 5px #b0b0b0
    &::after
      content ''
      height 100vh
      width 2px
      background #b9eee5
      position absolute
      left 7px
      top 14px
.reserved
  font-size 14px
  color #8c8c8c
  margin-top 10px
  text-align center
</style>

