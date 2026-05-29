<template>
  <div class="app-container tech-dashboard">
    <el-row :gutter="15">
      <el-col :span="24" class="card-box">
        <div class="tech-card animate-fade-in">
          <div class="tech-card-header">
            <span class="tech-title"><i class="el-icon-monitor"></i> REDIS 运行状态核心监控</span>
            <div class="tech-decoration"></div>
          </div>
          <div class="tech-table-wrapper">
            <table cellspacing="0" class="tech-table">
              <tbody>
                <tr>
                  <td class="tech-label">Redis版本</td>
                  <td class="tech-value"><span v-if="cache.info" class="highlight-cyan">{{ cache.info.redis_version }}</span></td>
                  <td class="tech-label">运行模式</td>
                  <td class="tech-value"><span v-if="cache.info">{{ cache.info.redis_mode == "standalone" ? "单机" : "集群" }}</span></td>
                  <td class="tech-label">端口</td>
                  <td class="tech-value"><span v-if="cache.info" class="highlight-blue">{{ cache.info.tcp_port }}</span></td>
                  <td class="tech-label">客户端数</td>
                  <td class="tech-value"><span v-if="cache.info" class="highlight-green">{{ cache.info.connected_clients }}</span></td>
                </tr>
                <tr>
                  <td class="tech-label">运行时间(天)</td>
                  <td class="tech-value"><span v-if="cache.info">{{ cache.info.uptime_in_days }}</span></td>
                  <td class="tech-label">使用内存</td>
                  <td class="tech-value"><span v-if="cache.info" class="highlight-orange">{{ cache.info.used_memory_human }}</span></td>
                  <td class="tech-label">使用CPU</td>
                  <td class="tech-value"><span v-if="cache.info">{{ parseFloat(cache.info.used_cpu_user_children).toFixed(2) }}%</span></td>
                  <td class="tech-label">内存配置</td>
                  <td class="tech-value"><span v-if="cache.info">{{ cache.info.maxmemory_human }}</span></td>
                </tr>
                <tr>
                  <td class="tech-label">AOF是否开启</td>
                  <td class="tech-value">
                    <span v-if="cache.info" :class="cache.info.aof_enabled == '0' ? 'text-muted' : 'highlight-green'">
                      {{ cache.info.aof_enabled == "0" ? "否" : "是" }}
                    </span>
                  </td>
                  <td class="tech-label">RDB是否成功</td>
                  <td class="tech-value">
                    <span v-if="cache.info" :class="cache.info.rdb_last_bgsave_status === 'ok' ? 'highlight-green' : 'highlight-red'">
                      {{ cache.info.rdb_last_bgsave_status }}
                    </span>
                  </td>
                  <td class="tech-label">Key数量</td>
                  <td class="tech-value"><span v-if="cache.dbSize" class="highlight-cyan">{{ cache.dbSize }}</span></td>
                  <td class="tech-label">网络入口/出口</td>
                  <td class="tech-value"><span v-if="cache.info" class="tech-net-speed">{{ cache.info.instantaneous_input_kbps }} / {{cache.info.instantaneous_output_kbps}} <small>kbps</small></span></td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </el-col>

      <el-col :span="12" class="card-box">
        <div class="tech-card animate-fade-in delay-1">
          <div class="tech-card-header">
            <span class="tech-title"><i class="el-icon-pie-chart"></i> 命令执行吞吐统计</span>
            <div class="tech-decoration"></div>
          </div>
          <div class="chart-wrapper">
            <div ref="commandstats" style="height: 400px" />
          </div>
        </div>
      </el-col>

      <el-col :span="12" class="card-box">
        <div class="tech-card animate-fade-in delay-2">
          <div class="tech-card-header">
            <span class="tech-title"><i class="el-icon-odometer"></i> 实时内存盘指针</span>
            <div class="tech-decoration"></div>
          </div>
          <div class="chart-wrapper">
            <div ref="usedmemory" style="height: 400px" />
          </div>
        </div>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import { getCache } from "@/api/monitor/cache"
