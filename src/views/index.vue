<template>
  <div class="app-container home bio-platform">
    <canvas id="dnaBackground" class="dna-bg"></canvas>

    <el-row :gutter="20" class="mb20 flex-relative">
      <el-col :span="24">
        <div class="hero-section">
          <h2>SDP-ChroNet 染色质相互作用预测平台</h2>
          <p>
            基于深度学习框架的高精度染色质空间互作分析系统。集成高并发数据中台，支持多细胞系关键基序（如 WT1、KLF9、KLF6 等）的结合位点特征提取与鲁棒性诊断。
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
              :class="['module-card', item.action === 'analyze' ? 'analyze-btn' : '']" 
              @click.native="handleModuleClick(item.action)"
            >
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
            action="YOUR_UPLOAD_API_URL"
            :data="uploadParam"
            :limit="1"
            accept=".fasta,.fa,.txt"
            :on-success="handleUploadSuccess"
            :before-upload="beforeUpload"
          >
            <i class="el-icon-upload"></i>
            <div class="el-upload__text">将 FASTA 文件拖到此处，或 <em>点击上传</em></div>
            <div class="el-upload__tip" slot="tip">支持上传标准格式的基因组序列文件，单文件大小限 50MB 以内</div>
          </el-upload>
        </el-card>
      </el-col>

      <el-col :xs="24" :sm="24" :md="10" :lg="10">
        <el-card shadow="never" class="table-card">
          <div slot="header" class="clearfix">
            <span class="card-title"><i class="el-icon-s-data"></i> 实时分析队列</span>
            <el-button style="float: right; padding: 3px 0" type="text">历史记录</el-button>
          </div>
          <el-table :data="recentResults" style="width: 100%" size="small" border stripe>
            <el-table-column prop="jobId" label="任务 ID" width="110"></el-table-column>
            <el-table-column prop="cellLine" label="细胞系" width="80"></el-table-column>
            <el-table-column prop="motif" label="基序 (Motif)"></el-table-column>
            <el-table-column prop="status" label="状态" width="85">
              <template slot-scope="scope">
                <el-tag :type="scope.row.status === '已完成' ? 'success' : 'warning'" size="mini" effect="dark">
                  {{ scope.row.status }}
                </el-tag>
              </template>
            </el-table-column>
            <el-table-column prop="score" label="互作概率" align="right" width="80"></el-table-column>
          </el-table>
        </el-card>
      </el-col>
    </el-row>

    <el-collapse-transition>
      <el-row :gutter="20" v-show="showAnalysis" class="mb20 flex-relative">
        <el-col :xs="24" :sm="24" :md="12" :lg="12">
          <el-card shadow="never" class="chart-card">
            <div slot="header" class="clearfix">
              <span class="card-title"><i class="el-icon-data-line"></i> 染色质互作预测评估 (ROC 曲线)</span>
              <el-tag size="mini" type="info" style="float: right">AUC: {{ generatedAuc }}</el-tag>
            </div>
            <div id="rocChart" style="height: 280px; width: 100%;"></div>
          </el-card>
        </el-col>
        <el-col :xs="24" :sm="24" :md="12" :lg="12">
          <el-card shadow="never" class="chart-card">
            <div slot="header" class="clearfix">
              <span class="card-title"><i class="el-icon-pie-chart"></i> 实时性能指标 (Accuracy)</span>
            </div>
            <div id="accChart" style="height: 280px; width: 100%;"></div>
          </el-card>
        </el-col>
      </el-row>
    </el-collapse-transition>
  </div>
</template>

<script>
import * as echarts from 'echarts';

