<template>
    <div class="login-homepage-form">
        <div class="container">
            <div class="content-login col-md-5 m-auto col-sm-8">
                <div class="logo-left">
                    <img src="/images/logo.jpg" class="size-img">
                </div>
                <div class="login-box-body">
                    <p class="green-strong">Team Collaborate</p>
                    <p>Join with us</p>
                    <transition name="fade">
                        <div class="alert alert-danger" v-if="timeout">
                            Request Timeout! Đăng nhập thất bại.
                        </div>
                        <div class="alert alert-danger" v-show="isLoginFbFail">
                            {{ errorLoginFbFail }}
                        </div>
                    </transition>

                    <login-form :logging-in="loggingIn" @setLogin="loggingIn = !loggingIn"/>
                    <div class="row margin-top-40 form-login">
                        <div class="row">
                            <div class="col-xs-12 button-left text-center margin-left-10">
                                <router-link :to="{ name: 'forgot_password' }">Quên mật khẩu</router-link>
                            </div>
                        </div>
                    </div>
                    <div class="social-auth-links text-center connect-facebook height-auto margin-top-20">
                        <p class="clearfix text-or text-center margin-top-20">Hoặc</p>
                        <a @click.prevent="register()" href="#"
                           class="register float-right"
                        >
                            Đăng ký tài khoản mới</a>

                        <a href="#" @click.prevent="loginByFB"
                           :disabled="loggingIn"
                           @disableLoginBtns="loggingIn = true"
                           @enableLoginBtns="loggingIn = false"
                           class="facebook float-right margin-bottom-20"
                        >
                            <i class="fab fa-google-plus-g"></i>
                            KẾT NỐI QUA GOOGLE</a>
                    </div>

                </div>
            </div>
        </div>
    </div>
</template>

<script>
  import LoginForm from './LoginForm'
  import Axios from 'axios'
  import {get} from "../../../helper/request";

  export default {
    data() {
      return {
        timeout: false,
        loggingIn: false,
        urlHomepage: '',
        isLoginFbFail: false,
        errorLoginFbFail: '',
        urlLoginFB: ''
      }
    },
    components: {
      'login-form': LoginForm
    },
    methods: {
      loginByFB: function () {
        this.loggingIn = true;

        let FBAuthPopup = window.open('http://' + this.urlLoginFB, 'Facebook Auth', 'height=600,width=450');
        let self = this;
        let INTERVAL = 500;
        let LOGIN_TIME_LIMIT = 5 * 60 * 1000;
        let loginTime = 0;
        let timeLoginFails = 0;
        let loginInterval = setInterval(function () {
          loginTime += INTERVAL;
          if (FBAuthPopup.closed) {
            let token = window.localStorage.getItem('token');
            if (token) {
              self.$store.dispatch('auth/saveToken', {token}).then();

              self.$store.dispatch('auth/fetchUser')
                .then(() => {
                  clearInterval(loginInterval);
                  self.loggingIn = false;
                  self.$router.push({name: 'homeIndex'});
                })
                .catch(() => {
                  clearInterval(loginInterval);
                  self.loggingIn = false;
                });
            } else {
              if(!self.isLoginFbFail) {
                timeLoginFails = loginTime;
              }
              self.isLoginFbFail = true;
              self.errorLoginFbFail = 'Có vấn đề khi đăng nhập bằng tài khoản Google của bạn. ' +
                'Vui lòng liên hệ quản trị viên để biết thêm chi tiết';
              self.loggingIn = false;
            }
          }
          if (timeLoginFails > 0 && (loginTime - timeLoginFails) > 3000 && self.isLoginFbFail) {
            self.isLoginFbFail = false;
            clearInterval(loginInterval);
            self.loggingIn = false;
          }
          if (loginTime > LOGIN_TIME_LIMIT) {
            clearInterval(loginInterval);
            self.timeout = true;
            self.loggingIn = false;
            self.isLoginFbFail = false
          }
        }, INTERVAL);
      },
      redirectHomepage: function () {
        get(`/api/user/auth/redirect-homepage`)
          .then((res) => {
            this.urlHomepage = res.data.urlHomepage
          })
      },
      loginFacebook: function () {
        get(`/api/user/auth/login-facebook`)
          .then((res) => {
            this.urlLoginFB = res.data.urlLoginFB
          })
      },

      register() {
        this.$router.push({
          name: 'register'
        })
      }

    },
    created: function () {
      this.redirectHomepage()
      this.loginFacebook()
    },
    beforeMount: function () {
      // $(function () {
      //   $('input').iCheck({
      //     checkboxClass: 'icheckbox_square-blue',
      //     radioClass: 'iradio_square-blue',
      //     increaseArea: '20%'
      //   });
      // });

      let classList = document.body.classList;
      while (classList.length > 0) {
        classList.remove(classList.item(0));
      }
      document.body.classList.add("hold-transition", "login-page", "login-homepage");
    }
  }
</script>
<style scoped>
    .size-img {
        max-height: 100px;
        max-width: 100px;
    }

    .height-auto{
        height: 120px;
    }
</style>