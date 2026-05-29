<template>
  <div class="cyber-monitor-container">
    <div class="cyber-header">
      <div class="header-line left"></div>
      <h2 class="header-title">系统核心资源监控中心</h2>
      <div class="header-line right"></div>
      <span class="header-sub">SYSTEM CORE RESOURCE MONITOR CENTER</span>
    </div>

    <el-row :gutter="16" class="cyber-grid">
      <el-col :span="12" class="cyber-col">
        <div class="cyber-panel-card">
          <div class="panel-header">
            <span class="panel-icon"><i class="el-icon-cpu"></i></span>
            <span class="panel-text">CPU 算力引擎状态</span>
          </div>
          
          <div class="panel-body flex-row" v-if="server.cpu">
            <div class="dashboard-ring-box">
              <svg class="ring-svg" viewBox="0 0 100 100">
                <circle class="ring-bg" cx="50" cy="50" r="42" />
                <circle class="ring-active cyan" cx="50" cy="50" r="42" 
                        :stroke-dasharray="263.8" 
                        :stroke-dashoffset="263.8 - (263.8 * server.cpu.used) / 100" />
              </svg>
              <div class="ring-center-text">
                <span class="percentage cyan-text">{{ server.cpu.used }}%</span>
                <span class="label">总使用率</span>
              </div>
            </div>
            <div class="dashboard-info-metrics">
              <div class="metric-row">
                <span class="m-label">物理核心总数</span>
                <span class="m-val highlight">{{ server.cpu.cpuNum }} 核</span>
              </div>
              <div class="metric-row">
                <span class="m-label">用户空间占用</span>
                <span class="m-val">{{ server.cpu.used }}%</span>
              </div>
              <div class="metric-row">
                <span class="m-label">系统内核占用</span>
                <span class="m-val">{{ server.cpu.sys }}%</span>
              </div>
              <div class="metric-row">
                <span class="m-label">当前空闲余量</span>
                <span class="m-val green-text">{{ server.cpu.free }}%</span>
              </div>
            </div>
          </div>
        </div>
      </el-col>

      <el-col :span="12" class="cyber-col">
        <div class="cyber-panel-card">
          <div class="panel-header">
            <span class="panel-icon"><i class="el-icon-tickets"></i></span>
            <span class="panel-text">内存双驱资源栈</span>
          </div>
          
          <div class="panel-body flex-row" v-if="server.mem && server.jvm">
            <div class="dashboard-ring-box split">
              <svg class="ring-svg" viewBox="0 0 100 100">
                <circle class="ring-bg" cx="50" cy="50" r="42" />
                <circle class="ring-active blue" cx="50" cy="50" r="42" 
                        :stroke-dasharray="263.8" 
                        :stroke-dashoffset="263.8 - (263.8 * server.mem.usage) / 100" />
              </svg>
              <div class="ring-center-text">
                <span class="percentage blue-text">{{ server.mem.usage }}%</span>
                <span class="label">系统内存</span>
              </div>
              <div class="ring-bottom-data">已用: {{ server.mem.used }}G / 总: {{ server.mem.total }}G</div>
            </div>

            <div class="dashboard-ring-box split">
              <svg class="ring-svg" viewBox="0 0 100 100">
                <circle class="ring-bg" cx="50" cy="50" r="42" />
                <circle class="ring-active purple" cx="50" cy="50" r="42" 
                        :stroke-dasharray="263.8" 
                        :stroke-dashoffset="263.8 - (263.8 * server.jvm.usage) / 100" />
              </svg>
              <div class="ring-center-text">
                <span class="percentage purple-text">{{ server.jvm.usage }}%</span>
                <span class="label">JVM虚拟机</span>
              </div>
              <div class="ring-bottom-data">已用: {{ server.jvm.used }}M / 总: {{ server.jvm.total }}M</div>
            </div>
          </div>
        </div>
      </el-col>

      <el-col :span="12" class="cyber-col">
        <div class="cyber-panel-card">
          <div class="panel-header">
            <span class="panel-icon"><i class="el-icon-monitor"></i></span>
            <span class="panel-text">硬件宿主与环境网关</span>
          </div>
          <div class="panel-body" v-if="server.sys">
            <div class="cyber-data-table">
              <div class="t-row">
                <div class="t-cell label">服务器主机名</div>
                <div class="t-cell val text-glow-blue">{{ server.sys.computerName }}</div>
                <div class="t-cell label">架构形态</div>
                <div class="t-cell val">{{ server.sys.osArch }}</div>
              </div>
              <div class="t-row">
                <div class="t-cell label">操作系统宿主</div>
                <div class="t-cell val">{{ server.sys.osName }}</div>
                <div class="t-cell label">内部网关IP</div>
                <div class="t-cell val text-glow-cyan">{{ server.sys.computerIp }}</div>
              </div>
              <div class="t-row单列">
                <span class="inner-label">工程部署根路径:</span>
                <span class="inner-val-path">{{ server.sys.userDir }}</span>
              </div>
            </div>
          </div>
        </div>
      </el-col>

      <el-col :span="12" class="cyber-col">
        <div class="cyber-panel-card">
          <div class="panel-header">
            <span class="panel-icon"><i class="el-icon-coffee-cup"></i></span>
            <span class="panel-text">Java 容器编译运行时环境</span>
          </div>
          <div class="panel-body" v-if="server.jvm">
            <div class="cyber-data-table">
              <div class="t-row">
                <div class="t-cell label">JVM引擎名称</div>
                <div class="t-cell val truncate" :title="server.jvm.name">{{ server.jvm.name }}</div>
                <div class="t-cell label">内核版本</div>
                <div class="t-cell val">{{ server.jvm.version }}</div>
              </div>
              <div class="t-row">
                <div class="t-cell label">容器启动时间</div>
                <div class="t-cell val font-mini">{{ server.jvm.startTime }}</div>
                <div class="t-cell label">节点运行时长</div>
                <div class="t-cell val text-glow-cyan">{{ server.jvm.runTime }}</div>
              </div>
              <div class="t-row单列">
                <span class="inner-label">SDK安装根路径:</span>
                <span class="inner-val-path" :title="server.jvm.home">{{ server.jvm.home }}</span>
              </div>
            </div>
          </div>
        </div>
      </el-col>

      <el-col :span="24" class="cyber-col">
        <div class="cyber-panel-card full-width">
          <div class="panel-header">
            <span class="panel-icon"><i class="el-icon-receiving"></i></span>
            <span class="panel-text">外存空间与磁盘列阵实时负载</span>
          </div>
          <div class="panel-body" v-if="server.sysFiles">
            <div class="disk-progress-container">
              <div class="disk-item-row" v-for="(sysFile, index) in server.sysFiles" :key="index">
                <div class="disk-meta">
                  <span class="disk-path"><i class="el-icon-folder"></i> 挂载点: <strong class="cyan-text">{{ sysFile.dirName }}</strong></span>
                  <span class="disk-type">系统: {{ sysFile.sysTypeName }} [{{ sysFile.typeName }}]</span>
                  <span class="disk-bytes">已用 {{ sysFile.used }} / 总量 {{ sysFile.total }}</span>
                </div>
                <div class="disk-bar-wrapper">
                  <div class="disk-bar-track">
                    <div class="disk-bar-fill" 
                         :class="{'danger': sysFile.usage > 85}"
                         :style="{ width: sysFile.usage + '%' }">
                      <div class="scan-light"></div>
                    </div>
                  </div>
                  <span class="disk-perc-text" :class="{'danger-text': sysFile.usage > 85}">{{ sysFile.usage }}%</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import { getServer } from "@/api/monitor/server"

