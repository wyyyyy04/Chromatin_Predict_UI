<template>
  <div class="app-container home bio-platform">
    <canvas id="dnaComplexBackground" class="dna-bg-complex"></canvas>

    <el-row :gutter="20" class="mb20 flex-relative">
      <el-col :span="24">
        <div class="hero-section animated-hero">
          <div class="hero-glow-layer"></div>
          <h2>SDP-ChroNet 染色质相互作用空间预测平台</h2>
          <p>
            本平台深度集成 <strong>Shared Residual Dilated Units (SRDU)</strong> 共享残差扩张卷积架构与 <strong>RC-SHINE</strong> 非负非对称集成训练策略。
            专门针对高复杂性基因组三维空间构象设计，支持 K562、GM12878 等多细胞系下对 WT1、KLF9、KLF6 等转录因子关键 Motif 匹配位点的跨细胞系鲁棒性诊断与三维交互概率预测。
          </p>
        </div>
      </el-col>
    </el-row>

    <el-row :gutter="20" class="mb20 flex-relative">
      <el-col :xs="24" :sm="24" :md="14" :lg="14">
        
        <el-row :gutter="15" class="mb20">
          <el-col :span="8" v-for="(item, index) in modules" :key="index">
            <el-card 
              shadow="hover" 
              :class="['module-card', item.action, getModuleStatusClass(item.action)]" 
              @click.native="handleModuleClick(item.action)"
            >
              <div class="card-status-dot" :class="{ 'active': checkStepActive(item.action) }"></div>
              <i :class="item.icon"></i>
              <div class="module-name">{{ item.name }}</div>
            </el-card>
          </el-col>
        </el-row>

        <el-card shadow="never" class="upload-card">
          <div slot="header" class="clearfix">
            <span class="card-title"><i class="el-icon-upload"></i> 染色质数据中台上传</span>
            <div class="data-type-selector">
              <span class="label-text">数据类型：</span>
              <el-radio-group v-model="uploadParam.dataType" size="mini">
                <el-radio-button label="genome">基因组数据 (.fa)</el-radio-button>
                <el-radio-button label="sequence">序列数据 (.fasta)</el-radio-button>
              </el-radio-group>
            </div>
          </div>
          <el-upload
            class="upload-demo"
            drag
            action="#"
            :auto-upload="false"
            :on-change="handleFileChange"
            :limit="1"
            accept=".fasta,.fa,.txt"
            :file-list="fileList"
          >
            <i class="el-icon-upload"></i>
            <div class="el-upload__text" v-if="!isUploaded">将 FASTA 文件拖到此处，或 <em>点击上传</em></div>
            <div class="el-upload__text uploaded-text" v-else><i class="el-icon-circle-check"></i> 序列加载成功: {{ fileName }}</div>
            <div class="el-upload__tip" slot="tip">系统将自动基于 SSRR (SHAP-Shift Risk Ranking) 算法体系执行预对齐校准</div>
          </el-upload>
        </el-card>
      </el-col>

      <el-col :xs="24" :sm="24" :md="10" :lg="10">
        <el-card shadow="never" class="table-card">
          <div slot="header" class="clearfix">
            <span class="card-title"><i class="el-icon-s-data"></i> 染色质空间互作分析队列</span>
            <el-tag size="mini" type="success" style="float: right">并发正常</el-tag>
          </div>
          <el-table :data="recentResults" style="width: 100%" size="small" border>
            <el-table-column prop="jobId" label="任务 ID" width="105"></el-table-column>
            <el-table-column prop="cellLine" label="细胞系" width="75"></el-table-column>
            <el-table-column prop="motif" label="关键 Motif" width="90"></el-table-column>
            <el-table-column prop="status" label="流程状态">
              <template slot-scope="scope">
                <span :class="'status-text-' + scope.row.statusCode">{{ scope.row.status }}</span>
              </template>
            </el-table-column>
          </el-table>
        </el-card>
      </el-col>
    </el-row>

    <el-collapse-transition>
      <el-row :gutter="20" v-show="showAnalysis" class="mb20 flex-relative">
        <el-col :xs="24" :sm="24" :md="12" :lg="12">
          <el-card shadow="never" class="chart-card">
            <div slot="header" class="clearfix">
              <span class="card-title"><i class="el-icon-data-line"></i> 空间互作预测评估 (ROC 曲线)</span>
              <el-tag size="mini" type="primary" style="float: right; font-family: monospace;">AUC: {{ generatedAuc }}</el-tag>
            </div>
            <div id="rocChart" style="height: 290px; width: 100%;"></div>
          </el-card>
        </el-col>
        <el-col :xs="24" :sm="24" :md="12" :lg="12">
          <el-card shadow="never" class="chart-card">
            <div slot="header" class="clearfix">
              <span class="card-title"><i class="el-icon-pie-chart"></i> 实时交叉验证性能指标 (Accuracy)</span>
            </div>
            <div id="accChart" style="height: 290px; width: 100%;"></div>
          </el-card>
        </el-col>
      </el-row>
    </el-collapse-transition>

    <div class="gene-loading-overlay" v-if="predictLoading">
      <div class="loading-box">
        <div class="sequencer-animation">
          <div class="scanner-line"></div>
          <div class="base-strand-view">{{ currentBaseSequence }}</div>
        </div>
        <h3 class="sand-box-title"><i class="el-icon-loading"></i> SDP-ChroNet 空间网络解析中...</h3>
        <p class="sand-box-sub">深度级联扩张卷积正在提取长程染色质基序特征...</p>
        
        <div class="gene-progress-wrapper">
          <el-progress :percentage="progressPercent" :stroke-width="12" color="#00f2fe" :show-text="true"></el-progress>
          <div class="sequencing-metrics">
            <span>Scan window: 200bp</span>
            <span>Est remaining: {{ remainingSeconds }}s</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import * as echarts from 'echarts';

