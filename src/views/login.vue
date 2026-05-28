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
      redirect: undefined
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
    this.initCyberBackground()
  },
  beforeDestroy() {
    if (this.animId) cancelAnimationFrame(this.animId)
    window.removeEventListener('resize', this.resizeCanvas)
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

    // --- 高级 Canvas 粒子液态系统 ---
    initCyberBackground() {
      const canvas = this.$refs.bgCanvas
      if (!canvas) return
      this.ctx = canvas.getContext('2d')
      
      this.resizeCanvas()
      window.addEventListener('resize', this.resizeCanvas)

      // 1. 初始化全屏及围绕漂浮的晶莹圆形水珠粒子 (Water Droplets)
      this.waterDroplets = Array.from({ length: 55 }, () => ({
        x: Math.random() * canvas.width,
        y: Math.random() * canvas.height,
        vx: (Math.random() - 0.5) * 0.4,
        vy: (Math.random() - 0.5) * 0.4,
        radius: Math.random() * 3.5 + 2, // 2px 到 5.5px 大小的精致水珠
        alpha: Math.random() * 0.4 + 0.2,
        color: Math.random() > 0.45 ? '#00ffcc' : '#0099ff',
        phase: Math.random() * Math.PI * 2, // 独立呼吸相位
        speed: 0.02 + Math.random() * 0.02
      }))

      // 2. 初始化紧密围绕在右侧 X 染色体周围的专属微型水珠气泡群
      this.chromDroplets = Array.from({ length: 25 }, () => ({
        angle: Math.random() * Math.PI * 2,
        distance: Math.random() * 80 + 40, // 环绕半径距离
        rotSpeed: (Math.random() - 0.5) * 0.015,
        radius: Math.random() * 2 + 1.5,
        alpha: Math.random() * 0.5 + 0.3,
        color: Math.random() > 0.5 ? '#00bcff' : '#00ffcc'
      }))

      this.globalTime = 0
      this.drawLoop()
    },
    resizeCanvas() {
      const canvas = this.$refs.bgCanvas
      if (canvas) {
        canvas.width = window.innerWidth
        canvas.height = window.innerHeight
      }
    },
    drawLoop() {
      const canvas = this.$refs.bgCanvas
      const ctx = this.ctx
      if (!canvas || !ctx) return

      const w = canvas.width
      const h = canvas.height

      this.globalTime += 0.015

      // 绘制深空科技背景底色
      ctx.fillStyle = '#020a1c'
      ctx.fillRect(0, 0, w, h)
      
      // 渲染复合大范围科幻暗蓝微弱发光
      let radialGlow = ctx.createRadialGradient(w/2, h/2, 10, w/2, h/2, Math.max(w, h))
      radialGlow.addColorStop(0, 'rgba(3, 28, 68, 0.55)')
      radialGlow.addColorStop(1, '#010612')
      ctx.fillStyle = radialGlow
      ctx.fillRect(0, 0, w, h)

      // ================= 特效一：全屏动感圆形水珠渲染 (替代原有碱基) =================
      this.waterDroplets.forEach(p => {
        p.x += p.vx
        p.y += p.vy
        p.phase += p.speed

        // 越界边缘平滑反弹
        if (p.x < 0 || p.x > w) p.vx *= -1
        if (p.y < 0 || p.y > h) p.vy *= -1

        // 水珠晶莹感：通过局部正弦计算出呼吸透明度
        const currentAlpha = p.alpha + Math.sin(p.phase) * 0.15

        ctx.save()
        ctx.beginPath()
        ctx.arc(p.x, p.y, p.radius, 0, Math.PI * 2)
        
        // 建立圆形水珠的高光高透渐变
        let dropGrad = ctx.createRadialGradient(p.x - p.radius*0.3, p.y - p.radius*0.3, p.radius * 0.1, p.x, p.y, p.radius)
        dropGrad.addColorStop(0, 'rgba(255, 255, 255, 0.85)') // 水珠表面的受光小高光
        dropGrad.addColorStop(0.3, p.color)
        dropGrad.addColorStop(1, 'rgba(0,0,0,0)')

        ctx.fillStyle = dropGrad
        ctx.globalAlpha = Math.max(0.1, Math.min(currentAlpha, 0.8))
        ctx.shadowBlur = 6
        ctx.shadowColor = p.color
        ctx.fill()
        ctx.restore()
      })

      // ================= 特效二：左侧完美复刻高精度 3D DNA 双螺旋 =================
      const dnaCenterX = w * 0.13
      const dnaAmplitude = 48  // 振幅
      const dnaFrequency = 0.0068 // 波长密集度

      for (let y = 50; y < h - 50; y += 22) {
        const angle1 = y * dnaFrequency + this.globalTime
        const angle2 = angle1 + Math.PI

        const x1 = dnaCenterX + Math.sin(angle1) * dnaAmplitude
        const x2 = dnaCenterX + Math.sin(angle2) * dnaAmplitude

        const z1 = Math.cos(angle1)
        const z2 = Math.cos(angle2)

        const alpha1 = (z1 + 1) / 2 * 0.45 + 0.2
        const alpha2 = (z2 + 1) / 2 * 0.45 + 0.2

        // 1. 绘制中轴氢键连接线
        ctx.beginPath()
        const lineGrad = ctx.createLinearGradient(x1, y, x2, y)
        lineGrad.addColorStop(0, `rgba(0, 255, 204, ${alpha1 * 0.25})`)
        lineGrad.addColorStop(1, `rgba(0, 153, 255, ${alpha2 * 0.25})`)
        ctx.strokeStyle = lineGrad
        ctx.lineWidth = 1.2
        ctx.moveTo(x1, y)
        ctx.lineTo(x2, y)
        ctx.stroke()

        // 2. 绘制链 A 上的水珠形节点
        const r1 = 4.5 + z1 * 1.8
        ctx.save()
        ctx.beginPath()
        ctx.arc(x1, y, r1, 0, Math.PI * 2)
        let nodeGrad1 = ctx.createRadialGradient(x1 - r1*0.2, y - r1*0.2, r1*0.1, x1, y, r1)
        nodeGrad1.addColorStop(0, 'rgba(255, 255, 255, 0.8)')
        nodeGrad1.addColorStop(0.4, '#00ffcc')
        nodeGrad1.addColorStop(1, 'rgba(0, 255, 204, 0)')
        ctx.fillStyle = nodeGrad1
        ctx.globalAlpha = alpha1 + 0.1
        if (z1 > 0) { ctx.shadowBlur = 8; ctx.shadowColor = '#00ffcc'; }
        ctx.fill()
        ctx.restore()

        // 3. 绘制链 B 上的水珠形节点
        const r2 = 4.5 + z2 * 1.8
        ctx.save()
        ctx.beginPath()
        ctx.arc(x2, y, r2, 0, Math.PI * 2)
        let nodeGrad2 = ctx.createRadialGradient(x2 - r2*0.2, y - r2*0.2, r2*0.1, x2, y, r2)
        nodeGrad2.addColorStop(0, 'rgba(255, 255, 255, 0.8)')
        nodeGrad2.addColorStop(0.4, '#0099ff')
        nodeGrad2.addColorStop(1, 'rgba(0, 153, 255, 0)')
        ctx.fillStyle = nodeGrad2
        ctx.globalAlpha = alpha2 + 0.1
        if (z2 > 0) { ctx.shadowBlur = 8; ctx.shadowColor = '#0099ff'; }
        ctx.fill()
        ctx.restore()
      }

      // ================= 特效三：右侧高科技 X 染色体（缓缓漂浮） =================
      const chromBaseX = w * 0.86
      const chromBaseY = h * 0.46
      
      // 计算缓缓漂浮的微调位移 (上下律动与微微摇摆)
      const driftX = Math.sin(this.globalTime * 0.4) * 8
      const driftY = Math.cos(this.globalTime * 0.5) * 16
      const chromX = chromBaseX + driftX
      const chromY = chromBaseY + driftY
      
      // 微小的慢速整体自旋
      const chromRotation = Math.sin(this.globalTime * 0.25) * 0.04

      ctx.save()
      ctx.translate(chromX, chromY)
      ctx.rotate(chromRotation)

      // 设定 X 染色体复合科幻渐变色
      let chromGrad = ctx.createLinearGradient(-30, -70, 30, 70)
      chromGrad.addColorStop(0, '#00ffcc')  // 顶部翠绿流光
      chromGrad.addColorStop(0.3, '#0072ff')// 中部冷蓝
      chromGrad.addColorStop(0.7, '#0052d4')
      chromGrad.addColorStop(1, '#00c6ff')  // 底部亮蓝

      // A. 绘制 X 染色体深色底层朦胧大光晕
      ctx.lineWidth = 26
      ctx.lineCap = 'round'
      ctx.strokeStyle = 'rgba(0, 114, 255, 0.12)'
      ctx.shadowBlur = 30
      ctx.shadowColor = '#0072ff'
      
      // 染色单体 1（左上到右下，中间向内弯曲缢缩）
      ctx.beginPath()
      ctx.moveTo(-28, -75)
      ctx.bezierCurveTo(-10, -35, -4, -10, -4, 0)
      ctx.bezierCurveTo(-4, 10, -10, 35, -28, 75)
      ctx.stroke()
      // 染色单体 2（右上到左下）
      ctx.beginPath()
      ctx.moveTo(28, -75)
      ctx.bezierCurveTo(10, -35, 4, -10, 4, 0)
      ctx.bezierCurveTo(4, 10, 10, 35, 28, 75)
      ctx.stroke()

      // B. 绘制 X 染色体核心高亮生动的多层次质感形态
      ctx.shadowBlur = 12
      ctx.shadowColor = '#00ffcc'
      ctx.lineWidth = 14
      ctx.strokeStyle = chromGrad
      
      ctx.beginPath()
      ctx.moveTo(-28, -75)
      ctx.bezierCurveTo(-10, -35, -3, -8, -3, 0)
      ctx.bezierCurveTo(-3, 8, -10, 35, -28, 75)
      ctx.stroke()

      ctx.beginPath()
      ctx.moveTo(28, -75)
      ctx.bezierCurveTo(10, -35, 3, -8, 3, 0)
      ctx.bezierCurveTo(3, 8, 10, 35, 28, 75)
      ctx.stroke()

      // C. 注入核心丝状高能染色质微细内核（原图内部的高亮白核线）
      ctx.shadowBlur = 4
      ctx.shadowColor = '#ffffff'
      ctx.lineWidth = 2.5
      ctx.strokeStyle = 'rgba(255, 255, 255, 0.85)'
      
      ctx.beginPath()
      ctx.moveTo(-28, -75)
      ctx.bezierCurveTo(-10, -35, -3, -8, -3, 0)
      ctx.bezierCurveTo(-3, 8, -10, 35, -28, 75)
      ctx.stroke()

      ctx.beginPath()
      ctx.moveTo(28, -75)
      ctx.bezierCurveTo(10, -35, 3, -8, 3, 0)
      ctx.bezierCurveTo(3, 8, 10, 35, 28, 75)
      ctx.stroke()
      ctx.restore()

      // ================= 特效四：专属环绕在 X 染色体周围的水珠气泡群 =================
      this.chromDroplets.forEach(dp => {
        dp.angle += dp.rotSpeed
        // 结合染色体的缓缓漂浮位移，使其完美的“粘附环绕”在 X 染色体周围
        const cDropX = chromX + Math.cos(dp.angle) * dp.distance
        const cDropY = chromY + Math.sin(dp.angle) * dp.distance * 0.85 // 略微压扁轨道

        const pulseAlpha = dp.alpha + Math.sin(this.globalTime * 2 + dp.distance) * 0.12

        ctx.save()
        ctx.beginPath()
        ctx.arc(cDropX, cDropY, dp.radius, 0, Math.PI * 2)
        
        let cDropGrad = ctx.createRadialGradient(cDropX - dp.radius*0.2, cDropY - dp.radius*0.2, dp.radius*0.1, cDropX, cDropY, dp.radius)
        cDropGrad.addColorStop(0, 'rgba(255, 255, 255, 0.9)')
        cDropGrad.addColorStop(0.4, dp.color)
        cDropGrad.addColorStop(1, 'rgba(0,0,0,0)')

        ctx.fillStyle = cDropGrad
        ctx.globalAlpha = Math.max(0.1, Math.min(pulseAlpha, 0.85))
        ctx.shadowBlur = 5
        ctx.shadowColor = dp.color
        ctx.fill()
        ctx.restore()
      })

      this.animId = requestAnimationFrame(this.drawLoop)
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
  background-color: #020a1c;
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
  background: rgba(4, 18, 48, 0.68);
  border: 1px solid rgba(0, 153, 255, 0.38);
  backdrop-filter: blur(14px);
  -webkit-backdrop-filter: blur(14px);
  box-shadow: 0 15px 45px rgba(0, 6, 20, 0.85), 
              inset 0 0 25px rgba(0, 170, 255, 0.18);
  
  /* 深度重写输入框样式 */
  ::v-deep .el-input__inner {
    height: 42px;
    line-height: 42px;
    background-color: rgba(4, 24, 60, 0.75) !important;
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