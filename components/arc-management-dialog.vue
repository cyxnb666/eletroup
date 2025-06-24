<template>
  <el-dialog :visible.sync="dialogVisible" width="550px" :show-close="false" :close-on-click-modal="false"
    @close="handleClose">
    <div slot="title" class="title">ARC管理</div>

    <div class="arc-content" v-loading="loading">
      <h3>{{ activityDisplayName }}</h3>

      <div class="info-section">
        <div class="info-line">
          <div class="info-label">活动状态：</div>
          <div class="info-value">
            <el-tag :type="activityStatus.type">{{ activityStatus.text }}</el-tag>
          </div>
        </div>

        <div class="info-line">
          <div class="info-label">活动时间：</div>
          <div class="info-value">{{ activityTimeRange }}</div>
        </div>
      </div>

      <!-- ARC 部分 -->
      <div class="section-title">ARC</div>
      <div class="button-row">
        <!-- 终审合格按钮 - 占2/3 -->
        <div class="button-container wide">
          <div v-if="!arcData.hasArcRelation" class="disabled-button">
            <span class="button-text">终审合格</span>
            <i class="el-icon-upload2 upload-icon disabled"></i>
          </div>
          <div v-else-if="arcData.finalAuditStatus === 'none'" class="active-button" @click="handleArcFinalAudit">
            <span class="button-text">终审合格</span>
            <i v-if="!arcData.uploading" class="el-icon-upload2 upload-icon active"></i>
            <i v-else class="el-icon-loading upload-icon active"></i>
          </div>
          <div v-else class="uploaded-button-with-title">
            <div class="button-title">终审合格</div>
            <div class="file-info">
              <span class="file-name clickable" @click="downloadFile(arcData.finalAuditFile.downloadUrl)">{{
                arcData.finalAuditFile.fileName }}</span>
            </div>
            <i class="el-icon-upload2 upload-icon reupload-right" @click="handleArcFinalAudit"></i>
          </div>
        </div>

        <!-- 明细清单按钮 - 占1/3 -->
        <div class="button-container narrow">
          <div v-if="!arcData.hasArcRelation" class="disabled-button">
            <span class="button-text">明细清单</span>
            <i class="el-icon-download download-icon disabled"></i>
          </div>
          <div v-else class="active-button" @click="handleArcDetailDownload">
            <span class="button-text">明细清单</span>
            <i class="el-icon-download download-icon active"></i>
          </div>
        </div>
      </div>

      <!-- ARC Premium 部分 -->
      <div class="section-title">ARC Premium</div>
      <div class="button-row">
        <!-- 新增凭证按钮 -->
        <div class="button-container">
          <div v-if="arcPremiumData.certificateCount === 0" class="active-button" @click="handleNewCertificate">
            <span class="button-text">新增凭证</span>
            <i v-if="!arcPremiumData.uploadingNew" class="el-icon-upload2 upload-icon active"></i>
            <i v-else class="el-icon-loading upload-icon active"></i>
          </div>
          <div v-else class="uploaded-button-with-title">
            <div class="button-title-row">
              <span class="button-title">新增凭证</span>
              <i v-if="!arcPremiumData.uploadingNew" class="el-icon-upload2 upload-icon active"
                @click="handleNewCertificate"></i>
              <i v-else class="el-icon-loading upload-icon active"></i>
            </div>
            <div class="certificate-data-row">
              <div class="certificate-info-inline">
                <span class="certificate-count">{{ formatNumber(arcPremiumData.certificateCount) }}</span>
                <i class="el-icon-s-data data-icon"></i>
              </div>
              <span class="upload-time">{{ arcPremiumData.lastUploadTime }}</span>
            </div>
          </div>
        </div>

        <!-- 凭证退保按钮 -->
        <div class="button-container">
          <div v-if="!arcPremiumData.hasArcPremiumRelation" class="disabled-button">
            <span class="button-text">凭证退保</span>
            <i class="el-icon-upload2 upload-icon disabled"></i>
          </div>
          <div v-else-if="!arcPremiumData.surrenderUploadTime" class="active-button"
            @click="handleCertificateSurrender">
            <span class="button-text">凭证退保</span>
            <i v-if="!arcPremiumData.uploadingSurrender" class="el-icon-upload2 upload-icon active"></i>
            <i v-else class="el-icon-loading upload-icon active"></i>
          </div>
          <div v-else class="uploaded-button-with-title surrender-style">
            <div class="button-title-row">
              <span class="button-title">凭证退保</span>
              <div class="surrender-upload-container">
                <i v-if="!arcPremiumData.uploadingSurrender" class="el-icon-upload2 upload-icon active"
                  @click="handleCertificateSurrender"></i>
                <i v-else class="el-icon-loading upload-icon active"></i>
              </div>
            </div>
            <div class="surrender-time">{{ arcPremiumData.surrenderUploadTime }}</div>
          </div>
        </div>
      </div>

      <div class="button-row">
        <!-- ARC Premium 终审合格按钮 - 占2/3 -->
        <div class="button-container wide">
          <div v-if="!arcPremiumData.hasArcPremiumRelation" class="disabled-button">
            <span class="button-text">终审合格</span>
            <i class="el-icon-upload2 upload-icon disabled"></i>
          </div>
          <div v-else-if="arcPremiumData.finalAuditStatus === 'none'" class="active-button"
            @click="handlePremiumFinalAudit">
            <span class="button-text">终审合格</span>
            <i v-if="!arcPremiumData.uploadingFinal" class="el-icon-upload2 upload-icon active"></i>
            <i v-else class="el-icon-loading upload-icon active"></i>
          </div>
          <div v-else class="uploaded-button-with-title">
            <div class="button-title">终审合格</div>
            <div class="file-info">
              <span class="file-name clickable" @click="downloadFile(arcPremiumData.finalAuditFile.downloadUrl)">{{
                arcPremiumData.finalAuditFile.fileName }}</span>
            </div>
            <i class="el-icon-upload2 upload-icon reupload-right" @click="handlePremiumFinalAudit"></i>
          </div>
        </div>

        <!-- ARC Premium 明细清单按钮 - 占1/3 -->
        <div class="button-container narrow">
          <div v-if="!arcPremiumData.hasArcPremiumRelation" class="disabled-button">
            <span class="button-text">明细清单</span>
            <i class="el-icon-download download-icon disabled"></i>
          </div>
          <div v-else class="active-button" @click="handlePremiumDetailDownload">
            <span class="button-text">明细清单</span>
            <i class="el-icon-download download-icon active"></i>
          </div>
        </div>
      </div>
    </div>

    <span slot="footer" class="dialog-footer">
      <el-button type="primary" @click="dialogVisible = false">关闭</el-button>
    </span>

    <!-- 隐藏的文件选择input -->
    <input ref="fileInput" type="file" style="display: none" accept=".xls,.xlsx" @change="handleFileSelect">
  </el-dialog>
