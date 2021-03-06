<template>
  <common-layout>
    <div class="top">
      <div class="header">
        <span class="title">{{ systemName }}</span>
      </div>
      <div class="desc">ConteMan Can't Stop</div>
    </div>
    <div class="login">
      <a-form :form="form" @submit="onSubmit">
        <div>
          <a-alert
            v-show="error"
            type="error"
            :closable="true"
            :message="error"
            show-icon
            style="margin-bottom: 24px;"
          />
          <a-form-item>
            <a-input
              v-decorator="['name', {rules: [{ required: true, message: '请输入用户名', whitespace: true}]}]"
              autocomplete="autocomplete"
              size="large"
              placeholder="请输入用户名"
            >
              <a-icon slot="prefix" type="user" />
            </a-input>
          </a-form-item>
          <a-form-item>
            <a-input
              v-decorator="['password', {rules: [{ required: true, message: '请输入密码', whitespace: true}]}]"
              size="large"
              placeholder="请输入密码"
              autocomplete="autocomplete"
              type="password"
            >
              <a-icon slot="prefix" type="lock" />
            </a-input>
          </a-form-item>
        </div>
        <a-form-item>
          <a-button
            :loading="logging"
            style="width: 100%;margin-top: 24px"
            size="large"
            html-type="submit"
            type="primary"
          >登录
          </a-button>
        </a-form-item>
      </a-form>
    </div>
  </common-layout>
</template>

<script>
import CommonLayout from '@/layouts/CommonLayout'
import { User } from '@/services'
import { setAuthorization } from '@/utils/request'
import { loadRoutes } from '@/utils/routerUtil'
import { mapMutations } from 'vuex'

export default {
  name: 'Login',
  components: {
    CommonLayout
  },
  data() {
    return {
      logging: false,
      error: '',
      form: this.$form.createForm(this)
    }
  },
  computed: {
    systemName() {
      return this.$store.state.setting.systemName
    }
  },
  methods: {
    ...mapMutations('account', ['setUser', 'setPermissions', 'setRoles']),
    onSubmit(e) {
      e.preventDefault()
      this.form.validateFields((err) => {
        if (!err) {
          this.logging = true
          const name = this.form.getFieldValue('name')
          const password = this.form.getFieldValue('password')
          User.login(name, password)
            .then(this.afterLogin)
            .catch(e => {
              this.logging = false
              console.log(e.message)
            })
        }
      })
    },
    afterLogin(res) {
      this.logging = false
      const loginRes = res.data
      if (loginRes.code === 0) {
        const { user, permissions, roles } = loginRes.data
        this.setUser(user)
        this.setPermissions(permissions)
        this.setRoles(roles)
        setAuthorization({
          token: loginRes.data.token,
          expireAt: new Date(loginRes.data.expireAt)
        })
        // 获取路由配置
        User.getRoutesConfig().then(result => {
          const routesConfig = result.data.data
          loadRoutes(routesConfig)
          this.$router.push('/dashboard')
          this.$message.success(loginRes.msg, 3)
        })
      } else {
        this.error = loginRes.msg
      }
    }
  }
}
</script>

<style lang="less" scoped>
@import "./src/theme/index";

.common-layout {
  .top {
    text-align: center;

    .header {
      height: 44px;
      line-height: 44px;

      a {
        text-decoration: none;
      }

      .logo {
        height: 44px;
        vertical-align: top;
        margin-right: 16px;
      }

      .title {
        font-size: 33px;
        color: @title-color;
        font-family: 'Myriad Pro', 'Helvetica Neue', Arial, Helvetica, sans-serif;
        font-weight: 600;
        position: relative;
        top: 2px;
      }
    }

    .desc {
      font-size: 14px;
      color: @text-color-second;
      margin-top: 12px;
      margin-bottom: 40px;
    }
  }

  .login {
    width: 368px;
    margin: 0 auto;
    @media screen and (max-width: 576px) {
      width: 95%;
    }
    @media screen and (max-width: 320px) {
      .captcha-button {
        font-size: 14px;
      }
    }

    .icon {
      font-size: 24px;
      color: @text-color-second;
      margin-left: 16px;
      vertical-align: middle;
      cursor: pointer;
      transition: color 0.3s;

      &:hover {
        color: @primary-color;
      }
    }
  }
}
</style>
