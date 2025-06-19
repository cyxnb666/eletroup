<template>
    <el-dialog title="上传历史" :visible.sync="visible" :show-close="false" :close-on-click-modal="false" width="55%"
        class="upload-history-dialog">
        <div class="history-table-container">
            <el-table :data="historyData" border stripe height="410px" element-loading-text="拼命加载中"
                :header-cell-style="{ background: '#EFEFF0', color: '#252628' }" v-loading="loading">
                <el-table-column prop="createTime" label="上传时间" align="center"></el-table-column>
                <el-table-column prop="batchTotal" label="文件数量" align="center"></el-table-column>
                <el-table-column v-if="showFileType" label="文件类型" align="center">
                    <template slot-scope="scope">
                        {{ getFileTypeText(scope.row.fileType) }}
                    </template>
                </el-table-column>
                <el-table-column prop="createByName" label="上传人" align="center"></el-table-column>
                <el-table-column label="完成识别" align="center">
                    <template slot-scope="scope">
                        {{ scope.row.identifyActual }} / {{ scope.row.batchTotal }}
                    </template>
                </el-table-column>
                <el-table-column prop="identifyErrorActual" label="识别错误" align="center">
                    <template slot-scope="scope">
                        <span v-if="scope.row.identifyErrorActual > 0" class="error-count clickable"
                            @click="showErrorDetails(scope.row)">
                            {{ scope.row.identifyErrorActual }}
                        </span>
                        <span v-else>{{ scope.row.identifyErrorActual }}</span>
                    </template>
                </el-table-column>
            </el-table>
        </div>
        <div class="pagination-container">
            <el-pagination @size-change="handleSizeChange" background @current-change="handleCurrentChange"
                :current-page="currentPage" :page-sizes="[10, 20, 50, 100]" :page-size="pageSize"
                layout="total, sizes, prev, pager, next, jumper" :total="total">
            </el-pagination>
        </div>
        <div slot="footer" class="dialog-footer">
            <el-button @click="refreshData" plain icon="el-icon-refresh">刷新</el-button>
            <el-button @click="close" type="primary">关闭</el-button>
        </div>
        <error-details-dialog :visible.sync="errorDialogVisible" :batch-no="selectedBatchNo" :show-file-type="showFileType" append-to-body>
        </error-details-dialog>
    </el-dialog>
</template>

<script>
import ErrorDetailsDialog from './ErrorDetailsDialog.vue';

export default {
    name: 'UploadHistoryDialog',
    components: {
        ErrorDetailsDialog
    },
    props: {
        visible: {
            type: Boolean,
            default: false
        },
        showFileType: {
            type: Boolean,
            default: false
        }
    },
    data() {
        return {
            historyData: [],
            loading: false,
            currentPage: 1,
            pageSize: 10,
            total: 0,
            errorDialogVisible: false,
            selectedBatchNo: ''
        };
    },
    watch: {
        visible(val) {
            if (val) {
                this.fetchData();
            }
        }
    },
    methods: {
        close() {
            this.$emit('update:visible', false);
        },
        refreshData() {
            this.fetchData();
        },
        handleSizeChange(val) {
            this.pageSize = val;
            this.fetchData();
        },
        handleCurrentChange(val) {
            this.currentPage = val;
            this.fetchData();
        },
        fetchData() {
            this.loading = true;
            
            // 根据是否显示文件类型来决定调用哪个API
            const apiUrl = this.showFileType 
                ? '/subsidyPolicy/getSubsidyPolicyFileBatchHistorys' 
                : '/activityPolicy/getActivityPolicyFileBatchHistorys';
            
            this.$https(apiUrl, {
                body: {
                    page: this.currentPage,
                    rows: this.pageSize
                }
            }).then(response => {
                if (response && response.body) {
                    this.historyData = response.body.rows || [];
                    this.total = response.body.total || 0;
                } else {
                    this.historyData = [];
                    this.total = 0;
                }
            }).catch(error => {
                console.error('获取上传历史失败:', error);
                this.$message.error('获取上传历史失败');
                this.historyData = [];
                this.total = 0;
            }).finally(() => {
                this.loading = false;
            });
        },
        showErrorDetails(row) {
            // 获取批次号
            const batchNo = row.batchNo;
            if (batchNo) {
                this.selectedBatchNo = batchNo;
                this.errorDialogVisible = true;
            } else {
                this.$message.warning('无法获取批次信息，无法查看详情');
            }
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
.upload-history-dialog {
    display: flex;
    flex-direction: column;
}

.history-table-container {
    margin-bottom: 20px;
}

.pagination-container {
    margin: 15px 0;
    text-align: right;
}

.dialog-footer {
    text-align: right;
}

:deep(.el-dialog__body) {
    max-height: 500px;
    overflow-y: auto;
}

.error-count.clickable {
    color: #3488ff;
    text-decoration: underline;
    cursor: pointer;
}

.error-count.clickable:hover {
    color: #66b1ff;
}
</style>