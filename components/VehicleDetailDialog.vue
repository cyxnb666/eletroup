<template>
    <el-dialog :visible.sync="dialogVisible" width="80%" :close-on-click-modal="false" class="vehicle-detail-dialog"
        :title="null" :show-close="false" v-loading="saveLoading">
        <div slot="title" class="custom-dialog-title">
            <div class="title-first-row">
                <span>车辆详情</span>
                <div class="view-buttons">
                    <el-button type="primary" size="small" class="view-form-btn"
                        @click="viewElectronicForm('policy')">查看商业险保单</el-button>
                    <el-button type="primary" size="small" class="view-form-btn"
                        @click="viewElectronicForm('invoice')">查看购车发票</el-button>
                </div>
            </div>
            <div class="status-notification" v-if="formData.remark">
                <i class="el-icon-info"></i>
                <span class="status-text">
                    <strong>未达标：</strong>
                    <span v-html="formatRemark"></span>
                </span>
            </div>
        </div>

        <div v-if="dialogVisible">
            <el-form ref="entireForm" :model="formData" label-width="100px" :disabled="true"
                label-position="top">
                <!-- 车辆信息 section -->
                <div class="section-header">车辆信息</div>
                <el-row :gutter="20">
                    <el-col :span="6">
                        <el-form-item label="车牌号">
                            <el-input v-model="formData.licensePlate" placeholder="请输入" maxlength="8"
                                :disabled="isNewCarWithoutPlate">
                                <template slot="append">
                                    <el-checkbox v-model="isNewCarWithoutPlate" disabled>新车未上牌</el-checkbox>
                                </template>
                            </el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="车架号">
                            <el-input v-model="formData.frameNo" placeholder="请输入" maxlength="17">
                            </el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="发动机号">
                            <el-input v-model="formData.engineNo" placeholder="请输入" maxlength="20">
                            </el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="车型">
                            <el-input v-model="formData.carModel" maxlength="50" placeholder="请输入"></el-input>
                        </el-form-item>
                    </el-col>
                </el-row>

                <el-row :gutter="20">
                    <el-col :span="6">
                        <el-form-item label="初登日期">
                            <el-date-picker v-model="formData.registrationDate" type="date" placeholder="选择日期"
                                format="yyyy-MM-dd" value-format="yyyy-MM-dd"></el-date-picker>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="核定载质量(kg)">
                            <el-input v-model="formData.approvedMass" placeholder="请输入">
                            </el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="核定载客(人)">
                            <el-input v-model="formData.seatingCapacity" placeholder="请输入">
                            </el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="使用性质">
                            <el-select v-model="formData.usageNature" placeholder="请选择">
                                <el-option v-for="item in dictionaryData.vhlUsageDicList" :key="item.dicCode"
                                    :label="item.dicName" :value="item.dicCode">
                                </el-option>
                            </el-select>
                        </el-form-item>
                    </el-col>
                </el-row>

                <el-row :gutter="20">
                    <el-col :span="6">
                        <el-form-item label="机动车种类">
                            <el-select v-model="formData.vehicleType" placeholder="请选择">
                                <el-option v-for="item in dictionaryData.carKindDicList" :key="item.dicCode"
                                    :label="item.dicName" :value="item.dicCode">
                                </el-option>
                            </el-select>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="能源类型">
                            <el-select v-model="formData.energyType" placeholder="请选择">
                                <el-option v-for="item in dictionaryData.carFuelTypeList" :key="item.dicCode"
                                    :label="item.dicName" :value="item.dicCode">
                                </el-option>
                            </el-select>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="排量(L)">
                            <el-input v-model="formData.displacement" placeholder="请输入">
                            </el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="功率(kw)">
                            <el-input v-model="formData.power" placeholder="请输入">
                            </el-input>
                        </el-form-item>
                    </el-col>
                </el-row>

                <el-divider></el-divider>

                <!-- 商业险信息 section -->
                <div class="section-header">商业险信息</div>
                <el-row :gutter="20">
                    <el-col :span="6">
                        <el-form-item label="保单号">
                            <el-input v-model="formData.policyNo" maxlength="50" placeholder="请输入"></el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="保险公司名称">
                            <el-select v-model="formData.insuranceCompany" filterable placeholder="请选择">
                                <el-option v-for="item in insuranceCompanies" :key="item.value" :label="item.label"
                                    :value="item.value">
                                </el-option>
                            </el-select>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="保险起期">
                            <el-date-picker v-model="formData.insuranceStartDate" type="datetime" placeholder="选择日期时间"
                                format="yyyy-MM-dd HH:mm:ss" value-format="yyyy-MM-dd HH:mm:ss">
                            </el-date-picker>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="保险止期">
                            <el-date-picker v-model="formData.insuranceEndDate" type="datetime" placeholder="选择日期时间"
                                format="yyyy-MM-dd HH:mm:ss" value-format="yyyy-MM-dd HH:mm:ss">
                            </el-date-picker>
                        </el-form-item>
                    </el-col>
                </el-row>

                <el-row :gutter="20">
                    <el-col :span="6">
                        <el-form-item label="车损险保额(元)">
                            <el-input v-model="formData.vehicleDamageAmount" placeholder="请输入">
                            </el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="三者险保额(元)">
                            <el-input v-model="formData.thirdPartyAmount" placeholder="请输入" maxlength="20">
                            </el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="保费合计(元)">
                            <el-input v-model="formData.totalPremium" placeholder="请输入">
                            </el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="收费确认时间">
                            <el-date-picker v-model="formData.feeConfirmTime" type="datetime" placeholder="选择日期时间"
                                format="yyyy-MM-dd HH:mm:ss" value-format="yyyy-MM-dd HH:mm:ss"></el-date-picker>
                        </el-form-item>
                    </el-col>
                </el-row>

                <el-row :gutter="20">
                    <el-col :span="6">
                        <el-form-item label="保单生成时间">
                            <el-date-picker v-model="formData.policyCreateTime" type="datetime" placeholder="选择日期时间"
                                format="yyyy-MM-dd HH:mm:ss" value-format="yyyy-MM-dd HH:mm:ss"></el-date-picker>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="投保确认时间">
                            <el-date-picker v-model="formData.insuranceConfirmTime" type="datetime" placeholder="选择日期时间"
                                format="yyyy-MM-dd HH:mm:ss" value-format="yyyy-MM-dd HH:mm:ss"></el-date-picker>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="投保确认码">
                            <el-input v-model="formData.insuranceConfirmCode" maxlength="50"
                                placeholder="请输入"></el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <!-- 占位 -->
                    </el-col>
                </el-row>

                <el-divider></el-divider>

                <!-- 投保险种 section -->
                <div class="insurance-types-header">
                    <div class="section-header">投保险种</div>
                </div>

                <div class="custom-insurance-table">
                    <!-- 表头 -->
                    <div class="table-header">
                        <div class="header-item">险种名称</div>
                        <div class="header-item">类型</div>
                        <div class="header-item">保额</div>
                        <div class="header-item">保费(元)</div>
                        <div class="header-item">操作</div>
                    </div>

                    <!-- 表格内容 -->
                    <div v-for="(item, index) in formData.insuranceTypes" :key="index" class="table-row">
                        <div class="row-item">
                            <el-select v-model="item.name" placeholder="请选择" disabled>
                                <el-option v-for="cvrgItem in availableCvrgList(index)" :key="cvrgItem.dicCode"
                                    :label="cvrgItem.dicName" :value="cvrgItem.dicCode">
                                </el-option>
                            </el-select>
                        </div>
                        <div class="row-item">{{ item.name ? getInsuranceTypeName(item.name) : '' }}</div>
                        <div class="row-item">
                            <el-input v-model="item.amount" placeholder="请输入保额" disabled maxlength="20">
                            </el-input>
                        </div>
                        <div class="row-item">
                            <el-input v-model="item.premium" placeholder="请输入保费" disabled>
                            </el-input>
                        </div>
                        <div class="row-item">
                            <span class="delete-btn-disabled">删除</span>
                        </div>
                    </div>

                    <!-- 没有数据时的提示 -->
                    <div v-if="formData.insuranceTypes.length === 0" class="no-data">
                        暂无险种数据
                    </div>
                </div>

                <el-divider></el-divider>

                <!-- 特别约定 section -->
                <div class="section-header">特别约定</div>
                <el-form-item>
                    <el-input type="textarea" v-model="formData.specialTerms" :rows="4" maxlength="500"
                        placeholder="请输入特别约定内容"></el-input>
                </el-form-item>

                <el-divider></el-divider>

                <!-- 购车发票 section -->
                <div class="section-header">购车发票</div>
                <el-row :gutter="20">
                    <el-col :span="6">
                        <el-form-item label="发票号">
                            <el-input v-model="formData.invoiceNo" maxlength="50" placeholder="请输入"></el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="开票日期">
                            <el-date-picker v-model="formData.invoiceDate" type="date" placeholder="选择日期"
                                format="yyyy-MM-dd" value-format="yyyy-MM-dd"></el-date-picker>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="加税合计(元)">
                            <el-input v-model="formData.totalAmount" placeholder="输入内容"></el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="销售单位名称">
                            <el-input v-model="formData.sellerName" placeholder="输入内容"></el-input>
                        </el-form-item>
                    </el-col>
                </el-row>

                <el-row :gutter="20">
                    <el-col :span="6">
                        <el-form-item label="厂牌型号">
                            <el-input v-model="formData.brandModel" placeholder="请输入"></el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="18">
                        <!-- 占位 -->
                    </el-col>
                </el-row>
            </el-form>
        </div>

        <div slot="footer" class="dialog-footer">
            <el-button @click="handleClose">关闭</el-button>
        </div>
    </el-dialog>
