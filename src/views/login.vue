<template>
  <div class="login-container">
    <canvas ref="bgCanvas" class="bg-canvas"></canvas>

    <el-form ref="loginForm" :model="loginForm" :rules="loginRules" class="login-form-cyber">
      
      <div class="platform-title-container">
        <div class="platform-logo">
          <svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M4.5 3C5.5 5 7 8.5 7 12C7 15.5 5.5 19 4.5 21" stroke="#00ffcc" stroke-width="2.5" stroke-linecap="round"/>
            <path d="M19.5 3C18.5 5 17 8.5 17 12C17 15.5 18.5 19 19.5 21" stroke="#0099ff" stroke-width="2.5" stroke-linecap="round"/>
            <path d="M6.5 6H17.5" stroke="rgba(0,255,204,0.4)" stroke-width="1.5"/>
            <path d="M7 11H17" stroke="rgba(0,153,255,0.4)" stroke-width="1.5"/>
            <path d="M7 15H17" stroke="rgba(0,255,204,0.4)" stroke-width="1.5"/>
            <path d="M6.5 18H17.5" stroke="rgba(0,153,255,0.4)" stroke-width="1.5"/>
          </svg>
        </div>
        <div class="platform-text">
          <h3 class="main-title">染色质相互作用预测平台</h3>
          <span class="sub-title">CHROMATIN INTERACTION PREDICTION PLATFORM</span>
        </div>
      </div>

      <el-form-item prop="username">
        <el-input v-model="loginForm.username" type="text" auto-complete="off" placeholder="用户名 / 账号">
          <svg-icon slot="prefix" icon-class="user" class="el-input__icon input-icon-cyber" />
        </el-input>
      </el-form-item>

      <el-form-item prop="password">
        <el-input v-model="loginForm.password" type="password" auto-complete="off" placeholder="密码" @keyup.enter.native="handleLogin" show-password>
          <svg-icon slot="prefix" icon-class="password" class="el-input__icon input-icon-cyber" />
        </el-input>
      </el-form-item>

      <el-form-item prop="code" v-if="captchaEnabled">
        <el-input v-model="loginForm.code" auto-complete="off" placeholder="验证码" style="width: 63%" @keyup.enter.native="handleLogin">
          <svg-icon slot="prefix" icon-class="validCode" class="el-input__icon input-icon-cyber" />
        </el-input>
        <div class="login-code-cyber">
          <img :src="codeUrl" @click="getCode" class="login-code-img-cyber"/>
        </div>
      </el-form-item>

      <div class="form-options">
        <el-checkbox v-model="loginForm.rememberMe">记住密码</el-checkbox>
        <div v-if="register" class="register-link-box">
          <router-link class="link-type-cyber" :to="'/register'">立即注册</router-link>
        </div>
      </div>

      <el-form-item style="width:100%; margin-top: 10px;">
        <el-button :loading="loading" size="medium" type="primary" class="btn-submit-cyber" @click.native.prevent="handleLogin">
          <span v-if="!loading">登 录</span>
          <span v-else>登 录 中...</span>
        </el-button>
      </el-form-item>
    </el-form>

    <div class="el-login-footer-cyber">
      <span>{{ footerContent }}</span>
    </div>
  </div>
</template>

<script>
import { getCodeImg } from "@/api/login"
import Cookies from "js-cookie"
import { encrypt, decrypt } from '@/utils/jsencrypt'
import defaultSettings from '@/settings'

