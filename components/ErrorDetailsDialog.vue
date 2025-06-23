<template>
    <el-dialog title="识别错误" :visible.sync="visible" :close-on-click-modal="false" :show-close="false" width="55%"
        class="error-details-dialog" append-to-body>
        <div class="error-table-container">
            <el-table :data="errorData" border stripe height="410px" element-loading-text="拼命加载中"
                :header-cell-style="{ background: '#EFEFF0', color: '#252628' }" v-loading="loading">
                <el-table-column prop="fileName" label="文件名称" align="center"></el-table-column>
                <!-- <el-table-column v-if="showFileType" label="文件类型" align="center">
                    <template slot-scope="scope">
                        {{ getFileTypeText(scope.row.fileType) }}
                    </template>
                </el-table-column> -->
                <el-table-column label="文件大小" align="center">
                    <template slot-scope="scope">
                        {{ formatFileSize(scope.row.fileSize) }}
                    </template>
                </el-table-column>
            </el-table>
        </div>
        <div slot="footer" class="dialog-footer">
            <el-button @click="close" type="primary">关闭</el-button>
        </div>
    </el-dialog>
</template>

<script>
export default {
    name: 'ErrorDetailsDialog',
    props: {
        visible: {
            type: Boolean,
            default: false
        },
        batchNo: {
            type: String,
            default: ''
        },
        showFileType: {
            type: Boolean,
            default: false
        }
    },
    data() {
        return {
            errorData: [],
            loading: false
        };
    },
    watch: {
        visible(val) {
            if (val && this.batchNo) {
                this.fetchErrorDetails();
            }
        },
        batchNo(val) {
            if (this.visible && val) {
                this.fetchErrorDetails();
            }
        }
    },
    methods: {
        formatFileSize(bytes) {
            if (bytes === null || bytes === undefined) return '';
            return (bytes / 1024).toFixed(2) + ' KB';
        },
        close() {
            this.$emit('update:visible', false);
        },
        fetchErrorDetails() {
            if (!this.batchNo) return;

            this.loading = true;
            
            // 根据是否显示文件类型来决定调用哪个接口
            const apiUrl = this.showFileType 
                ? '/ncdController/getNcdFileBatchErrorHistorysByBatchNo' 
                : '/activityPolicy/getFileBatchErrorHistorysByBatchNo';
            
            this.$https(apiUrl, {
                body: {
                    batchNo: this.batchNo
                }
            }).then(response => {
                if (response && response.body) {
                    this.errorData = Array.isArray(response.body) ? response.body : [response.body];
                } else {
                    this.errorData = [];
                }
            }).catch(error => {
                console.error('获取错误详情失败:', error);
                this.$message.error('获取错误详情失败');
                this.errorData = [];
            }).finally(() => {
                this.loading = false;
            });
        },
        getFileTypeText(fileType) {
            const fileTypeMap = {
                'policy': '商业险保单',
                'invoice': '购车发票',
                '1': '商业险保单',
                '2': '购车发票'
            };
            return fileTypeMap[fileType] || fileType || '-';
        }
    }
}
</script>

<style scoped>
.error-details-dialog {
    display: flex;
    flex-direction: column;
}

.error-table-container {
    margin-bottom: 20px;
}

.dialog-footer {
    text-align: right;
}

:deep(.el-dialog__body) {
    max-height: 500px;
    overflow-y: auto;
}
</style>