</template>

<script>
import { isMainInsurance, insuranceTypeName } from '@/views/modules/car-insurance/utils/index.js';

export default {
    props: {
        visible: {
            type: Boolean,
            default: false
        },
        vehicleData: {
            type: Object,
            default: () => ({})
        },
        insuranceCompanies: {
            type: Array,
            default: () => []
        },
        dictionaryData: {
            type: Object,
            default: () => ({
                vhlUsageDicList: [],
                carKindDicList: [],
                carFuelTypeList: [],
                cvrgList: [],
                newEnergyCvrgList: [],
            })
        }
    },
    data() {
        return {
            isNewCarWithoutPlate: false,
            loading: false,
            saveLoading: false,
            dialogVisible: false,
            formData: {
                // 车辆信息
                licensePlate: '',
                frameNo: '',
                engineNo: '',
                carModel: '',
                registrationDate: '',
                approvedMass: '',
                seatingCapacity: '',
                usageNature: '',
                vehicleType: '',
                energyType: '',
                displacement: '',
                power: '',

                // 商业险信息
                policyNo: '',
                insuranceCompany: '',
                insuranceStartDate: '',
                insuranceEndDate: '',
                vehicleDamageAmount: '',
                thirdPartyAmount: '',
                totalPremium: '',
                feeConfirmTime: '',
                policyCreateTime: '',
                insuranceConfirmTime: '',
                insuranceConfirmCode: '',

                // 投保险种
                insuranceTypes: [],

                // 特别约定
                specialTerms: '',

                // 购车发票
                invoiceNo: '',
                invoiceDate: '',
                totalAmount: '',
                sellerName: '',
                brandModel: '',

                // 其他
                remark: '',
                policyFileUrl: '',
                invoiceFileUrl: '',
                activeUrl: ''
            }
        };
    },
    computed: {
        formatRemark() {
            if (!this.formData.remark) return '';

            if (this.formData.remark.includes('活动详情')) {
                const linkUrl = this.formData.activeUrl || 'javascript:void(0)';
                const parts = this.formData.remark.split('活动详情');

                return parts[0] +
                    `<a href="${linkUrl}" target="_blank" style="color:#3488ff; text-decoration:underline; cursor:pointer;">活动详情</a>` +
                    (parts[1] || '');
            }

            return this.formData.remark;
        }
    },
    watch: {
        visible(val) {
            this.dialogVisible = val;
            if (val) {
                this.initFormData();
            }
        },
        dialogVisible(val) {
            if (!val) {
                this.$emit('update:visible', false);
            }
        }
    },
    methods: {
        initFormData() {
            this.loading = true;

            // 使用传入的车辆数据
            if (Object.keys(this.vehicleData || {}).length > 0) {
                // 如果数据格式是新的API格式（包含vehicleInfo, policyInfo等），先进行映射
                if (this.vehicleData.vehicleInfo || this.vehicleData.policyInfo) {
                    this.formData = this.mapVehicleData(this.vehicleData);
                } else {
                    // 如果是旧格式的平铺数据，直接深拷贝
                    this.formData = JSON.parse(JSON.stringify(this.vehicleData));
                }

                if (!this.formData.activeUrl && this.vehicleData.activeUrl) {
                    this.formData.activeUrl = this.vehicleData.activeUrl;
                }

                if (this.formData.licensePlate === "新车未上牌") {
                    this.isNewCarWithoutPlate = true;
                    this.formData.licensePlate = ''; // 清空输入框，不显示"新车未上牌"
                } else {
                    this.isNewCarWithoutPlate = false;
                }

                // 确保 insuranceTypes 是一个数组
                if (!this.formData.insuranceTypes) {
                    this.formData.insuranceTypes = [];
                }
            } else {
                // 空数据处理
                this.resetFormData();
            }

            this.loading = false;
        },
        resetFormData() {
            this.formData = {
                // 车辆信息
                licensePlate: '',
                frameNo: '',
                engineNo: '',
                carModel: '',
                registrationDate: '',
                approvedMass: '',
                seatingCapacity: '',
                usageNature: '',
                vehicleType: '',
                energyType: '',
                displacement: '',
                power: '',

                // 商业险信息
                policyNo: '',
                insuranceCompany: '',
                insuranceStartDate: '',
                insuranceEndDate: '',
                vehicleDamageAmount: '',
                thirdPartyAmount: '',
                totalPremium: '',
                feeConfirmTime: '',
                policyCreateTime: '',
                insuranceConfirmTime: '',
                insuranceConfirmCode: '',

                // 投保险种
                insuranceTypes: [],

                // 特别约定
                specialTerms: '',

                // 购车发票
                invoiceNo: '',
                invoiceDate: '',
                totalAmount: '',
                sellerName: '',
                brandModel: ''
            };
        },
        handleClose() {
            this.dialogVisible = false;
            this.$emit('cancel');
        },
        getInsuranceTypeName(code) {
            return insuranceTypeName(code);
        },
        isMainInsurance(code) {
            return isMainInsurance(code);
        },
        // 获取当前可用的险种列表
        availableCvrgList(currentIndex) {
            // 结合新能源和普通险种列表
            let combinedList = [...this.dictionaryData.cvrgList];

            // 如果车辆是新能源汽车，也加入新能源险种
            if (this.formData.energyType === '02') {
                combinedList = [...combinedList, ...this.dictionaryData.newEnergyCvrgList];
            }

            return combinedList;
        },
        viewElectronicForm(type) {
            if (type === 'policy') {
                if (this.formData.policyFileUrl) {
                    window.open(this.formData.policyFileUrl, '_blank');
                } else {
                    this.$message.info('没有可查看的商业险保单');
                }
            } else if (type === 'invoice') {
                if (this.formData.invoiceFileUrl) {
                    window.open(this.formData.invoiceFileUrl, '_blank');
                } else {
                    this.$message.info('没有可查看的购车发票');
                }
            }
        }
    }
};
</script>

