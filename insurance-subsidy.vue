<template>
  <div class="insurance-subsidy-management" v-loading="deleteLoading">
    <!-- 顶部活动选择和上传区域 -->
    <div class="activity-section">
      <div class="activity-name-section">
        <div class="stat-header">
          <span>活动名称<span class="required">*</span></span>
        </div>
        <div class="input-with-button">
          <el-select v-model="activityName" placeholder="请选择" @change="handleActivityChange" no-data-text="无活动">
            <el-option v-for="item in activityList" :key="item.activeId" :label="item.activeName"
              :value="item.activeId"> </el-option>
          </el-select>
          <el-button type="primary" icon="el-icon-upload" @click="showUploadDialog" :disabled="isUploadDisabled"
            v-if="isDealerPermission">
            上传资料(PDF)
          </el-button>
        </div>
      </div>
    </div>

    <!-- 查询条件表单 -->
    <div class="insurance-search">
      <el-form :model="queryForm" class="search-form" ref="queryForm" label-position="top">
        <!-- 有经销商权限的布局 -->
        <div v-if="hasProductManagementCopyPermission">
          <el-row :gutter="20">
            <el-col :span="6">
              <el-form-item label="经销商" prop="deptName">
                <el-select v-model="queryForm.deptName" filterable placeholder="请选择" clearable style="width: 100%">
                  <el-option v-for="item in dealerList" :key="item.value" :label="item.label" :value="item.value">
                  </el-option>
                </el-select>
              </el-form-item>
            </el-col>
            <el-col :span="6">
              <el-form-item label="车架号" prop="frmNo">
                <el-input v-model="queryForm.frmNo" placeholder="请输入" maxlength="17" clearable></el-input>
              </el-form-item>
            </el-col>
            <el-col :span="6">
              <el-form-item label="商业险保单号" prop="vciPolicyNo">
                <el-input v-model="queryForm.vciPolicyNo" placeholder="请输入" maxlength="50" clearable></el-input>
              </el-form-item>
            </el-col>
            <el-col :span="6">
              <el-form-item label="购车发票号" prop="invoiceNo">
                <el-input v-model="queryForm.invoiceNo" placeholder="请输入" maxlength="50" clearable></el-input>
              </el-form-item>
            </el-col>
          </el-row>

          <div v-show="showAllConditions">
            <el-row :gutter="20">
              <el-col :span="6">
                <el-form-item label="保险公司" prop="vciInsId">
                  <el-select v-model="queryForm.vciInsId" placeholder="请选择" clearable filterable style="width: 100%">
                    <el-option v-for="item in insCompanyList" :key="item.value" :label="item.label"
                      :value="item.value"></el-option>
                  </el-select>
                </el-form-item>
              </el-col>
              <el-col :span="6">
                <el-form-item label="更新时间" prop="updateTime">
                  <el-date-picker v-model="queryForm.updateTime" type="daterange" range-separator="至"
                    start-placeholder="开始日期" end-placeholder="结束日期" value-format="yyyy-MM-dd" style="width: 100%">
                  </el-date-picker>
                </el-form-item>
              </el-col>
              <el-col :span="6">
                <el-form-item label="车型" prop="vehicleName">
                  <el-input v-model="queryForm.vehicleName" placeholder="请输入" maxlength="50" clearable></el-input>
                </el-form-item>
              </el-col>
              <el-col :span="6">
                <!-- 占位，保持布局一致 -->
              </el-col>
            </el-row>
          </div>
        </div>

        <!-- 没有经销商权限的布局 -->
        <div v-else>
          <el-row :gutter="20">
            <el-col :span="6">
              <el-form-item label="车架号" prop="frmNo">
                <el-input v-model="queryForm.frmNo" placeholder="请输入" maxlength="17" clearable></el-input>
              </el-form-item>
            </el-col>
            <el-col :span="6">
              <el-form-item label="商业险保单号" prop="vciPolicyNo">
                <el-input v-model="queryForm.vciPolicyNo" placeholder="请输入" maxlength="50" clearable></el-input>
              </el-form-item>
            </el-col>
            <el-col :span="6">
              <el-form-item label="购车发票号" prop="invoiceNo">
                <el-input v-model="queryForm.invoiceNo" placeholder="请输入" maxlength="50" clearable></el-input>
              </el-form-item>
            </el-col>
            <el-col :span="6">
              <el-form-item label="保险公司" prop="vciInsId">
                <el-select v-model="queryForm.vciInsId" placeholder="请选择" clearable filterable style="width: 100%">
                  <el-option v-for="item in insCompanyList" :key="item.value" :label="item.label"
                    :value="item.value"></el-option>
                </el-select>
              </el-form-item>
            </el-col>
          </el-row>

          <div v-show="showAllConditions">
            <el-row :gutter="20">
              <el-col :span="6">
                <el-form-item label="更新时间" prop="updateTime">
                  <el-date-picker v-model="queryForm.updateTime" type="daterange" range-separator="至"
                    start-placeholder="开始日期" end-placeholder="结束日期" value-format="yyyy-MM-dd" style="width: 100%">
                  </el-date-picker>
                </el-form-item>
              </el-col>
              <el-col :span="6">
                <el-form-item label="车型" prop="vehicleName">
                  <el-input v-model="queryForm.vehicleName" placeholder="请输入" maxlength="50" clearable></el-input>
                </el-form-item>
              </el-col>
              <el-col :span="12">
                <!-- 占位，保持布局一致 -->
              </el-col>
            </el-row>
          </div>
        </div>

        <!-- 操作按钮部分 -->
        <div class="operate-condition">
          <span @click="showAllConditions = !showAllConditions">
            {{ showAllConditions ? '收起查询条件' : '更多查询条件' }}
            <i :class="['el-icon-arrow-' + (showAllConditions ? 'up' : 'down')]"></i>
          </span>
        </div>

        <div class="actions-container">
          <el-button type="primary" icon="el-icon-search" @click="search">查询</el-button>
          <el-button plain icon="el-icon-refresh-left" @click="reset">重置</el-button>
          <div class="vertical-divider" v-if="isDealerPermission"></div>
          <el-button plain icon="el-icon-delete" @click="deleteItems" v-if="isDealerPermission"
            :disabled="multipleSelection.length === 0 && !checkedAll">删除</el-button>
          <div class="vertical-divider" v-if="isDealerPermission"></div>
          <el-button plain icon="el-icon-time" @click="showUploadHistory" v-if="isDealerPermission">上传历史</el-button>
          <div class="double-click-tip">
            <img src="@/assets/icon/error-circle.svg" class="error-icon" alt="error-icon" />
            <span>双击数据查看保单详情</span>
          </div>
        </div>
      </el-form>
    </div>

    <!-- 数据表格 -->
    <div class="table-container">
      <el-table ref="dataTable" :data="tableData" element-loading-text="拼命加载中" border stripe :height="tableHeight"
        @selection-change="handleSelectionChange" @row-dblclick="detail"
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
        <el-table-column prop="deptName" label="经销商" align="center"
          v-if="hasProductManagementCopyPermission"></el-table-column>
        <el-table-column prop="frmNo" label="车架号" align="center"></el-table-column>
        <el-table-column prop="vciInsName" label="保险公司" align="center"></el-table-column>
        <el-table-column prop="vciPolicyNo" label="商业险保单号" align="center" width="150"></el-table-column>
        <el-table-column prop="invoiceNo" label="购车发票号" align="center"></el-table-column>
        <el-table-column prop="isElectricVehicle" label="是否纯电车" align="center">
          <template slot-scope="scope">
            {{ scope.row.isElectricVehicle === '1' ? '是' : '否' }}
          </template>
        </el-table-column>
        <el-table-column prop="invoiceDate" label="开票日期" align="center"></el-table-column>
        <el-table-column prop="fstRegNo" label="初登日期" align="center"></el-table-column>
        <el-table-column prop="updateTime" label="更新时间" align="center"></el-table-column>
      </el-table>
    </div>

    <!-- 分页 -->
    <div class="paginationPart">
      <div class="service-contact">
        <span>技术支持热线：</span>
        <img src="@/assets/icon/service.svg" alt="service" class="service-icon" />
        <span class="service-number">400-801-0756</span>
        <img src="@/assets/icon/mail.svg" alt="mail" class="mail-icon" />
        <span class="mail-address">bmw_insurance@zhixunchelian.cn</span>
      </div>
      <p class="statistics">已选{{ checkedAll ? total : multipleSelection.length }}个数据</p>
      <el-pagination background @size-change="handleSizeChange" @current-change="handleCurrentChange"
        :current-page="currentPage" :page-sizes="[10, 20, 30, 50]" :page-size="pageSize"
        layout="total, sizes, prev, pager, next, jumper" :total="total"> </el-pagination>
    </div>

    <!-- 上传对话框 -->
    <upload-material-dialog :visible.sync="uploadDialogVisible" :protocol-content="protocolContent"
      :activity-name="currentActivity ? currentActivity.activeName : ''" :active-id="activityName"
      @cancel="uploadDialogVisible = false" @upload="handleUpload" @showHistory="showUploadHistory" />

    <!-- 上传历史对话框 -->
    <upload-history-dialog :visible.sync="uploadHistoryVisible" :show-file-type="true" />

    <!-- 车辆详情对话框 -->
    <vehicle-detail-dialog :visible.sync="vehicleDetailVisible" :vehicle-data="currentVehicleData || {}"
      :insurance-companies="insCompanyList" :dictionary-data="dictionaryData" @cancel="vehicleDetailVisible = false" />
  </div>
