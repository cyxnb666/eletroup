<template>
  <div class="insurance-policy-management" v-loading="exportLoading">
    <!-- 顶部统计卡片 -->
    <div class="statistics-cards" v-loading="refreshLoading">
      <div class="stat-card">
        <div class="stat-header">
          <span>剩余保单额度</span>
          <el-tooltip effect="dark" content="当前选择活动的剩余奖励总额度。" placement="top">
            <i class="el-icon-info"></i>
          </el-tooltip>
        </div>
        <div class="stat-value blue">{{ remainingQuota }}</div>
      </div>

      <div class="stat-card">
        <div class="stat-header">
          <span>保底目标完成量</span>
          <el-tooltip effect="dark" content="当前选择活动需要最少上传的“合格”保单数量，上传超过保底数量要求的“合格”的保单才能获得奖励" placement="top">
            <i class="el-icon-info"></i>
          </el-tooltip>
        </div>
        <div class="stat-value error">{{ completionRate }}</div>
      </div>

      <div class="stat-card">
        <div class="stat-header">
          <span>获得支持保单数</span>
          <el-tooltip effect="dark" content="当前选择活动获得奖励的上传保单数量。" placement="top">
            <i class="el-icon-info"></i>
          </el-tooltip>
        </div>
        <div class="stat-value green">{{ supportedPolicies }}</div>
      </div>

      <div class="activity-name-section">
        <div class="stat-header">
          <span>活动名称<span class="required">*</span></span>
          <el-button size="large" icon="el-icon-refresh" type="text" @click="refreshdata"
            :disabled="!activityList.length">刷新</el-button>
        </div>
        <div class="input-with-button">
          <el-select v-model="activityName" placeholder="请选择" @change="handleActivityChange" no-data-text="无活动">
            <el-option v-for="item in activityList" :key="item.activeId" :label="item.activeName"
              :value="item.activeId">
            </el-option>
          </el-select>
        </div>
      </div>
    </div>

    <!-- 查询条件表单 -->
    <div class="insurance-search">
      <el-form :model="queryForm" class="search-form" ref="queryForm" label-position="top">
        <el-row :gutter="20">
          <el-col :span="6" v-if="hasArea">
            <el-form-item label="区域" prop="region">
              <el-select v-model="queryForm.region" placeholder="请选择" clearable style="width: 100%">
                <el-option v-for="item in regionList" :key="item.value" :label="item.label" :value="item.value">
                </el-option>
              </el-select>
            </el-form-item>
          </el-col>

          <el-col :span="6">
            <el-form-item label="CBU" prop="cbu">
              <el-input v-model="queryForm.cbu" placeholder="请输入" maxlength="10"
                oninput="value=value.replace(/[^\d]/g,'')" clearable></el-input>
            </el-form-item>
          </el-col>

          <el-col :span="6">
            <el-form-item label="CKD" prop="ckd">
              <el-input v-model="queryForm.ckd" placeholder="请输入" maxlength="10"
                oninput="value=value.replace(/[^\d]/g,'')" clearable></el-input>
            </el-form-item>
          </el-col>

          <el-col :span="6" v-if="hasJxs">
            <el-form-item label="经销商" prop="deptName">
              <el-select v-model="queryForm.deptName" filterable placeholder="请选择" clearable style="width: 100%">
                <el-option v-for="item in dealerList" :key="item.value" :label="item.label" :value="item.value">
                </el-option>
              </el-select>
            </el-form-item>
          </el-col>
        </el-row>

        <el-form-item>
          <el-button type="primary" icon="el-icon-search" @click="search">查询</el-button>
          <el-button plain icon="el-icon-refresh-left" @click="reset">重置</el-button>
          <el-button plain icon="el-icon-download" @click="exportDetails"
            :disabled="multipleSelection.length === 0">导出明细</el-button>
          <el-button plain icon="el-icon-setting" @click="openActivityManagement" :disabled="!activityName"
            v-if="hasFactoryPermission">活动管理</el-button>
          <el-button plain icon="el-icon-setting" @click="openArcManagement" 
            :disabled="multipleSelection.length !== 1">ARC管理</el-button>
        </el-form-item>
      </el-form>
    </div>

    <!-- 数据表格 -->
    <div class="table-container">
      <el-table ref="dataTable" :data="tableData" element-loading-text="拼命加载中" border stripe
        height="calc(100vh - 453px)" @selection-change="handleSelectionChange"
        :header-cell-style="{ background: '#EFEFF0', color: '#252628' }" v-loading="tableLoading">
        <el-table-column align="center" width="70">
          <template slot="header" slot-scope="scope">
            <el-checkbox v-model="checkedAll" @change="checkedAllOnChange">全选</el-checkbox>
          </template>
          <template slot-scope="scope">
            <el-checkbox :disabled="checkedAll" @change="handleRowSelect(scope.row)"
              v-model="scope.row.isSelected"></el-checkbox>
          </template>
        </el-table-column>
        <el-table-column prop="region" label="区域" align="center"></el-table-column>
        <el-table-column prop="cbu" label="cbu" align="center"></el-table-column>
        <el-table-column prop="ckd" label="ckd" align="center"></el-table-column>
        <el-table-column prop="companyName" label="经销商" align="center"></el-table-column>
        <el-table-column prop="batchTotal" label="上传保单总数" align="center"></el-table-column>
         <el-table-column prop="repairAndContSupCount" label="标签活动支持保单数" align="center"></el-table-column>
        <el-table-column prop="otherSupCount" label="其它获取支持保单数" align="center"></el-table-column>
        <el-table-column prop="supPolCountActual" label="合计获得支持保单数" align="center"></el-table-column>
      </el-table>
    </div>

    <!-- 分页 -->
    <div class="paginationCont">
      <p class="statistics">
        已选{{ checkedAll ? total : multipleSelection.length }}个数据
      </p>
      <el-pagination background @size-change="handleSizeChange" @current-change="handleCurrentChange"
        :current-page="currentPage" :page-sizes="[10, 20, 30, 50]" :page-size="pageSize"
        layout="total, sizes, prev, pager, next, jumper" :total="total">
      </el-pagination>
    </div>
    <activity-management :visible.sync="activityManagementVisible"
      :activity-name="currentActivity ? currentActivity.activeName : ''" :active-id="activityName"
      :current-activity="currentActivity" />
    <arc-management-dialog :visible.sync="arcManagementVisible"
  :activity-name="currentActivity ? currentActivity.activeName : ''" 
  :active-id="activityName"
  :dept-code="multipleSelection.length > 0 ? multipleSelection[0].deptCode : ''"
  :current-activity="currentActivity" />
  </div>
