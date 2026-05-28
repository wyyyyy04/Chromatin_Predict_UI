<template>
  <div class="app-container home bio-platform">
    <el-row :gutter="20" class="mb20">
      <el-col :span="24">
        <div class="hero-section">
          <h2>SDP-ChroNet 染色质相互作用预测平台</h2>
          <p>
            基于 Shared Residual Dilated Units 的深度学习框架，支持多细胞系的高精度染色质空间互作分析。
            请在下方上传 FASTA 序列数据以启动预测分析流程。
          </p>
        </div>
      </el-col>
    </el-row>

    <el-row :gutter="20">
      <el-col :xs="24" :sm="24" :md="16" :lg="16">
        
        <el-row :gutter="15" class="mb20">
          <el-col :span="6" v-for="(item, index) in modules" :key="index">
            <el-card shadow="hover" class="module-card" @click.native="handleModuleClick(item.action)">
              <i :class="item.icon"></i>
              <div class="module-name">{{ item.name }}</div>
            </el-card>
          </el-col>
        </el-row>

        <el-card shadow="never" class="mb20 upload-card">
          <div slot="header" class="clearfix">
            <span class="card-title"><i class="el-icon-upload cloud-icon"></i> 序列数据上传 (FASTA)</span>
          </div>
          <el-upload
            class="upload-demo"
            drag
            action="YOUR_UPLOAD_API_URL"
            :limit="1"
            accept=".fasta,.fa,.txt"
            :on-success="handleUploadSuccess"
            :before-upload="beforeUpload"
          >
            <i class="el-icon-upload"></i>
            <div class="el-upload__text">将 FASTA 文件拖到此处，或 <em>点击上传</em></div>
            <div class="el-upload__tip" slot="tip">只能上传 .fasta / .fa 文件，且大小不超过 50MB</div>
          </el-upload>
        </el-card>

        <el-card shadow="never">
          <div slot="header" class="clearfix">
            <span class="card-title"><i class="el-icon-s-data"></i> 最近预测结果</span>
            <el-button style="float: right; padding: 3px 0" type="text">查看全部</el-button>
          </div>
          <el-table :data="recentResults" style="width: 100%" size="small">
            <el-table-column prop="jobId" label="任务 ID" width="120"></el-table-column>
            <el-table-column prop="cellLine" label="细胞系" width="100"></el-table-column>
            <el-table-column prop="motif" label="关键 Motif" width="120"></el-table-column>
            <el-table-column prop="status" label="状态" width="100">
              <template slot-scope="scope">
                <el-tag :type="scope.row.status === '完成' ? 'success' : 'primary'" size="mini">
                  {{ scope.row.status }}
                </el-tag>
              </template>
            </el-table-column>
            <el-table-column prop="score" label="互作概率" align="right"></el-table-column>
          </el-table>
        </el-card>
      </el-col>

      <el-col :xs="24" :sm="24" :md="8" :lg="8">
        
        <el-card shadow="never" class="mb20">
          <div slot="header" class="clearfix">
            <span class="card-title"><i class="el-icon-data-line"></i> 模型评估 (ROC)</span>
          </div>
          <div id="rocChart" style="height: 250px;"></div>
        </el-card>

        <el-card shadow="never">
          <div slot="header" class="clearfix">
            <span class="card-title"><i class="el-icon-pie-chart"></i> 实时性能指标</span>
          </div>
          <div id="accChart" style="height: 250px;"></div>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import * as echarts from 'echarts';