export default {
  name: "Login",
  data() {
    return {
      title: process.env.VUE_APP_TITLE,
      footerContent: defaultSettings.footerContent,
      codeUrl: "",
      loginForm: {
        username: "admin",
        password: "admin123",
        rememberMe: false,
        code: "",
        uuid: ""
      },
      loginRules: {
        username: [{ required: true, trigger: "blur", message: "请输入您的账号" }],
        password: [{ required: true, trigger: "blur", message: "请输入您的密码" }],
        code: [{ required: true, trigger: "change", message: "请输入验证码" }]
      },
      loading: false,
      captchaEnabled: true,
      register: false,
      redirect: undefined,
      canvasCtx: null,
      animationFrameId: null,
      baseParticles: [],
      helixRotation: 0
    }
  },
  watch: {
    $route: {
      handler: function(route) {
        this.redirect = route.query && route.query.redirect
      },
      immediate: true
    }
  },
  created() {
    this.getCode()
    this.getCookie()
  },
  mounted() {
    this.initCanvasBackground()
  },
  beforeDestroy() {
    if (this.animationFrameId) {
      cancelAnimationFrame(this.animationFrameId)
    }
    window.removeEventListener('resize', this.handleResize)
  },
  methods: {
    getCode() {
      getCodeImg().then(res => {
        this.captchaEnabled = res.captchaEnabled === undefined ? true : res.captchaEnabled
        if (this.captchaEnabled) {
          this.codeUrl = "data:image/gif;base64," + res.img
          this.loginForm.uuid = res.uuid
        }
      })
    },
    getCookie() {
      const username = Cookies.get("username")
      const password = Cookies.get("password")
      const rememberMe = Cookies.get('rememberMe')
      this.loginForm = {
        username: username === undefined ? this.loginForm.username : username,
        password: password === undefined ? this.loginForm.password : decrypt(password),
        rememberMe: rememberMe === undefined ? false : Boolean(rememberMe)
      }
    },
    handleLogin() {
      this.$refs.loginForm.validate(valid => {
        if (valid) {
          this.loading = true
          if (this.loginForm.rememberMe) {
            Cookies.set("username", this.loginForm.username, { expires: 30 })
            Cookies.set("password", encrypt(this.loginForm.password), { expires: 30 })
            Cookies.set('rememberMe', this.loginForm.rememberMe, { expires: 30 })
          } else {
            Cookies.remove("username")
            Cookies.remove("password")
            Cookies.remove('rememberMe')
          }
          this.$store.dispatch("Login", this.loginForm).then(() => {
            this.$router.push({ path: this.redirect || "/" }).catch(()=>{})
          }).catch(() => {
            this.loading = false
            if (this.captchaEnabled) {
              this.getCode()
            }
          })
        }
      })
    },

    // --- Canvas 背景：双螺旋碱基粒子 ---
    initCanvasBackground() {
      const canvas = this.$refs.bgCanvas
      this.canvasCtx = canvas.getContext('2d')

      this.handleResize()
      window.addEventListener('resize', this.handleResize)

      const letters = ['A', 'T', 'C', 'G']
      const particleCount = 60
      this.baseParticles = []

      for (let i = 0; i < particleCount; i++) {
        this.baseParticles.push({
          x: Math.random() * canvas.width,
          y: Math.random() * canvas.height,
          vx: (Math.random() - 0.5) * 0.8,
          vy: (Math.random() - 0.5) * 0.8,
          text: letters[Math.floor(Math.random() * letters.length)],
          fontSize: Math.random() * 12 + 12,
          alpha: Math.random() * 0.4 + 0.2,
          colorType: Math.floor(Math.random() * 2)
        })
      }

      this.renderLoop()
    },
    handleResize() {
      const canvas = this.$refs.bgCanvas
      if (canvas) {
        canvas.width = window.innerWidth
        canvas.height = window.innerHeight
      }
    },
    renderLoop() {
      const canvas = this.$refs.bgCanvas
      const ctx = this.canvasCtx
      if (!canvas || !ctx) return

      const w = canvas.width
      const h = canvas.height

      ctx.clearRect(0, 0, w, h)
      const bgGradient = ctx.createLinearGradient(0, 0, w, h)
      bgGradient.addColorStop(0, '#040814')
      bgGradient.addColorStop(0.5, '#081226')
      bgGradient.addColorStop(1, '#030712')
      ctx.fillStyle = bgGradient
      ctx.fillRect(0, 0, w, h)

      this.helixRotation += 0.012
      const helixPositions = [w * 0.15, w * 0.85]
      const amplitude = 45
      const frequency = 0.007

      helixPositions.forEach(centerX => {
        for (let y = 0; y < h; y += 24) {
          const angleA = y * frequency + this.helixRotation
          const xA = centerX + Math.sin(angleA) * amplitude
          const zA = Math.cos(angleA)

          const angleB = y * frequency + this.helixRotation + Math.PI
          const xB = centerX + Math.sin(angleB) * amplitude
          const zB = Math.cos(angleB)

          const alphaA = (zA + 1) / 2 * 0.4 + 0.1
          const alphaB = (zB + 1) / 2 * 0.4 + 0.1

          ctx.beginPath()
          const lineGrad = ctx.createLinearGradient(xA, y, xB, y)
          lineGrad.addColorStop(0, `rgba(0, 255, 204, ${alphaA * 0.25})`)
          lineGrad.addColorStop(1, `rgba(0, 153, 255, ${alphaB * 0.25})`)
          ctx.strokeStyle = lineGrad
          ctx.lineWidth = 1.2
          ctx.moveTo(xA, y)
          ctx.lineTo(xB, y)
          ctx.stroke()

          ctx.beginPath()
          ctx.arc(xA, y, 3.5 + zA * 1.5, 0, Math.PI * 2)
          ctx.fillStyle = `rgba(0, 255, 204, ${alphaA + 0.2})`
          ctx.shadowBlur = 8
          ctx.shadowColor = '#00ffcc'
          ctx.fill()

          ctx.beginPath()
          ctx.arc(xB, y, 3.5 + zB * 1.5, 0, Math.PI * 2)
          ctx.fillStyle = `rgba(0, 153, 255, ${alphaB + 0.2})`
          ctx.shadowColor = '#0099ff'
          ctx.fill()

          ctx.shadowBlur = 0
        }
      })

      this.baseParticles.forEach(p => {
        p.x += p.vx
        p.y += p.vy

        if (p.x < 0 || p.x > w) p.vx *= -1
        if (p.y < 0 || p.y > h) p.vy *= -1

        ctx.save()
        ctx.globalAlpha = p.alpha
        ctx.font = `bold ${p.fontSize}px Arial, Helvetica, sans-serif`

        ctx.shadowBlur = 10
        if (p.colorType === 0) {
          ctx.fillStyle = '#00ffcc'
          ctx.shadowColor = '#00ffcc'
        } else {
          ctx.fillStyle = '#0099ff'
          ctx.shadowColor = '#0099ff'
        }

        ctx.fillText(p.text, p.x, p.y)
        ctx.restore()
      })

      this.animationFrameId = requestAnimationFrame(this.renderLoop)
    }
  }
}
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
.login-container {
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
  background-color: #02091a;
  overflow: hidden;
}

