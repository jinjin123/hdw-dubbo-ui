<template>
  <nav class="site-navbar" :class="'site-navbar--' + navbarLayoutType">
    <div class="site-navbar__header">
      <h1 class="site-navbar__brand" @click="$router.push({ name: 'home' })">
        <a class="site-navbar__brand-lg" href="javascript:;">数据采集系统</a>
        <a class="site-navbar__brand-mini" href="javascript:;">HDW</a>
      </h1>
    </div>
    <div class="site-navbar__body clearfix">
      <el-menu
        class="site-navbar__menu"
        mode="horizontal">
        <el-menu-item class="site-navbar__switch" index="0" @click="sidebarFold = !sidebarFold">
          <icon-svg name="zhedie"></icon-svg>
        </el-menu-item>
      </el-menu>
      <el-menu
        class="site-navbar__menu site-navbar__menu--right"
        mode="horizontal">
        <el-menu-item index="1" @click="$router.push({ name: 'theme' })">
          <template slot="title">
            <el-badge>
              <icon-svg name="shezhi" class="el-icon-setting"></icon-svg>
            </el-badge>
          </template>
        </el-menu-item>
        <el-menu-item  index="2">
          <i class="el-icon-rank" title="浏览器内全屏"></i>
        </el-menu-item>
        <el-menu-item index="3" @click="bigMsgShow">
          <el-dropdown :show-timeout="0" placement="bottom">
            <span class="el-dropdown-link" style="outline: none;">
              <i class="el-icon-bell"></i>
            </span>
            <a class="messageHide" v-if="unreadSmsCount>0">{{unreadSmsCount}}</a>
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item :key="index" v-for="(messageContent,index) in messageContentFive"
                                @click.native="smallMsgShow(messageContent)">
                <el-row style="max-width: 200px;">
                  <el-col :span="24" style="
                  display: -webkit-box;
                  -webkit-box-orient: vertical;
                  -webkit-line-clamp: 2;
                  overflow: hidden;
                  line-height: 1.5em;
                  padding-top:6px;
                  color: #606266;">{{messageContent.content}}
                  </el-col>
                  <el-col :span="24" style="font-size: 12px; color: #999;">{{messageContent.timeStr}}</el-col>
                </el-row>
              </el-dropdown-item>
              <el-dropdown-item>
                <el-row :gutter="20" v-if="unreadSmsCount>0" style="max-width: 200px;">
                  <el-col :span="12" @click.native="handleUpdateMsgStatus2">标记当前为已读</el-col>
                  <el-col :span="12" @click.native="bigMsgShow">查看全部</el-col>
                </el-row>
              </el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
        </el-menu-item>
        <el-menu-item class="site-navbar__avatar" index="4">
          <el-dropdown :show-timeout="0" placement="bottom">
            <span class="el-dropdown-link">
              <img src="~@/assets/img/avatar.png" :alt="userName">{{ userName }}
            </span>
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item @click.native="updatePasswordHandle()">修改密码</el-dropdown-item>
              <el-dropdown-item @click.native="logoutHandle()">退出</el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
        </el-menu-item>
      </el-menu>
    </div>
    <!-- 弹窗, 修改密码 -->
    <update-password v-if="updatePassowrdVisible" ref="updatePassowrd"></update-password>
    <!-- 全部消息 -->
    <big-message v-if="bigMessageVisible" ref="bigMessage"></big-message>
  </nav>
</template>

