<template>
  <div>
    <div v-if="illustDetail" class="detail">
      <List :list="pictureList" @infinite="infinite">
        <div class="detail-top">
          <div class="detail-img animated zoomIn" :style="{height: `${illustDetail.itemHeight}px`}" @click="seePreview">
            <img
              :src="illustDetail.originalSrc"
              width="100%"
              height="100%"
              alt="detail"
              :style="imgStyle"
            >
            <Like
              :width="80"
              :like="illustDetail.isLiked"
              @handleLike="handleLike"
            />
            <v-btn
              absolute
              top
              style="left: 0;top: 0;"
              icon
              @click.stop="$router.back()"
            >
              <svg font-size="20" class="icon" aria-hidden="true">
                <use xlink:href="#picfanhui" />
              </svg>
            </v-btn>
            <v-menu>
              <template v-slot:activator="{ on }">
                <v-btn
                  icon
                  absolute
                  style="right: 0;top: 0;"
                  @click.stop
                  v-on="on"
                >
                  <svg font-size="20" class="icon" aria-hidden="true">
                    <use xlink:href="#piccaidan" />
                  </svg>
                </v-btn>
              </template>
              <v-list>
                <v-list-item
                  v-for="(item, index) in items"
                  :key="index"
                  @click="clickMenu(item.val)"
                >
                  <v-list-item-title>{{ item.title }}</v-list-item-title>
                </v-list-item>
              </v-list>
            </v-menu>
          </div>
          <div class="detail-info">
            <div class="info-top">
              <h3 class="text-no-wrap text-truncate"></h3>
              <p>ID:  {{ illustDetail.id }}</p>
              <v-btn class="collects" outlined small fab color="rgb(212, 35, 122)" @click="addCollects">
                <svg font-size="30" class="icon" aria-hidden="true">
                  <use xlink:href="#pictianjia1" />
                </svg>
              </v-btn>
            </div>
            <p class="caption" v-html="" />
            <div class="tags">
              <span
                v-for="(item, index) in illustDetail.tags"
                :key="index"
                class="caption tag"
                @click="searchByTag(item.name)"
              >
                <a></a>
                <a></a>
              </span>
            </div>
            <div class="work-stats">
              <a class="work-stats-a">
                <svg class="icon" aria-hidden="true">
                  <use xlink:href="#picyanjing" />
                </svg>
                <span>0</span>
              </a>
              <a class="work-stats-a">
                <svg class="icon" aria-hidden="true">
                  <use xlink:href="#picaixin" />
                </svg>
                <span>0</span>
              </a>
              <a class="work-stats-a">
                <span>{{ illustDetail.createDate }}</span>
              </a>
              <a v-if="likeUsers" class="work-stats-a user-avatar">
                <v-avatar v-for="item in likeUsers" :key="item.userId" size="30" @click="goUsers">
                  <img :src="item.userId | replaceAvatar" style="object-fit: cover;">
                </v-avatar>
              </a>
            </div>
            <v-divider />
            <div class="user">
              <router-link :to="`/artist/${illustDetail.artistId}`" class="text-no-wrap text-truncate">
                <v-avatar>
                  <img :src="illustDetail.avatarSrc" alt="avatar">
                </v-avatar>
                <span>{{ illustDetail.artistPreView.name }}</span>
              </router-link>
              <v-btn color="primary" rounded @click="follow">
                {{ illustDetail.artistPreView.isFollowed ? '已关注' : '+加关注' }}
              </v-btn>
            </div>
            <v-divider />

            <div class="detail-comment">
              <div class="comment-title">评论</div>
              <div v-if="commentList.length">
                <comment-list :list="commentList.slice(0, 5)" @reply="openComment" />
              </div>
              <div v-else class="comment-no">
                <svg font-size="36" class="icon" aria-hidden="true">
                  <use xlink:href="#picpinglun" />
                </svg>
                <!-- <span>留下你的足迹o(*￣▽￣*)o</span> -->
              </div>
              <v-btn
                color="rgba(0, 0, 0, 0.04)"
                width="100%"
                depressed
                rounded
                @click="openComment"
              >
                添加评论
              </v-btn>
            </div>

            <v-divider />
            <div class="title">
              <h2>相关作品</h2>
            </div>
          </div>
        </div>
      </List>
      <div v-if="preview" class="detail-preview">
        <v-carousel
          height="100vh"
          hide-delimiter-background
          hide-delimiters
          show-arrows-on-hover
        >
          <v-carousel-item v-for="(item, i) in illustDetail.imgs" :key="i">
            <v-row class="fill-height" align="center" justify="center">
              <img
                width="100%"
                :src="item"
                :style="{filter: illustDetail.setu ? 'blur(20px)' : ''}"
              >
            </v-row>
            <v-btn
              color="#b9eee5"
              dark
              small
              fixed
              top
              right
              fab
              @click="jumpUrl(item)"
            >
              <svg font-size="20" class="icon" aria-hidden="true">
                <use xlink:href="#picfasong" />
              </svg>
            </v-btn>
          </v-carousel-item>
        </v-carousel>
        <v-btn
          color="#b9eee5"
          dark
          small
          fixed
          top
          left
          fab
          @click="preview = false"
        >
          <svg font-size="20" class="icon" aria-hidden="true">
            <use xlink:href="#picguanbi7" />
          </svg>
        </v-btn>
      </div>
    </div>
    <Comment :show.sync="show" :pid="pid" @reply="reply" />
  </div>
