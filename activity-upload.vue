<template>
  <div class="insurance-accident-car-management" v-loading="deleteLoading || exportLoading">
    <!-- 顶部统计卡片 -->
    <div class="activity-selection-area">
      <span>活动名称<span class="required">*</span></span>
      <el-select v-model="activityName" placeholder="请选择" @change="handleActivityChange" no-data-text="无活动"
        class="activity-select">
        <el-option v-for="item in activityList" :key="item.activeId" :label="item.activeName" :value="item.activeId">
        </el-option>
      </el-select>
      <el-button type="primary" icon="el-icon-upload" @click="showUploadDialog" :disabled="isUploadDisabled"
        v-if="!shouldHideOperationButtons">
        上传保单(PDF)
      </el-button>
      <el-button size="large" icon="el-icon-refresh" type="text" @click="refreshdata"
        :disabled="!activityList.length">刷新</el-button>
    </div>

    <!-- 统计卡片 -->
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

      <div class="stat-card">
        <div class="stat-header">
          <span>宝驾计划（基础版）单数</span>
          <el-tooltip effect="dark" content="当前选择活动获得奖励的ARC单数" placement="top">
            <i class="el-icon-info"></i>
          </el-tooltip>
        </div>
        <div class="stat-value-with-download">
          <span class="stat-value green downloadable-number" @click="downloadBaoJiaBasic">{{ baoJiaBasicCount }}</span>
          <el-button size="mini" icon="el-icon-download" type="text" @click="downloadBaoJiaBasic"></el-button>
        </div>
      </div>

      <div class="stat-card">
        <div class="stat-header">
          <span>宝驾计划（尊享版）单数</span>
          <el-tooltip effect="dark" content="当前选择活动获得奖励的ARC Premium单数" placement="top">
            <i class="el-icon-info"></i>
          </el-tooltip>
        </div>
        <div class="stat-value-with-download">
          <span class="stat-value green downloadable-number" @click="downloadBaoJiaVip">{{ baoJiaVipCount }}</span>
          <el-button size="mini" icon="el-icon-download" type="text" @click="downloadBaoJiaVip"></el-button>
        </div>
      </div>
    </div>

    <!-- 查询条件表单 -->
    <div class="insurance-search">
      <el-form :model="queryForm" :rules="rules" class="search-form" ref="queryForm" label-position="top">
        <!-- 没有经销商权限的布局 -->
        <div v-if="!hasProductManagementCopyPermission">
          <el-row :gutter="20">
            <el-col :span="6">
              <el-form-item label="保单状态" prop="uploadStatus">
                <el-select v-model="queryForm.uploadStatus" placeholder="请选择" clearable multiple style="width: 100%">
                  <el-option label="信息不全" value="1"></el-option>
                  <el-option label="未达标" value="2"></el-option>
                  <el-option label="初审合格" value="3"></el-option>
                  <el-option label="终审合格" value="4"></el-option>
                </el-select>
              </el-form-item>
            </el-col>
            <el-col :span="6">
              <el-form-item label="上传时间" prop="uploadTime">
                <el-date-picker v-model="queryForm.uploadTime" type="daterange" range-separator="至"
                  start-placeholder="开始日期" end-placeholder="结束日期" value-format="yyyy-MM-dd" style="width: 100%">
                </el-date-picker>
              </el-form-item>
            </el-col>
            <el-col :span="6">
              <el-form-item label="更新时间" prop="dateRange">
                <el-date-picker v-model="queryForm.dateRange" type="daterange" range-separator="至"
                  start-placeholder="开始日期" end-placeholder="结束日期" value-format="yyyy-MM-dd" style="width: 100%">
                </el-date-picker>
              </el-form-item>
            </el-col>
            <el-col :span="6">
              <el-form-item label="车架号" prop="frameNo">
                <el-input v-model="queryForm.frameNo" placeholder="请输入" maxlength="17" clearable></el-input>
              </el-form-item>
            </el-col>
          </el-row>

          <div v-show="showAllConditions">
            <el-row :gutter="20">
              <el-col :span="6">
                <el-form-item label="保单号" prop="policyNo">
                  <el-input v-model="queryForm.policyNo" placeholder="请输入" maxlength="50" clearable></el-input>
                </el-form-item>
              </el-col>
              <!-- <el-col :span="6">
                <el-form-item label="投保类型" prop="insureType">
                  <el-select v-model="queryForm.insureType" placeholder="请选择" clearable style="width: 100%">
                    <el-option label="修转续" value="1"></el-option>
                    <el-option label="其它" value="0"></el-option>
                  </el-select>
                </el-form-item>
              </el-col> -->
              <el-col :span="6">
                <el-form-item label="保险公司" prop="insCompany">
                  <el-select v-model="queryForm.insCompany" placeholder="请选择" clearable filterable style="width: 100%">
                    <el-option v-for="item in insCompanyList" :key="item.value" :label="item.label"
                      :value="item.value"></el-option>
                  </el-select>
                </el-form-item>
              </el-col>
              <el-col :span="6">
                <el-form-item label="车型" prop="carModel">
                  <el-input v-model="queryForm.carModel" placeholder="请输入" maxlength="50" clearable></el-input>
                </el-form-item>
              </el-col>
              <el-col :span="6">
                <el-form-item label="车牌号" prop="licensePlate">
                  <el-input v-model="queryForm.licensePlate" placeholder="请输入" maxlength="10" clearable></el-input>
                </el-form-item>
              </el-col>
            </el-row>
            <el-row :gutter="20">
              <el-col :span="6">
                <el-form-item label="标签" prop="tag">
                  <el-select v-model="queryForm.tag" filterable clearable placeholder="请选择" style="width: 100%">
                    <el-option v-for="tag in tagOptions" :key="tag.value" :label="tag.label" :value="tag.value" />
                  </el-select>
                </el-form-item>
              </el-col>
              <el-col :span="18">
                <!-- 占位的，保持布局一致 -->
              </el-col>
            </el-row>
          </div>
        </div>

        <!-- 有经销商权限的布局 -->
        <div v-else>
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
              <el-form-item label="保单状态" prop="uploadStatus">
                <el-select v-model="queryForm.uploadStatus" placeholder="请选择" clearable multiple style="width: 100%">
                  <el-option label="信息不全" value="1"></el-option>
                  <el-option label="未达标" value="2"></el-option>
                  <el-option label="初审合格" value="3"></el-option>
                  <el-option label="终审合格" value="4"></el-option>
                </el-select>
              </el-form-item>
            </el-col>
            <el-col :span="6">
              <el-form-item label="上传时间" prop="uploadTime">
                <el-date-picker v-model="queryForm.uploadTime" type="daterange" range-separator="至"
                  start-placeholder="开始日期" end-placeholder="结束日期" value-format="yyyy-MM-dd" style="width: 100%">
                </el-date-picker>
              </el-form-item>
            </el-col>
            <el-col :span="6">
              <el-form-item label="更新时间" prop="dateRange">
                <el-date-picker v-model="queryForm.dateRange" type="daterange" range-separator="至"
                  start-placeholder="开始日期" end-placeholder="结束日期" value-format="yyyy-MM-dd" style="width: 100%">
                </el-date-picker>
              </el-form-item>
            </el-col>
          </el-row>

          <div v-show="showAllConditions">
            <el-row :gutter="20">
              <el-col :span="6">
                <el-form-item label="车架号" prop="frameNo">
                  <el-input v-model="queryForm.frameNo" placeholder="请输入" maxlength="17" clearable></el-input>
                </el-form-item>
              </el-col>
              <el-col :span="6">
                <el-form-item label="保单号" prop="policyNo">
                  <el-input v-model="queryForm.policyNo" placeholder="请输入" maxlength="50" clearable></el-input>
                </el-form-item>
              </el-col>
              <!-- <el-col :span="6">
                <el-form-item label="投保类型" prop="insureType">
                  <el-select v-model="queryForm.insureType" placeholder="请选择" clearable style="width: 100%">
                    <el-option label="修转续" value="1"></el-option>
                    <el-option label="其它" value="0"></el-option>
                  </el-select>
                </el-form-item>
              </el-col> -->
              <el-col :span="6">
                <el-form-item label="保险公司" prop="insCompany">
                  <el-select v-model="queryForm.insCompany" placeholder="请选择" clearable filterable style="width: 100%">
                    <el-option v-for="item in insCompanyList" :key="item.value" :label="item.label"
                      :value="item.value"></el-option>
                  </el-select>
                </el-form-item>
              </el-col>
              <el-col :span="6">
                <el-form-item label="车型" prop="carModel">
                  <el-input v-model="queryForm.carModel" placeholder="请输入" maxlength="50" clearable></el-input>
                </el-form-item>
              </el-col>
            </el-row>

            <el-row :gutter="20">
              <el-col :span="6">
                <el-form-item label="车牌号" prop="licensePlate">
                  <el-input v-model="queryForm.licensePlate" placeholder="请输入" maxlength="10" clearable></el-input>
                </el-form-item>
              </el-col>
              <el-col :span="6">
                <el-form-item label="标签" prop="tag">
                  <el-select v-model="queryForm.tag" filterable clearable placeholder="请选择" style="width: 100%">
                    <el-option v-for="tag in tagOptions" :key="tag.value" :label="tag.label" :value="tag.value" />
                  </el-select>
                </el-form-item>
              </el-col>
              <el-col :span="6">
                <!-- 占位的，保持布局一致 -->
              </el-col>
            </el-row>
          </div>
        </div>

        <!-- 操作按钮部分不变 -->
        <div class="operate-condition">
          <span @click="showAllConditions = !showAllConditions">
            {{ showAllConditions ? '收起查询条件' : '更多查询条件' }}
            <i :class="['el-icon-arrow-' + (showAllConditions ? 'up' : 'down')]"></i>
          </span>
        </div>

        <div class="actions-container">
          <el-button type="primary" icon="el-icon-search" @click="search">查询</el-button>
          <el-button plain icon="el-icon-refresh-left" @click="reset">重置</el-button>
          <div class="vertical-divider"></div>
          <el-button type="primary" icon="el-icon-edit" @click="edit" :disabled="multipleSelection.length !== 1"
            v-if="!shouldHideOperationButtons">编辑</el-button>
          <el-button plain icon="el-icon-delete" @click="deleteItems"
            :disabled="multipleSelection.length === 0 && !checkedAll" v-if="!shouldHideOperationButtons">删除</el-button>
          <div class="vertical-divider" v-if="!shouldHideOperationButtons"></div>
          <el-button plain icon="el-icon-time" @click="showUploadHistory"
            v-if="!shouldHideOperationButtons">上传历史</el-button>
          <el-button plain icon="el-icon-download" @click="exportDetails"
            :disabled="multipleSelection.length === 0 && !checkedAll">导出明细</el-button>
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
        <el-table-column prop="deptName" label="经销商" align="center" v-if="hasProductManagementCopyPermission">
        </el-table-column>
        <el-table-column prop="dataStatus" label="状态" align="center">
          <template slot-scope="scope">
            <span :class="'dataStatus-' + scope.row.dataStatus">
              {{ scope.row.dataStatus === '1' ? '信息不全' : scope.row.dataStatus === '2' ? '未达标' : scope.row.dataStatus ===
                '3' ? '初审合格' : scope.row.dataStatus === '4' ? '终审合格' : scope.row.dataStatus }}
            </span>
          </template>
        </el-table-column>
        <el-table-column prop="tag" label="标签" align="center"></el-table-column>
        <!-- <el-table-column prop="insureTypeName" label="投保类型" align="center"></el-table-column> -->
        <el-table-column prop="isElectricName" label="是否纯电车" align="center"></el-table-column>
        <el-table-column prop="plateNo" label="车牌号" align="center"></el-table-column>
        <el-table-column prop="frmNo" label="车架号" align="center"></el-table-column>
        <el-table-column prop="insName" label="保险公司" align="center"></el-table-column>
        <el-table-column prop="policyNo" label="保单号" width="150" align="center"></el-table-column>
        <el-table-column prop="chargeConfirmTime" label="收费确认时间" align="center"></el-table-column>
        <el-table-column prop="updateTime" label="更新时间" align="center"></el-table-column>
        <el-table-column prop="uploadTime" label="上传时间" align="center"></el-table-column>
        <el-table-column prop="policyRemark" label="备注" align="center">
          <template slot-scope="scope">
            <div class="remark-cell">
              <el-tooltip v-if="scope.row.policyRemark" effect="dark" :content="scope.row.policyRemark" placement="top"
                :disabled="!isLongText(scope.row.policyRemark)">
                <span class="remark-text">{{ scope.row.policyRemark }}</span>
              </el-tooltip>
              <span v-else>-</span>
            </div>
          </template>
        </el-table-column>
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

    <upload-policy-dialog :visible.sync="uploadDialogVisible" :protocol-content="protocolContent"
      :activity-name="currentActivity ? currentActivity.activeName : ''" :active-id="activityName"
      @cancel="uploadDialogVisible = false" @upload="handleUpload" @showHistory="showUploadHistory" />
    <insurance-policy-dialog :visible.sync="policyDialogVisible" :mode="policyDialogMode"
      :policy-data="currentPolicyData || {}" :insurance-companies="insCompanyList" :dictionary-data="dictionaryData"
      @save="handleSavePolicy" @cancel="policyDialogVisible = false" />
    <upload-history-dialog :visible.sync="uploadHistoryVisible" />
  </div>