export default {
  name: "Index",
  data() {
    return {
      uploadParam: { dataType: 'sequence' },
      modules: [
        { name: '1. 数据上传', icon: 'el-icon-upload2', action: 'upload' },
        { name: '2. 序列预测', icon: 'el-icon-cpu', action: 'predict' },
        { name: '3. 结果分析', icon: 'el-icon-data-analysis', action: 'analyze' }
      ],
      recentResults: [
        { jobId: 'CHRO-8812X', cellLine: 'K562', motif: 'WT1', status: '✅ 已入库分析', statusCode: 'success' },
        { jobId: 'CHRO-8813Y', cellLine: 'K562', motif: 'KLF9', status: '✅ 已入库分析', statusCode: 'success' },
        { jobId: 'CHRO-8814Z', cellLine: 'K562', motif: 'KLF6', status: '⏳ 待提取', statusCode: 'wait' }
      ],
      
      // 状态控制守卫
      isUploaded: false,
      isPredicted: false,
      showAnalysis: false,
      fileName: '',
      fileList: [],

      // 加载条逻辑数据
      predictLoading: false,
      progressPercent: 0,
      remainingSeconds: 0,
      currentBaseSequence: 'A T C G G C T A A T C G',

      // 图表实例
      generatedAuc: '0.00',
      rocChartInstance: null,
      accChartInstance: null,
      canvasAnimationId: null
    };
  },
  mounted() {
    this.initComplexDnaCanvas();
    window.addEventListener('resize', this.handleResize);
  },
  beforeDestroy() {
    if (this.canvasAnimationId) cancelAnimationFrame(this.canvasAnimationId);
    window.removeEventListener('resize', this.handleResize);
  },
  methods: {
    // 拦截与高管权限判定样式
    getModuleStatusClass(action) {
      if (action === 'predict' && !this.isUploaded) return 'is-disabled';
      if (action === 'analyze' && !this.isPredicted) return 'is-disabled';
      return 'is-ready';
    },
    checkStepActive(action) {
      if (action === 'upload' && this.isUploaded) return true;
      if (action === 'predict' && this.isPredicted) return true;
      if (action === 'analyze' && this.showAnalysis) return true;
      return false;
    },
    // 文件选择逻辑
    handleFileChange(file) {
      const isFa = file.name.endsWith('.fasta') || file.name.endsWith('.fa') || file.name.endsWith('.txt');
      if (!isFa) {
        this.$message.error('系统仅解析标准格式的生物学 .fasta / .fa 文件！');
        this.fileList = [];
        return;
      }
      this.isUploaded = true;
      this.fileName = file.name;
      this.$message.success('FASTA 数据读取成功，核心计算集群已就绪，请执行第2步[序列预测]！');
    },

    // 核心工作流点击处理器
    handleModuleClick(action) {
      if (action === 'upload') {
        this.$message.info('请在下方的拖拽面板选择您的序列数据文件。');
      } 
      else if (action === 'predict') {
        if (!this.isUploaded) {
          this.$message({ message: '【工作流前置拦截】请先在下方上传 FASTA 基因数据序列！', type: 'warning' });
          return;
        }
        this.startGenePredicting();
      } 
      else if (action === 'analyze') {
        if (!this.isPredicted) {
          this.$message({ message: '【工作流前置拦截】当前模型未运行，请先点击并运行“序列预测”模块！', type: 'error' });
          return;
        }
        this.showAnalysis = true;
        this.$nextTick(() => { this.renderScientificCharts(); });
      }
    },

    // 🧬 仿真基因加载计时器（5-20秒随机数）
    startGenePredicting() {
      this.predictLoading = true;
      this.progressPercent = 0;
      
      const randomDuration = Math.floor(Math.random() * (20 - 5 + 1)) + 5; 
      this.remainingSeconds = randomDuration;

      const intervalStep = 100; 
      const totalTicks = (randomDuration * 1000) / intervalStep;
      let currentTick = 0;

      const baseTypes = ['A', 'T', 'C', 'G'];

      const timer = setInterval(() => {
        currentTick++;
        this.progressPercent = Math.floor((currentTick / totalTicks) * 100);
        this.remainingSeconds = Math.ceil(randomDuration - (currentTick * intervalStep / 1000));

        // 模拟高频碱基突变错配流
        let mockRow = [];
        for (let k = 0; k < 14; k++) {
          mockRow.push(baseTypes[Math.floor(Math.random() * 4)]);
        }
        this.currentBaseSequence = mockRow.join(' - ');

        if (currentTick >= totalTicks) {
          clearInterval(timer);
          this.predictLoading = false;
          this.isPredicted = true;
          
          // 预测成功后更新队列模拟数据状态
          this.recentResults.unshift({
            jobId: 'CHRO-NEW99',
            cellLine: 'K562',
            motif: this.uploadParam.dataType === 'genome' ? 'Global-Fa' : 'User-Fasta',
            status: '⚡ 预测就绪(可分析)',
            statusCode: 'ready'
          });

          this.$message({ message: 'SDP-ChroNet 空间风险诊断收敛完成！第3步[结果分析]已解锁。', type: 'success' });
        }
      }, intervalStep);
    },

    // 绘制包含动态特征的 ECharts 图表
    renderScientificCharts() {
      const randomAcc = (Math.random() * (96.8 - 91.2) + 91.2).toFixed(2);
      this.generatedAuc = (randomAcc / 100 + 0.015).toFixed(3);

      if (!this.rocChartInstance) this.rocChartInstance = echarts.init(document.getElementById('rocChart'));
      const p1 = (Math.random() * (0.68 - 0.58) + 0.58).toFixed(2);
      const p2 = (Math.random() * (0.89 - 0.82) + 0.82).toFixed(2);
      
      this.rocChartInstance.setOption({
        grid: { top: '12%', left: '10%', right: '8%', bottom: '15%' },
        tooltip: { trigger: 'axis' },
        xAxis: { type: 'value', min: 0, max: 1, name: 'False Positive Rate (FPR)', nameLocation: 'center', nameGap: 25 },
        yAxis: { type: 'value', min: 0, max: 1, name: 'True Positive Rate (TPR)' },
        series: [
          {
            name: 'ROC (SDP-ChroNet)', type: 'line', smooth: true, symbol: 'none', color: '#00f2fe', lineStyle: { width: 3 },
            areaStyle: {
              color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
                { offset: 0, color: 'rgba(0, 242, 254, 0.35)' },
                { offset: 1, color: 'rgba(0, 242, 254, 0.0)' }
              ])
            },
            data: [[0, 0], [0.04, p1], [0.14, p2], [0.35, 0.95], [0.65, 0.99], [1, 1]]
          },
          { name: 'Baseline', type: 'line', symbol: 'none', lineStyle: { type: 'dashed', color: '#94a3b8' }, data: [[0, 0], [1, 1]] }
        ]
      });

      if (!this.accChartInstance) this.accChartInstance = echarts.init(document.getElementById('accChart'));
      this.accChartInstance.setOption({
        series: [{
          type: 'gauge', startAngle: 225, endAngle: -45, min: 0, max: 100, radius: '95%', center: ['50%', '55%'],
          axisLine: { lineStyle: { width: 10, color: [[0.4, '#ef4444'], [0.75, '#f59e0b'], [1, '#10b981']] } },
          pointer: { width: 4, itemStyle: { color: '#1e293b' } },
          progress: { show: true, width: 10 },
          detail: { offsetCenter: [0, '70%'], valueAnimation: true, formatter: '{value}%', fontSize: 24, fontWeight: 'bold' },
          data: [{ value: randomAcc, name: 'Accuracy' }]
        }]
      });
    },

    // 🧬 升级版 Canvas：双级3D旋转双螺旋 + 游离碱基飘散动画
    initComplexDnaCanvas() {
      const canvas = document.getElementById('dnaComplexBackground');
      const ctx = canvas.getContext('2d');
      
      const resize = () => {
        canvas.width = canvas.parentElement.clientWidth;
        canvas.height = canvas.parentElement.clientHeight || window.innerHeight;
      };
      resize();

      let tick = 0;
      
      // 生成游离的碱基碎片背景数据
      const floatingBases = [];
      const baseChars = ['A', 'T', 'C', 'G'];
      for (let b = 0; b < 20; b++) {
        floatingBases.push({
          x: Math.random() * window.innerWidth,
          y: Math.random() * window.innerHeight,
          char: baseChars[Math.floor(Math.random() * 4)],
          speed: Math.random() * 0.4 + 0.2,
          alpha: Math.random() * 0.2 + 0.05,
          fontSize: Math.floor(Math.random() * 8 + 10)
        });
      }

      const drawComplex = () => {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        tick += 0.012; 

        // 1. 渲染游离自由碱基层
        floatingBases.forEach(base => {
          base.y -= base.speed;
          if (base.y < -20) base.y = canvas.height + 20;
          ctx.fillStyle = `rgba(15, 32, 67, ${base.alpha})`;
          ctx.font = `bold ${base.fontSize}px Courier New`;
          ctx.fillText(base.char, base.x, base.y);
        });

        // 2. 渲染高密度的双螺旋串联主链
        const cy = canvas.height * 0.52;
        const startX = 40;
        const endX = canvas.width - 40;
        const nodesCount = 35; 
        const gap = (endX - startX) / nodesCount;

        for (let i = 0; i <= nodesCount; i++) {
          const x = startX + i * gap;
          
          // 计算主链1与主链2的周期正弦值
          const rad1 = tick + i * 0.25;
          const rad2 = tick + i * 0.25 + Math.PI;

          const y1 = cy + Math.sin(rad1) * 75;
          const y2 = cy + Math.sin(rad2) * 75;

          // 借助余弦值来模拟深度 Z-Axis 导致的缩放与透明度变化
          const depth1 = (Math.cos(rad1) + 1) / 2; // 0 ~ 1
          const depth2 = (Math.cos(rad2) + 1) / 2; // 0 ~ 1

          // 绘制碱基配对氢键（基于景深控制深浅）
          ctx.beginPath();
          ctx.strokeStyle = `rgba(14, 165, 233, ${0.06 + depth1 * 0.1})`;
          ctx.lineWidth = 1 + depth1 * 1.5;
          ctx.moveTo(x, y1);
          ctx.lineTo(x, y2);
          ctx.stroke();

          // 主链节点1 - 数码冷蓝
          ctx.beginPath();
          ctx.fillStyle = `rgba(0, 242, 254, ${0.2 + depth1 * 0.6})`;
          ctx.arc(x, y1, 2 + depth1 * 4, 0, Math.PI * 2);
          ctx.fill();

          // 主链节点2 - 科技冷绿
          ctx.beginPath();
          ctx.fillStyle = `rgba(52, 211, 153, ${0.2 + depth2 * 0.6})`;
          ctx.arc(x, y2, 2 + depth2 * 4, 0, Math.PI * 2);
          ctx.fill();
        }

        this.canvasAnimationId = requestAnimationFrame(drawComplex);
      };
      drawComplex();
    },
    handleResize() {
      const canvas = document.getElementById('dnaComplexBackground');
      if (canvas) {
        canvas.width = canvas.parentElement.clientWidth;
        canvas.height = canvas.parentElement.clientHeight || window.innerHeight;
      }
      if (this.rocChartInstance) this.rocChartInstance.resize();
      if (this.accChartInstance) this.accChartInstance.resize();
    }
  }
};
</script>