</template>

<script>
export default {
  name: 'ArcManagementDialog',
  props: {
    visible: {
      type: Boolean,
      default: false
    },
    activityName: {
      type: String,
      default: ''
    },
    activeId: {
      type: String,
      default: ''
    },
    deptCode: {
      type: String,
      default: ''
    },
    currentActivity: {
      type: Object,
      default: () => ({})
    }
  },
  data() {
    return {
      dialogVisible: false,
      loading: false,
      currentUploadType: '', // 'arc_final' | 'premium_new' | 'premium_surrender' | 'premium_final'

      // 模拟数据 - 设置对比状态
      arcData: {
        hasArcRelation: true,        // ARC有关联凭证
        finalAuditStatus: 'none',    // 没有上传文件 -> 应该显示蓝色按钮
        uploading: false,
        finalAuditFile: null         // 确保没有文件
      },

      arcPremiumData: {
        hasArcPremiumRelation: false, // ARC Premium无关联凭证 -> 终审合格应该显示灰色按钮
        certificateCount: 0,
        lastUploadTime: null,
        surrenderUploadTime: null,
        finalAuditStatus: 'none',    // 没有上传文件
        finalAuditFile: null,        // 确保没有文件
        uploadingNew: false,
        uploadingSurrender: false,
        uploadingFinal: false
      }
    };
  },
  computed: {
    activityDisplayName() {
      return this.activityName || '';
    },
    activityStatus() {
      if (!this.currentActivity) {
        return { text: '未知', type: 'info' };
      }

      const overdueStatus = this.currentActivity.overdueStatus;
      return overdueStatus === "0"
        ? { text: '有效', type: 'success' }
        : { text: '无效', type: 'warning' };
    },
    activityTimeRange() {
      if (!this.currentActivity || !this.currentActivity.startTime) {
        return '未知';
      }

      const startDate = new Date(this.currentActivity.startTime);
      const startYear = startDate.getFullYear();
      const startMonth = String(startDate.getMonth() + 1).padStart(2, '0');
      const startDay = String(startDate.getDate()).padStart(2, '0');
      const formattedStartDate = `${startYear}年${startMonth}月${startDay}日`;

      if (this.currentActivity.endTime) {
        const endDate = new Date(this.currentActivity.endTime);
        const endYear = endDate.getFullYear();
        const endMonth = String(endDate.getMonth() + 1).padStart(2, '0');
        const endDay = String(endDate.getDate()).padStart(2, '0');
        const formattedEndDate = `${endYear}年${endMonth}月${endDay}日`;
        return `${formattedStartDate} - ${formattedEndDate}`;
      } else {
        return `${formattedStartDate} - 永久`;
      }
    }
  },
  watch: {
    visible(val) {
      this.dialogVisible = val;
      if (val && this.activeId && this.deptCode) {
        this.fetchArcData();
      }
    },
    dialogVisible(val) {
      if (!val) {
        this.$emit('update:visible', false);
      }
    }
  },
  methods: {
    handleClose() {
      this.$emit('update:visible', false);
    },

    fetchArcData() {
      this.loading = true;

      setTimeout(() => {
        // 复杂场景：混合多种状态
        this.arcData = {
          hasArcRelation: true,         // ARC有关联凭证
          finalAuditStatus: 'none',     // 终审合格未上传
          uploading: true,              // 正在上传终审合格文件（loading状态）
          finalAuditFile: null
        };

        this.arcPremiumData = {
          hasArcPremiumRelation: true,  // ARC Premium有关联凭证
          certificateCount: 856,        // 有凭证数据
          lastUploadTime: '2025/06/20 09:30',
          surrenderUploadTime: null,    // 凭证退保未上传
          finalAuditStatus: 'none',     // 终审合格未上传
          finalAuditFile: null,
          uploadingNew: false,          // 新增凭证不在上传中
          uploadingSurrender: true,     // 凭证退保正在上传中（loading状态）
          uploadingFinal: false         // 终审合格不在上传中
        };

        this.loading = false;
      }, 500);
    },

    formatNumber(value) {
      if (value === null || value === undefined) return '0';
      return value.toLocaleString();
    },

    // ARC 终审合格
    handleArcFinalAudit() {
      if (this.arcData.uploading) return;
      this.currentUploadType = 'arc_final';
      this.$refs.fileInput.click();
    },

    // ARC 明细清单下载
    handleArcDetailDownload() {
      this.$message.info('开始下载ARC明细清单');
      // 模拟下载
      const link = document.createElement('a');
      link.href = '#'; // 实际项目中这里会是真实的下载URL
      link.download = 'ARC明细清单.xlsx';
      link.click();
    },

    // ARC Premium 新增凭证
    handleNewCertificate() {
      if (this.arcPremiumData.uploadingNew) return;
      this.currentUploadType = 'premium_new';
      this.$refs.fileInput.click();
    },

    // ARC Premium 凭证退保
    handleCertificateSurrender() {
      if (this.arcPremiumData.uploadingSurrender) return;
      this.currentUploadType = 'premium_surrender';
      this.$refs.fileInput.click();
    },

    // ARC Premium 终审合格
    handlePremiumFinalAudit() {
      if (this.arcPremiumData.uploadingFinal) return;
      this.currentUploadType = 'premium_final';
      this.$refs.fileInput.click();
    },

    // ARC Premium 明细清单下载
    handlePremiumDetailDownload() {
      this.$message.info('开始下载ARC Premium明细清单');
      // 模拟下载
      const link = document.createElement('a');
      link.href = '#';
      link.download = 'ARC_Premium明细清单.xlsx';
      link.click();
    },

    // 处理文件选择
    handleFileSelect(event) {
      const file = event.target.files[0];
      if (!file) return;

      // 检查文件类型
      if (!file.name.match(/\.(xls|xlsx)$/i)) {
        this.$message.error('只能上传 .xls/.xlsx 格式的文件');
        return;
      }

      this.handleFileUpload(file);

      // 重置input，允许选择相同文件
      event.target.value = '';
    },

    // 处理文件上传
    handleFileUpload(file) {
      // 设置loading状态
      switch (this.currentUploadType) {
        case 'arc_final':
          this.arcData.uploading = true;
          break;
        case 'premium_new':
          this.arcPremiumData.uploadingNew = true;
          break;
        case 'premium_surrender':
          this.arcPremiumData.uploadingSurrender = true;
          break;
        case 'premium_final':
          this.arcPremiumData.uploadingFinal = true;
          break;
      }

      // 模拟上传过程
      setTimeout(() => {
        // 根据上传类型更新对应数据
        switch (this.currentUploadType) {
          case 'arc_final':
            this.arcData.uploading = false;
            this.arcData.finalAuditStatus = 'uploaded';
            this.arcData.finalAuditFile = {
              fileName: file.name,
              downloadUrl: 'https://example.com/uploaded-file.xlsx',
              uploadTime: this.formatDate(new Date())
            };
            break;
          case 'premium_new':
            this.arcPremiumData.uploadingNew = false;
            this.arcPremiumData.hasArcPremiumRelation = true;
            this.arcPremiumData.certificateCount += Math.floor(Math.random() * 1000) + 1000;
            this.arcPremiumData.lastUploadTime = this.formatDate(new Date());
            break;
          case 'premium_surrender':
            this.arcPremiumData.uploadingSurrender = false;
            this.arcPremiumData.surrenderUploadTime = this.formatDate(new Date());
            break;
          case 'premium_final':
            this.arcPremiumData.uploadingFinal = false;
            this.arcPremiumData.finalAuditStatus = 'uploaded';
            this.arcPremiumData.finalAuditFile = {
              fileName: file.name,
              downloadUrl: 'https://example.com/uploaded-file.xlsx',
              uploadTime: this.formatDate(new Date())
            };
            break;
        }

        this.$message.success('上传成功');
      }, 2000);
    },

    // 下载文件
    downloadFile(url) {
      if (url) {
        window.open(url, '_blank');
      } else {
        this.$message.warning('暂无下载链接');
      }
    },

    formatDate(date) {
      if (!date) return '';
      const year = date.getFullYear();
      const month = String(date.getMonth() + 1).padStart(2, '0');
      const day = String(date.getDate()).padStart(2, '0');
      const hours = String(date.getHours()).padStart(2, '0');
      const minutes = String(date.getMinutes()).padStart(2, '0');
      return `${year}/${month}/${day} ${hours}:${minutes}`;
    }
  }
};
</script>