</template>

<script>
import { getReportDropdownList } from '@/views/modules/daibu-insurance/daibuche-common.js';
import ActivityManagement from './components/activity-management.vue';
import ArcManagementDialog from './components/arc-management-dialog.vue';
export default {
  components: {
    ActivityManagement,
    ArcManagementDialog
  },
  data() {
    return {
      arcManagementVisible: false,
      checkedAll: false,
      activityList: [], // 存储活动列表
      dealerList: [], //存储经销商列表
      regionList: [],
      currentActivity: null, // 当前选中的活动详情
      activityDetails: null, // 第二个接口返回的详细数据
      isActivityExpired: false, // 活动是否过期
      activityName: '',
      queryForm: {
        region: '',
        cbu: '',
        ckd: '',
        deptName: ''
      },
      tableData: [],
      multipleSelection: [],
      currentPage: 1,
      pageSize: 10,
      total: 0,
      tableLoading: false,
      exportLoading: false,
      refreshLoading: false,
      activityManagementVisible: false
    };
  },
  computed: {
    hasFactoryPermission() {
      const permissions = JSON.parse(sessionStorage.getItem('permissions') || '[]');
      return permissions.some(permission => [
        'daibu-report-overview-changshang', //厂商
      ].includes(permission));
    },
    hasArea() {
      const permissions = JSON.parse(sessionStorage.getItem('permissions') || '[]');
      return permissions.some(permission => [
        'daibu-report-overview-changshang',
      ].includes(permission));
    },
    hasJxs() {
      const permissions = JSON.parse(sessionStorage.getItem('permissions') || '[]');
      return permissions.some(permission => [
        'daibu-report-overview-changshang',
        'daibu-report-overview-dq',
        'daibu-report-overview-jt'
      ].includes(permission));
    },
    // 剩余保单额度显示
    remainingQuota() {
      if (!this.activityList.length) return 'N/A';

      if (this.activityDetails && this.activityDetails.policyQuotaActual !== undefined) {
        return this.formatNumber(this.activityDetails.policyQuotaActual);
      }

      return 'N/A';
    },

    // 保底目标完成量显示
    completionRate() {
      if (!this.activityList.length) return 'N/A';

      if (this.activityDetails) {
        const actual = this.formatNumber(this.activityDetails.compQtyActual || 0);
        const target = this.formatNumber(this.currentActivity.compQty || 0);
        return `${actual} / ${target}`;
      }
      return 'N/A';
    },

    // 获得支持保单数显示
    supportedPolicies() {
      if (!this.activityList.length) return 'N/A';

      if (this.activityDetails) {
        return this.formatNumber(this.activityDetails.supPolCountActual || 0);
      }
      return 'N/A';
    },
  },
  created() {
    this.getActivityList();
    this.getDealerList();
  },
  methods: {
    search() {
      this.currentPage = 1;

      // 清空选择状态
      this.multipleSelection = [];
      this.checkedAll = false;

      // 对现有表格数据清除选中状态
      if (this.tableData && this.tableData.length > 0) {
        this.tableData.forEach(row => {
          this.$set(row, 'isSelected', false);
        });
      }

      this.fetchDealerStatistics();
    },
    reset() {
      this.$refs.queryForm.resetFields();
    },
    // 获取活动列表
    getActivityList() {
      this.$https('/activityPolicy/getActivityPolicyTypeList', {
        body: {}
      }).then(response => {
        if (response) {
          this.activityList = Array.isArray(response.body) ? response.body :
            (response.body ? [response.body] : []);

          if (this.activityList.length > 0 && !this.activityName) {
            const firstActivity = this.activityList[0];
            this.activityName = firstActivity.activeId;
            this.currentActivity = firstActivity;
            this.isActivityExpired = firstActivity.overdueStatus === "1";

            // 获取该活动的详细数据
            this.getActivityDetails(firstActivity.activeId);

            // 获取表格数据
            this.fetchDealerStatistics();
          } else if (this.activityList.length === 0) {
            this.activityName = '';
            this.currentActivity = null;
            this.activityDetails = null;

            // 清空表格数据
            this.tableData = [];
            this.total = 0;
          }
        } else {
          this.activityList = [];
        }
      }).catch(error => {
        console.error('获取活动列表失败:', error);
        this.activityList = [];
      });
    },

    // 获取活动详细数据
    getActivityDetails(activeId) {
      if (!activeId) return;

      this.$https('/activityPolicy/getActivityPolicyTotal', {
        body: { activeId }
      }).then(response => {
        if (response) {
          this.activityDetails = response.body;
        } else {
          this.activityDetails = null;
        }
      }).catch(error => {
        console.error('获取活动详情失败:', error);
        this.activityDetails = null;
      });
    },
    refreshdata() {
      const currentActivityId = this.activityName || '';
      this.refreshLoading = true;
      this.$https('/activityPolicy/getActivityPolicyTypeList', {
        body: {}
      }).then(response => {
        if (response) {
          this.activityList = Array.isArray(response.body) ? response.body :
            (response.body ? [response.body] : []);

          if (this.activityList.length > 0) {
            // 找到并保持当前选中的活动
            if (currentActivityId) {
              const currentActivity = this.activityList.find(item => item.activeId === currentActivityId);
              if (currentActivity) {
                this.currentActivity = currentActivity;
                this.isActivityExpired = currentActivity.overdueStatus === "1";

                // 获取该活动的详细统计数据
                this.getActivityDetails(currentActivityId);
              }
            }
          }
        }
      }).catch(error => {
        console.error('获取活动列表失败:', error);
      }).finally(() => {
        this.refreshLoading = false;
      });
    },
    // 处理活动变更
    handleActivityChange(value) {
      // 找到选中的活动详情
      const selectedActivity = this.activityList.find(item => item.activeId === value);
      if (selectedActivity) {
        this.currentActivity = selectedActivity;
        this.isActivityExpired = selectedActivity.overdueStatus === "1";

        // 获取新选中活动的详细数据
        this.getActivityDetails(value);

        // 重置表单和分页
        this.reset();
        this.currentPage = 1;

        // 获取新的表格数据
        this.fetchDealerStatistics();
      }
    },
    fetchDealerStatistics() {
      if (!this.activityName) {
        this.$message.warning('请先选择活动');
        this.tableData = [];
        this.total = 0;
        this.multipleSelection = [];
        this.checkedAll = false;
        this.tableLoading = false; // 确保loading状态被重置
        return; // 如果没有活动名称，直接返回不调用接口
      }

      this.tableLoading = true;

      // 构建查询参数
      const params = {
        activeId: this.activityName || '',
        region: this.queryForm.region || '',
        cbu: this.queryForm.cbu || '',
        ckd: this.queryForm.ckd || '',
        deptCodeList: this.queryForm.deptName ? [this.queryForm.deptName] : [],
        page: this.currentPage,
        rows: this.pageSize
      };

      this.$https('/eventDashboard/getDealerActivityPolicyStatistics', {
        body: params
      }).then(response => {
        this.tableLoading = false;

        if (response) {
          if (response.body) {
            this.tableData = response.body.rows || [];
            this.total = response.body.total || 0;

            this.tableData.forEach(row => {
              this.$set(row, 'isSelected', false);
            });
          } else {
            this.tableData = [];
            this.total = 0;
          }
        } else {
          this.$message.error('获取经销商统计数据失败');
          this.tableData = [];
          this.total = 0;
        }

        // 清除选中状态
        this.multipleSelection = [];
        this.checkedAll = false;
      }).catch(error => {
        this.tableLoading = false;
        console.error('获取经销商统计数据失败:', error);
        this.$message.error('获取经销商统计数据失败');
        this.tableData = [];
        this.total = 0;
      });
    },
    exportDetails() {
      if (this.multipleSelection.length === 0 && !this.checkedAll) {
        this.$message.warning('请至少选择一条记录');
        return;
      }

      this.exportLoading = true;

      let params = {};

      if (this.checkedAll) {
        // 勾选"全选"时，传递查询条件参数
        params = {
          activeId: this.activityName || '',
          region: this.queryForm.region || '',
          cbu: this.queryForm.cbu || '',
          ckd: this.queryForm.ckd || '',
          deptCodeList: this.queryForm.deptName ? [this.queryForm.deptName] : [],
          isAllChecked: true
        };
      } else {
        // 勾选具体行时，只传递选中行的ID
        params = {
          activeId: this.activityName || '',
          deptCodeList: this.multipleSelection.map(item => item.deptCode),
          isAllChecked: false
        };
      }

      this.$https('/eventDashboard/exportDealerActivityPolicyStatistics', {
        body: params
      }).then(res => {
        if (res) {
          const downloadUrl = res.body;
          if (downloadUrl) {
            const downloadLink = document.createElement('a');
            downloadLink.href = downloadUrl;
            // downloadLink.download = '名字-' + new Date().getTime() + '.xlsx';
            // downloadLink.style.display = 'none';
            document.body.appendChild(downloadLink);
            downloadLink.click();

            // 清理DOM
            document.body.removeChild(downloadLink);

            this.$message.success('导出成功');
          } else {
            this.$message.error('导出失败：未获取到下载链接');
          }
        } else {
          this.$message.error(res.msg || '导出失败');
        }
      }).catch(error => {
        console.error('导出失败:', error);
        this.$message.error('导出失败，请重试');
      }).finally(() => {
        this.exportLoading = false;
      });
    },
    handleRowSelect(row) {
      // 因为禁用了行复选框，所以这个方法在全选状态下不会被调用
      if (this.multipleSelection.includes(row)) {
        this.multipleSelection = this.multipleSelection.filter(item => item !== row);
      } else {
        this.multipleSelection.push(row);
      }
    },
    formatNumber(value) {
      // 处理非数值情况
      if (value === null || value === undefined || value === '' || value === 'N/A') {
        return value;
      }

      // 尝试转换为数字
      const num = Number(value);

      // 如果是有效数字，添加千分位分隔符
      if (!isNaN(num)) {
        return num.toLocaleString();
      }

      // 非数字情况直接返回原值
      return value;
    },
    // 添加全选事件处理方法
    checkedAllOnChange(val) {
      if (val) {
        // 首先清空所有行的选中状态
        this.tableData.forEach(row => {
          // 确保每行有isSelected属性并设为false
          this.$set(row, 'isSelected', false);
        });

        // 全选时，设置所有表格行为选中状态（用于删除和导出按钮的逻辑）
        this.multipleSelection = [...this.tableData];
      } else {
        // 取消全选时，清空选中数组
        this.multipleSelection = [];
      }
    },
    handleSelectionChange(val) {
      if (!this.checkedAll) {
        this.multipleSelection = val;
      }
    },
    handleSizeChange(val) {
      this.pageSize = val;
      this.fetchDealerStatistics();
    },
    handleCurrentChange(val) {
      this.currentPage = val;
      this.fetchDealerStatistics();
    },
    getDealerList() {
      getReportDropdownList().then(result => {
        if (result && result.dealerList && Array.isArray(result.dealerList)) {
          this.dealerList = result.dealerList.map(item => ({
            label: item.label,
            value: item.value
          }));
        } else {
          this.dealerList = [];
        }

        // 处理区域数据
        if (result && result.regionList && Array.isArray(result.regionList)) {
          this.regionList = result.regionList.map(item => ({
            label: item.label,
            value: item.value
          }));
        } else {
          this.regionList = [];
        }
      }).catch(error => {
        console.error('获取失败:', error);
        this.$message.error('获取失败');
        this.dealerList = [];
        this.regionList = [];
      });
    },
    openActivityManagement() {
      if (this.activityName) {
        this.activityManagementVisible = true;
      }
    },
    openArcManagement() {
      if (this.multipleSelection.length === 1) {
        this.arcManagementVisible = true;
      } else {
        this.$message.warning('请先选择一条记录');
      }
    }
  }
};
</script>