<style scoped lang="scss">
.bio-platform {
  background-color: #030712; /* 改用生物实验室高级深邃黑底色 */
  min-height: calc(100vh - 84px);
  padding: 24px;
  position: relative;
  overflow: hidden;
}

/* 🧬 高维基因画布 */
.dna-bg-complex {
  position: absolute;
  top: 0; left: 0; width: 100%; height: 100%;
  z-index: 0;
  pointer-events: none;
}

.flex-relative {
  position: relative;
  z-index: 1;
}

.mb20 {
  margin-bottom: 20px;
}

/* 🧪 顶部模型展示：极光多阶流动渐变特效 */
.hero-section {
  position: relative;
  background: linear-gradient(-45deg, #020617, #0f172a, #1e1b4b, #030712);
  background-size: 300% 300%;
  animation: auroraShift 12s ease infinite;
  color: #f8fafc;
  padding: 26px 36px;
  border-radius: 6px;
  box-shadow: 0 4px 30px rgba(0, 0, 0, 0.4);
  border: 1px solid rgba(99, 102, 241, 0.15);
  overflow: hidden;

  h2 {
    margin-top: 0;
    font-size: 21px;
    font-weight: 600;
    color: #38bdf8;
    text-shadow: 0 0 10px rgba(56, 189, 248, 0.3);
  }
  p {
    margin: 8px 0 0 0;
    font-size: 13.5px;
    line-height: 1.7;
    color: #94a3b8;
    strong { color: #34d399; }
  }
}

@keyframes auroraShift {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}

/* 🎛️ 模块卡片：微粒质感与锁闭权限样式 */
.module-card {
  position: relative;
  text-align: center;
  padding: 20px 0;
  cursor: pointer;
  background: rgba(15, 23, 42, 0.65) !important;
  border: 1px solid rgba(56, 189, 248, 0.12) !important;
  backdrop-filter: blur(8px);
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);

  i {
    font-size: 30px;
    color: #64748b;
    margin-bottom: 10px;
    transition: color 0.3s;
  }
  .module-name {
    font-size: 14px;
    font-weight: 500;
    color: #94a3b8;
  }

  // 状态闪烁指示灯
  .card-status-dot {
    position: absolute;
    top: 10px; right: 10px;
    width: 6px; height: 6px;
    border-radius: 50%;
    background: #475569;
    &.active {
      background: #00f2fe;
      box-shadow: 0 0 8px #00f2fe;
    }
  }
}

/* 被守卫拦截锁定的样式 */
.is-disabled {
  cursor: not-allowed !important;
  opacity: 0.45;
  border: 1px dashed rgba(148, 163, 184, 0.2) !important;
  &:hover {
    transform: none !important;
    box-shadow: none !important;
  }
}

/* 激活可点击样式 */
.is-ready:hover {
  transform: translateY(-4px);
  border-color: #00f2fe !important;
  box-shadow: 0 8px 24px rgba(0, 242, 254, 0.15);
  i { color: #00f2fe; }
  .module-name { color: #f1f5f9; }
}

/* 数据上传区样式 */
.upload-card {
  background: rgba(15, 23, 42, 0.4) !important;
  border: 1px solid rgba(255, 255, 255, 0.05) !important;
  
  .data-type-selector {
    float: right; margin-top: -4px;
    .label-text { font-size: 12px; color: #64748b; }
  }

  ::v-deep .el-upload-dragger {
    width: 100%; height: 140px;
    background-color: rgba(30, 41, 59, 0.3) !important;
    border: 1px dashed rgba(56, 189, 248, 0.2);
    
    &:hover { border-color: #00f2fe; }
    .el-icon-upload { margin: 20px 0 10px; font-size: 40px; color: #475569; }
  }
  
  .uploaded-text {
    color: #34d399; font-weight: bold; margin-top: 40px;
    i { margin-right: 5px; }
  }
}

/* 队列及表格组件美化 */
.table-card {
  background: rgba(15, 23, 42, 0.4) !important;
  border: 1px solid rgba(255, 255, 255, 0.05) !important;
  height: 285px;
  
  ::v-deep .el-table {
    background: transparent;
    color: #94a3b8;
    th { background: #1e293b !important; color: #cbd5e1; border-bottom: 1px solid #334155; }
    tr { background: rgba(15, 23, 42, 0.5) !important; }
    td { border-bottom: 1px solid #1e293b; }
  }
  .status-text-success { color: #10b981; }
  .status-text-wait { color: #f59e0b; }
  .status-text-ready { color: #00f2fe; font-weight: bold; animation: pulse 2s infinite; }
}

.card-title {
  font-weight: 600; font-size: 14px; color: #e2e8f0;
  i { color: #00f2fe; margin-right: 6px; }
}

/* 🧪 全屏高通量测序加载沙箱遮罩 */
.gene-loading-overlay {
  position: fixed;
  top: 0; left: 0; right: 0; bottom: 0;
  background: rgba(3, 7, 18, 0.9);
  backdrop-filter: blur(12px);
  z-index: 9999;
  display: flex; align-items: center; justify-content: center;

  .loading-box {
    width: 480px; background: #0f172a; border-radius: 8px;
    padding: 30px; border: 1px solid #00f2fe;
    box-shadow: 0 0 30px rgba(0, 242, 254, 0.25);
    text-align: center;
  }
  
  .sequencer-animation {
    background: #020617; border-radius: 4px; padding: 15px;
    margin-bottom: 20px; position: relative; overflow: hidden;
    border: 1px solid #1e293b;

    .base-strand-view {
      font-family: 'Courier New', Courier, monospace;
      color: #34d399; font-weight: bold; font-size: 16px; letter-spacing: 2px;
    }
    .scanner-line {
      position: absolute; top: 0; left: 0; width: 100%; height: 2px;
      background: linear-gradient(90deg, transparent, #00f2fe, transparent);
      animation: scan 1.5s linear infinite;
    }
  }

  .sand-box-title { color: #f8fafc; margin: 0 0 5px 0; font-size: 16px; }
  .sand-box-sub { color: #64748b; font-size: 12px; margin: 0 0 20px 0; }
  
  .gene-progress-wrapper {
    ::v-deep .el-progress-bar__inner {
      background: linear-gradient(90deg, #00f2fe 0%, #4facfe 100%);
    }
    ::v-deep .el-progress__text { color: #00f2fe; font-weight: bold; }
    
    .sequencing-metrics {
      display: flex; justify-content: space-between;
      margin-top: 10px; font-size: 11px; color: #475569; font-family: monospace;
    }
  }
}

/* 下方科研评估卡片 */
.chart-card {
  background: #0f172a !important;
  border: 1px solid rgba(255, 255, 255, 0.05) !important;
  border-top: 3px solid #00f2fe !important;
}

/* 动画特效库 */
@keyframes scan {
  0% { top: 0%; }
  50% { top: 100%; }
  100% { top: 0%; }
}
@keyframes pulse {
  0% { opacity: 0.6; }
  50% { opacity: 1; }
  100% { opacity: 0.6; }
}
</style>