export default {
  name: "Server",
  data() {
    return {
      server: []
    }
  },
  created() {
    this.getList()
    this.openLoading()
  },
  methods: {
    /** 查询服务器信息 */
    getList() {
      getServer().then(response => {
        this.server = response.data
        this.$modal.closeLoading()
      })
    },
    // 打开加载层
    openLoading() {
      this.$modal.loading("正在智算调配数字面板，同步集群数据...")
    }
  }
}
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
/* 全局暗色科技主题底色 */
.cyber-monitor-container {
  background-color: #02091a;
  min-height: 100vh;
  padding: 24px;
  color: #bfe7ff;
  font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;
  box-sizing: border-box;
}

/* 顶部一体化大标题 */
.cyber-header {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  margin-bottom: 25px;
  padding-top: 10px;

  .header-title {
    margin: 0;
    font-size: 24px;
    font-weight: 700;
    color: #ffffff;
    letter-spacing: 3px;
    text-shadow: 0 0 15px rgba(0, 153, 255, 0.85);
  }

  .header-sub {
    font-size: 10px;
    color: #0072ff;
    letter-spacing: 1.5px;
    margin-top: 6px;
    font-weight: 600;
  }

  .header-line {
    position: absolute;
    top: 50%;
    width: 30%;
    height: 1px;
    background: linear-gradient(90deg, rgba(0,255,204,0), rgba(0,114,255,0.6), rgba(0,255,204,0));
    
    &.left { left: 2%; }
    &.right { right: 2%; }
  }
}

