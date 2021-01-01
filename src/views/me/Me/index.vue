<template>
  <div class="me">
    <div class="me-header">
      <v-card class="header-box" width="95%" outlined>
        <div class="link">
          <svg v-if="isVip" class="icon vip" font-size="25" aria-hidden="true">
            <use xlink:href="#pichuiyuan" />
          </svg>
          <v-avatar class="user-avatar" size="80" @click="$router.push('/avatar')">
            <img :src="avatar" alt="Avatar">
          </v-avatar>
          <div class="user-info">
            <p class="font-weight-bold text-no-wrap text-truncate" :style="{'color': isVip ? 'rgb(251, 114, 153)' : ''}">{{ user.username }}</p>
            <div class="info-tabs">
              <router-link to="" class="tab">
                <span>0</span>
                <p>消息</p>
              </router-link>
              <router-link to="" class="tab">
                <span>0</span>
                <p>粉丝</p>
              </router-link>
            </div>
          </div>
        </div>
      </v-card>
    </div>
    <v-list class="me-lists">
      <v-list-item
        v-for="item in list"
        :key="item.val"
        @click="clickItem(item.val)"
      >
        <v-list-item-icon>
          <svg font-size="20" class="icon" aria-hidden="true">
            <use :xlink:href="item.icon" />
          </svg>
        </v-list-item-icon>
        <v-list-item-content>
          <v-list-item-title>{{ item.text }}</v-list-item-title>
        </v-list-item-content>
        <v-list-item-icon>
          <svg class="icon" aria-hidden="true">
            <use xlink:href="#picyoujiantou" />
          </svg>
        </v-list-item-icon>
      </v-list-item>
    </v-list>
    <v-dialog
      v-model="dialog"
      max-width="290"
    >
      <v-card>
        <v-card-title class="headline">
          会员图片加速
        </v-card-title>
        <v-card-text class="text-center">
          <span v-if="isVip">{{ '当前会员加速中 有效期到' }}{{ user.permissionLevelExpireDate | dateFormat }}</span>
          <span v-else>您当前还不是会员</span>
        </v-card-text>
        <v-card-text class="text-center">
          <v-btn class="ma-2" depressed small color="primary" href="/handbook">使用说明</v-btn>
          <div class="d-flex justify-center">
            <v-btn class="mr-2" depressed small color="success" target="_blank" href="https://mall.pixivic.net/product/">支付宝购买</v-btn>
            <v-btn depressed small color="success" target="_blank" href="https://weidian.com/?userid=1676062924">微信购买</v-btn>
          </div>
          <span>需要微信付款请下载微店app或在微信打开https://weidian.com/?userid=1676062924</span>
        </v-card-text>
        <v-card-text class="text-center">
          <v-text-field v-model="code" label="输入兑换码（日期可以叠加）" />
          <v-btn depressed color="success" :disabled="!code" @click="exchange">立即兑换</v-btn>
        </v-card-text>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
import { mapGetters } from 'vuex';
import Cookie from 'js-cookie';
import Confirm from '@/components/confirm';
import Toast from '@/components/toast';

export default {
  name: 'Me',
  data() {
    return {
      list: [
        { icon: '#picA34', text: '会员加速', val: 'vip' },
        { icon: '#pictianjia1', text: '我的画集', val: '/collects' },
        { icon: '#picshoucang-copy-copy-copy', text: '我的收藏', val: '/collect' },
        { icon: '#picicon-copy', text: '我的关注', val: '/artistCollect' },
        { icon: '#piclishi', text: '浏览历史', val: '/history' },
        { icon: '#picshezhi', text: '用户设置', val: '/setting' },
        { icon: '#piclog_out', text: '退出登录', val: 'logout' }
      ],
      dialog: false,
      code: null,
      permissionLevelExpireDate: null
    };
  },
  computed: {
    ...mapGetters([
      'user',
      'avatar',
      'isVip'
    ])
  },
  mounted() {
    if (this.user.id) {
      this.getUsersInfo();
    }
  },
  methods: {
    async clickItem(val) {
      if (val === 'logout') {
        const res = await Confirm({
          title: '确认退出吗?'
        });
        if (res === 'submit') {
          Cookie.remove('jwt');
          localStorage.removeItem('user');
          window.location.href = process.env.VUE_APP_HOME_URL;
        }
      } else if (val === 'vip') {
        this.dialog = true;
      } else {
        this.$router.push(val);
      }
    },
    exchange() {
      this.code = this.code.trim();
      this.$api.user
        .exchangeVip({
          userId: this.user.id,
          exchangeCode: this.code
        })
        .then(res => {
          Toast({ content: res.data.message });
          if (res.status === 200) {
            this.$store.dispatch('setUser', res.data.data);
            this.$store.dispatch('vipProxyServer');
          }
        });
    },
    getUsersInfo() {
      this.$api.user.getUsers(this.user.id)
        .then((res) => {
          const { data } = res.data;
          this.$store.dispatch('setUser', data);
        });
    }
  }
};
</script>

<style scoped lang="stylus">
.me
  width 100%
  height 100%
  position relative
  background-color #fff
  &-header
    height 200px
    overflow hidden
    background url('~@/assets/images/girl.jpg') no-repeat
    background-size 100%
    background-position 0 -100px
    border-bottom-right-radius 70% 60%
    border-bottom-left-radius 70% 60%
    // 937 283
    .header-box
      position absolute
      left 50%
      top 100px
      transform translateX(-50%)
      border-radius 20px
      .link
        position relative
        height 100%
        .vip
          position absolute
          top 30px
          left 80px
          z-index 999
        .user-avatar
          position absolute
          top -30px
          left 30px
        .user-info
          padding-top 20px
          text-align center
          >p
            margin-left 120px
            text-align left
          .info-tabs
            display flex
            .tab
              flex 1
              p
                color #5e5e5e
  &-lists
    margin-top 30px
    padding-bottom 80px
  .btn
    margin-left 50%
    margin-top 50px
    transform translateX(-50%)
    span
      display inline-block
      margin-left 5px
      vertical-align middle
</style>

