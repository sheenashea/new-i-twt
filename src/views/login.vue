<template>
  <v-app class="bg">
    <v-main>
      <v-container py-5>
        <v-layout align-center justify-center py-3>
          <v-flex xs12 sm8 md4>
            <v-card flat class="px-2 pb-3 login-card">
              <v-card-text>
                <v-layout justify-center pt-2>
                  <v-avatar size="60px">
                    <img src="@/assets/logo.png" alt="twt" />
                  </v-avatar>
                </v-layout>
                <v-layout justify-center py-3>
                  <div class="headline">天外天账户登录</div>
                </v-layout>
                <v-tabs centered fixed-tabs v-model="tab">
                  <v-tabs-slider color="#1E88E5"> </v-tabs-slider>
                  <v-tab href="#tab-1">密码登录</v-tab>
                  <v-tab href="#tab-2">短信登录</v-tab>
                </v-tabs>
                <v-tabs-items v-model="tab">
                  <v-tab-item value="tab-1" style="paddingtop: 20px">
                    <v-form>
                      <v-text-field
                        name="username"
                        label="邮箱/用户名/学号"
                        type="text"
                        required
                        v-model="username"
                        :rules="usernameRules"
                      ></v-text-field>
                      <v-text-field
                        id="password"
                        name="密码"
                        label="密码"
                        :append-icon="showPsw ? 'mdi-eye' : 'mdi-eye-off'"
                        @click:append="showPsw = !showPsw"
                        :type="showPsw ? 'text' : 'password'"
                        required
                        v-model="password"
                        :rules="passwordRules"
                      ></v-text-field>
                    </v-form>
                    <v-card-actions>
                      <v-btn
                        block
                        depressed
                        color="info"
                        @click="loginCommon"
                        :loading="loginBtnLoading"
                        >登录
                      </v-btn>
                    </v-card-actions>
                  </v-tab-item>
                  <v-tab-item value="tab-2" style="paddingtop: 20px">
                    <v-form>
                      <v-text-field
                        name="phone"
                        label="手机号"
                        type="text"
                        required
                        v-model="phone"
                        :rules="phoneRules"
                      >
                        <template v-slot:append>
                          <v-btn @click="sendVerifyCode" text
                            >发送验证码
                          </v-btn>
                        </template>
                      </v-text-field>
                      <v-text-field
                        autocomplete="off"
                        id="verify"
                        name="验证码"
                        label="验证码"
                        type="text"
                        required
                        v-model="verify"
                        :rules="verifyRules"
                      ></v-text-field>
                      <v-layout v-if="recaptchaLoaded"
                        ><div id="grecaptcha"></div
                      ></v-layout>
                    </v-form>
                    <v-card-actions>
                      <v-btn
                        block
                        :loading="loginBtnLoading"
                        depressed
                        color="info"
                        @click="loginPhone"
                        >登录
                      </v-btn>
                    </v-card-actions>
                  </v-tab-item>
                </v-tabs-items>
                <div class="func-group">
                  <a href="#/create">注册账号</a>
                  <a @click="dialog = true">忘记密码？去重置</a>
                </div>
              </v-card-text>
            </v-card>
          </v-flex>
        </v-layout>
      </v-container>
    </v-main>
    <div class="footer">
      <span>© {{ getYear() }} 天外天</span>
    </div>
    <v-dialog v-model="dialog" max-width="600px">
      <v-card>
        <v-card-title>
          <span class="headline">重置密码</span>
        </v-card-title>
        <v-card-text>
          <v-form ref="form">
            <v-text-field
              v-model="rePWD.userNumber"
              label="用户编号"
              required
            ></v-text-field>
            <v-text-field
              v-model="rePWD.idNumber"
              label="身份证号"
              required
            ></v-text-field>
            <v-text-field
              v-model="rePWD.newPassword"
              label="新密码"
              type="password"
              required
            ></v-text-field>
          </v-form>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="primary" @click="handleResetPwd">重置密码 </v-btn>
          <v-btn text @click="dialog = false">取消 </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-app>
</template>

<script>
import {
  login,
  loginWithPhone,
  loginVerifyCode,
  googlerecaptcha,
} from "@/api/user";
import Message from "@/components/message";
import { setToken } from "@/utils/auth";
import { rePWDByID } from "../api/user";