import * as echarts from "echarts"

export default {
  name: "Cache",
  data() {
    return {
      commandstats: null,
      usedmemory: null,
      cache: []
    }
  },
  created() {
    this.getList()
    this.openLoading()
  },
  beforeDestroy() {
    // 组件销毁时移除监听，防止内存泄漏
    window.removeEventListener("resize", this.handleResize)
  },
  methods: {
    getList() {
      getCache().then((response) => {
        this.cache = response.data
        this.$modal.closeLoading()

        // 1. 命令统计饼图（玫瑰图优化为科技感配色）
        this.commandstats = echarts.init(this.$refs.commandstats)
        this.commandstats.setOption({
          backgroundColor: 'transparent',
          tooltip: {
            trigger: "item",
            formatter: "{a} <br/>{b} : {c} ({d}%)",
            backgroundColor: 'rgba(6, 18, 44, 0.85)',
            borderColor: '#00f6ff',
            textStyle: { color: '#fff' }
          },
          legend: {
            orient: 'vertical',
            right: '5%',
            top: 'center',
            textStyle: { color: '#a2c0d8' }
          },
          color: ['#00f6ff', '#00ff87', '#fecb00', '#ff4b7d', '#9d56ff', '#3b77ff'],
          series: [
            {
              name: "命令占比",
              type: "pie",
              roseType: "area",
              radius: ['20%', '75%'],
              center: ["40%", "50%"],
              data: response.data.commandStats,
              itemStyle: {
                borderRadius: 4,
                borderColor: '#061a35',
                borderWidth: 2
              },
              label: {
                color: '#a2c0d8'
              },
              animationEasing: "cubicInOut",
              animationDuration: 1200,
            }
          ]
        })

        // 2. 内存仪表盘（优化为流线型科技仪表盘）
        const usedMemValue = parseFloat(this.cache.info.used_memory_human) || 0
        // 动态根据其单位（M或G）来粗略预估最大值，或者固定为max
        const isG = this.cache.info.used_memory_human.includes('G')
        const maxRange = isG ? 16 : 1024 

        this.usedmemory = echarts.init(this.$refs.usedmemory)
        this.usedmemory.setOption({
          backgroundColor: 'transparent',
          tooltip: {
            backgroundColor: 'rgba(6, 18, 44, 0.85)',
            borderColor: '#00ff87',
            textStyle: { color: '#fff' }
          },
          series: [
            {
              name: "内存指标",
              type: "gauge",
              min: 0,
              max: maxRange,
              radius: '85%',
              center: ["50%", "55%"],
              progress: {
                show: true,
                width: 12,
                itemStyle: {
                  color: new echarts.graphic.LinearGradient(0, 0, 1, 0, [
                    { offset: 0, color: '#00f6ff' },
                    { offset: 1, color: '#00ff87' }
                  ])
                }
              },
              axisLine: {
                lineStyle: { width: 12, color: [[1, '#112b4d']] }
              },
              axisTick: { show: false },
              splitLine: { length: 12, lineStyle: { color: '#00f6ff', width: 2 } },
              axisLabel: { color: '#a2c0d8', distance: 20 },
              pointer: {
                itemStyle: { color: '#00ff87' }
              },
              title: {
                offsetCenter: [0, '70%'],
                style: { color: '#a2c0d8', fontSize: 14 }
              },
              detail: {
                valueAnimation: true,
                formatter: () => this.cache.info.used_memory_human,
                color: '#ffffff',
                fontSize: 24,
                fontWeight: 'bold',
                offsetCenter: [0, '40%']
              },
              data: [
                {
                  value: usedMemValue,
                  name: `当前消耗 (Max: ${this.cache.info.maxmemory_human || '未限制'})`,
                }
              ]
            }
          ]
        })

        window.addEventListener("resize", this.handleResize)
      })
    },
    handleResize() {
      this.commandstats && this.commandstats.resize()
      this.usedmemory && this.usedmemory.resize()
    },
    openLoading() {
      this.$modal.loading("正在接入数字孪生监控系统，请稍候...")
    }
  }
}
</script>