<style scoped>
.title {
  font-weight: bold;
}

.arc-content {
  padding: 0;
}

h3 {
  margin: 0 0 15px;
  font-size: 16px;
  border-bottom: 1px solid #EBEEF5;
  padding-bottom: 10px;
}

.info-section {
  margin-bottom: 15px;
}

.info-line {
  display: flex;
  margin: 6px 0;
  align-items: center;
}

.info-label {
  min-width: 80px;
  color: #606266;
  font-size: 14px;
}

.info-value {
  flex: 1;
  display: flex;
  align-items: center;
}

.section-title {
  font-size: 14px;
  font-weight: bold;
  margin: 15px 0 10px 0;
  color: #000000;
}

.button-row {
  display: flex;
  gap: 12px;
  margin-bottom: 12px;
}

.button-container {
  height: 65px;
}

.button-container.wide {
  flex: 2;
}

.button-container.narrow {
  flex: 1;
}

.button-container:not(.wide):not(.narrow) {
  flex: 1;
}

.disabled-button {
  display: flex;
  justify-content: space-between;
  align-items: center;
  height: 100%;
  padding: 12px;
  border-radius: 6px;
  background-color: #F3F3F3;
  cursor: not-allowed;
}

.disabled-button .button-text {
  color: #00000099;
  font-size: 14px;
  font-weight: 400;
}