</template>

<script>
import UploadPolicyDialog from './components/UploadPolicyDialog.vue'
import InsurancePolicyDialog from './components/InsurancePolicyDialog.vue'
import UploadHistoryDialog from './components/UploadHistoryDialog.vue'
import { getInsuranceCompanies } from '@/utils/commonApi'
import { getReportDropdownList } from '@/views/modules/daibu-insurance/daibuche-common.js'
export default {
  components: {
    UploadPolicyDialog,
    InsurancePolicyDialog,
    UploadHistoryDialog,
  },
  data() {
    return {
      checkedAll: false,
      activityName: '',
      activityList: [], // 存储活动列表
      dealerList: [], //存储经销商列表
      currentActivity: null, // 当前选中的活动详情
      activityDetails: null, // 第二个接口返回的详细数据
      isActivityExpired: false, // 活动是否过期
      showAllConditions: false,
      uploadDialogVisible: false,
      policyDialogVisible: false,
      policyDialogMode: 'edit', // 'edit' 或 'view'
      currentPolicyData: {},
      uploadHistoryVisible: false,
      protocolContent: '',
      insCompanyList: [],
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
      queryForm: {
        deptName: '',
        uploadStatus: [],
        dateRange: [],
        uploadTime: [],
        frameNo: '',
        policyNo: '',
        // insureType: '',
        insCompany: '',
        carModel: '',
        licensePlate: '',
        tag: '',
      },
      tagOptions: [],
      tableData: [],
      multipleSelection: [],
      currentPage: 1,
      pageSize: 10,
      total: 0,
      tableLoading: false,
      deleteLoading: false,
      exportLoading: false,
      refreshLoading: false,
    }
  },
  computed: {
    tableHeight() {
      // 如果未展开查询条件，使用基础高度
      if (!this.showAllConditions) {
        return 'calc(100vh - 525px)'
      }

      // 展开查询条件时的高度
      return 'calc(100vh - 648px)'
    },
    // 权限检查
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
    shouldHideOperationButtons() {
      const permissions = JSON.parse(sessionStorage.getItem('permissions') || '[]')
      return permissions.some(permission => ['daibu-report-overview-changshang', 'daibu-report-overview-dq', 'daibu-report-overview-ins', 'daibu-report-overview-jt'].includes(permission))
    },
    remainingQuota() {
      if (!this.activityList.length) return 'N/A'

      if (this.activityDetails && this.activityDetails.policyQuotaActual !== undefined) {
        return this.formatNumber(this.activityDetails.policyQuotaActual)
      }

      return 'N/A'
    },

    // 保底目标完成量显示
    completionRate() {
      if (!this.activityList.length) return 'N/A'

      if (this.activityDetails) {
        const actual = this.formatNumber(this.activityDetails.compQtyActual || 0)
        const target = this.formatNumber(this.currentActivity.compQty || 0)
        return `${actual} / ${target}`
      }
      return 'N/A'
    },

    // 获得支持保单数
    supportedPolicies() {
      if (!this.activityList.length) return 'N/A'

      if (this.activityDetails) {
        return this.formatNumber(this.activityDetails.supPolCountActual || 0)
      }
      return 'N/A'
    },

    // 上传按钮是否禁用
    isUploadDisabled() {
      return this.isActivityExpired || !this.activityList.length
    },

    // 宝驾计划基础版单数
    baoJiaBasicCount() {
      if (!this.activityList.length) return 'N/A';

      if (this.activityDetails) {
        return this.formatNumber(this.activityDetails.arcSupCount || 0);
      }
      return 'N/A';
    },

    // 宝驾计划尊享版单数  
    baoJiaVipCount() {
      if (!this.activityList.length) return 'N/A';

      if (this.activityDetails) {
        return this.formatNumber(this.activityDetails.arcPremiumSupCount || 0);
      }
      return 'N/A';
    },
  },
  created() {
    this.getActivityPolicyBaseInfo()
    this.getInsuranceCompanies()
    this.getActivityList()
    this.getDealerList()
    this.getActivityPolicyUploadProtocol()
    this.getTagOptions()
  },
  methods: {
    search() {
      this.currentPage = 1 // 重置到第一页

      // 清空选择状态
      this.multipleSelection = []
      this.checkedAll = false

      // 对现有表格数据清除选中状态
      if (this.tableData && this.tableData.length > 0) {
        this.tableData.forEach(row => {
          this.$set(row, 'isSelected', false)
        })
      }

      this.fetchPolicyList()
    },
    reset() {
      this.$refs.queryForm.resetFields()
      this.queryForm.dateRange = []
      this.queryForm.uploadTime = []
    },
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
    edit() {
      // 确保 multipleSelection 已初始化且只选择了一条记录
      if (!this.multipleSelection || this.multipleSelection.length !== 1) {
        this.$message.warning('请选择一条记录进行编辑')
        return
      }

      // 获取选中记录的ID
      const policyId = this.multipleSelection[0].policyId
      this.fetchPolicyDetails(policyId, 'edit')
    },

    detail(row) {
      const policyId = row.policyId
      this.fetchPolicyDetails(policyId, 'view')
    },
    // 获取标签选项数据
    getTagOptions() {
      this.$https('/activityPolicy/getActivityPolicyAllTags', {
        body: {}
      })
        .then(res => {
          if (res.header.code === '10000') {
            this.tagOptions = (res.body || []).map(tag => ({
              label: tag,
              value: tag
            }))
          }
        })
        .catch(err => {
          console.error('获取标签数据失败：', err)
        })
    },
    // 获取保单详情数据
    fetchPolicyDetails(policyId, mode) {
      this.loading = true

      this.$https('/activityPolicy/getActivityPolicyInfo', {
        body: { policyId },
      })
        .then(response => {
          if (response && response.body) {
            const data = response.body
            const vehicleInfo = data.vehicleInfo || {}
            const policyInfo = data.policyInfo || {}
            const cvrgList = data.cvrgList || []
            const dmoInfo = data.dmoInfo || {}

            // 创建表单数据对象
            this.currentPolicyData = {
              policyId: policyId,
              // 车辆信息
              licensePlate: vehicleInfo.plateNo || '',
              frameNo: vehicleInfo.frmNo || '',
              engineNo: vehicleInfo.engNo || '',
              policyRemark: policyInfo.policyRemark || '',
              carModel: vehicleInfo.vehicleName || vehicleInfo.modelName || '',
              registrationDate: vehicleInfo.fstRegNo || '',
              approvedMass: vehicleInfo.ton || '',
              seatingCapacity: vehicleInfo.seatNum || '',
              usageNature: vehicleInfo.vhlUsageCode || '',
              vehicleType: vehicleInfo.carKindCode || '',
              energyType: vehicleInfo.carFuelType || '',
              displacement: vehicleInfo.desplacement || '',
              power: vehicleInfo.vhlPower || '',

              // 保单信息
              policyType: policyInfo.policyType || '',
              insuranceCompany: policyInfo.insId || '',
              policyNo: policyInfo.policyNo || '',
              fileUrl: policyInfo.fileUrl || '',
              activeUrl: policyInfo.activeUrl || '',
              insuranceStartDate: policyInfo.comenceTime || '',
              insuranceEndDate: policyInfo.terminateTime || '',
              vehicleDamageAmount: policyInfo.vehicleDamageCoverage !== undefined && policyInfo.vehicleDamageCoverage !== null ? policyInfo.vehicleDamageCoverage.toString() : '',
              thirdPartyAmount: policyInfo.thirdPartyCoverage ? policyInfo.thirdPartyCoverage.toString() : '',
              totalPremium: policyInfo.premiumTotal !== undefined && policyInfo.premiumTotal !== null ? policyInfo.premiumTotal.toString() : policyInfo.premium !== undefined && policyInfo.premium !== null ? policyInfo.premium.toString() : '',
              travelTax: policyInfo.travelTax || '',
              feeConfirmTime: policyInfo.chargeConfirmTime || '',
              policyCreateTime: policyInfo.policyGenTime || '',
              insuranceConfirmTime: policyInfo.policyConfirmTime || '',
              policyPrintTime: policyInfo.policyPrintTime || '',
              insuranceConfirmCode: policyInfo.policyConfirmCode || '',
              policyVerificationCode: policyInfo.policyVerCode || '',
              bankTransactionNo: policyInfo.bankTxnNo || '',
              remark: policyInfo.remark || '',

              // 投保险种
              insuranceTypes: this.mapInsuranceTypes(cvrgList),

              // 特别约定
              specialTerms: policyInfo.specialAgreement || '',

              // DMO字段信息
              actualDriver: dmoInfo.actualFollower || '',
              vehicleId: dmoInfo.vehicleId || '',
              personalInfoPolicy: dmoInfo.premiumPolicyStatus || '',
              lastYearInsurer: dmoInfo.insureIns || '',
              businessType: dmoInfo.busType || '',
              hasBMWAccidentRepairProduct: dmoInfo.bmwArcStatus || '',
            }

            // 设置对话框模式并显示
            this.policyDialogMode = mode
            this.policyDialogVisible = true
          } else {
            this.$message.error('获取保单信息失败')
          }
        })
        .catch(error => {
          console.error('获取保单信息失败:', error)
        })
        .finally(() => {
          this.loading = false
        })
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
    refreshdata() {
      const currentActivityId = this.activityName || ''
      this.refreshLoading = true
      this.$https('/activityPolicy/getActivityPolicyTypeList', {
        body: {},
      })
        .then(response => {
          if (response) {
            this.activityList = Array.isArray(response.body) ? response.body : response.body ? [response.body] : []

            if (this.activityList.length > 0) {
              // 找到并保持当前选中的活动
              if (currentActivityId) {
                const currentActivity = this.activityList.find(item => item.activeId === currentActivityId)
                if (currentActivity) {
                  this.currentActivity = currentActivity
                  this.isActivityExpired = currentActivity.overdueStatus === '1'

                  // 获取该活动的详细统计数据
                  this.getActivityDetails(currentActivityId)
                }
              }
            }
          }
        })
        .catch(error => {
          console.error('获取活动列表失败:', error)
        })
        .finally(() => {
          this.refreshLoading = false
        })
    },
    deleteItems() {
      if (this.multipleSelection.length === 0 && !this.checkedAll) {
        this.$message.warning('请至少选择一条记录')
        return
      }

      this.$confirm('是否删除选择的保单数据？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      })
        .then(() => {
          this.deleteLoading = true
          // 记住当前选中的活动ID
          const currentActivityId = this.activityName || ''

          const params = {
            tag: this.queryForm.tag || '',
            policyIds: [],
            deptCode: this.queryForm.deptName || '',
            isAllChecked: this.checkedAll,
            activeId: currentActivityId,
            dataStatusList: Array.isArray(this.queryForm.uploadStatus) ? this.queryForm.uploadStatus : this.queryForm.uploadStatus ? [this.queryForm.uploadStatus] : [],
            updateStartTime: this.queryForm.dateRange && this.queryForm.dateRange[0] ? this.queryForm.dateRange[0] : '',
            updateEndTime: this.queryForm.dateRange && this.queryForm.dateRange[1] ? this.queryForm.dateRange[1] : '',
            uploadFileStartTime: this.queryForm.uploadTime && this.queryForm.uploadTime[0] ? this.queryForm.uploadTime[0] : '', // 添加上传开始时间
            uploadFileEndTime: this.queryForm.uploadTime && this.queryForm.uploadTime[1] ? this.queryForm.uploadTime[1] : '', // 添加上传结束时间
            frmNo: this.queryForm.frameNo || '',
            policyNo: this.queryForm.policyNo || '',
            // insureType: this.queryForm.insureType || '',
            insId: this.queryForm.insCompany || '',
            vehicleName: this.queryForm.carModel || '',
            plateNo: this.queryForm.licensePlate || '',
          }

          // 如果不是全选，填充policyIds数组
          if (!this.checkedAll) {
            params.policyIds = this.multipleSelection.map(item => item.policyId)
          }

          this.$https('/activityPolicy/deleteActivityPolicys', {
            body: params,
          })
            .then(res => {
              if (res && res.header && res.header.code === '10000') {
                // 只有当返回码为10000时才显示成功消息
                this.$message.success('删除成功')

                // 刷新活动列表数据
                this.$https('/activityPolicy/getActivityPolicyTypeList', {
                  body: {},
                })
                  .then(response => {
                    if (response) {
                      this.activityList = Array.isArray(response.body) ? response.body : response.body ? [response.body] : []

                      if (this.activityList.length > 0) {
                        // 找到并保持当前选中的活动
                        if (currentActivityId) {
                          const currentActivity = this.activityList.find(item => item.activeId === currentActivityId)
                          if (currentActivity) {
                            this.currentActivity = currentActivity
                            this.isActivityExpired = currentActivity.overdueStatus === '1'

                            // 获取该活动的详细统计数据
                            this.getActivityDetails(currentActivityId)
                            this.search() // 刷新表格数据
                          }
                        }
                      }
                    }
                  })
                  .catch(error => {
                    console.error('获取活动列表失败:', error)
                  })
              }
            })
            .catch(error => {
              console.error('删除失败:', error)
              this.$message.error('删除失败，请重试')
            })
            .finally(() => {
              this.deleteLoading = false
            })
        })
        .catch(() => { })
    },
    showUploadHistory() {
      this.uploadHistoryVisible = true
    },
    exportDetails() {
      if (this.multipleSelection.length === 0 && !this.checkedAll) {
        this.$message.warning('请至少选择一条记录')
        return
      }

      this.exportLoading = true

      let params = {}

      if (this.checkedAll) {
        // 勾选"全选"时，传递查询条件参数
        params = {
          tag: this.queryForm.tag || '',
          isAllChecked: true,
          activeId: this.activityName || '',
          dataStatusList: Array.isArray(this.queryForm.uploadStatus) ? this.queryForm.uploadStatus : this.queryForm.uploadStatus ? [this.queryForm.uploadStatus] : [],
          deptCodeList: this.queryForm.deptName ? [this.queryForm.deptName] : [],
          updateStartTime: this.queryForm.dateRange && this.queryForm.dateRange[0] ? this.queryForm.dateRange[0] : '',
          updateEndTime: this.queryForm.dateRange && this.queryForm.dateRange[1] ? this.queryForm.dateRange[1] : '',
          uploadFileStartTime: this.queryForm.uploadTime && this.queryForm.uploadTime[0] ? this.queryForm.uploadTime[0] : '', // 添加上传开始时间
          uploadFileEndTime: this.queryForm.uploadTime && this.queryForm.uploadTime[1] ? this.queryForm.uploadTime[1] : '', // 添加上传结束时间
          frmNo: this.queryForm.frameNo || '',
          policyNo: this.queryForm.policyNo || '',
          // insureType: this.queryForm.insureType || '',
          insId: this.queryForm.insCompany || '',
          vehicleName: this.queryForm.carModel || '',
          plateNo: this.queryForm.licensePlate || '',
        }
      } else {
        // 勾选具体行时，只传递选中行的ID
        params = {
          policyIds: this.multipleSelection.map(item => item.policyId),
          activeId: this.activityName || '',
          isAllChecked: false,
        }
      }

      this.$https('/activityPolicy/exportActivityPolicys', {
        body: params,
      })
        .then(res => {
          if (res) {
            const downloadUrl = res.body
            if (downloadUrl) {
              const downloadLink = document.createElement('a')
              downloadLink.href = downloadUrl
              // downloadLink.download = '名字-' + new Date().getTime() + '.xlsx';
              // downloadLink.style.display = 'none';
              document.body.appendChild(downloadLink)
              downloadLink.click()

              // 清理DOM
              document.body.removeChild(downloadLink)

              this.$message.success('导出成功')
            } else {
              this.$message.error('导出失败：未获取到下载链接')
            }
          } else {
            this.$message.error(res.msg || '导出失败')
          }
        })
        .catch(error => {
          console.error('导出失败:', error)
          this.$message.error('导出失败，请重试')
        })
        .finally(() => {
          this.exportLoading = false
        })
    },
    handleRowSelect(row) {
      // 因为禁用了行复选框，所以这个方法在全选状态下不会被调用
      if (this.multipleSelection.includes(row)) {
        this.multipleSelection = this.multipleSelection.filter(item => item !== row)
      } else {
        this.multipleSelection.push(row)
      }
    },
    isLongText(text) {
      if (!text) return false
      return text.length > 20
    },
    checkedAllOnChange(val) {
      if (val) {
        // 首先清空所有行的选中状态
        this.tableData.forEach(row => {
          // 确保每行有isSelected属性并设为false
          this.$set(row, 'isSelected', false)
        })
        this.multipleSelection = [...this.tableData]
      } else {
        this.multipleSelection = []
      }
    },
    handleSelectionChange(val) {
      if (!this.checkedAll) {
        this.multipleSelection = val
      }
    },
    handleSizeChange(val) {
      this.pageSize = val
      this.fetchPolicyList()
    },
    handleCurrentChange(val) {
      this.currentPage = val
      this.fetchPolicyList()
    },
    showUploadDialog() {
      this.uploadDialogVisible = true
    },
    handleUpload(fileList) {
      this.$message({
        message: '上传成功',
        type: 'success',
        duration: 3000,
      })
    },
    handleSavePolicy(formData) {
      console.log('保存策略数据:', formData)

      // 先记住当前选中的活动ID
      const currentActivityId = this.activityName

      // 刷新活动列表数据
      this.$https('/activityPolicy/getActivityPolicyTypeList', {
        body: {},
      })
        .then(response => {
          if (response && response.header && response.header.code === '10000') {
            // 只有当列表获取成功时才显示成功消息
            this.$message.success('保存成功')

            this.activityList = Array.isArray(response.body) ? response.body : response.body ? [response.body] : []

            if (this.activityList.length > 0) {
              // 找到并保持当前选中的活动
              if (currentActivityId) {
                const currentActivity = this.activityList.find(item => item.activeId === currentActivityId)
                if (currentActivity) {
                  this.currentActivity = currentActivity
                  this.isActivityExpired = currentActivity.overdueStatus === '1'

                  // 获取该活动的详细统计数据
                  this.getActivityDetails(currentActivityId)
                  // 刷新表格数据
                  this.search()
                }
              }
            }
          }
        })
        .catch(error => {
          console.error('获取活动列表失败:', error)
        })
    },
    // 好多数据的接口
    getActivityPolicyBaseInfo() {
      this.$https('/activityPolicy/getActivityPolicyBaseInfo', {
        body: {},
      })
        .then(res => {
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
            }
          }
        })
        .catch(error => {
          console.error('获取数据字典失败:', error)
        })
    },
    getActivityPolicyUploadProtocol() {
      this.$https('/activityPolicy/getActivityPolicyUploadProtocol', {
        body: {},
      })
        .then(response => {
          if (response && response.body) {
            const content = response.body.replace(/<a\s+(?:[^>]*?\s+)?href=(["'])(.*?)\1/g, '<a target="_blank" href=$1$2$1')
            this.protocolContent = content
          }
        })
        .catch(error => {
          console.error('获取协议失败:', error)
        })
    },
    getActivityList() {
      this.$https('/activityPolicy/getActivityPolicyTypeList', {
        body: {},
      })
        .then(response => {
          if (response) {
            this.activityList = Array.isArray(response.body) ? response.body : response.body ? [response.body] : []

            if (this.activityList.length > 0 && !this.activityName) {
              const firstActivity = this.activityList[0]
              this.activityName = firstActivity.activeId
              this.currentActivity = firstActivity
              this.isActivityExpired = firstActivity.overdueStatus === '1'
              this.getActivityDetails(firstActivity.activeId)
              this.fetchPolicyList()
            } else if (this.activityList.length === 0) {
              this.activityName = ''
              this.currentActivity = null
              this.activityDetails = null
            }
          } else {
            this.activityList = []
          }
        })
        .catch(error => {
          console.error('获取活动列表失败:', error)
          this.$message.error('获取活动列表失败')
          this.activityList = []
        })
    },

    fetchPolicyList() {
      if (!this.activityName) {
        this.$message.warning('请先选择活动')
        this.tableData = []
        this.total = 0
        this.multipleSelection = []
        this.checkedAll = false
        this.tableLoading = false // 确保loading状态被重置
        return // 如果没有活动名称，直接返回不调用接口
      }

      this.tableLoading = true

      // 构建查询参数
      const params = {
        tag: this.queryForm.tag || '',
        activeId: this.activityName || '',
        dataStatusList: Array.isArray(this.queryForm.uploadStatus) ? this.queryForm.uploadStatus : this.queryForm.uploadStatus ? [this.queryForm.uploadStatus] : [],
        updateStartTime: this.queryForm.dateRange && this.queryForm.dateRange[0] ? this.queryForm.dateRange[0] : '',
        updateEndTime: this.queryForm.dateRange && this.queryForm.dateRange[1] ? this.queryForm.dateRange[1] : '',
        uploadFileStartTime: this.queryForm.uploadTime && this.queryForm.uploadTime[0] ? this.queryForm.uploadTime[0] : '', // 添加上传开始时间
        uploadFileEndTime: this.queryForm.uploadTime && this.queryForm.uploadTime[1] ? this.queryForm.uploadTime[1] : '', // 添加上传结束时间
        frmNo: this.queryForm.frameNo || '',
        policyNo: this.queryForm.policyNo || '',
        // insureType: this.queryForm.insureType || '',
        insId: this.queryForm.insCompany || '',
        vehicleName: this.queryForm.carModel || '',
        plateNo: this.queryForm.licensePlate || '',
        deptCodeList: this.queryForm.deptName ? [this.queryForm.deptName] : [],
        page: this.currentPage,
        rows: this.pageSize,
      }

      this.$https('/activityPolicy/getActivityPolicyList', {
        body: params,
      })
        .then(response => {
          this.tableLoading = false

          if (response) {
            if (response.body) {
              this.tableData = response.body.rows || []
              this.total = response.body.total || 0

              this.tableData.forEach(row => {
                this.$set(row, 'isSelected', false)
              })
            } else {
              this.tableData = []
              this.total = 0
            }
          } else {
            this.$message.error('获取保单列表失败')
            this.tableData = []
            this.total = 0
          }
          this.multipleSelection = []
          this.checkedAll = false
        })
        .catch(error => {
          this.tableLoading = false
          console.error('获取保单列表失败:', error)
          this.$message.error('获取保单列表失败')
          this.total = 0
        })
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
    // 获取活动详细数据
    getActivityDetails(activeId) {
      if (!activeId) return;

      this.$https('/activityPolicy/getActivityPolicyTotal', {
        body: {
          activeId,
          arcStatisticsStatus: false
        },
      })
        .then(response => {
          if (response && response.body) {
            this.activityDetails = response.body;
          } else {
            this.$message.error('获取活动详情失败');
            this.activityDetails = null;
          }
        })
        .catch(error => {
          console.error('获取活动详情失败:', error);
          this.$message.error('获取活动详情失败');
          this.activityDetails = null;
        });
    },

    formatNumber(value) {
      // 处理非数值情况
      if (value === null || value === undefined || value === '' || value === 'N/A') {
        return value
      }

      // 转换为数字
      const num = Number(value)

      // 如果是有效数字，添加千分位分隔符
      if (!isNaN(num)) {
        return num.toLocaleString()
      }

      // 非数字情况直接返回原值
      return value
    },

    // 处理活动变更
    handleActivityChange(value) {
      // 找到选中的活动详情
      const selectedActivity = this.activityList.find(item => item.activeId === value)
      if (selectedActivity) {
        this.currentActivity = selectedActivity
        this.isActivityExpired = selectedActivity.overdueStatus === '1'

        // 获取新选中活动的详细数据
        this.getActivityDetails(value)

        // 重置查询条件
        this.reset()

        // 重新获取表格数据
        this.currentPage = 1 // 重置到第一页
        this.fetchPolicyList()
      }
    },
    // 宝驾计划基础版下载
    downloadBaoJiaBasic() {
      if (!this.activityName) {
        this.$message.warning('请先选择活动');
        return;
      }

      this.exportLoading = true;

      this.$https('/eventDashboard/exportArcStatisticsIssueFile', {
        body: {
          activeId: this.activityName,
          arcIssueType: "1"
        }
      }).then(res => {
        if (res) {
          const downloadUrl = res.body;
          if (downloadUrl) {
            const downloadLink = document.createElement('a');
            downloadLink.href = downloadUrl;
            document.body.appendChild(downloadLink);
            downloadLink.click();
            document.body.removeChild(downloadLink);

            this.$message.success('宝驾计划基础版数据导出成功');
          } else {
            this.$message.error('导出失败：未获取到下载链接');
          }
        } else {
          this.$message.error('宝驾计划基础版数据导出失败');
        }
      }).catch(error => {
        console.error('宝驾计划基础版数据导出失败:', error);
        this.$message.error('导出失败，请重试');
      }).finally(() => {
        this.exportLoading = false;
      });
    },

    // 宝驾计划尊享版下载  
    downloadBaoJiaVip() {
      if (!this.activityName) {
        this.$message.warning('请先选择活动');
        return;
      }

      this.exportLoading = true;

      this.$https('/eventDashboard/exportArcStatisticsIssueFile', {
        body: {
          activeId: this.activityName,
          arcIssueType: "2"
        }
      }).then(res => {
        if (res) {
          const downloadUrl = res.body;
          if (downloadUrl) {
            const downloadLink = document.createElement('a');
            downloadLink.href = downloadUrl;
            document.body.appendChild(downloadLink);
            downloadLink.click();
            document.body.removeChild(downloadLink);

            this.$message.success('宝驾计划尊享版数据导出成功');
          } else {
            this.$message.error('导出失败：未获取到下载链接');
          }
        } else {
          this.$message.error('宝驾计划尊享版数据导出失败');
        }
      }).catch(error => {
        console.error('宝驾计划尊享版数据导出失败:', error);
        this.$message.error('导出失败，请重试');
      }).finally(() => {
        this.exportLoading = false;
      });
    },
  },
}
</script>

<style scoped>
.insurance-accident-car-management {
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
  color: #1c69d4;
}

.error {
  color: #d54941;
}

.green {
  color: #2ba471;
}

.activity-name {
  flex: 2;
}

.insurance-search {
  margin-top: 20px;
  height: 100%;
  width: 100%;
}

.operate-condition {
  cursor: pointer;
  color: #667181;
  font-size: 12px;
  margin: 0px 0px 10px 0px;
}

.vertical-divider {
  display: inline-block;
  width: 1px;
  height: 24px;
  background-color: #ccc;
  margin: 0 10px;
  vertical-align: middle;
}

.input-with-button {
  display: flex;
  align-items: center;
}

.input-with-button .el-select {
  flex: 1;
  margin-right: 10px;
}

.search-form {
  margin-bottom: 20px;
  height: 100%;
  width: 100%;
}

.operate-condition i {
  margin-left: 5px;
}

.action-buttons {
  margin-top: 10px;
}

.table-container {
  margin-bottom: 5px;
}

.dataStatus-1 {
  color: #e37318;
}

.dataStatus-2 {
  color: #d54941;
}

.dataStatus-3 {
  color: #2ba471;
}

.dataStatus-4 {
  color: #2ba471;
}

.statistics {
  margin: 0;
  margin-right: 15px;
  color: #737f91;
  font-size: 14px;
  font-weight: 700;
}

.statistics-cards {
  display: flex;
  margin-bottom: 20px;
  gap: 15px;
  border: 1px solid #e0e0e0;
  border-radius: 5px;
  padding: 12px;
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

.actions-container {
  display: flex;
  align-items: center;
  flex-wrap: nowrap;
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

.remark-cell {
  width: 100%;
  overflow: hidden;
}

.remark-text {
  display: inline-block;
  max-width: 100%;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.activity-selection-area {
  display: flex;
  align-items: center;
  gap: 15px;
  padding: 10px 0;
  color: #606266;
  font-size: 14px;
}

.activity-selection-area .el-button {
  margin-left: auto;
}

.activity-selection-area .el-button:hover {
  text-decoration: underline;
}

.activity-select {
  width: 300px;
}

.stat-value-with-download {
  display: flex;
  align-items: center;
  gap: 12px;
}

.stat-value-with-download .el-button {
  color: #2BA471;
  font-size: 20px;
  padding: 0;
  min-height: auto;
}

.stat-value-with-download .el-button:hover {
  color: #2BA471;
}

.downloadable-number {
  cursor: pointer;
  transition: all 0.3s ease;
}

.downloadable-number:hover {
  text-decoration: underline;
}

/deep/ .el-form-item--mini .el-form-item__label {
  line-height: 8px;
}

/deep/ .el-form-item--mini.el-form-item {
  margin-bottom: 10px;
}
</style>