<style scoped lang="scss">
/* 科技风核心样式面板 */
.tech-dashboard {
  background-color: #040d1a; /* 深邃暗黑底色 */
  min-height: calc(100vh - 84px);
  padding: 20px;
  color: #d1f0ff;
  font-family: 'Helvetica Neue', Helvetica, 'PingFang SC', sans-serif;

  // 自定义科技卡片替代 el-card
  .tech-card {
    background: linear-gradient(135deg, rgba(6, 30, 66, 0.8) 0%, rgba(4, 17, 41, 0.9) 100%);
    border: 1px solid #0059b3;
    box-shadow: 0 0 15px rgba(0, 110, 255, 0.2), inset 0 0 20px rgba(0, 110, 255, 0.1);
    border-radius: 4px;
    margin-bottom: 20px;
    padding: 18px;
    position: relative;
    overflow: hidden;

    &::before {
      content: "";
      position: absolute;
      top: 0; left: 0; width: 10px; height: 10px;
      border-top: 2px solid #00f6ff;
      border-left: 2px solid #00f6ff;
    }
    &::after {
      content: "";
      position: absolute;
      bottom: 0; right: 0; width: 10px; height: 10px;
      border-bottom: 2px solid #00f6ff;
      border-right: 2px solid #00f6ff;
    }
  }

  .tech-card-header {
    display: flex;
    flex-direction: column;
    margin-bottom: 15px;
    border-bottom: 1px solid rgba(0, 89, 179, 0.4);
    padding-bottom: 10px;

    .tech-title {
      font-size: 16px;
      font-weight: bold;
      color: #00f6ff;
      text-shadow: 0 0 10px rgba(0, 246, 255, 0.5);
      i { margin-right: 6px; }
    }
  }

  /* 数字化表格定制 */
  .tech-table-wrapper {
    overflow-x: auto;
  }
  .tech-table {
    width: 100%;
    border-collapse: collapse;
    background: rgba(4, 21, 48, 0.5);

    td {
      padding: 12px 14px;
      border: 1px solid rgba(0, 89, 179, 0.3);
      font-size: 14px;
    }

    .tech-label {
      background: rgba(0, 89, 179, 0.15);
      color: #a2c0d8;
      width: 12%;
      text-align: right;
      font-weight: 500;
    }

    .tech-value {
      background: rgba(6, 26, 53, 0.3);
      color: #ffffff;
      width: 13%;
    }
  }

  /* 霓虹高亮色系 */
  .highlight-cyan { color: #00f6ff; text-shadow: 0 0 8px rgba(0,246,255,0.4); font-weight: bold;}
  .highlight-blue { color: #3b77ff; }
  .highlight-green { color: #00ff87; text-shadow: 0 0 8px rgba(0,255,135,0.4); }
  .highlight-orange { color: #fecb00; text-shadow: 0 0 8px rgba(254,203,0,0.4); font-weight: bold;}
  .highlight-red { color: #ff4b7d; }
  .text-muted { color: #5a7b9a; }
  
  .tech-net-speed {
    font-size: 13px;
    small { color: #a2c0d8; margin-left: 2px;}
  }

  .chart-wrapper {
    background: rgba(2, 12, 31, 0.4);
    border-radius: 4px;
    border: 1px dashed rgba(0, 89, 179, 0.3);
  }

  /* 动效 */
  .animate-fade-in {
    animation: fadeIn 0.6s ease-out forwards;
  }
  .delay-1 { animation-delay: 0.15s; opacity: 0; }
  .delay-2 { animation-delay: 0.3s; opacity: 0; }
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}
</style>