/* 全屏画布置底 */
.bg-canvas {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 0;
}

/* 完美同步原图：高级感暗色磨砂玻璃面板 */
.login-form-cyber {
  position: relative;
  z-index: 10;
  width: 440px;
  padding: 35px 35px 20px 35px;
  border-radius: 16px;
  background: rgba(4, 17, 44, 0.68);
  border: 1px solid rgba(0, 153, 255, 0.38);
  backdrop-filter: blur(14px);
  -webkit-backdrop-filter: blur(14px);
  box-shadow: 0 15px 45px rgba(0, 5, 18, 0.85), 
              inset 0 0 25px rgba(0, 170, 255, 0.18);
  
  /* 深度重写输入框样式 */
  ::v-deep .el-input__inner {
    height: 42px;
    line-height: 42px;
    background-color: rgba(4, 22, 54, 0.75) !important;
    border: 1px solid rgba(0, 114, 255, 0.38) !important;
    color: #bfe7ff !important;
    font-size: 14px;
    border-radius: 6px;
    padding-left: 40px;
    
    &:focus {
      border-color: #00ffcc !important;
      box-shadow: 0 0 10px rgba(0, 255, 204, 0.35) !important;
    }
  }

  ::v-deep .el-input__inner::placeholder {
    color: #467399 !important;
  }
}

/* 中英文双层复合平台标题 */
.platform-title-container {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 35px;
  gap: 12px;
}

.platform-logo {
  width: 36px;
  height: 36px;
  display: flex;
  align-items: center;
}

.platform-text {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}

.main-title {
  margin: 0;
  padding: 0;
  font-size: 21px;
  font-weight: 600;
  color: #ffffff;
  letter-spacing: 1.5px;
  text-shadow: 0 0 12px rgba(0, 153, 255, 0.65);
}

.sub-title {
  font-size: 9px;
  font-weight: 500;
  color: #0099ff;
  letter-spacing: 0.3px;
  margin-top: 2px;
  opacity: 0.95;
}

/* 增强前缀图标的荧光质感 */
.input-icon-cyber {
  height: 100%;
  width: 15px;
  margin-left: 5px;
  color: #00bfff !important;
  filter: drop-shadow(0 0 4px rgba(0, 191, 255, 0.65));
}

/* 验证码排列 */
.login-code-cyber {
  width: 33%;
  height: 42px;
  float: right;
  border-radius: 6px;
  overflow: hidden;
  border: 1px solid rgba(0, 114, 255, 0.38);
  box-sizing: border-box;
  
  img {
    cursor: pointer;
    width: 100%;
    height: 100%;
    object-fit: cover;
  }
}

.login-code-img-cyber {
  height: 42px;
}

/* 辅助选项样式适配 */
.form-options {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin: -5px 0 25px 0;
}

::v-deep .el-checkbox {
  color: #608eb3 !important;
  
  .el-checkbox__inner {
    background-color: rgba(4, 24, 60, 0.85);
    border-color: rgba(0, 114, 255, 0.5);
  }
  
  .el-checkbox__input.is-checked .el-checkbox__inner {
    background-color: #0072ff;
    border-color: #00ffcc;
  }
  
  .el-checkbox__input.is-checked + .el-checkbox__label {
    color: #00ffcc !important;
  }
}

.register-link-box {
  font-size: 14px;
}

.link-type-cyber {
  color: #00bfff;
  text-decoration: none;
  transition: color 0.3s;
  
  &:hover {
    color: #00ffcc;
  }
}

/* 高亮流光登录按钮 */
.btn-submit-cyber {
  width: 100%;
  height: 42px;
  font-size: 16px;
  font-weight: 600;
  letter-spacing: 4px;
  border: none !important;
  background: linear-gradient(90deg, #0052d4, #0072ff, #00c6ff) !important;
  box-shadow: 0 4px 15px rgba(0, 114, 255, 0.45);
  transition: all 0.3s ease;

  &:hover {
    background: linear-gradient(90deg, #0072ff, #00c6ff, #00ffcc) !important;
    box-shadow: 0 4px 20px rgba(0, 255, 204, 0.55);
    transform: translateY(-1px);
  }
  
  &:active {
    transform: translateY(1px);
  }
}

/* 底部版权说明 */
.el-login-footer-cyber {
  height: 40px;
  line-height: 40px;
  position: fixed;
  bottom: 0;
  width: 100%;
  text-align: center;
  color: rgba(96, 142, 179, 0.65);
  font-family: Arial, sans-serif;
  font-size: 12px;
  letter-spacing: 0.8px;
  z-index: 5;
}
</style>