export default {
  name: "login",
  data() {
    return {
      sitekey: "6LeIxAcTAAAAAJcZVRqyHh71UMIEGNQ_MXjiZKhI",
      newPas: "",
      rePWD: {
        userNumber: "",
        idNumber: "",
        newPassword: "",
      },
      loginBtnLoading: false,
      dialog: false,
      from: this.$route.query.from,
      showPsw: false,
      tab: "",
      username: "",
      usernameRules: [(v) => !!v || "请输入邮箱/用户名/学号"],
      password: "",
      passwordRules: [(v) => !!v || "请输入密码"],
      phone: "",
      phoneRules: [
        (v) => !!v || "请输入手机号",
        (v) =>
          (v &&
            (function (x) {
              let s =
                /^(13[0-9]|14[01456879]|15[0-3,5-9]|16[2567]|17[0-8]|18[0-9]|19[0-3,5-9])\d{8}$/;
              return s.test(x);
            })(v)) ||
          "请您输入正确的手机号",
      ],
      verify: "",
      verifyRules: [
        (v) => !!v || "请输入验证码",
        (v) =>
          (v &&
            (function (x) {
              let s = /^\d{6}$/;
              return s.test(x);
            })(v)) ||
          "请您输入合法的验证码",
      ],
      recaptchaLoaded: false, // 控制 reCAPTCHA 组件是否显示
    };
  },
  methods: {
    getYear() {
      let date = new Date();
      return date.getFullYear();
    },
    sendVerifyCode() {
      // 验证手机号格式
      const phoneValid = this.phoneRules.every(
        (rule) => rule(this.phone) === true
      );
      if (!phoneValid) {
        this.$message.error("请输入合法的手机号");
        return;
      }

      if (!this.recaptchaLoaded) {
        this.recaptchaLoaded = true;
        this.$nextTick(() => {
          window.grecaptcha.render("grecaptcha", {
            sitekey: this.sitekey,
            callback: this.handleRecaptchaSuccess,
            "expired-callback": this.expiredCallback,
          });
        });
      }
      Message.warning("请点击按钮进行验证");
    },

    handleRecaptchaSuccess(token) {
      //console.log("11111111", token);
      // 处理验证码发送的逻辑
      googlerecaptcha(token)
        .then((res) => {
          console.log(res);
          if (res) {
            // 发送验证码
            this.processSendVerifyCode();
          }
        })
        .catch((error) => {
          console.error(error);
        });
    },

    expiredCallback() {
      console.log("expire");
    },

    processSendVerifyCode() {
      //console.log("1111112222");
      let data = { phone: this.phone };
      loginVerifyCode(data)
        .then(() => {
          Message.success("短信发送成功");
        })
        .catch((error) => {
          console.error(error);
          Message.error("短信发送失败");
        });
    },

    loginCommon() {
      this.loginBtnLoading = true;
      let data = {
        account: this.username,
        password: this.password,
      };
      login(data)
        .then((value) => {
          this.$message.success(`登录成功`);
          setToken(value.result.token);
          sessionStorage.setItem("basicInfo", JSON.stringify(value.result));
          if (this.from) {
            this.$router.push({ path: this.from });
          } else {
            this.$router.push({ path: "/home" });
          }
          this.loginBtnLoading = false;
        })
        .catch((value) => {
          console.log(value);
          this.loginBtnLoading = false;
        });
    },

    loginPhone() {
      this.loginBtnLoading = true;
      let data = {
        phone: this.phone,
        code: this.verify,
      };
      loginWithPhone(data)
        .then((value) => {
          Message.success(`登录成功`);
          setToken(value.result.token);
          sessionStorage.setItem("basicInfo", JSON.stringify(value.result));
          if (this.from) {
            this.$router.push({ path: this.from });
          } else {
            this.$router.push({ path: "/home" });
          }
          this.loginBtnLoading = false;
        })
        .catch((value) => {
          console.log(value);
          this.loginBtnLoading = false;
        });
    },

    handleResetPwd() {
      let flag = true;
      for (let ref in this.$refs) {
        flag &&= this.$refs[ref].validate(true);
      }
      if (flag) {
        rePWDByID(this.rePWD).then(() => {
          this.$message.success("密码已重置");
        });
      }
    },
  },
  mounted() {
    //初始化
  },
  created() {
    sessionStorage.clear();
  },
};
</script>

<style scoped>
.bg {
  background-color: #f5f5f5;
}
.login-card {
  width: 100%;
}
.headline {
  font-size: 24px;
  font-weight: bold;
}
.func-group {
  margin-top: 10px;
  text-align: center;
}
.footer {
  text-align: center;
  padding: 10px;
}
</style>

<style lang="scss" scoped>
div.bg {
  background: #026fce;
  background-image: linear-gradient(
    45deg,
    hsl(170deg, 80%, 70%),
    hsl(190deg, 80%, 70%),
    hsl(250deg, 80%, 70%),
    hsl(3200deg, 80%, 70%)
  );
  overflow: hidden;
  background-size: 200% 200%;
  animation: gradient-move 3s ease alternate infinite;

  .login-card {
    margin-top: 20%;
  }

  .func-group {
    width: 100%;
    display: flex;
    justify-content: space-between;
    margin-top: 14px;

    a {
      text-decoration: none;
    }
  }
}

.footer {
  bottom: 30px;
  position: absolute;
  width: calc(100% - 24px);
  margin: 0 12px;
  text-align: center;

  .copyright {
    position: relative;
    display: inline-block;
    padding: 8px 20px;
    margin: 0 auto;
    border-radius: 50px;
    background: rgba(255, 255, 255, 0.5);
    text-align: center;
    max-width: 940px;
    font-size: 14px;
    line-height: 24px;

    a {
      text-decoration: none;
      color: #000;
    }
  }
}

@keyframes gradient-move {
  0% {
    background-position: 0% 0%;
  }
  100% {
    background-position: 100% 100%;
  }
}
</style>