<style scoped>
.vehicle-detail-dialog {
    font-size: 14px;
}

.custom-dialog-title {
    width: 100%;
    font-size: 16px;
    font-weight: bold;
    display: flex;
    flex-direction: column;
}

.title-first-row {
    display: flex;
    justify-content: space-between;
    align-items: center;
    width: 100%;
    margin-bottom: 8px;
}

.view-buttons {
    display: flex;
    gap: 10px;
}

.status-notification {
    height: 40px;
    display: flex;
    align-items: center;
    background-color: #FA9550;
    color: #000000;
    padding: 4px 10px;
    border-radius: 4px;
    font-size: 12px;
    font-weight: normal;
    width: 100%;
}

.status-notification i {
    margin-right: 5px;
}

.status-text strong {
    font-size: 14px;
    font-weight: bold;
    margin-right: 3px;
}

.view-form-btn {
    font-size: 12px;
    padding: 7px 12px;
}

.section-header {
    font-size: 16px;
    font-weight: bold;
    margin: 20px 0 15px 0;
    color: #000000E5;
}

/* 投保险种标题和按钮的容器 */
.insurance-types-header {
    display: flex;
    justify-content: flex-start;
    align-items: center;
    margin: 15px 0 15px;
}

/* 投保险种的标题 */
.insurance-types-header .section-header {
    margin: 0;
    padding: 0;
    border-bottom: none;
    margin-right: 15px;
}