<script>
    import UpdatePassword from './main-navbar-update-password'
    import {clearLoginInfo, getUUID} from '@/utils'
    import BigMessage from './common/message'

    export default {
      data () {
        return {
          updatePassowrdVisible: false,
          messageContentFive: [],
          unreadSmsCount: 0,
          fiveMoreVisible: false,
          websocket: null,
          websocket2: null,
          bigMessageVisible: false
        }
      },
      components: {
        UpdatePassword,
        BigMessage
      },
      computed: {
        navbarLayoutType: {
          get () {
            return this.$store.state.common.navbarLayoutType
          }
        },
        sidebarFold: {
          get () {
            return this.$store.state.common.sidebarFold
          },
          set (val) {
            this.$store.commit('common/updateSidebarFold', val)
          }
        },
        mainTabs: {
          get () {
            return this.$store.state.common.mainTabs
          },
          set (val) {
            this.$store.commit('common/updateMainTabs', val)
          }
        },
        userName: {
          get () {
            return this.$store.state.user.name
          }
        },
        userId: {
          get () {
            return this.$store.state.user.id
          }
        }
      },
      created () {
        this.initWebSocket()
        this.initWebSocket2()
      },
      destroyed () {
        this.over()
        this.over2()
      },
      methods: {
            // 修改密码
        updatePasswordHandle () {
          this.updatePassowrdVisible = true
          this.$nextTick(() => {
            this.$refs.updatePassowrd.init()
          })
        },
            // 退出
        logoutHandle () {
          this.$confirm('确定进行[退出]操作?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }).then(() => {
            this.$http({
              url: this.$http.adornUrl('/sys/logout'),
              method: 'post',
              data: this.$http.adornData()
            }).then(({data}) => {
              if (data && data.code === 0) {
                clearLoginInfo()
                this.$router.push({name: 'login'})
              }
            })
          }).catch(() => {
          })
        },
            // 接收最近5条未读消息及未读消息数量
        initWebSocket () {
          const wsUrl = window.SITE_CONFIG.wsUrl + '/ws/homeSms/' + this.userId + '_' + getUUID()
          this.websocket = new WebSocket(wsUrl)
          this.websocket.onopen = this.webSocketOnOpen
          this.websocket.onerror = this.webSocketOnError
          this.websocket.onmessage = this.webSocketOnMessage
          this.websocket.onclose = this.webSocketClose()
          this.over = () => {
            this.websocket.close()
          }
        },
            // 连接成功
        webSocketOnOpen () {
          console.log('WebSocket连接成功')
        },
            // 错误
        webSocketOnError (e) {
          console.log('WebSocket连接发生错误')
        },
            // 数据接收
        webSocketOnMessage (e) {
          const receiveData = JSON.parse(e.data)
          this.messageContentFive = []
          if (receiveData.list) {
            this.messageContentFive = receiveData.list
          }
          if (receiveData.count) {
            this.unreadSmsCount = receiveData.count
            this.fiveMoreVisible = true
          } else {
            this.fiveMoreVisible = false
            this.unreadSmsCount = 0
          }
        },
            // 数据发送
        webSocketSend (data) {
          this.websocket.send(data)
        },
            // 关闭
        webSocketClose (e) {
          console.log('WebSocket连接关闭')
        },
            // 接收实时页面消息
        initWebSocket2 () {
          const wsUrl = window.SITE_CONFIG.wsUrl + '/ws/sms/' + this.userId + '_' + getUUID()
          this.websocket2 = new WebSocket(wsUrl)
          this.websocket2.onopen = this.webSocketOnOpen2
          this.websocket2.onerror = this.webSocketOnError2
          this.websocket2.onmessage = this.webSocketOnMessage2
          this.websocket2.onclose = this.webSocketClose2()
          this.over2 = () => {
            this.websocket2.close()
          }
        },
            // 连接成功
        webSocketOnOpen2 () {
          console.log('2 WebSocket连接成功')
        },
            // 错误
        webSocketOnError2 (e) {
          console.log('2 WebSocket连接发生错误')
        },
            // 数据接收
        webSocketOnMessage2 (e) {
          const receiveData = JSON.parse(e.data)
          this.smallMsgShow(receiveData)
        },
            // 数据发送
        webSocketSend2 (data) {
          this.websocket.send(data)
        },
            // 关闭
        webSocketClose2 (e) {
          console.log('2 WebSocket连接关闭')
        },
        smallMsgShow (messageContent) {
          this.handleUpdateMsgStatus(messageContent)
          this.$notify({
            title: messageContent.title,
            message: messageContent.content,
            position: 'bottom-right'
          })
        },
        bigMsgShow () {
          this.bigMessageVisible = true
          this.$nextTick(() => {
            this.$refs.bigMessage.init()
          })
        },
            // 用于单条消息修改状态
        handleUpdateMsgStatus (messageContent) {
          this.webSocketSend(messageContent.id)
        },
            // 用于当前标记为已读
        handleUpdateMsgStatus2 () {
          if (this.messageContentFive) {
            let s = ''
            for (let i = 0; i < this.messageContentFive.length; i++) {
              if (i === this.messageContentFive.length - 1) {
                s += this.messageContentFive[i].id
              } else {
                s += this.messageContentFive[i].id
                s += ','
              }
            }
            this.webSocketSend(s)
          }
        }
      }
    }
</script>

<style scoped>
  .messageHide {
    position: absolute !important;
    top: 14px !important;
    right: -20px !important;
    background: #d00 !important;
    height: 16px !important;
    line-height: 16px !important;
    font-size: 12px !important;
    -webkit-border-radius: 50px !important;
    -moz-border-radius: 50px !important;
    border-radius: 50px !important;
    padding: 0 8px !important;
    color: #fff !important;
  }
</style>