</template>

<script>
import UploadMaterialDialog from './components/UploadMaterialDialog.vue'
import UploadHistoryDialog from './components/UploadHistoryDialog.vue'
import VehicleDetailDialog from './components/VehicleDetailDialog.vue'
import { getInsuranceCompanies } from '@/utils/commonApi'
import { getReportDropdownList } from '@/views/modules/daibu-insurance/daibuche-common.js'

export default {
  name: 'InsuranceSubsidy',
  components: {
    UploadMaterialDialog,
    UploadHistoryDialog,
    VehicleDetailDialog,
  },
  data() {
    return {
      checkedAll: false,
      activityName: '',
      activityList: [], // 存储活动列表
      isActivityExpired: false, // 活动是否过期
      currentActivity: null, // 当前选中的活动详情
      showAllConditions: false,
      uploadDialogVisible: false,
      uploadHistoryVisible: false,
      vehicleDetailVisible: false,
      currentVehicleData: {},
      protocolContent: '', // 协议内容
      insCompanyList: [], // 保险公司列表
      dictionaryData: {
        vhlUsageDicList: [], // 使用性质
        carKindDicList: [], // 机动车种类
        carFuelTypeList: [], // 能源类型
        premiumPolicyList: [], // 个人信息保护政策
        busTypeList: [], // 业务类别
        bmwArcList: [], // 是否购买宝马事故维修补偿产品
        cvrgList: [], // 险种列表
        newEnergyCvrgList: [], // 新能源险种列表
      },
      dealerList: [], // 经销商列表
      queryForm: {
        deptName: '',
        updateTime: [],
        frmNo: '',
        vciPolicyNo: '',
        invoiceNo: '',
        vciInsId: '',
        vehicleName: '',
      },
      tableData: [],
      multipleSelection: [],
      currentPage: 1,
      pageSize: 10,
      total: 0,
      tableLoading: false,
      deleteLoading: false,
    }
  },
  computed: {
    tableHeight() {
      // 如果未展开查询条件，使用基础高度
      if (!this.showAllConditions) {
        return 'calc(100vh - 450px)'
      }

      // 展开查询条件时的高度
      return 'calc(100vh - 510px)'
    },
    isDealerPermission() {
      const permissions = JSON.parse(sessionStorage.getItem('permissions') || '[]');
      // 如果没有厂商、大区、保险公司、集团权限，则认为是经销商权限
      return !permissions.some(permission => [
        'daibu-report-overview-changshang', // 厂商
        'daibu-report-overview-dq',         // 大区
        'daibu-report-overview-ins',        // 保险公司
        'daibu-report-overview-jt'          // 集团
      ].includes(permission));
    },
    isUploadDisabled() {
      return this.isActivityExpired || !this.activityList.length;
    },
    hasProductManagementCopyPermission() {
      const permissions = JSON.parse(sessionStorage.getItem('permissions') || '[]')
      return permissions.some(permission =>
        [
          'daibu-report-overview-changshang', //厂商
          'daibu-report-overview-dq', // 大区
          'daibu-report-overview-ins', // 保险公司
          'daibu-report-overview-jt', // 集团
        ].includes(permission),
      )
    },
  },
  created() {
    this.getActivityList();
    this.getInsuranceCompanies();
    this.getUploadProtocol();
    this.getDictionaryData();
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

      this.fetchSubsidyList();
    },
    reset() {
      this.$refs.queryForm.resetFields();
      this.queryForm.updateTime = [];
    },
    // 获取经销商列表
    getDealerList() {
      getReportDropdownList()
        .then(result => {
          if (result && result.dealerList && Array.isArray(result.dealerList)) {
            this.dealerList = result.dealerList.map(item => ({
              label: item.label,
              value: item.value,
            }))
          } else {
            this.dealerList = []
          }
        })
        .catch(error => {
          console.error('获取经销商列表失败:', error)
          this.$message.error('获取经销商列表失败')
          this.dealerList = []
        })
    },
    // 获取活动列表
    getActivityList() {
      this.$https('/ncdController/getNcdActiveTypeList', {
        body: {}
      }).then(response => {
        if (response) {
          this.activityList = Array.isArray(response.body) ? response.body : (response.body ? [response.body] : []);
          if (this.activityList.length > 0 && !this.activityName) {
            const firstActivity = this.activityList[0];
            this.activityName = firstActivity.activeId;
            this.currentActivity = firstActivity;
            this.isActivityExpired = firstActivity.overdueStatus === '1';
            this.fetchSubsidyList();
          } else if (this.activityList.length === 0) {
            this.activityName = '';
            this.currentActivity = null;
            this.isActivityExpired = false;
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

    // 获取保险公司列表
    async getInsuranceCompanies() {
      try {
        const result = await getInsuranceCompanies()
        if (result && result.length > 0) {
          this.insCompanyList = result.map(item => ({
            label: item.insName,
            value: item.insId,
          }))
        }
      } catch (error) {
        console.error('获取保险公司列表失败:', error)
        this.$message.error('获取保险公司列表失败')
      }
    },

    // 刷新活动数据
    refreshdata() {
      const currentActivityId = this.activityName || '';
      this.getActivityList();
      // 保持当前选中的活动
      if (currentActivityId) {
        this.$nextTick(() => {
          this.activityName = currentActivityId;
        });
      }
    },

    // 处理活动变更
    handleActivityChange(value) {
      const selectedActivity = this.activityList.find(item => item.activeId === value);
      if (selectedActivity) {
        this.currentActivity = selectedActivity;
        // 检查活动是否过期
        this.isActivityExpired = selectedActivity.overdueStatus === '1';

        this.reset();
        this.currentPage = 1;
        this.multipleSelection = [];
        this.checkedAll = false;
        this.fetchSubsidyList();
      }
    },

    // 获取补贴列表数据
    fetchSubsidyList() {
      if (!this.activityName) {
        this.$message.warning('请先选择活动');
        this.tableData = [];
        this.total = 0;
        this.multipleSelection = [];
        this.checkedAll = false;
        this.tableLoading = false;
        return;
      }

      this.tableLoading = true;

      const params = {
        activeId: this.activityName || '',
        deptCodeList: this.queryForm.deptName ? [this.queryForm.deptName] : [],
        updateStartTime: this.queryForm.updateTime && this.queryForm.updateTime[0] ? this.queryForm.updateTime[0] : '',
        updateEndTime: this.queryForm.updateTime && this.queryForm.updateTime[1] ? this.queryForm.updateTime[1] : '',
        frmNo: this.queryForm.frmNo || '',
        vciPolicyNo: this.queryForm.vciPolicyNo || '',
        invoiceNo: this.queryForm.invoiceNo || '',
        vciInsId: this.queryForm.vciInsId || '',
        vehicleName: this.queryForm.vehicleName || '',
        page: this.currentPage,
        rows: this.pageSize,
      };

      this.$https('/ncdController/getNcdList', {
        body: params,
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
          this.$message.error('获取补贴列表失败');
          this.tableData = [];
          this.total = 0;
        }
        this.multipleSelection = [];
        this.checkedAll = false;
      }).catch(error => {
        this.tableLoading = false;
        console.error('获取补贴列表失败:', error);
        this.$message.error('获取补贴列表失败');
        this.tableData = [];
        this.total = 0;
      });
    },

    // 删除
    deleteItems() {
      if (this.multipleSelection.length === 0 && !this.checkedAll) {
        this.$message.warning('请至少选择一条记录');
        return;
      }

      this.$confirm('是否删除选择的保单数据？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      }).then(() => {
        this.deleteLoading = true;

        let params = {};

        if (this.checkedAll) {
          // 勾选"全选"时，传递查询条件参数
          params = {
            activeId: this.activityName || '',
            deptCode: this.queryForm.deptName || '',
            isAllChecked: true,
            updateStartTime: this.queryForm.updateTime && this.queryForm.updateTime[0] ? this.queryForm.updateTime[0] : '',
            updateEndTime: this.queryForm.updateTime && this.queryForm.updateTime[1] ? this.queryForm.updateTime[1] : '',
            frmNo: this.queryForm.frmNo || '',
            vciPolicyNo: this.queryForm.vciPolicyNo || '',
            invoiceNo: this.queryForm.invoiceNo || '',
            vciInsId: this.queryForm.vciInsId || '',
            vehicleName: this.queryForm.vehicleName || '',
          };
        } else {
          // 勾选具体行时，只传递选中行的ID
          params = {
            activeId: this.activityName || '',
            ids: this.multipleSelection.map(item => item.id),
            isAllChecked: false,
          };
        }

        this.$https('/ncdController/deleteNcds', {
          body: params
        }).then(res => {
          if (res && res.header && res.header.code === '10000') {
            this.$message.success('删除成功');
            this.search(); // 刷新表格数据
          }
        }).catch(error => {
          console.error('删除失败:', error);
          this.$message.error('删除失败，请重试');
        }).finally(() => {
          this.deleteLoading = false;
        });
      }).catch(() => { });
    },

    // 获取上传协议内容
    getUploadProtocol() {
      this.$https('/activityPolicy/getActivityPolicyUploadProtocol', {
        body: {}
      }).then(response => {
        if (response && response.body) {
          const content = response.body.replace(/<a\s+(?:[^>]*?\s+)?href=(["'])(.*?)\1/g, '<a target="_blank" href=$1$2$1');
          this.protocolContent = content;
        }
      }).catch(error => {
        console.error('获取协议失败:', error);
      });
    },

    // 显示上传对话框
    showUploadDialog() {
      this.uploadDialogVisible = true;
    },

    // 显示上传历史
    showUploadHistory() {
      this.uploadHistoryVisible = true;
    },

    // 获取字典数据
    getDictionaryData() {
      this.$https('/activityPolicy/getActivityPolicyBaseInfo', {
        body: {}
      }).then(res => {
        if (res && res.body) {
          // 保存所有字典数据
          this.dictionaryData = {
            vhlUsageDicList: res.body.vhlUsageDicList || [],
            carKindDicList: res.body.carKindDicList || [],
            carFuelTypeList: res.body.carFuelTypeList || [],
            premiumPolicyList: res.body.premiumPolicyList || [],
            busTypeList: res.body.busTypeList || [],
            bmwArcList: res.body.bmwArcList || [],
            cvrgList: res.body.cvrgList || [],
            newEnergyCvrgList: res.body.newEnergyCvrgList || [],
          };
        }
      }).catch(error => {
        console.error('获取数据字典失败:', error);
      });
    },

    // 查看详情
    detail(row) {
      const id = row.id;
      this.fetchVehicleDetails(id);
    },

    // 映射险种数据
    mapInsuranceTypes(cvrgList) {
      if (!cvrgList || !cvrgList.length) return []

      return cvrgList.map(item => {
        return {
          name: item.code || '', // 使用险种代码作为name
          type: item.code || '', // 使用险种代码作为type
          amount: item.amount || '',
          premium: item.cvrgPrem || '',
        }
      })
    },

    // 获取车辆详情数据
    fetchVehicleDetails(id) {
      this.loading = true;

      this.$https('/ncdController/getNcdInfo', {
        body: { id },
      }).then(response => {
        if (response && response.body) {
          const data = response.body;
          const cvrgList = data.cvrgList || [];

          this.currentVehicleData = {
            // 标识字段 - 控制显示逻辑
            hasPolicyData: data.policyDataStatus === '1',
            hasInvoiceData: data.carInvoiceDataStatus === '1',

            // 车辆信息
            licensePlate: data.plateNo || '',
            frameNo: data.frmNo || '',
            engineNo: data.engNo || '',
            carModel: data.vehicleName || '',
            registrationDate: data.fstRegNo || '',
            approvedMass: data.ton !== undefined && data.ton !== null ? data.ton.toString() : '',
            seatingCapacity: data.seatNum !== undefined && data.seatNum !== null ? data.seatNum.toString() : '',
            usageNature: data.vhlUsageCode || '',
            vehicleType: data.carKindCode || '',
            energyType: data.carFuelType || '',
            displacement: data.desplacement || '',
            power: data.vhlPower !== undefined && data.vhlPower !== null ? data.vhlPower.toString() : '',

            // 商业险信息
            policyNo: data.vciPolicyNo || '',
            insuranceCompany: data.vciInsId || '',
            insuranceStartDate: data.vciComenceTime || '',
            insuranceEndDate: data.vciTerminateTime || '',
            vehicleDamageAmount: data.vehicleDamageCoverage || '',
            thirdPartyAmount: data.thirdPartyCoverage || '',
            totalPremium: data.vciPremiumTotal !== undefined && data.vciPremiumTotal !== null ? data.vciPremiumTotal.toString() : '',
            feeConfirmTime: data.vciChargeConfirmTime || '',
            policyCreateTime: data.vciPolicyGenTime || '',
            insuranceConfirmTime: data.vciPolicyConfirmTime || '',
            insuranceConfirmCode: data.vciPolicyConfirmCode || '',

            // 投保险种
            insuranceTypes: this.mapInsuranceTypes(cvrgList),

            // 特别约定
            specialTerms: data.vciSpecialAgreement || '',

            // 购车发票
            invoiceNo: data.invoiceNo || '',
            invoiceDate: data.invoiceDate || '',
            totalAmount: data.totalTaxAmount || '',
            sellerName: data.sellerName || '',
            brandModel: data.carInvoiceBrandModel || '',

            // 其他字段
            remark: data.remark || '',
            policyFileUrl: data.vciFileUrl || '',
            invoiceFileUrl: data.carInvoiceFileUrl || '',
            activeUrl: ''
          };

          this.vehicleDetailVisible = true;
        } else {
          this.$message.error('获取车辆信息失败');
        }
      }).catch(error => {
        console.error('获取车辆信息失败:', error);
        this.$message.error('获取车辆信息失败');
      }).finally(() => {
        this.loading = false;
      });
    },

    // 处理上传
    handleUpload(fileList) {
      this.$message({
        message: '上传成功',
        type: 'success',
        duration: 3000,
      });
    },

    // 全选处理方法
    checkedAllOnChange(val) {
      if (val) {
        // 首先清空所有行的选中状态
        this.tableData.forEach(row => {
          // 确保每行有isSelected属性并设为false
          this.$set(row, 'isSelected', false);
        });

        // 全选时，设置所有表格行为选中状态（用于删除和导出按钮）
        this.multipleSelection = [...this.tableData];
      } else {
        // 取消全选时，清空选中数组
        this.multipleSelection = [];
      }
    },

    handleRowSelect(row) {
      // 因为禁用了行复选框，所以这个方法在全选状态下不会被调用
      if (this.multipleSelection.includes(row)) {
        this.multipleSelection = this.multipleSelection.filter(item => item !== row);
      } else {
        this.multipleSelection.push(row);
      }
    },

    handleSelectionChange(val) {
      if (!this.checkedAll) {
        this.multipleSelection = val;
      }
    },

    // 分页大小变化
    handleSizeChange(val) {
      this.pageSize = val;
      this.fetchSubsidyList();
    },

    // 当前页变化
    handleCurrentChange(val) {
      this.currentPage = val;
      this.fetchSubsidyList();
    },
  }
}
</script>

<style scoped>
.insurance-subsidy-management {
  padding: 10px 0px 10px 0px;
}

.activity-section {
  margin-bottom: 20px;
  border: 1px solid #e0e0e0;
  border-radius: 5px;
  padding: 12px;
}

.activity-name-section {
  width: 50%;
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

.required {
  color: #f56c6c;
  margin-left: 2px;
}

.input-with-button {
  display: flex;
  align-items: center;
}

.input-with-button .el-select {
  flex: 1;
  margin-right: 10px;
}

.insurance-search {
  margin-top: 20px;
  height: 100%;
  width: 100%;
}

.search-form {
  margin-bottom: 20px;
  height: 100%;
  width: 100%;
}

.operate-condition {
  cursor: pointer;
  color: #667181;
  font-size: 12px;
  margin: 0px 0px 10px 0px;
}

.operate-condition i {
  margin-left: 5px;
}

.actions-container {
  display: flex;
  align-items: center;
  flex-wrap: nowrap;
}

.vertical-divider {
  display: inline-block;
  width: 1px;
  height: 24px;
  background-color: #ccc;
  margin: 0 10px;
  vertical-align: middle;
}

.double-click-tip {
  display: inline-flex;
  align-items: center;
  margin-left: 15px;
  color: #f6685d;
  font-size: 14px;
  height: 32px;
}

.double-click-tip .error-icon {
  width: 16px;
  height: 16px;
  margin-right: 5px;
}

.table-container {
  margin-bottom: 5px;
}

.statistics {
  margin: 0;
  margin-right: 15px;
  color: #737f91;
  font-size: 14px;
  font-weight: 700;
}

.paginationPart {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 0;
  box-sizing: border-box;
}

.service-contact {
  display: flex;
  align-items: center;
  margin-right: auto;
  color: gray;
}

.service-icon,
.mail-icon {
  width: 16px;
  height: 16px;
  margin-right: 5px;
}

.service-number {
  margin-right: 15px;
}

/deep/ .el-form-item--mini .el-form-item__label {
  line-height: 8px;
}

/deep/ .el-form-item--mini.el-form-item {
  margin-bottom: 10px;
}
</style>