<style scoped>
.insurance-policy-management {
  padding: 10px 0px 10px 0px;
}

.stat-card {
  flex: 1;
  padding: 15px 20px;
  background-color: #f3f3f3;
  border-radius: 5px;
  box-sizing: border-box;
}

.stat-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 10px;
  color: #606266;
  font-size: 14px;
}

.stat-header i {
  margin-left: auto;
  color: #909399;
  cursor: pointer;
}

.required {
  color: #f56c6c;
  margin-left: 2px;
}

.stat-value {
  font-size: 20px;
  /* font-weight: bold; */
}

.blue {
  color: #1C69D4;
}

.error {
  color: #D54941;
}

.green {
  color: #2BA471;
}

.activity-name-section {
  flex: 3;
  padding: 15px 20px;
  box-sizing: border-box;
}

.insurance-search {
  margin-top: 20px;
  height: 100%;
  width: 100%;
}

.search-form {
  margin-bottom: 20px;
}

.search-form .el-select,
.search-form .el-date-editor {
  width: 100%;
}

.input-with-button {
  display: flex;
  align-items: center;
}

.input-with-button .el-select {
  flex: 1;
}

.table-container {
  /* margin-bottom: 5px; */
}

.statistics {
  margin: 0;
  margin-right: 15px;
  color: #737f91;
  font-size: 14px;
  font-weight: 700;
}

.pagination-container {
  text-align: right;
}

.statistics-cards {
  display: flex;
  margin-bottom: 20px;
  gap: 15px;
  border: 1px solid #e0e0e0;
  border-radius: 5px;
  padding: 12px;
}

/deep/ .el-form-item--mini .el-form-item__label {
  line-height: 8px;
}

/deep/ .el-form-item--mini.el-form-item {
  margin-bottom: 15px;
}
</style>