export default {
  name: "Index",
  data() {
    return {
      modules: [
        { name: '数据上传', icon: 'el-icon-upload', action: 'upload' },
        { name: '模型训练', icon: 'el-icon-cpu', action: 'train' },
        { name: '序列预测', icon: 'el-icon-magic-stick', action: 'predict' },
        { name: '结果分析', icon: 'el-icon-data-analysis', action: 'analyze' }
      ],
      recentResults: [
        { jobId: 'JOB-20260401', cellLine: 'K562', motif: 'WT1', status: '完成', score: '0.942' },
        { jobId: 'JOB-20260402', cellLine: 'K562', motif: 'KLF9', status: '完成', score: '0.891' },
        { jobId: 'JOB-20260403', cellLine: 'GM12878', motif: 'CTCF', status: '预测中', score: '-' },
        { jobId: 'JOB-20260404', cellLine: 'K562', motif: 'KLF6', status: '完成', score: '0.915' }
      ]
    };
  },
  mounted() {
    this.initRocChart();
    this.initAccChart();
  },
  methods: {
    handleModuleClick(action) {
      // 在这里处理模块卡片的点击跳转逻辑
      this.$message.info(`触发操作: ${action}`);
    },
    beforeUpload(file) {
      const isFasta = file.name.endsWith('.fasta') || file.name.endsWith('.fa') || file.name.endsWith('.txt');
      if (!isFasta) {
        this.$message.error('上传文件只能是 FASTA 格式!');
      }
      return isFasta;
    },
    handleUploadSuccess(res, file) {
      this.$message.success('FASTA 数据上传成功，已加入分析队列。');
    },
    initRocChart() {
      const chartDom = document.getElementById('rocChart');
      const myChart = echarts.init(chartDom);
      const option = {
        tooltip: { trigger: 'axis' },
        grid: { left: '3%', right: '4%', bottom: '3%', containLabel: true },
        xAxis: { type: 'value', name: 'FPR', nameLocation: 'middle', nameGap: 25 },
        yAxis: { type: 'value', name: 'TPR' },
        series: [
          {
            name: 'ROC',
            type: 'line',
            smooth: true,
            color: '#1890ff',
            areaStyle: {
              color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
                { offset: 0, color: 'rgba(24,144,255,0.5)' },
                { offset: 1, color: 'rgba(24,144,255,0.1)' }
              ])
            },
            data: [[0, 0], [0.1, 0.6], [0.2, 0.82], [0.4, 0.93], [0.6, 0.97], [1, 1]]
          },
          {
            name: 'Baseline',
            type: 'line',
            lineStyle: { type: 'dashed', color: '#999' },
            data: [[0, 0], [1, 1]]
          }
        ]
      };
      myChart.setOption(option);
      window.addEventListener("resize", () => myChart.resize());
    },
    initAccChart() {
      const chartDom = document.getElementById('accChart');
      const myChart = echarts.init(chartDom);
      const option = {
        tooltip: { formatter: '{a} <br/>{b} : {c}%' },
        series: [
          {
            name: '模型准确率',
            type: 'gauge',
            progress: { show: true, width: 10 },
            axisLine: { lineStyle: { width: 10 } },
            axisTick: { show: false },
            splitLine: { length: 12, lineStyle: { width: 2, color: '#999' } },
            pointer: { width: 5 },
            detail: { valueAnimation: true, formatter: '{value}%', fontSize: 20 },
            data: [{ value: 92.4, name: 'Accuracy' }]
          }
        ]
      };
      myChart.setOption(option);
      window.addEventListener("resize", () => myChart.resize());
    }
  }
};
</script>

<style scoped lang="scss">
/* 基础容器样式 */
.bio-platform {
  background-color: #f4f7fc;
  min-height: calc(100vh - 84px);
  padding: 20px;
  
  /* 注入伪造的 DNA/节点网络 背景水印效果 */
  position: relative;
  &::before {
    content: "";
    position: absolute;
    top: 0; left: 0; right: 0; bottom: 0;
    background-image: radial-gradient(circle at 15% 50%, rgba(24, 144, 255, 0.04) 0%, transparent 50%),
                      radial-gradient(circle at 85% 30%, rgba(24, 144, 255, 0.04) 0%, transparent 50%);
    pointer-events: none;
    z-index: 0;
  }
}

.mb20 {
  margin-bottom: 20px;
}

/* 顶部科研风欢迎区 */
.hero-section {
  background: linear-gradient(135deg, #0f2027 0%, #203a43 50%, #2c5364 100%);
  color: #fff;
  padding: 30px 40px;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
  position: relative;
  z-index: 1;

  h2 {
    margin-top: 0;
    font-weight: 600;
    font-size: 24px;
    letter-spacing: 1px;
  }
  
  p {
    margin: 10px 0 0 0;
    font-size: 14px;
    opacity: 0.85;
    max-width: 700px;
    line-height: 1.6;
  }
}

/* 卡片通用样式 */
.el-card {
  border: none;
  border-radius: 8px;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.03);
  position: relative;
  z-index: 1;
}

.card-title {
  font-weight: 600;
  color: #333;
  font-size: 15px;
  i {
    margin-right: 5px;
    color: #1890ff;
  }
}

/* 核心功能卡片 */
.module-card {
  text-align: center;
  padding: 20px 0;
  cursor: pointer;
  transition: all 0.3s ease;
  background-color: #fff;
  
  &:hover {
    transform: translateY(-5px);
    box-shadow: 0 8px 16px rgba(24, 144, 255, 0.15);
    i { color: #1890ff; }
  }

  i {
    font-size: 32px;
    color: #606266;
    margin-bottom: 15px;
    transition: color 0.3s ease;
  }

  .module-name {
    font-size: 15px;
    font-weight: 500;
    color: #303133;
  }
}

/* FASTA 上传区域覆盖样式 */
.upload-card {
  ::v-deep .el-upload {
    width: 100%;
  }
  ::v-deep .el-upload-dragger {
    width: 100%;
    height: 160px;
    background-color: #fafbfc;
    border: 1px dashed #d9d9d9;
    border-radius: 8px;
    transition: border-color 0.3s;
    
    &:hover {
      border-color: #1890ff;
      background-color: #f0f7ff;
    }
    
    .el-icon-upload {
      font-size: 50px;
      margin: 30px 0 10px;
      color: #8c939d;
    }
  }
}
</style>