export default {
  name: "Index",
  data() {
    return {
      // 上传参数绑定
      uploadParam: {
        dataType: 'sequence' // 默认选择序列数据
      },
      // 三大核心模块
      modules: [
        { name: '数据上传', icon: 'el-icon-upload2', action: 'upload' },
        { name: '序列预测', icon: 'el-icon-cpu', action: 'predict' },
        { name: '结果分析', icon: 'el-icon-data-analysis', action: 'analyze' }
      ],
      // 表格模拟数据
      recentResults: [
        { jobId: 'CHRO-1024A', cellLine: 'K562', motif: 'WT1', status: '已完成', score: '0.941' },
        { jobId: 'CHRO-1025B', cellLine: 'K562', motif: 'KLF9', status: '已完成', score: '0.887' },
        { jobId: 'CHRO-1026C', cellLine: 'K562', motif: 'KLF6', status: '已完成', score: '0.912' },
        { jobId: 'CHRO-1027D', cellLine: 'GM12878', motif: 'CTCF', status: '分析中', score: '---' }
      ],
      showAnalysis: false, // 控制图表显示隐藏
      generatedAuc: '0.00',
      rocChartInstance: null,
      accChartInstance: null,
      animationFrameId: null
    };
  },
  mounted() {
    this.initDnaCanvas(); // 初始化背景动态基因链条
    window.addEventListener('resize', this.handleResize);
  },
  beforeDestroy() {
    // 销毁组件时释放动画和图表资源
    if (this.animationFrameId) cancelAnimationFrame(this.animationFrameId);
    window.removeEventListener('resize', this.handleResize);
    if (this.rocChartInstance) this.rocChartInstance.dispose();
    if (this.accChartInstance) this.accChartInstance.dispose();
  },
  methods: {
    // 点击功能模块
    handleModuleClick(action) {
      if (action === 'analyze') {
        this.showAnalysis = true;
        this.$nextTick(() => {
          this.generateRandomCharts();
        });
        this.$message({ message: '已成功模拟读取当前模型评估参数！', type: 'success' });
      } else if (action === 'predict') {
        this.$message({ message: '已启动后台神经网络预测服务...', type: 'info' });
      } else {
        this.$message({ message: '请使用下方拖拽面板完成数据提交。', type: 'info' });
      }
    },
    // 上传前的校验
    beforeUpload(file) {
      const isFa = file.name.endsWith('.fasta') || file.name.endsWith('.fa') || file.name.endsWith('.txt');
      if (!isFa) {
        this.$message.error('系统仅支持标准格式的 .fasta 或 .fa 文件！');
      }
      return isFa;
    },
    handleUploadSuccess() {
      this.$message.success('序列上传成功，已挂载至分析服务器。');
    },
    // 动态生成包含随机数的科研图表
    generateRandomCharts() {
      // 随机生成 90% - 97% 之间的准确率和对应的 AUC
      const randomAcc = (Math.random() * (97 - 90) + 90).toFixed(2);
      this.generatedAuc = (randomAcc / 100 + 0.02).toFixed(3);

      // 1. 初始化或刷新 ROC 曲线
      if (!this.rocChartInstance) {
        this.rocChartInstance = echarts.init(document.getElementById('rocChart'));
      }
      // 构建一条贴近随机生成AUC的平滑曲线
      const p1 = (Math.random() * (0.65 - 0.55) + 0.55).toFixed(2);
      const p2 = (Math.random() * (0.88 - 0.80) + 0.80).toFixed(2);
      
      const rocOption = {
        grid: { top: '10%', left: '8%', right: '8%', bottom: '15%' },
        tooltip: { trigger: 'axis', formatter: 'FPR: {b}<br/>TPR: {c}' },
        xAxis: { type: 'value', min: 0, max: 1, name: '伪阳性率 (FPR)', nameLocation: 'center', nameGap: 25 },
        yAxis: { type: 'value', min: 0, max: 1, name: '真阳性率 (TPR)' },
        series: [
          {
            name: 'ROC',
            type: 'line',
            smooth: true,
            symbol: 'none',
            color: '#0052d9',
            lineStyle: { width: 3 },
            areaStyle: {
              color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
                { offset: 0, color: 'rgba(0, 82, 217, 0.3)' },
                { offset: 1, color: 'rgba(0, 82, 217, 0.0)' }
              ])
            },
            data: [[0, 0], [0.05, p1], [0.15, p2], [0.4, 0.94], [0.7, 0.98], [1, 1]]
          },
          {
            name: 'Diagonal Baseline',
            type: 'line',
            symbol: 'none',
            lineStyle: { type: 'dashed', color: '#bbb' },
            data: [[0, 0], [1, 1]]
          }
        ]
      };
      this.rocChartInstance.setOption(rocOption);

      // 2. 初始化或刷新仪表盘
      if (!this.accChartInstance) {
        this.accChartInstance = echarts.init(document.getElementById('accChart'));
      }
      const accOption = {
        series: [
          {
            type: 'gauge',
            startAngle: 180,
            endAngle: 0,
            min: 0,
            max: 100,
            radius: '100%',
            center: ['50%', '75%'],
            axisLine: { lineStyle: { width: 8, color: [[0.3, '#ff4d4f'], [0.7, '#ffc069'], [1, '#07c160']] } },
            pointer: { width: 4, length: '70%' },
            progress: { show: true, width: 8 },
            detail: { offsetCenter: [0, '-10%'], valueAnimation: true, formatter: '{value}%', fontSize: 24, fontWeight: 'bold' },
            data: [{ value: randomAcc, name: 'Model Accuracy' }]
          }
        ]
      };
      this.accChartInstance.setOption(accOption);
    },
    // HTML5 Canvas 绘制动态缠绕基因链条
    initDnaCanvas() {
      const canvas = document.getElementById('dnaBackground');
      const ctx = canvas.getContext('2d');
      
      const resizeCanvas = () => {
        canvas.width = canvas.parentElement.clientWidth;
        canvas.height = canvas.parentElement.clientHeight || window.innerHeight;
      };
      resizeCanvas();

      let len = 0;
      const dots = [];
      const numDots = 25; // 链条节点数量
      
      const draw = () => {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        len += 0.015; // 控制转动速度

        const centerY = canvas.height * 0.55;
        const startX = 50;
        const endX = canvas.width - 50;
        const step = (endX - startX) / numDots;

        for (let i = 0; i <= numDots; i++) {
          const x = startX + i * step;
          // 第一条螺旋线
          const y1 = centerY + Math.sin(len + i * 0.4) * 60;
          // 第二条逆相螺旋线
          const y2 = centerY + Math.sin(len + i * 0.4 + Math.PI) * 60;
          
          // 计算节点的视差大小深度
          const size1 = (Math.cos(len + i * 0.4) + 1) * 3 + 2;
          const size2 = (Math.cos(len + i * 0.4 + Math.PI) + 1) * 3 + 2;

          // 绘制连接碱基对中间的氢键线段
          ctx.beginPath();
          ctx.strokeStyle = 'rgba(0, 82, 217, 0.12)';
          ctx.lineWidth = 1.5;
          ctx.moveTo(x, y1);
          ctx.lineTo(x, y2);
          ctx.stroke();

          // 绘制螺旋主链节点（腺嘌呤、胸腺嘧啶等生信隐喻色调）
          ctx.beginPath();
          ctx.fillStyle = size1 > 4 ? 'rgba(0, 82, 217, 0.45)' : 'rgba(0, 168, 112, 0.3)';
          ctx.arc(x, y1, size1, 0, Math.PI * 2);
          ctx.fill();

          ctx.beginPath();
          ctx.fillStyle = size2 > 4 ? 'rgba(0, 168, 112, 0.45)' : 'rgba(0, 82, 217, 0.3)';
          ctx.arc(x, y2, size2, 0, Math.PI * 2);
          ctx.fill();
        }

        // 连贯主链骨架线
        ctx.beginPath();
        ctx.strokeStyle = 'rgba(0, 82, 217, 0.15)';
        ctx.lineWidth = 1;
        for (let i = 0; i <= numDots; i++) {
          const x = startX + i * step;
          const y1 = centerY + Math.sin(len + i * 0.4) * 60;
          if (i === 0) ctx.moveTo(x, y1); else ctx.lineTo(x, y1);
        }
        ctx.stroke();

        ctx.beginPath();
        ctx.strokeStyle = 'rgba(0, 168, 112, 0.1)';
        ctx.lineWidth = 1;
        for (let i = 0; i <= numDots; i++) {
          const x = startX + i * step;
          const y2 = centerY + Math.sin(len + i * 0.4 + Math.PI) * 60;
          if (i === 0) ctx.moveTo(x, y2); else ctx.lineTo(x, y2);
        }
        ctx.stroke();

        this.animationFrameId = requestAnimationFrame(draw);
      };
      draw();
    },
    handleResize() {
      if (this.rocChartInstance) this.rocChartInstance.resize();
      if (this.accChartInstance) this.accChartInstance.resize();
      const canvas = document.getElementById('dnaBackground');
      if (canvas) {
        canvas.width = canvas.parentElement.clientWidth;
        canvas.height = canvas.parentElement.clientHeight || window.innerHeight;
      }
    }
  }
};
</script>

