<template>
  <div>
    <Header
      :showLoginBtn="false"
    />
    <div class="oauth-login">
      <Oauth
        :app="app"
        :userData="userData"
      />
    </div>
  </div>
</template>

<script>

import Header from '@/components/Header.vue'
import Oauth from '@/components/Login/Oauth.vue'

import Axios from 'axios'

import env from '../../../env.json'
import { getUserProfile, getAvatarUrl, getUserProfileTest } from '../../api/user'
import { disassemble } from '../../util/cookie'
import { mapActions, mapState } from 'vuex'
import i18n from '../../locale'

export default {
  components: {
    Header,
    Oauth
  },
  data () {
    return {
      app: {
        img: null,
        name: '',
        id: 0,
        callback: '',
        token: ''
      },
      userData: {
        avatar: '',
        nickname: ''
      }
    }
  },
  computed: {
    ...mapState([])
  },
  methods: {
    ...mapActions(['setThirdPartyUserId', 'setThirdPartyUserProfile'])
  },
  created () {
    Axios(env.DEVELOPERAPI + '/app/detail?appId=' + this.$route.params.id).then(app => {
      if (this.app.name === undefined) {
        this.$message({
          message: i18n.t('elMessage.error.oAuth'),
          type: 'error',
          duration: 4000
        })
        this.$router.push({ name: 'Landing' })
      } else {
        this.app = {
          id: this.$route.params.id,
          img: app.data.img === '' || app.data.img === undefined ? require('@/assets/img/app-default.png') : env.DEVELOPERAPI + '/img/' + app.data.img,
          ...app.data.detail
        }
      }
      document.title = i18n.t('siteTitle.oAuth.callback', { name: app.data.detail.name })
    }).catch(e => {
      this.$message({
        message: i18n.t('elMessage.error.oAuth'),
        type: 'error',
        duration: 4000
      })
      this.$router.push({ name: 'Landing' })
    })

    const token = this.$route.params.callback.replace('type=token&token=', '')
    const thirdPartyUser = disassemble(token)
    this.setThirdPartyUserId(thirdPartyUser.id)
    this.setThirdPartyUserProfile(thirdPartyUser)
    this.app.token = token

    const ls = window.localStorage || localStorage
    const network = JSON.parse(ls.getItem(this.$route.params.id))
    ls.removeItem(this.$route.params.id)

    if (!network) return

    if (network.network === 'test') {
      getUserProfileTest(thirdPartyUser.id).then(res => {
        this.userData.avatar = getAvatarUrl(res.data.data.avatar)
        this.userData.nickname = res.data.data.nickname
      })
    } else {
      getUserProfile(thirdPartyUser.id).then(res => {
        this.userData.avatar = getAvatarUrl(res.data.data.avatar)
        this.userData.nickname = res.data.data.nickname
      })
    }
  }
}
</script>

<style lang="less" scoped>
.oauth-login {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 70px 0 10px;
  box-sizing: border-box;
}

@media screen and (max-width: 500px) {
  .oauth-login {
    padding: 10px 0 10px;
  }
  .header {
    display: none;
  }
}
</style>
