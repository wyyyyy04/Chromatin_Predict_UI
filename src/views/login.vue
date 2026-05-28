<template>
  <div class="login">
    <canvas ref="loginCanvas" class="login-canvas"></canvas>

    <el-form ref="loginForm" :model="loginForm" :rules="loginRules" class="login-form">
      <h3 class="title">{{title}}</h3>
      <el-form-item prop="username">
        <el-input
          v-model="loginForm.username"
          type="text"
          auto-complete="off"
          placeholder="账号"
        >
          <svg-icon slot="prefix" icon-class="user" class="el-input__icon input-icon" />
        </el-input>
      </el-form-item>
      <el-form-item prop="password">
        <el-input
          v-model="loginForm.password"
          type="password"
          auto-complete="off"
          placeholder="密码"
          @keyup.enter.native="handleLogin"
        >
          <svg-icon slot="prefix" icon-class="password" class="el-input__icon input-icon" />
        </el-input>
      </el-form-item>
      <el-form-item prop="code" v-if="captchaEnabled">
        <el-input
          v-model="loginForm.code"
          auto-complete="off"
          placeholder="验证码"
          style="width: 63%"
          @keyup.enter.native="handleLogin"
        >
          <svg-icon slot="prefix" icon-class="validCode" class="el-input__icon input-icon" />
        </el-input>
        <div class="login-code">
          <img :src="codeUrl" @click="getCode" class="login-code-img"/>
        </div>
      </el-form-item>
      <el-checkbox v-model="loginForm.rememberMe" style="margin:0px 0px 25px 0px;">记住密码</el-checkbox>
      <el-form-item style="width:100%;">
        <el-button
          :loading="loading"
          size="medium"
          type="primary"
          style="width:100%;"
          @click.native.prevent="handleLogin"
        >
          <span v-if="!loading">登 录</span>
          <span v-else>登 录 中...</span>
        </el-button>
        <div style="float: right;" v-if="register">
          <router-link class="link-type" :to="'/register'">立即注册</router-link>
        </div>
      </el-form-item>
    </el-form>
    <div class="el-login-footer">
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
        username: [
          { required: true, trigger: "blur", message: "请输入您的账号" }
        ],
        password: [
          { required: true, trigger: "blur", message: "请输入您的密码" }
        ],
        code: [{ required: true, trigger: "change", message: "请输入验证码" }]
      },
      loading: false,
      // 验证码开关
      captchaEnabled: true,
      // 注册开关
      register: false,
      redirect: undefined,
      
      // Canvas 动画内部控制变量（非响应式，避免性能开销）
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
    // 销毁组件时释放定时器与事件监听，防止内存泄漏
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

    // --- Canvas 特效核心逻辑 ---
    initCanvasBackground() {
      const canvas = this.$refs.loginCanvas
      this.canvasCtx = canvas.getContext('2d')
      
      this.handleResize()
      window.addEventListener('resize', this.handleResize)

      // 初始化随机漂浮的 ATCG 碱基粒子
      const letters = ['A', 'T', 'C', 'G']
      const particleCount = 60 // 粒子总数量
      this.baseParticles = []

      for (let i = 0; i < particleCount; i++) {
        this.baseParticles.push({
          x: Math.random() * canvas.width,
          y: Math.random() * canvas.height,
          vx: (Math.random() - 0.5) * 0.8, // X轴漂移速度
          vy: (Math.random() - 0.5) * 0.8, // Y轴漂移速度
          text: letters[Math.floor(Math.random() * letters.length)],
          fontSize: Math.random() * 12 + 12, // 12px ~ 24px
          alpha: Math.random() * 0.4 + 0.2,  // 柔和透明度
          colorType: Math.floor(Math.random() * 2) // 两种科技配色的随机选择
        })
      }

      this.renderLoop()
    },
    handleResize() {
      const canvas = this.$refs.loginCanvas
      if (canvas) {
        canvas.width = window.innerWidth
        canvas.height = window.innerHeight
      }
    },
    renderLoop() {
      const canvas = this.$refs.loginCanvas
      const ctx = this.canvasCtx
      if (!canvas || !ctx) return

      const w = canvas.width
      const h = canvas.height

      // 1. 渲染深邃科技感渐变背景
      ctx.clearRect(0, 0, w, h)
      const bgGradient = ctx.createLinearGradient(0, 0, w, h)
      bgGradient.addColorStop(0, '#040814')
      bgGradient.addColorStop(0.5, '#081226')
      bgGradient.addColorStop(1, '#030712')
      ctx.fillStyle = bgGradient
      ctx.fillRect(0, 0, w, h)

      // 2. 渲染动态旋转的 DNA 双螺旋结构（左、右两侧对称分布，不干扰主登录框）
      this.helixRotation += 0.012
      const helixPositions = [w * 0.15, w * 0.85] // 左侧 15% 处，右侧 85% 处
      const amplitude = 45  // 螺旋形态振幅
      const frequency = 0.007 // 螺旋密集波长

      helixPositions.forEach(centerX => {
        // 每隔 24 像素绘制一组碱基对连线
        for (let y = 0; y < h; y += 24) {
          // 链 A 的参数坐标
          const angleA = y * frequency + this.helixRotation
          const xA = centerX + Math.sin(angleA) * amplitude
          const zA = Math.cos(angleA) // 用余弦模拟空间 3D 深度的缩放与透明度变化

          // 链 B 的参数坐标 (相位刚好相差 PI 即 180 度)
          const angleB = y * frequency + this.helixRotation + Math.PI
          const xB = centerX + Math.sin(angleB) * amplitude
          const zB = Math.cos(angleB)

          // 转化深度为柔和的透明度与大小
          const alphaA = (zA + 1) / 2 * 0.4 + 0.1
          const alphaB = (zB + 1) / 2 * 0.4 + 0.1

          // 绘制碱基氢键连接线
          ctx.beginPath()
          const lineGrad = ctx.createLinearGradient(xA, y, xB, y)
          lineGrad.addColorStop(0, `rgba(0, 255, 204, ${alphaA * 0.25})`)
          lineGrad.addColorStop(1, `rgba(0, 153, 255, ${alphaB * 0.25})`)
          ctx.strokeStyle = lineGrad
          ctx.lineWidth = 1.2
          ctx.moveTo(xA, y)
          ctx.lineTo(xB, y)
          ctx.stroke()

          // 绘制两条骨架单链上的节点磷酸基团
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
          
          // 重置外部光晕，不污染后续绘制
          ctx.shadowBlur = 0 
        }
      })

      // 3. 渲染全屏动态漂浮围绕的 ATCG 碱基粒子
      this.baseParticles.forEach(p => {
        p.x += p.vx
        p.y += p.vy

        // 碰壁边缘反弹逻辑
        if (p.x < 0 || p.x > w) p.vx *= -1
        if (p.y < 0 || p.y > h) p.vy *= -1

        ctx.save()
        ctx.globalAlpha = p.alpha
        ctx.font = `bold ${p.fontSize}px Arial, Helvetica, sans-serif`
        
        // 赋予粒子霓虹荧光特效
        ctx.shadowBlur = 10
        if (p.colorType === 0) {
          ctx.fillStyle = '#00ffcc' // 冷翠绿
          ctx.shadowColor = '#00ffcc'
        } else {
          ctx.fillStyle = '#0099ff' // 科技蓝
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
.login {
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
  background-color: #040814; /* 兜底暗色背景 */
  overflow: hidden;
}

/* 后台画布全屏置底样式 */
.login-canvas {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 0;
}

.title {
  margin: 0px auto 30px auto;
  text-align: center;
  color: #c0c6d3; /* 适配暗色背景，微调成亮灰色 */
  font-weight: 600;
  letter-spacing: 1px;
}

.login-form {
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.94); /* 增加微弱透明感，隐约透出背景特效 */
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.5);
  width: 400px;
  padding: 25px 25px 5px 25px;
  z-index: 1; /* 保证表单组件高亮在画布之上 */
  
  .el-input {
    height: 38px;
    input {
      height: 38px;
    }
  }
  .input-icon {
    height: 39px;
    width: 14px;
    margin-left: 2px;
  }
}

.login-tip {
  font-size: 13px;
  text-align: center;
  color: #bfbfbf;
}

.login-code {
  width: 33%;
  height: 38px;
  float: right;
  img {
    cursor: pointer;
    vertical-align: middle;
  }
}

.el-login-footer {
  height: 40px;
  line-height: 40px;
  position: fixed;
  bottom: 0;
  width: 100%;
  text-align: center;
  color: rgba(255, 255, 255, 0.6);
  font-family: Arial;
  font-size: 12px;
  letter-spacing: 1px;
  z-index: 1;
}

.login-code-img {
  height: 38px;
}
</style>