.custom-insurance-table {
    margin-bottom: 20px;
    overflow: hidden;
}

.table-header {
    display: flex;
    background-color: #EFEFF0;
    height: 40px;
    line-height: 40px;
    font-weight: bold;
    color: #252628;
}

.header-item,
.row-item {
    flex: 1;
    padding: 0 10px;
    text-align: center;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
}

.table-row {
    display: flex;
    border-top: 1px solid #EBEEF5;
    height: 50px;
    align-items: center;
}

.table-row:first-child {
    border-top: none;
}

/* 表格中的选择框和输入框样式 */
.row-item .el-select {
    width: 80%;
    margin: 0 auto;
}

.row-item .el-input {
    width: 80%;
    margin: 0 auto;
}

/* 设置内部input元素宽度 */
.row-item /deep/ .el-input__inner,
.row-item /deep/ .el-select .el-input__inner {
    text-align: center;
}

.dialog-footer {
    text-align: right;
}

.delete-btn-disabled {
    color: #C0C4CC;
    cursor: not-allowed;
}

.no-data {
    padding: 20px;
    text-align: center;
    color: #909399;
}

/deep/ .el-date-editor.el-input,
/deep/ .el-date-editor.el-input__inner,
/deep/ .el-select {
    width: 100%;
}

/* 表格中的选择框和输入框例外 */
.row-item /deep/ .el-date-editor.el-input,
.row-item /deep/ .el-date-editor.el-input__inner,
.row-item /deep/ .el-select {
    width: 80%;
}

/deep/ .el-dialog__body {
    border: 1px solid #EBEEF5;
    margin: 10px 20px 0px 20px;
    border-radius: 4px;
}

/deep/ .el-form-item {
    margin-bottom: 18px;
}

/deep/ .el-form-item--mini .el-form-item__label {
    line-height: 8px;
}

/deep/ .el-form-item--mini.el-form-item {
    margin-bottom: 20px;
}

/deep/ .el-dialog__header {
    padding: 15px 20px 0px 20px;
}

/deep/ .el-divider--horizontal {
    margin: 0px;
}

.row-item .el-form-item {
    margin-bottom: 0;
}
</style>