</template>

<script>
import { mapGetters } from 'vuex';
import dayjs from 'dayjs';
import List from '@/components/virtual-list/VirtualList';
import Like from '@/components/like/Like';
import Comment from './components/Comment';
import Alert from '@/components/alert';
import CommentList from './components/List';
import { replaceBigImg, replaceSmallImg } from '@/util';
import { SET_COLLECT_STATUS } from '@/store/mutation-types';

export default {
  name: 'Detail',
  components: {
    List,
    Like,
    Comment,
    CommentList
  },
  props: {
    pid: {
      required: true,
      type: [String, Number]
    }
  },
  data() {
    return {
      pictureList: [],
      illustDetail: null,
      page: 1,
      preview: false,
     /* items: [
        { val: 'pixiv', title: '跳转pixiv详情' },
        { val: 'artist', title: '跳转pixiv画师' }
      ],*/
      like: false,
      commentList: [],
      likeUsers: [],
      show: false
    };
  },
  computed: {
    ...mapGetters(['user', 'likeStatus', 'followStatus', 'detail', 'serverAddress']),
    imgStyle() {
      return {
        filter: this.illustDetail.setu ? `blur(25px)` : ''
      };
    }
  },
  watch: {
    // 详情去画师那里更改点赞状态 然后后退回来详情 状态也要变
    likeStatus(val, old) {
      const { illustId, like } = val;
      if (illustId === this.illustDetail.id) {
        this.illustDetail.isLiked = like;
      }
    },
    followStatus(val) {
      if (val.artistId === this.illustDetail.artistPreView.id) {
        this.illustDetail.artistPreView.isFollowed = val.follow;
      }
    }
  },
  mounted() {
    if (this.detail) {
      this.illustDetail = this.handleData(JSON.parse(JSON.stringify(this.detail)));
    } else {
      this.getIllustDetail();
    }
    this.getCommentsList();
    this.bookmarkedUsers();
    this.illustHistory();
  },
  methods: {
    getIllustDetail() {
      this.$api.detail
        .reqIllustDetail(this.pid)
        .then(res => {
          const data = res.data.data;
          this.illustDetail = this.handleData(data);
        });
    },
    infinite($state) {
      this.$api.detail
        .reqRelatedIllust({
          page: this.page++,
          illustId: this.pid
        })
        .then(res => {
          if (!res.data.data) {
            $state.complete();
          } else {
            this.pictureList = this.pictureList.concat(res.data.data);
            $state.loaded();
          }
        })
        .catch(err => {
          console.error(err);
        });
    },
    clickMenu(id) {
      switch (id) {
        case 'pixiv':
          window.open(`https://www.pixiv.net/artworks/${this.pid}`);
          break;
        case 'artist':
          window.open(`https://www.pixiv.net/users/${this.illustDetail.artistId}`);
          break;
      }
    },
    handleLike() {
      if (!this.user.id) {
        this.$router.push({
          name: 'Login',
          query: {
            return_to: window.location.href
          }
        });
        return;
      }
      const params = {
        userId: this.user.id,
        illustId: this.illustDetail.id,
        username: this.user.username
      };
      if (!this.illustDetail.isLiked) {
        this.illustDetail.isLiked = true;
        this.$store.dispatch('handleCollectIllust', params)
          .then(() => {})
          .catch(() => {
            this.illustDetail.isLiked = false;
            Alert({
              content: '收藏失败'
            });
          });
      } else {
        this.illustDetail.isLiked = false;
        this.$store.dispatch('deleteCollectIllust', params)
          .then(() => {})
          .catch(() => {
            this.illustDetail.isLiked = true;
            Alert({
              content: '取消收藏失败'
            });
          });
      }
    },
    follow() {
      if (!this.user.id) {
        this.$router.push({
          name: 'Login',
          query: {
            return_to: window.location.href
          }
        });
        return;
      }
      const data = {
        artistId: this.illustDetail.artistPreView.id,
        userId: this.user.id,
        username: this.user.username
      };
      if (!this.illustDetail.artistPreView.isFollowed) {
        this.illustDetail.artistPreView.isFollowed = true;
        this.$store.dispatch('handleFollowArtist', { ...data, follow: true })
          .then(res => {})
          .catch(() => {
            this.illustDetail.artistPreView.isFollowed = false;
            Alert({
              content: '关注失败'
            });
          });
      } else {
        this.illustDetail.artistPreView.isFollowed = false;
        this.$store.dispatch('handleFollowArtist', { ...data, follow: false })
          .then(res => {})
          .catch(() => {
            this.illustDetail.artistPreView.isFollowed = true;
            Alert({
              content: '取消关注失败'
            });
          });
      }
    },
    seePreview() {
      if (this.illustDetail.xrestrict === 1 && this.user.username !== 'pixivic') return;
      this.preview = true;
      this.$store.dispatch('changeTab', false);
    },
    openComment() {
      if (this.user.id) {
        this.show = true;
      } else {
        this.$router.push({
          name: 'Login',
          query: {
            return_to: window.location.href
          }
        });
      }
    },
    searchByTag(tag) {
      this.$router.push({
        name: 'SearchIllusts',
        query: {
          tag,
          source: 'detail'
        }
      });
    },
    jumpUrl(url) {
      window.open(url);
    },
    // 等待后端分页处理
    getCommentsList() {
      this.$api.comment.getComments({
        commentAppType: 'illusts',
        commentAppId: this.pid,
        pageSize: 5
      })
        .then(res => {
          if (res.status === 200) {
            this.commentList = res.data.data || [];
          }
        });
    },
    reply(list) {
      this.commentList = list;
    },
    bookmarkedUsers() {
      this.$api.detail.bookmarkedUsers({
        illustId: this.pid,
        pageSize: 3
      })
        .then(res => {
          this.likeUsers = res.data.data;
        });
    },
    goUsers() {
      this.$router.push(`/bookmark/${this.pid}`);
    },
    handleData(data) {
      return {
        ...data,
        itemHeight: data.itemHeight || parseInt((data.height / data.width) * document.body.clientWidth),
        originalSrc: data.originalSrc || replaceBigImg(data.imageUrls[0].original),
        src: data.src || replaceSmallImg(data.imageUrls[0].medium),
        avatarSrc: data.avatarSrc || replaceBigImg(data.artistPreView.avatar),
        createDate: dayjs(data.createDate).format('YYYY-MM-DD'),
        setu: data.setu || !!((data.xrestrict === 1 || data.sanityLevel > 5)) && this.user.username !== 'pixivic',
        imgs: data.imgs || data.imageUrls.reduce((pre, cur) => {
          return pre.concat(replaceBigImg(cur.original));
        }, [])
      };
    },
    illustHistory() {
      if (this.user.id) {
        this.$api.detail.illustHistory({
          userId: this.user.id,
          illustId: this.pid
        });
      }
    },
    addCollects() {
      const data = {
        show: true,
        id: this.pid
      };
      this.$store.commit(SET_COLLECT_STATUS, data);
    }
  }
};
</script>
<style lang="stylus" scoped>
@import '~@/assets/style/color.styl'
.detail
  background-size contain
  width 100%
  background-color #fff
  z-index 3
  font-size 16px
  &-img
    position relative
    margin 0 -8px
    box-sizing border-box
    background-color grey
    overflow hidden
    img
      width 100%
      transition opacity .3s
      vertical-align bottom
      object-fit cover
  &-comment
    margin-bottom 12px
    .comment-title
      padding-top 10px
      font-size 14px
      font-weight 700
    .comment-no
      display flex
      justify-content center
      align-items center
      flex-direction column
      padding 30px
      span
        font-size 14px
        font-weight 700px
        color rgba(0, 0, 0, 0.32)
        margin 12px 0
  &-info
    padding 8px
    overflow hidden
    >.caption
      word-wrap break-word
      margin-top 2px
    .info-top
      position relative
      h3
        max-width 80%
      .collects
        position absolute
        right 0
        top 6px
    .user
      display flex
      align-items center
      margin 14px 0
      a
        flex 1
        span
          margin-left 10px
          color rgba(0, 0, 0, 0.88)
    .tags
      .tag
        user-select none
        a:first-child
          color $primary
          margin-right 4px
          &::before
            content '#'
        a:last-child
          color #adadad
          margin-right 8px
    .work-stats
      margin 10px 0 15px
      user-select none
      position relative
      display flex
      align-items center
      &-a
        margin-right 10px
        height 30px
        line-height 30px
        span
          font-size 12px
          margin-left 2px
          color rgba(0, 0, 0, 0.32)
          vertical-align middle
      .user-avatar
        flex 1
        text-align right
        margin 0
        .v-avatar
          margin-right 5px
    .title
      margin-top 12px
      h2
        font-size 18px
        color #333
  &-preview
    position fixed
    top 0
    left 0
    right 0
    height 100vh
    overflow scroll
    background #fff
    >>>.v-carousel
      overflow-y scroll !important
    >>>.v-responsive
      overflow-y scroll !important
</style>
