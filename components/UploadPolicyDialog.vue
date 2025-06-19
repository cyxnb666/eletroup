<template>
  <el-dialog title="上传保单(PDF)" :visible.sync="dialogVisible" width="600px" :before-close="handleClose"
    :show-close="false" :close-on-click-modal="false" class="upload-policy-dialog">
    <div class="dialog-content">
      <!-- 活动名称 -->
      <div class="activity-name">{{ activityName }}</div>

      <!-- 上传区域 -->
      <div class="upload-container">
        <div class="upload-button-area" v-if="!isUploading && !uploadComplete">
          <el-button class="select-file-btn" size="medium" @click="triggerUpload">选择文件</el-button>
          <span class="upload-tips">支持批量上传文件，文件格式仅限PDF，最多只能上传 30 份文件</span>
        </div>

        <div class="upload-button-area" v-if="uploadComplete">
          <span class="upload-success-tip">* 已成功上传，点击下方"关闭"按钮，打开"上传历史"查看保单识别进度。</span>
        </div>

        <div class="file-container" :class="{ 'drag-over': isDragging && !uploadComplete }"
          @dragenter="!uploadComplete && handleDragEnter($event)"
          @dragleave="!uploadComplete && handleDragLeave($event)" @dragover="!uploadComplete && handleDragOver($event)"
          @drop="!uploadComplete && handleDrop($event)">
          <input ref="fileInput" type="file" multiple accept=".pdf" style="display: none" @change="handleFileSelect">

          <div v-if="fileList.length === 0 && !uploadComplete" class="empty-upload-area">
            <p class="drag-text">点击上方"选择文件"或将文件拖拽到此区域</p>
          </div>

          <div v-else class="file-list-container">
            <div v-for="(file, index) in fileList" :key="index" class="file-item-container">
              <div class="file-item">
                <div class="file-icon">
                  <i class="el-icon-document"></i>
                  <span class="file-type">PDF</span>
                </div>
                <span class="file-name">{{ file.name }}</span>

                <div class="file-status">
                  <i class="el-icon-circle-check success-icon" v-if="file.status === 'success'"></i>
                  <template v-else-if="file.status === 'error'">
                    <i class="el-icon-circle-close error-icon"></i>
                    <i class="el-icon-close remove-icon" @click="removeFile(index)" v-if="!uploadComplete"></i>
                  </template>
                  <i class="el-icon-close remove-icon" @click="removeFile(index)"
                    v-else-if="(!file.status || file.status === 'ready') && !isUploading && !uploadComplete"></i>
                </div>
              </div>

              <!-- 进度条占位区域 - 始终存在但内容可变 -->
              <div class="progress-container">
                <el-progress v-if="file.status === 'uploading' || file.status === 'uploaded'"
                  :percentage="file.percentage || 0" :stroke-width="4" class="upload-progress"
                  :status="file.status === 'uploaded' ? 'success' : ''">
                </el-progress>
              </div>
            </div>
          </div>
        </div>
      </div>
      <!-- 协议同意区域 -->
      <div class="agreement-area">
        <div v-if="protocolContent" v-html="protocolContent"></div>

        <div class="agreement-options">
          <el-checkbox v-model="agreementAccepted" :disabled="isUploading || uploadComplete">
            客户已完全同意《宝马个人信息保护政策》中的全部内容
          </el-checkbox>
        </div>
      </div>
    </div>

    <span slot="footer" class="dialog-footer">
      <el-button @click="handleCancel" v-if="!isUploading && !uploadComplete">取消</el-button>
      <el-button type="primary" :disabled="isUploadButtonDisabled" @click="handleUpload" :loading="isUploading"
        v-if="!uploadComplete">
        {{ isUploading ? '上传中' : '点击上传' }}
      </el-button>
      <el-button type="primary" @click="handleClose" v-show="shouldShowCloseButton">关闭</el-button>
    </span>
  </el-dialog>
</template>

<script>
import { uploadToOss } from '@/utils/commonApi'