.disabled-button .upload-icon.disabled,
.disabled-button .download-icon.disabled {
  color: #00000099;
  font-size: 18px;
}

.active-button {
  display: flex;
  justify-content: space-between;
  align-items: center;
  height: 100%;
  padding: 12px;
  border: 1px solid #00000099;
  border-radius: 6px;
  background-color: #F3F3F3;
  cursor: pointer;
}

.active-button .button-text {
  color: #000000E5;
  font-size: 14px;
  font-weight: 400;
}

.active-button .upload-icon.active,
.active-button .download-icon.active {
  color: #1C69D4;
  font-size: 18px;
}

.uploaded-button {
  display: flex;
  justify-content: space-between;
  align-items: center;
  height: 100%;
  padding: 12px;
  border: 1px solid #00000099;
  border-radius: 6px;
  background-color: #F3F3F3;
}

.uploaded-button-with-title {
  display: flex;
  flex-direction: column;
  height: 100%;
  padding: 8px 12px;
  border: 1px solid #00000099;
  border-radius: 6px;
  background-color: #F3F3F3;
  position: relative;
}

.button-title {
  color: #000000E5;
  font-size: 14px;
  font-weight: 400;
  margin-bottom: 4px;
}

.button-title-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 4px;
}

.certificate-data-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.certificate-info-inline {
  display: flex;
  align-items: center;
  gap: 4px;
}

