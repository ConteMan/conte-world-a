<template>
  <a-dropdown>
    <div class="header-avatar" style="cursor: pointer">
      <!--      <a-avatar class="avatar" size="small" shape="circle" :src="user.avatar"/>-->
      <span class="name">{{ user.name }}</span>
    </div>
    <a-menu slot="overlay" :class="['avatar-menu']">
      <a-menu-item @click="() => $router.push({path: '/mine/info'})">
        <a-icon type="user" />
        <span>修改密码</span>
      </a-menu-item>
      <a-menu-item @click="setSettingStatus(true)">
        <a-icon type="setting" />
        <span>主题配置</span>
      </a-menu-item>
      <a-menu-divider />
      <a-menu-item @click="logout">
        <a-icon style="margin-right: 8px;" type="poweroff" />
        <span>退出登录</span>
      </a-menu-item>
    </a-menu>
  </a-dropdown>
</template>

<script>
import { mapGetters, mapMutations } from 'vuex'
import { User } from '@/services'

export default {
  name: 'HeaderAvatar',
  computed: {
    ...mapGetters('account', ['user']),
  },
  methods: {
    ...mapMutations('setting', ['setSettingStatus']),
    logout() {
      User.logout()
      this.$router.push('/login')
    }
  }
}
</script>

<style lang="less">
.header-avatar {
  display: inline-flex;

  .avatar, .name {
    align-self: center;
  }

  .avatar {
    margin-right: 8px;
  }

  .name {
    font-weight: 500;
  }
}

.avatar-menu {
  width: 150px;
}

</style>