export default {
  name: 'UploadPolicyDialog',
  props: {
    visible: {
      type: Boolean,
      default: false
    },
    activityName: {  // 这里接收的是活动名称
      type: String,
      default: ''
    },
    activeId: {  // 这是活动ID
      type: String,
      default: ''
    },
    protocolContent: {
      type: String,
      default: ''
    }
  },
  data() {
    return {
      dialogVisible: this.visible,
      fileList: [],
      agreementAccepted: false,
      isDragging: false,
      dragCounter: 0, // 用于处理多层嵌套的拖拽事件
      isUploading: false, // 标记是否正在上传
      uploadComplete: false, // 标记上传是否完成
      uploadResults: [] // 存储上传成功的文件信息
    }
  },
  computed: {
    shouldShowCloseButton() {
      return this.uploadComplete && !this.isUploading;
    },
    isUploadButtonDisabled() {
      return !this.agreementAccepted || this.fileList.length === 0 || this.isUploading;
    },
    // 计算成功上传的文件数量
    successCount() {
      return this.fileList.filter(file => file.status === 'success').length;
    }
  },
  watch: {
    visible(val) {
      this.dialogVisible = val
      if (val) {
        // 重置状态
        this.isUploading = false
        this.uploadComplete = false
        this.fileList = []
        this.agreementAccepted = false
        this.uploadResults = []
      }
    },
    dialogVisible(val) {
      if (!val) {
        this.$emit('update:visible', false)
      }
    }
  },
  methods: {
    // 触发文件选择
    triggerUpload() {
      this.$refs.fileInput.click()
    },

    // 处理文件选择
    handleFileSelect(event) {
      const files = event.target.files
      if (files) {
        for (let i = 0; i < files.length; i++) {
          if (this.fileList.length < 30) {
            // 只接受PDF文件
            if (files[i].type === 'application/pdf' || files[i].name.toLowerCase().endsWith('.pdf')) {
              const file = files[i];
              // 为每个文件添加状态属性
              this.$set(file, 'status', 'ready');
              this.$set(file, 'percentage', 0);
              this.fileList.push(file);
            } else {
              this.$message.error('只能上传PDF文件');
            }
          } else {
            this.$message.warning('最多只能上传30份文件');
            break;
          }
        }
      }
      // 重置input，允许选择相同文件
      this.$refs.fileInput.value = '';
    },

    // 移除文件
    removeFile(index) {
      const file = this.fileList[index];

      if (file.status === 'error') {
        if (file.ossResult) {
          const resultIndex = this.uploadResults.findIndex(result =>
            result.fileId === file.ossResult.id);
          if (resultIndex !== -1) {
            this.uploadResults.splice(resultIndex, 1);
          }
        }
      }

      this.fileList.splice(index, 1);
    },

    // 处理拖拽进入
    handleDragEnter(event) {
      event.preventDefault();
      event.stopPropagation();
      this.dragCounter++;
      this.isDragging = true;
    },

    // 处理拖拽离开
    handleDragLeave(event) {
      event.preventDefault();
      event.stopPropagation();
      this.dragCounter--;
      if (this.dragCounter === 0) {
        this.isDragging = false;
      }
    },

    // 处理拖拽悬停
    handleDragOver(event) {
      event.preventDefault();
      event.stopPropagation();
      this.isDragging = true;
    },

    // 处理文件放置
    handleDrop(event) {
      event.preventDefault();
      event.stopPropagation();
      this.isDragging = false;
      this.dragCounter = 0;

      const files = event.dataTransfer.files;
      for (let i = 0; i < files.length; i++) {
        if (files[i].type === 'application/pdf' || files[i].name.toLowerCase().endsWith('.pdf')) {
          if (this.fileList.length < 30) {
            const file = files[i];
            // 为每个文件添加状态属性
            this.$set(file, 'status', 'ready');
            this.$set(file, 'percentage', 0);
            this.fileList.push(file);
          } else {
            this.$message.warning('最多只能上传30份文件');
            break;
          }
        } else {
          this.$message.error('只能上传PDF文件');
        }
      }
    },

    // 关闭对话框
    handleClose() {
      this.dialogVisible = false;
      if (this.uploadComplete) {
        // 打开上传历史对话框
        this.$nextTick(() => {
          this.$emit('showHistory');
        });
      }
      this.$emit('update:visible', false);
    },

    // 取消上传
    handleCancel() {
      this.dialogVisible = false;
      this.$emit('cancel');
    },

    // 上传文件
    async handleUpload() {
      if (!this.agreementAccepted || this.fileList.length === 0 || this.isUploading) return;

      this.isUploading = true;
      this.uploadResults = [];

      try {
        // 逐个上传文件到OSS
        for (let i = 0; i < this.fileList.length; i++) {
          const file = this.fileList[i];

          // 给每个文件添加唯一ID以便跟踪
          const fileId = `file_${Date.now()}_${i}`;
          this.$set(file, 'id', fileId);
          this.$set(file, 'status', 'uploading');
          this.$set(file, 'percentage', 0);

          // 添加进度条显示的强制更新
          this.$forceUpdate();

          try {
            // 设置上限为85%，留出实际完成的空间
            this.simulateProgress(i, 85);

            // 上传到阿里云OSS，添加进度回调
            const params = {
              file: file,
            };

            const ossResult = await uploadToOss(params);

            // 清除模拟进度的定时器
            if (file.progressInterval) {
              clearInterval(file.progressInterval);
            }

            // 更新为100%完成
            this.$set(file, 'percentage', 100);
            this.$set(file, 'status', 'uploaded');
            this.$set(file, 'ossResult', ossResult);

            // 强制Vue更新视图
            this.$forceUpdate();

            // 存储上传结果
            this.uploadResults.push({
              fileName: file.name,
              fileUrl: ossResult.fileUrl,
              fileId: ossResult.id
            });
          } catch (error) {
            console.error('文件上传失败:', error);

            // 清除模拟进度的定时器
            if (file.progressInterval) {
              clearInterval(file.progressInterval);
            }

            this.$set(file, 'status', 'error');
            this.$message.error(`文件 ${file.name} 上传失败`);
          }
        }

        if (this.uploadResults.length > 0) {
          try {
            const response = await this.saveFilesToBackend();

            if (response && response.header && response.header.code === '10000') {
              this.fileList.forEach(file => {
                if (file.status === 'uploaded') {
                  this.$set(file, 'status', 'success');
                }
              });

              this.uploadComplete = true;
              this.$emit('upload', this.uploadResults);
            } else {
              this.fileList.forEach(file => {
                if (file.status === 'uploaded') {
                  this.$set(file, 'status', 'error');
                }
              });

              this.uploadComplete = false;
            }
          } catch (error) {
            console.error('保存文件信息到后端失败:', error);

            // 如果后端API调用失败，将所有已上传文件标记为错误
            this.fileList.forEach(file => {
              if (file.status === 'uploaded') {
                this.$set(file, 'status', 'error');
              }
            });

            // 将上传完成标志设为false，允许用户重试
            this.uploadComplete = false;
            this.$message.error('保存文件信息失败，请删除错误文件后重试');
          }
        } else {
          this.$message.error('没有文件上传成功');
        }
      } catch (error) {
        console.error('上传过程发生错误:', error);
        this.$message.error('上传过程发生错误');
      } finally {
        this.isUploading = false;

        // 清理所有模拟进度的定时器
        this.fileList.forEach(file => {
          if (file.progressInterval) {
            clearInterval(file.progressInterval);
            this.$delete(file, 'progressInterval');
          }
        });
      }
    },

    // 改进的模拟进度方法，设置上限
    simulateProgress(fileIndex, maxPercent = 90) {
      const file = this.fileList[fileIndex];
      let percentage = 0;
      const interval = setInterval(() => {
        if (percentage < maxPercent && file.status === 'uploading') {
          // 逐渐减慢增长速度，模拟真实上传
          const increment = Math.max(1, Math.floor((maxPercent - percentage) / 10));
          percentage += increment;
          this.$set(file, 'percentage', Math.min(percentage, maxPercent));

          // 强制Vue更新视图
          this.$forceUpdate();
        } else {
          clearInterval(interval);
        }
      }, 500);

      // 存储interval ID
      this.$set(file, 'progressInterval', interval);
    },

    async saveFilesToBackend() {
      try {
        const files = this.uploadResults.map(item => {
          // 移除域名部分，只保留路径
          const urlWithoutDomain = item.fileUrl.replace(/^https?:\/\/[^\/]+\//, '');

          const originalFile = this.fileList.find(file =>
            file.name === item.fileName && file.ossResult && file.ossResult.id === item.fileId);

          return {
            fileName: item.fileName,
            fileUrl: urlWithoutDomain,
            fileSize: originalFile ? originalFile.size : 0 // bytes
          };
        });

        const response = await this.$https('/activityPolicy/saveActivityPolicyFiles', {
          body: {
            activeId: this.activeId || '',
            files: files
          }
        });

        return response;
      } catch (error) {
        console.error('保存文件信息到后端失败:', error);
        throw error;
      }
    },

    // 组件销毁前清理所有interval
    beforeDestroy() {
      this.fileList.forEach(file => {
        if (file.progressInterval) {
          clearInterval(file.progressInterval);
        }
      });
    }
  }
}
</script>

<style scoped>
.upload-policy-dialog>>>.el-dialog__header {
  padding: 15px 20px 0px 20px;
  /* border-bottom: 1px solid #e6e6e6; */
}

.upload-policy-dialog>>>.el-dialog__body {
  padding: 20px;
}

.dialog-content {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.activity-name {
  font-size: 16px;
  color: #00000066;
  /* margin-bottom: 10px; */
}

.upload-button-area {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}

.select-file-btn {
  margin-right: 15px;
}

.upload-tips {
  color: #00000066;
  font-size: 14px;
}

.upload-success-tip {
  color: #1C69D4;
  font-size: 14px;
}


.file-container {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  height: 200px;
  overflow-y: auto;
  background-color: #fafafa;
  transition: border-color 0.3s, background-color 0.3s;
}

.upload-policy-dialog>>>.el-dialog__body {
  padding: 20px;
  overflow: hidden;
}

.upload-complete .file-container {
  border: 1px solid #d9d9d9;
}

.file-item-container {
  position: relative;
}

.file-item {
  display: flex;
  align-items: center;
  padding: 4px 10px;
  border-radius: 4px;
  position: relative;
  margin-bottom: 0;
}

.progress-container {
  height: 12px;
  /* 减小高度 */
  padding: 0 10px;
  margin-top: 0;
  /* 与文件项之间无间距 */
}


.drag-over {
  border-color: #409EFF;
  background-color: rgba(64, 158, 255, 0.06);
}

.empty-upload-area {
  height: 200px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.drag-text {
  font-size: 14px;
  color: #909399;
}

.file-list-container {
  height: 100%;
  padding: 8px;
}

.file-item-wrapper {
  margin-bottom: 12px;
}

.file-item:hover {
  background-color: #f5f7fa;
}

.file-icon {
  position: relative;
  margin-right: 8px;
  display: flex;
  align-items: center;
}

.file-icon i {
  font-size: 20px;
  color: #909399;
}

.file-type {
  position: absolute;
  left: 0;
  bottom: -2px;
  font-size: 8px;
  background-color: #909399;
  color: white;
  padding: 0 2px;
  border-radius: 2px;
}

.file-name {
  flex: 1;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.file-status {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  width: 30px;
}

.remove-icon {
  cursor: pointer;
  color: #909399;
  font-size: 16px;
  padding: 4px;
}

.remove-icon:hover {
  color: #f56c6c;
}

.success-icon {
  color: #67C23A;
  font-size: 18px;
}

.error-icon {
  color: #F56C6C;
  font-size: 18px;
}

.file-progress {
  padding: 0 10px;
  margin-top: 4px;
}

.upload-progress {
  width: 100%;
}

.agreement-area {
  margin-top: 15px;
  border: 1px solid #e6e6e6;
  border-radius: 4px;
  padding: 15px;
}

.agreement-area>>>.agreement-text {
  line-height: 1.6;
  margin-bottom: 15px;
}

.agreement-area>>>.agreement-link {
  color: #409EFF;
  text-decoration: none;
}

.agreement-options {
  margin-top: 10px;
}

.dialog-footer {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
}
</style>