<style scoped lang="scss">
.bio-platform {
  background-color: #f0f4f8;
  min-height: calc(100vh - 84px);
  padding: 24px;
  position: relative;
  overflow: hidden;
}

/* 动态网格与DNA背景 */
.dna-bg {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 0;
  pointer-events: none;
}

/* 保证上层组件定位不被背景覆盖 */
.flex-relative {
  position: relative;
  z-index: 1;
}

.mb20 {
  margin-bottom: 20px;
}

/* 科研头部区域 */
.hero-section {
  background: linear-gradient(145deg, #0a1931 0%, #15305b 100%);
  color: #ffffff;
  padding: 24px 32px;
  border-radius: 6px;
  box-shadow: 0 4px 16px rgba(10, 25, 49, 0.12);
  
  h2 {
    margin-top: 0;
    margin-bottom: 10px;
    font-weight: 600;
    font-size: 22px;
    letter-spacing: 0.5px;
  }
  p {
    margin: 0;
    font-size: 13.5px;
    line-height: 1.6;
    opacity: 0.85;
  }
}

/* 功能控制卡片 */
.module-card {
  text-align: center;
  padding: 18px 0;
  cursor: pointer;
  transition: all 0.25s ease;
  background: rgba(255, 255, 255, 0.85);
  border: 1px solid rgba(0, 82, 217, 0.08);
  backdrop-filter: blur(4px);
  
  &:hover {
    transform: translateY(-3px);
    background: #ffffff;
    box-shadow: 0 6px 16px rgba(0, 82, 217, 0.12);
    border-color: #0052d9;
    i { color: #0052d9; }
  }

  i {
    font-size: 28px;
    color: #4e5969;
    margin-bottom: 10px;
    transition: color 0.25s ease;
  }

  .module-name {
    font-size: 14px;
    font-weight: 500;
    color: #1d2129;
  }
}

/* 突出显示结果分析按钮 */
.analyze-btn {
  background: rgba(242, 243, 245, 0.9);
  border-left: 3px solid #0052d9;
}

/* 精准控制数据类型选择器位置 */
.data-type-selector {
  float: right;
  margin-top: -4px;
  .label-text {
    font-size: 12px;
    color: #86909c;
    margin-right: 4px;
  }
}

.card-title {
  font-weight: 600;
  font-size: 14.5px;
  color: #1d2129;
  i {
    color: #0052d9;
    margin-right: 6px;
  }
}

/* 上传容器自适应与美化 */
.upload-card {
  background: rgba(255, 255, 255, 0.9);
  ::v-deep .el-upload {
    width: 100%;
  }
  ::v-deep .el-upload-dragger {
    width: 100%;
    height: 155px;
    background-color: #f8fafc;
    border-radius: 4px;
    
    &:hover {
      border-color: #0052d9;
      background-color: #f2f3ff;
    }
    .el-icon-upload {
      margin: 25px 0 12px;
      font-size: 46px;
      color: #86909c;
    }
  }
}

/* 数据队列卡片 */
.table-card {
  background: rgba(255, 255, 255, 0.9);
  height: 298px;
  ::v-deep .el-card__body {
    padding: 10px;
  }
}

/* 底部图表卡片样式 */
.chart-card {
  background: #ffffff;
  border-top: 3px solid #0052d9;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.02);
}
</style>