.cyber-grid {
  margin-bottom: -16px;
  align-items: stretch !important;
}

.cyber-col {
  margin-bottom: 16px;
  display: flex;
  flex-direction: column;
}

/* 媲美参考图的赛博磨砂玻璃卡片 */
.cyber-panel-card {
  position: relative;
  flex: 1 1 auto;
  background: rgba(4, 22, 54, 0.65);
  border: 1px solid rgba(0, 153, 255, 0.32);
  border-radius: 4px;
  box-shadow: 0 8px 25px rgba(0, 5, 15, 0.55),
              inset 0 0 15px rgba(0, 114, 255, 0.15);
  padding: 16px;
  min-height: 190px;
  box-sizing: border-box;
  overflow: hidden;

  /* 数字化尖角切角装饰线 - 还原真实大屏质感 */
  &::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    width: 10px;
    height: 10px;
    border-top: 2px solid #00ffcc;
    border-left: 2px solid #00ffcc;
  }
  &::after {
    content: '';
    position: absolute;
    bottom: 0;
    right: 0;
    width: 10px;
    height: 10px;
    border-bottom: 2px solid #00ffcc;
    border-right: 2px solid #00ffcc;
  }
  
  &.full-width {
    min-height: auto;
  }
}

/* 面板头部 */
.panel-header {
  display: flex;
  align-items: center;
  border-bottom: 1px solid rgba(0, 114, 255, 0.25);
  padding-bottom: 10px;
  margin-bottom: 14px;

  .panel-icon {
    font-size: 18px;
    color: #00ffcc;
    margin-right: 8px;
    filter: drop-shadow(0 0 4px rgba(0, 255, 204, 0.6));
  }

  .panel-text {
    font-size: 15px;
    font-weight: 600;
    color: #ffffff;
    letter-spacing: 0.5px;
  }
}

/* 布局辅助 */
.flex-row {
  display: flex;
  align-items: center;
  justify-content: space-around;
  height: calc(100% - 40px);
}

