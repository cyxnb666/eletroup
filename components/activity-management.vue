<template>
  <el-dialog :visible.sync="dialogVisible" width="700px" :show-close="false" :close-on-click-modal="false"
    @close="handleClose">
    <div slot="title" class="title">活动管理</div>

    <div class="activity-content" v-loading="loading">
      <h3>{{ activityDisplayName }}</h3>

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

      <div class="info-line">
        <div class="info-label">标签清单：</div>
        <div class="info-value">
          <template v-if="renovationFiles.length > 0">
            <div class="file-info-wrapper">
              <div class="file-timestamp" v-if="renovationUploadTime">
                <div class="timestamp-line">
                  {{ renovationUploadTime }}
                  <a href="javascript:void(0)" class="download-link" @click="downloadFile('renovation')">下载文件</a>
                  <a href="javascript:void(0)" class="delete-link" @click="deleteFile('renovation')">删除</a>
                </div>
                <div v-if="renovationErrorMsg" class="error-message">
                  * {{ renovationErrorMsg }}
                </div>
              </div>
              <div class="file-list scrollable-file-list">
                <div v-for="(fileName, index) in renovationFiles" :key="index" class="file-item">
                  {{ fileName }}
                </div>
              </div>
            </div>
          </template>
          <template v-else-if="renovationLoading">
            <div class="loading-wrapper">
              <i class="el-icon-loading"></i> 处理中...
            </div>
          </template>
          <template v-else>
            <div class="upload-container">
              <el-upload class="upload-button" action="#" :http-request="handleRenovationUpload" :show-file-list="false"
                :multiple="true" accept=".xlsx" :limit="10" :file-list="[]">
                <el-button icon="el-icon-upload2">上传文件</el-button>
              </el-upload>
              <div class="upload-tip">* 可同时选择多个文件，最多10个文件。</div>
            </div>
          </template>
        </div>
      </div>

      <div class="info-line">
        <div class="info-label">终审合格：</div>
        <div class="info-value">
          <template v-if="finalFiles.length > 0">
            <div class="file-info-wrapper">
              <div class="file-timestamp" v-if="finalUploadTime">
                {{ finalUploadTime }}
                <a href="javascript:void(0)" class="download-link" @click="downloadFile('final')">下载文件</a>
                <el-upload class="inline-upload" action="#" :http-request="handleFinalUpload" :show-file-list="false"
                  accept=".xlsx" :disabled="!isActivityInvalid">
                  <a href="javascript:void(0)" class="upload-link"
                    :class="{ 'disabled-link': !isActivityInvalid }">重新上传</a>
                </el-upload>
              </div>
              <div class="file-list">
                <div v-for="(fileName, index) in finalFiles" :key="index" class="file-item">
                  {{ fileName }}
                </div>
              </div>
            </div>
          </template>
          <template v-else-if="finalLoading">
            <div class="loading-wrapper">
              <i class="el-icon-loading"></i> 处理中...
            </div>
          </template>
          <template v-else>
            <div class="upload-container">
              <el-upload class="upload-button" action="#" :http-request="handleFinalUpload" :show-file-list="false"
                :multiple="false" accept=".xlsx" :disabled="!isActivityInvalid" :file-list="[]" ref="finalUpload">
                <el-button icon="el-icon-upload2" :disabled="!isActivityInvalid">上传文件</el-button>
              </el-upload>
              <div class="upload-tip" v-if="!isActivityInvalid">* 只有活动状态为"无效"时才能上传终审合格文件。</div>
            </div>
          </template>
        </div>
      </div>
    </div>

    <span slot="footer" class="dialog-footer">
      <el-button type="primary" @click="dialogVisible = false">关闭</el-button>
    </span>
  </el-dialog>
</template>

<script>
import { uploadToOss } from '@/utils/commonApi'

