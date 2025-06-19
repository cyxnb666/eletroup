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
          <el-button type="primary" icon="el-icon-upload" @click="showUploadDialog" :disabled="!activityName">
            上传资料(PDF)
          </el-button>
        </div>
      </div>
    </div>

    <!-- 查询条件表单 -->
    <div class="insurance-search">
      <el-form :model="queryForm" class="search-form" ref="queryForm" label-position="top">
        <el-row :gutter="20">
          <el-col :span="6">
            <el-form-item label="车架号" prop="frameNo">
              <el-input v-model="queryForm.frameNo" placeholder="请输入" maxlength="17" clearable></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="商业险保单号" prop="policyNo">
              <el-input v-model="queryForm.policyNo" placeholder="请输入" maxlength="50" clearable></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="购车发票号" prop="invoiceNo">
              <el-input v-model="queryForm.invoiceNo" placeholder="请输入" maxlength="50" clearable></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="保险公司" prop="insCompany">
              <el-select v-model="queryForm.insCompany" placeholder="请选择" clearable filterable style="width: 100%">
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
              <el-form-item label="车型" prop="carModel">
                <el-input v-model="queryForm.carModel" placeholder="请输入" maxlength="50" clearable></el-input>
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <!-- 占位，保持布局一致 -->
            </el-col>
          </el-row>
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
          <div class="vertical-divider"></div>
          <el-button plain icon="el-icon-delete" @click="deleteItems"
            :disabled="multipleSelection.length === 0 && !checkedAll">删除</el-button>
          <div class="vertical-divider"></div>
          <el-button plain icon="el-icon-time" @click="showUploadHistory">上传历史</el-button>
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
        <el-table-column prop="deptName" label="经销商" align="center"></el-table-column>
        <el-table-column prop="frameNo" label="车架号" align="center"></el-table-column>
        <el-table-column prop="insCompany" label="保险公司" align="center"></el-table-column>
        <el-table-column prop="policyNo" label="商业险保单号" align="center" width="150"></el-table-column>
        <el-table-column prop="invoiceNo" label="购车发票号" align="center"></el-table-column>
        <el-table-column prop="isElectricVehicle" label="是否纯电车" align="center">
          <template slot-scope="scope">
            {{ scope.row.isElectricVehicle === '1' ? '是' : '否' }}
          </template>
        </el-table-column>
        <el-table-column prop="registrationDate" label="初登日期" align="center"></el-table-column>
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
        cvrgList: [], // 险种列表
      },
      queryForm: {
        updateTime: [],
        frameNo: '',
        policyNo: '',
        invoiceNo: '',
        insCompany: '',
        carModel: '',
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
  },
  created() {
    this.getActivityList();
    this.getInsuranceCompanies();
    this.getUploadProtocol();
    this.getDictionaryData();
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

    // 获取活动列表
    getActivityList() {
      // TODO: 调用实际API获取活动列表
      // this.$https('/subsidyActivity/getActivityList', {
      //   body: {}
      // }).then(response => {
      //   if (response) {
      //     this.activityList = Array.isArray(response.body) ? response.body : (response.body ? [response.body] : []);
      //     if (this.activityList.length > 0 && !this.activityName) {
      //       const firstActivity = this.activityList[0];
      //       this.activityName = firstActivity.activeId;
      //       this.currentActivity = firstActivity;
      //       this.fetchSubsidyList();
      //     }
      //   }
      // }).catch(error => {
      //   console.error('获取活动列表失败:', error);
      //   this.activityList = [];
      // });

      // 模拟数据
      this.activityList = [
        { activeId: '1', activeName: '2025年三季度买车送保险' },
        { activeId: '2', activeName: '2025年四季度买车送保险' }
      ];
      if (this.activityList.length > 0) {
        this.activityName = this.activityList[0].activeId;
        this.currentActivity = this.activityList[0];
        this.fetchSubsidyList();
      }
    },

    // 获取保险公司列表
    getInsuranceCompanies() {
      // TODO: 调用实际API获取保险公司列表
      // 模拟数据
      this.insCompanyList = [
        { label: '中国人民财产保险股份有限公司', value: '1' },
        { label: '平安财险', value: '2' },
        { label: '太平洋保险', value: '3' }
      ];
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
        this.reset();
        this.currentPage = 1;

        // 清空选择状态
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

      // 构建查询参数
      const params = {
        activeId: this.activityName || '',
        updateStartTime: this.queryForm.updateTime && this.queryForm.updateTime[0] ? this.queryForm.updateTime[0] : '',
        updateEndTime: this.queryForm.updateTime && this.queryForm.updateTime[1] ? this.queryForm.updateTime[1] : '',
        frameNo: this.queryForm.frameNo || '',
        policyNo: this.queryForm.policyNo || '',
        invoiceNo: this.queryForm.invoiceNo || '',
        insCompany: this.queryForm.insCompany || '',
        carModel: this.queryForm.carModel || '',
        page: this.currentPage,
        rows: this.pageSize,
      };

      // TODO: 调用实际API
      // this.$https('/subsidyPolicy/getSubsidyPolicyList', {
      //   body: params,
      // }).then(response => {
      //   this.tableLoading = false;
      //   if (response) {
      //     if (response.body) {
      //       this.tableData = response.body.rows || [];
      //       this.total = response.body.total || 0;
      //       
      //       this.tableData.forEach(row => {
      //         this.$set(row, 'isSelected', false);
      //       });
      //     } else {
      //       this.tableData = [];
      //       this.total = 0;
      //     }
      //   } else {
      //     this.$message.error('获取补贴列表失败');
      //     this.tableData = [];
      //     this.total = 0;
      //   }
      //   this.multipleSelection = [];
      //   this.checkedAll = false;
      // }).catch(error => {
      //   this.tableLoading = false;
      //   console.error('获取补贴列表失败:', error);
      //   this.$message.error('获取补贴列表失败');
      //   this.tableData = [];
      //   this.total = 0;
      // });

      // 模拟数据
      setTimeout(() => {
        this.tableData = [
          {
            id: 1,
            frameNo: 'FAKEVIN0123456789',
            deptName: '宝马汽车经销商A',
            insCompany: '中国人保',
            policyNo: 'FAKEPN_0123456789',
            invoiceNo: '2225647',
            isElectricVehicle: '1',
            registrationDate: '2025/03/10',
            updateTime: '2025/03/10 14:00'
          },
          {
            id: 2,
            frameNo: 'FAKEVIN0123456789',
            deptName: '宝马汽车经销商B',
            insCompany: '平安财险',
            policyNo: 'FAKEPN_0123456789',
            invoiceNo: '2225648',
            isElectricVehicle: '0',
            registrationDate: '2025/03/10',
            updateTime: '2025/03/10 14:00'
          },
          {
            id: 3,
            frameNo: 'FAKEVIN0123456789',
            deptName: '宝马汽车经销商C',
            insCompany: '太平洋保险',
            policyNo: 'FAKEPN_0123456789',
            invoiceNo: '2225649',
            isElectricVehicle: '1',
            registrationDate: '2025/03/10',
            updateTime: '2025/03/10 14:00'
          },
          {
            id: 4,
            frameNo: 'FAKEVIN0123456789',
            deptName: '宝马汽车经销商D',
            insCompany: '太平洋保险',
            policyNo: 'FAKEPN_0123456789',
            invoiceNo: '2225650',
            isElectricVehicle: '0',
            registrationDate: '2025/03/10',
            updateTime: '2025/03/10 14:00'
          }
        ];

        this.tableData.forEach(row => {
          this.$set(row, 'isSelected', false);
        });

        this.total = 4;
        this.tableLoading = false;
        this.multipleSelection = [];
        this.checkedAll = false;
      }, 1000);
    },

    // 删除选中项
    deleteItems() {
      if (this.multipleSelection.length === 0 && !this.checkedAll) {
        this.$message.warning('请至少选择一条记录');
        return;
      }

      this.$confirm('是否删除选择的补贴数据？', '提示', {
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
            isAllChecked: true,
            updateStartTime: this.queryForm.updateTime && this.queryForm.updateTime[0] ? this.queryForm.updateTime[0] : '',
            updateEndTime: this.queryForm.updateTime && this.queryForm.updateTime[1] ? this.queryForm.updateTime[1] : '',
            frameNo: this.queryForm.frameNo || '',
            policyNo: this.queryForm.policyNo || '',
            invoiceNo: this.queryForm.invoiceNo || '',
            insCompany: this.queryForm.insCompany || '',
            carModel: this.queryForm.carModel || '',
          };
        } else {
          // 勾选具体行时，只传递选中行的ID
          params = {
            activeId: this.activityName || '',
            vehicleIds: this.multipleSelection.map(item => item.id),
            isAllChecked: false,
          };
        }

        // TODO: 调用实际删除API
        // this.$https('/subsidyPolicy/deleteSubsidyVehicles', {
        //   body: params
        // }).then(res => {
        //   if (res && res.header && res.header.code === '10000') {
        //     this.$message.success('删除成功');
        //     this.search(); // 刷新表格数据
        //   }
        // }).catch(error => {
        //   console.error('删除失败:', error);
        //   this.$message.error('删除失败，请重试');
        // }).finally(() => {
        //   this.deleteLoading = false;
        // });

        // 模拟删除操作
        setTimeout(() => {
          this.$message.success('删除成功');
          this.deleteLoading = false;
          this.search(); // 刷新表格数据
        }, 1000);
      }).catch(() => { });
    },

    // 获取上传协议内容
    getUploadProtocol() {
      // TODO: 调用实际API获取协议内容
      // this.$https('/subsidyPolicy/getUploadProtocol', {
      //   body: {}
      // }).then(response => {
      //   if (response && response.body) {
      //     const content = response.body.replace(/<a\s+(?:[^>]*?\s+)?href=(["'])(.*?)\1/g, '<a target="_blank" href=$1$2$1');
      //     this.protocolContent = content;
      //   }
      // }).catch(error => {
      //   console.error('获取协议失败:', error);
      // });

      // 模拟协议内容
      this.protocolContent = `
        <div class="agreement-text">
          <p>上传保单前请仔细阅读并确认已与用户签署<a href="javascript:void(0)" class="agreement-link">《宝马个人信息保护政策》</a>：</p>
        </div>
      `;
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
      // TODO: 调用实际API获取字典数据
      // this.$https('/subsidyPolicy/getBaseInfo', {
      //   body: {}
      // }).then(res => {
      //   if (res && res.body) {
      //     this.dictionaryData = {
      //       vhlUsageDicList: res.body.vhlUsageDicList || [],
      //       carKindDicList: res.body.carKindDicList || [],
      //       carFuelTypeList: res.body.carFuelTypeList || [],
      //       cvrgList: res.body.cvrgList || [],
      //     };
      //   }
      // }).catch(error => {
      //   console.error('获取数据字典失败:', error);
      // });

      // 模拟字典数据
      this.dictionaryData = {
        vhlUsageDicList: [
          { dicCode: '01', dicName: '非营运' },
          { dicCode: '02', dicName: '营运' }
        ],
        carKindDicList: [
          { dicCode: '01', dicName: '六座以下客车' },
          { dicCode: '02', dicName: '六座以上客车' }
        ],
        carFuelTypeList: [
          { dicCode: '01', dicName: '燃油' },
          { dicCode: '02', dicName: '纯电动' }
        ],
        cvrgList: [
          { dicCode: '0301100', dicName: '机动车损失保险' },
          { dicCode: '0301600', dicName: '第三者责任保险' },
          { dicCode: '0301700', dicName: '车上人员责任保险（司机）' },
          { dicCode: '0301800', dicName: '附加修理期间费用补偿保险' }
        ]
      };
    },

    // 查看详情
    detail(row) {
      const vehicleId = row.id || row.vehicleId;
      this.fetchVehicleDetails(vehicleId);
    },

    // 获取车辆详情数据
    fetchVehicleDetails(vehicleId) {
      this.loading = true;

      // TODO: 调用实际API获取车辆详情
      // this.$https('/subsidyPolicy/getVehicleInfo', {
      //   body: { vehicleId },
      // }).then(response => {
      //   if (response && response.body) {
      //     const data = response.body;
      //     this.currentVehicleData = this.mapVehicleData(data);
      //     this.vehicleDetailVisible = true;
      //   } else {
      //     this.$message.error('获取车辆信息失败');
      //   }
      // }).catch(error => {
      //   console.error('获取车辆信息失败:', error);
      //   this.$message.error('获取车辆信息失败');
      // }).finally(() => {
      //   this.loading = false;
      // });

      // 模拟获取车辆详情数据
      setTimeout(() => {
        this.currentVehicleData = {
          // 车辆信息
          licensePlate: '宝马MW6462C5',
          frameNo: 'LBV31FX05RM896107',
          engineNo: 'A008F231',
          carModel: '宝马MW6462C5多用途乘用车',
          registrationDate: '2024-09-03',
          approvedMass: '3999',
          seatingCapacity: '5',
          usageNature: '01',
          vehicleType: '01',
          energyType: '01',
          displacement: '2.99',
          power: '',

          // 商业险信息
          policyNo: 'PDAA202544010000522184',
          insuranceCompany: '1',
          insuranceStartDate: '2024-09-03 00:00:00',
          insuranceEndDate: '2024-09-03 00:00:00',
          vehicleDamageAmount: '177155.20',
          thirdPartyAmount: '3000000.00',
          totalPremium: '5248.67',
          feeConfirmTime: '2024-09-03 00:00:00',
          policyCreateTime: '2024-09-03 00:00:00',
          insuranceConfirmTime: '2024-09-03 00:00:00',
          insuranceConfirmCode: '',

          // 投保险种
          insuranceTypes: [
            {
              name: '0301100',
              type: '0301100',
              amount: '150,000',
              premium: '150,000'
            },
            {
              name: '0301600',
              type: '0301600',
              amount: '150,000',
              premium: '150,000'
            },
            {
              name: '0301700',
              type: '0301700',
              amount: '150,000',
              premium: '150,000'
            },
            {
              name: '0301800',
              type: '0301800',
              amount: '150,000',
              premium: '150,000'
            }
          ],

          // 特别约定
          specialTerms: '',

          // 购车发票
          invoiceNo: '222555464',
          invoiceDate: '2024-09-03',
          totalAmount: '',
          sellerName: '',
          brandModel: '宝马牌系列车',

          // 文件URL
          policyFileUrl: '',
          invoiceFileUrl: '',

          // 其他
          remark: '',
          activeUrl: ''
        };

        this.vehicleDetailVisible = true;
        this.loading = false;
      }, 1000);
    },

    // 处理上传
    handleUpload(fileList) {
      this.$message({
        message: '上传成功',
        type: 'success',
        duration: 3000,
      });
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