/* SVG 动态核心仪表盘圆环 */
.dashboard-ring-box {
  position: relative;
  width: 110px;
  height: 110px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;

  &.split {
    width: 45%;
    height: auto;
    margin-top: -5px;
  }

  .ring-svg {
    transform: rotate(-90deg);
    width: 100%;
    height: 100%;
  }

  circle {
    fill: none;
    stroke-width: 7;
  }

  .ring-bg {
    stroke: rgba(0, 114, 255, 0.12);
  }

  .ring-active {
    stroke-linecap: round;
    transition: stroke-dashoffset 0.8s ease-in-out;
    
    &.cyan { stroke: #00ffcc; filter: drop-shadow(0 0 5px #00ffcc); }
    &.blue { stroke: #0072ff; filter: drop-shadow(0 0 5px #0072ff); }
    &.purple { stroke: #a052ff; filter: drop-shadow(0 0 5px #a052ff); }
  }

  .ring-center-text {
    position: absolute;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;

    .percentage {
      font-size: 18px;
      font-weight: bold;
      font-family: 'Courier New', Courier, monospace;
    }
    .label {
      font-size: 9px;
      color: #608eb3;
      margin-top: 2px;
    }
  }

  .ring-bottom-data {
    font-size: 10px;
    color: #5580a6;
    margin-top: 8px;
    white-space: nowrap;
    text-align: center;
    background: rgba(0, 114, 255, 0.08);
    padding: 2px 6px;
    border-radius: 10px;
  }
}

/* 右侧多量指标样式 */
.dashboard-info-metrics {
  width: 55%;
  display: flex;
  flex-direction: column;
  gap: 8px;

  .metric-row {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 4px 8px;
    background: rgba(4, 32, 77, 0.4);
    border-left: 3px solid #0072ff;
    border-radius: 0 4px 4px 0;
    font-size: 12px;

    .m-label { color: #8cb6d9; }
    .m-val { font-weight: 600; color: #ffffff; }
  }
}

/* 信息卡数据扁平表格模式 */
.cyber-data-table {
  display: flex;
  flex-direction: column;
  gap: 6px;
  font-size: 12px;

  .t-row {
    display: flex;
    background: rgba(4, 32, 77, 0.3);
    border-bottom: 1px solid rgba(0, 114, 255, 0.12);
    
    .t-cell {
      padding: 8px;
      
      &.label {
        width: 25%;
        color: #729cb3;
        background: rgba(0, 114, 255, 0.08);
        font-weight: 500;
      }
      &.val {
        width: 25%;
        color: #ffffff;
        font-weight: 600;
      }
    }
  }

  .t-row单列 {
    display: flex;
    align-items: center;
    padding: 8px;
    background: rgba(4, 32, 77, 0.2);
    border-radius: 4px;
    margin-top: 4px;

    .inner-label {
      color: #729cb3;
      margin-right: 8px;
      white-space: nowrap;
    }
    .inner-val-path {
      color: #00ffcc;
      font-family: monospace;
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
    }
  }
}

/* 存储矩阵外层进度条容器 */
.disk-progress-container {
  display: flex;
  flex-direction: column;
  gap: 14px;
  padding: 4px 0;
}

.disk-item-row {
  background: rgba(4, 32, 77, 0.3);
  padding: 12px 16px;
  border-radius: 4px;
  border: 1px solid rgba(0, 114, 255, 0.15);
}

.disk-meta {
  display: flex;
  justify-content: space-between;
  font-size: 12px;
  margin-bottom: 8px;
  color: #8cb6d9;

  .disk-path strong { font-weight: 600; }
  .disk-bytes { font-family: monospace; color: #ffffff; }
}

.disk-bar-wrapper {
  display: flex;
  align-items: center;
  gap: 12px;

  .disk-bar-track {
    flex-grow: 1;
    height: 10px;
    background: #031533;
    border-radius: 5px;
    overflow: hidden;
    position: relative;
    border: 1px solid rgba(0, 114, 255, 0.2);
  }

  .disk-bar-fill {
    height: 100%;
    border-radius: 5px;
    background: linear-gradient(90deg, #0052d4, #0072ff, #00ffcc);
    box-shadow: 0 0 8px rgba(0, 255, 204, 0.5);
    transition: width 0.6s ease;
    position: relative;
    
    &.danger {
      background: linear-gradient(90deg, #b21f1f, #f12711);
      box-shadow: 0 0 8px rgba(241, 39, 17, 0.6);
    }
  }

  /* 进度条内部高动态扫描流光 */
  .scan-light {
    position: absolute;
    top: 0;
    left: 0;
    height: 100%;
    width: 30%;
    background: linear-gradient(90deg, rgba(255,255,255,0), rgba(255,255,255,0.4), rgba(255,255,255,0));
    animation: cyber-scan 2.5s infinite linear;
  }

  .disk-perc-text {
    font-size: 13px;
    font-weight: bold;
    font-family: monospace;
    width: 42px;
    text-align: right;
    color: #00ffcc;
    
    &.danger-text { color: #ff4949; }
  }
}

/* 高光发光滤镜类 */
.cyan-text { color: #00ffcc !important; }
.blue-text { color: #0072ff !important; }
.purple-text { color: #a052ff !important; }
.green-text { color: #00ff87 !important; }
.text-glow-cyan { text-shadow: 0 0 6px rgba(0, 255, 204, 0.6); color: #00ffcc !important; }
.text-glow-blue { text-shadow: 0 0 6px rgba(0, 114, 255, 0.6); color: #4fa3ff !important; }
.truncate { overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
.font-mini { font-size: 11px !important; }

@keyframes cyber-scan {
  0% { transform: translateX(-100%); }
  100% { transform: translateX(350%); }
}
</style>