.surrender-style {
  position: relative;
}

.surrender-upload-container {
  position: absolute;
  right: 12px;
  top: 50%;
  transform: translateY(-50%);
}

.surrender-time {
  color: #909399;
  font-size: 12px;
  text-align: left;
}

.file-info {
  display: flex;
  align-items: center;
  margin-top: 4px;
  padding-right: 30px;
}

.file-name {
  color: #909399;
  font-size: 12px;
  flex: 1;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.file-name.clickable {
  text-decoration: underline;
  cursor: pointer;
}

.file-name.clickable:hover {
  color: #1C69D4;
}

.upload-icon.reupload {
  color: #D54941;
  font-size: 18px;
  cursor: pointer;
  flex-shrink: 0;
}

.upload-icon.reupload-right {
  color: #D54941;
  font-size: 18px;
  cursor: pointer;
  position: absolute;
  right: 12px;
  top: 50%;
  transform: translateY(-50%);
}

.upload-icon.center {
  align-self: center;
  margin-top: auto;
  margin-bottom: auto;
}

.certificate-count {
  color: #00000099;
  font-size: 13px;
  font-weight: 290;
}

.data-icon {
  color: #00000099;
  font-size: 12px;
}

.upload-time {
  color: #909399;
  font-size: 12px;
}

.uploaded-button .upload-icon.active,
.uploaded-button-with-title .upload-icon.active {
  color: #1C69D4;
  font-size: 18px;
  cursor: pointer;
  flex-shrink: 0;
}

/* loading状态图标 */
.el-icon-loading {
  animation: rotating 2s linear infinite;
  color: #1C69D4;
}

@keyframes rotating {
  0% {
    transform: rotate(0deg);
  }

  100% {
    transform: rotate(360deg);
  }
}

.dialog-footer {
  text-align: right;
}

/deep/ .el-dialog {
  margin-top: 8vh !important;
}

/deep/ .el-dialog__header {
  padding: 15px 20px 0px 20px;
}

/deep/ .el-dialog__body {
  padding: 15px 20px;
  max-height: 75vh;
  overflow-y: auto;
}

/deep/ .el-dialog__footer {
  padding: 10px 20px 15px;
}

/deep/ .el-tag--success {
  background-color: #67C23A;
  border-color: #67C23A;
  color: white;
}

/deep/ .el-tag--warning {
  background-color: #E6A23C;
  border-color: #E6A23C;
  color: white;
}

/deep/ .el-tag--info {
  background-color: #909399;
  border-color: #909399;
  color: white;
}
</style>