export default {
  name: 'ActivityManagement',
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
    }
  },
  data() {
    return {
      dialogVisible: false,
      loading: false,
      currentActivity: null,
      renovationFiles: [],
      finalFiles: [],
      renovationLoading: false,
      finalLoading: false,
      renovationUploadTime: '',
      finalUploadTime: '',
      pendingRenovationFiles: [], // 用于收集待处理的文件
      uploadTimer: null, // 上传定时器
      renovationDownloadUrl: '',
      finalDownloadUrl: '',
      renovationErrorMsg: '',
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
    isActivityInvalid() {
      return this.currentActivity && this.currentActivity.overdueStatus !== "0";
    },
    activityTimeRange() {
      if (!this.currentActivity || !this.currentActivity.startTime) {
        return '未知';
      }

      // 格式化开始时间
      const startDate = new Date(this.currentActivity.startTime);
      const startYear = startDate.getFullYear();
      const startMonth = String(startDate.getMonth() + 1).padStart(2, '0');
      const startDay = String(startDate.getDate()).padStart(2, '0');
      const formattedStartDate = `${startYear}年${startMonth}月${startDay}日`;

      // 检查是否有结束时间
      if (this.currentActivity.endTime) {
        // 格式化结束时间
        const endDate = new Date(this.currentActivity.endTime);
        const endYear = endDate.getFullYear();
        const endMonth = String(endDate.getMonth() + 1).padStart(2, '0');
        const endDay = String(endDate.getDate()).padStart(2, '0');
        const formattedEndDate = `${endYear}年${endMonth}月${endDay}日`;

        return `${formattedStartDate} - ${formattedEndDate}`;
      } else {
        // 如果没有结束时间，则显示"永久"
        return `${formattedStartDate} - 永久`;
      }
    }
  },
  watch: {
    visible(val) {
      this.dialogVisible = val;
      if (val && this.activeId) {
        this.fetchActivityData();
        this.fetchActivityFiles();
      }
    },
    dialogVisible(val) {
      if (!val) {
        this.$emit('update:visible', false);
      }
    },
    activeId(val) {
      if (val && this.dialogVisible) {
        this.fetchActivityData();
        this.fetchActivityFiles();
      }
    }
  },
  methods: {
    handleClose() {
      this.$emit('update:visible', false);
    },

    fetchActivityData() {
      if (!this.activeId) return;

      this.loading = true;
      this.$https('/activityPolicy/getActivityPolicyTypeList', {
        body: {}
      }).then(response => {
        if (response && response.body) {
          const activities = Array.isArray(response.body) ? response.body : [response.body];
          this.currentActivity = activities.find(item => item.activeId === this.activeId) || null;
        }
      }).catch(error => {
        console.error('获取活动数据失败:', error);
      }).finally(() => {
        this.loading = false;
      });
    },

    fetchActivityFiles() {
      if (!this.activeId) return;

      this.loading = true;
      this.renovationFiles = [];
      this.finalFiles = [];
      this.renovationUploadTime = '';
      this.finalUploadTime = '';
      this.renovationDownloadUrl = '';
      this.finalDownloadUrl = '';
      this.renovationErrorMsg = '';

      this.$https('/eventDashboard/getUpToDateActivityFiles', {
        body: { activeId: this.activeId }
      }).then(response => {
        if (response && response.body) {
          const fileList = response.body;

          // 处理标签清单文件
          const renovationFileData = fileList.filter(file => file.fileType === "1");
          if (renovationFileData.length > 0) {
            // 从时间降序排序，取最新的时间
            renovationFileData.sort((a, b) => new Date(b.updateTime) - new Date(a.updateTime));
            this.renovationUploadTime = this.formatDate(new Date(renovationFileData[0].updateTime));

            // 直接使用fileNames数组
            this.renovationFiles = renovationFileData[0].fileNames || [];

            // 保存下载URL
            this.renovationDownloadUrl = renovationFileData[0].handFileUrl || '';

            // 保存错误信息
            this.renovationErrorMsg = renovationFileData[0].errorMsg || '';
          }

          // 处理终审合格文件
          const finalFileData = fileList.filter(file => file.fileType === "2");
          if (finalFileData.length > 0) {
            // 从时间降序排序，取最新的时间
            finalFileData.sort((a, b) => new Date(b.updateTime) - new Date(a.updateTime));
            this.finalUploadTime = this.formatDate(new Date(finalFileData[0].updateTime));

            // 直接使用fileNames数组
            this.finalFiles = finalFileData[0].fileNames || [];

            // 保存下载URL
            this.finalDownloadUrl = finalFileData[0].handFileUrl || '';
          }
        }
      }).catch(error => {
        console.error('获取活动文件信息失败:', error);
        this.$message.error('获取活动文件信息失败');
      }).finally(() => {
        this.loading = false;
      });
    },

    async handleRenovationUpload(options) {
      if (!this.activeId) {
        this.$message.error('活动ID不存在，无法上传文件');
        return;
      }

      // 确保处理的是文件
      const file = options.file.raw || options.file;

      // 检查文件类型
      if (!file.name.toLowerCase().endsWith('.xlsx')) {
        this.$message.error('只能上传xlsx格式的文件');
        return;
      }

      // 添加到待处理文件列表
      this.pendingRenovationFiles.push(file);

      // 如果已经有定时器，先清除
      if (this.uploadTimer) {
        clearTimeout(this.uploadTimer);
      }

      // 设置loading状态
      this.renovationLoading = true;

      // 设置新的定时器，延迟处理以收集所有文件
      this.uploadTimer = setTimeout(async () => {
        try {
          // 上传所有文件到OSS
          const uploadPromises = this.pendingRenovationFiles.map(file => uploadToOss({ file }));
          const uploadResults = await Promise.all(uploadPromises);

          // 准备fileFormList数组
          const fileFormList = uploadResults.map((result, index) => ({
            fileUrl: result.fileUrl.replace(/^https?:\/\/[^\/]+\//, ''),
            fileName: this.pendingRenovationFiles[index].name
          }));

          const response = await this.$https('/eventDashboard/uploadActivityFile', {
            body: {
              activeId: this.activeId,
              fileType: "1",  // 1修转续文件
              fileFormList: fileFormList
            }
          });

          if (response && response.header && response.header.code === "10000") {
            this.$message.success('上传成功');

            // 重新获取最新的文件数据
            this.fetchActivityFiles();
          } else {
            // this.$message.error('上传失败: ' + (response.header.message || '未知错误'));
          }
        } catch (error) {
          console.error('上传失败:', error);
          this.$message.error('上传失败: ' + (error.message || '请重试'));
        } finally {
          this.renovationLoading = false;
          this.pendingRenovationFiles = []; // 清空待处理文件
          this.uploadTimer = null;
        }
      }, 100); // 短暂延迟以收集所有文件
    },

    async handleFinalUpload(options) {
      if (!this.activeId) {
        this.$message.error('活动ID不存在，无法上传文件');
        return;
      }

      if (!this.isActivityInvalid) {
        this.$message.warning('只有活动状态为"无效"时才能上传终审合格文件');
        return;
      }

      const file = options.file;

      // 检查文件类型
      if (!file.name.toLowerCase().endsWith('.xlsx')) {
        this.$message.error('只能上传xlsx格式的文件');
        return;
      }

      this.finalLoading = true;

      try {
        // 上传文件到OSS
        const ossResult = await uploadToOss({ file });

        // 获取文件URL，移除域名部分
        const fileUrl = ossResult.fileUrl.replace(/^https?:\/\/[^\/]+\//, '');
        const requestBody = {
          activeId: this.activeId,
          fileType: "2",  // 2终审合格文件
          fileFormList: [{
            fileUrl: fileUrl,
            fileName: file.name
          }]
        };

        const response = await this.$https('/eventDashboard/uploadActivityFile', {
          body: requestBody
        });

        if (response && response.header && response.header.code === "10000") {
          this.$message.success('上传成功');

          // 重新获取最新的文件数据
          this.fetchActivityFiles();
        } else {
          // this.$message.error('上传失败: ' + (response.header.message || '未知错误'));
        }
      } catch (error) {
        console.error('上传失败:', error);
        this.$message.error('上传失败: ' + (error.message || '请重试'));
      } finally {
        this.finalLoading = false;
      }
    },

    deleteFile(type) {
      if (!this.activeId) {
        this.$message.warning('没有活动ID，无法删除文件');
        return;
      }

      const fileType = type === 'renovation' ? "1" : "2";
      const fileTypeName = type === 'renovation' ? '标签清单' : '终审合格文件';

      this.$confirm(`确定要删除${fileTypeName}吗?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.loading = true;

        this.$https('/eventDashboard/deleteActivityFile', {
          body: {
            activeId: this.activeId,
            fileType: fileType
          }
        }).then(res => {
          if (res && res.header && res.header.code === "10000") {
            // 删除成功
            // if (type === 'renovation') {
            //   this.renovationFiles = [];
            //   this.renovationUploadTime = '';
            // } else {
            //   this.finalFiles = [];
            //   this.finalUploadTime = '';
            // }
            this.$message.success('删除成功');

            // 重新获取最新的文件数据
            this.fetchActivityFiles();
          } else {
            // this.$message.error('删除失败: ' + (res.header.message || '未知错误'));
          }
        }).catch(error => {
          console.error('删除文件失败:', error);
          this.$message.error('删除文件失败，请重试');
        }).finally(() => {
          this.loading = false;
        });
      }).catch(() => {
      });
    },

    downloadFile(type) {
      if (type === 'renovation' && this.renovationDownloadUrl) {
        this.$message.info('开始下载标签清单文件');
        this.triggerDirectDownload(this.renovationDownloadUrl, '标签清单');
      } else if (type === 'final' && this.finalDownloadUrl) {
        this.$message.info('开始下载终审合格文件');
        this.triggerDirectDownload(this.finalDownloadUrl, '终审合格文件');
      } else {
        this.$message.warning('没有可下载的文件');
      }
    },

    triggerDirectDownload(url, suggestedName) {
      const link = document.createElement('a');
      link.href = url;
      link.download = suggestedName || '';
      link.style.display = 'none';
      document.body.appendChild(link);
      link.click();

      setTimeout(() => {
        document.body.removeChild(link);
      }, 100);
    },

    formatDate(date) {
      if (!date) return '';

      try {
        const year = date.getFullYear();
        const month = String(date.getMonth() + 1).padStart(2, '0');
        const day = String(date.getDate()).padStart(2, '0');
        const hours = String(date.getHours()).padStart(2, '0');
        const minutes = String(date.getMinutes()).padStart(2, '0');

        return `${year}/${month}/${day} ${hours}:${minutes}`;
      } catch (e) {
        console.error('时间格式化错误:', e);
        return '';
      }
    }
  }
};
</script>

<style scoped>
.title {
  font-weight: bold;
}

.activity-content {
  padding: 0;
}

h3 {
  margin: 0 0 20px;
  font-size: 16px;
  border-bottom: 1px solid #EBEEF5;
  padding-bottom: 15px;
}

.info-line {
  display: flex;
  margin: 15px 0;
  align-items: flex-start;
}

.info-label {
  min-width: 100px;
  color: #606266;
  padding-top: 2px;
}

.info-value {
  flex: 1;
  display: flex;
  align-items: center;
  min-height: 32px;
}

.file-info-wrapper {
  width: 100%;
}

.timestamp-line {
  display: flex;
  align-items: center;
  flex-wrap: nowrap;
}

.error-message {
  color: #D54941;
  font-size: 14px;
  margin: 4px 0;
  line-height: 1.4;
}

.file-timestamp {
  font-size: 14px;
  color: #606266;
  margin-bottom: 10px;
  border-bottom: 1px solid #EBEEF5;
  padding-bottom: 5px;
}

.download-link,
.upload-link,
.delete-link {
  color: #409EFF;
  text-decoration: none;
  margin-left: 15px;
  cursor: pointer;
  white-space: nowrap;
}

.delete-link {
  color: #D54941;
}

.file-list {
  width: 100%;
}

.file-item {
  padding: 5px 0;
  font-size: 14px;
  color: #909399;
}

.loading-wrapper {
  display: flex;
  align-items: center;
  color: #909399;
  height: 32px;
}

.upload-container {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}

.upload-button {
  margin-bottom: 5px;
}

.upload-tip {
  font-size: 12px;
  color: #909399;
  line-height: 1.2;
}

.dialog-footer {
  text-align: right;
}

.inline-upload {
  display: inline-block;
}

.disabled-link {
  color: #C0C4CC;
  cursor: not-allowed;
}

.scrollable-file-list {
  max-height: 150px;
  overflow-y: auto;
}

/deep/ .el-button--primary {
  background-color: #409EFF;
  border-color: #409EFF;
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