<template>
    <el-dialog :visible.sync="dialogVisible" width="80%" :close-on-click-modal="false" class="insurance-policy-dialog"
        :title="null" :show-close="false" v-loading="saveLoading">
        <div slot="title" class="custom-dialog-title">
            <div class="title-first-row">
                <span>{{ isViewMode ? '保单详情' : '编辑保单' }}</span>
                <el-button type="primary" size="small" class="view-form-btn"
                    @click="viewElectronicForm">查看电子保单</el-button>
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
            <el-form ref="entireForm" :model="formData" :rules="formRules" label-width="100px" :disabled="isViewMode"
                label-position="top">
                <!-- 车辆信息 section -->
                <div class="section-header">车辆信息</div>
                <el-row :gutter="20">
                    <el-col :span="6">
                        <el-form-item label="车牌号" prop="licensePlate">
                            <el-input v-model="formData.licensePlate" placeholder="请输入" maxlength="8"
                                @input="handleLicensePlateInput" :disabled="isNewCarWithoutPlate">
                                <template slot="append">
                                    <el-checkbox v-model="isNewCarWithoutPlate"
                                        @change="handleNewCarCheckboxChange">新车未上牌</el-checkbox>
                                </template>
                            </el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="车架号" prop="frameNo">
                            <el-input v-model="formData.frameNo" placeholder="请输入" maxlength="17"
                                @input="handleFrameNoInput">
                            </el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="发动机号" prop="engineNo">
                            <el-input v-model="formData.engineNo" placeholder="请输入" maxlength="20"
                                @input="handleEngineNoInput">
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
                        <el-form-item label="初登日期" prop="registrationDate">
                            <el-date-picker v-model="formData.registrationDate" type="date" placeholder="选择日期"
                                format="yyyy-MM-dd" value-format="yyyy-MM-dd"></el-date-picker>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="核定载质量(kg)">
                            <el-input v-model="formData.approvedMass" placeholder="请输入"
                                @input="handleApprovedMassInput">
                            </el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="核定载客(人)">
                            <el-input v-model="formData.seatingCapacity" placeholder="请输入"
                                @input="handleSeatingCapacityInput">
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
                            <el-input v-model="formData.displacement" placeholder="请输入"
                                @input="handleDisplacementInput">
                            </el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="功率(kw)">
                            <el-input v-model="formData.power" placeholder="请输入" @input="handlePowerInput">
                            </el-input>
                        </el-form-item>
                    </el-col>
                </el-row>

                <el-divider></el-divider>

                <!-- 保单信息 section -->
                <div class="section-header">保单信息</div>
                <el-row :gutter="20">
                    <el-col :span="6">
                        <el-form-item label="保单类型" prop="policyType">
                            <el-radio-group v-model="formData.policyType">
                                <el-radio label="1">交强险</el-radio>
                                <el-radio label="0">商业险</el-radio>
                            </el-radio-group>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="保险公司名称" prop="insuranceCompany">
                            <el-select v-model="formData.insuranceCompany" filterable placeholder="请选择">
                                <el-option v-for="item in insuranceCompanies" :key="item.value" :label="item.label"
                                    :value="item.value">
                                </el-option>
                            </el-select>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="保单号" prop="policyNo">
                            <el-input v-model="formData.policyNo" maxlength="50" placeholder="请输入"></el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="保险起期" prop="insuranceStartDate">
                            <el-date-picker v-model="formData.insuranceStartDate" type="datetime" placeholder="选择日期时间"
                                format="yyyy-MM-dd HH:mm:ss" value-format="yyyy-MM-dd HH:mm:ss"
                                :picker-options="startDatePickerOptions" @change="handleStartDateChange">
                            </el-date-picker>
                        </el-form-item>
                    </el-col>
                </el-row>

                <el-row :gutter="20">
                    <el-col :span="6">
                        <el-form-item label="保险止期" prop="insuranceEndDate">
                            <el-date-picker v-model="formData.insuranceEndDate" type="datetime" placeholder="请先选择保险起期"
                                format="yyyy-MM-dd HH:mm:ss" value-format="yyyy-MM-dd HH:mm:ss"
                                :disabled="!formData.insuranceStartDate" :picker-options="endDatePickerOptions"
                                @focus="onEndDatePickerOpen" @change="handleEndDateChange"
                                @calendar-change="onEndDateSelect">
                            </el-date-picker>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="车损险保额(元)" prop="vehicleDamageAmount">
                            <el-input v-model="formData.vehicleDamageAmount" placeholder="请输入"
                                @input="handleVehicleDamageAmountInput">
                            </el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="三者险保额(元)" prop="thirdPartyAmount">
                            <!-- <el-select v-model="formData.thirdPartyAmount" placeholder="请选择" :disabled="isViewMode">
                                <el-option v-for="option in thirdPartyAmountOptions" :key="option.dicCode"
                                    :label="option.dicName" :value="option.dicCode">
                                </el-option>
                            </el-select> -->
                            <el-input v-model="formData.thirdPartyAmount" placeholder="请输入" maxlength="20">
                            </el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="保费合计(元)" prop="totalPremium">
                            <el-input v-model="formData.totalPremium" placeholder="请输入"
                                @input="handleTotalPremiumInput">
                            </el-input>
                        </el-form-item>
                    </el-col>
                </el-row>


                <el-row :gutter="20">
                    <el-col :span="6" v-if="formData.policyType === '1'">
                        <el-form-item label="车船税">
                            <el-input v-model="formData.travelTax" placeholder="请输入" @input="handletravelTax">
                            </el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6" v-else>
                        <el-form-item label="保单打印时间">
                            <el-date-picker v-model="formData.policyPrintTime" type="date" placeholder="选择日期时间"
                                format="yyyy-MM-dd" value-format="yyyy-MM-dd"></el-date-picker>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="收费确认时间" prop="feeConfirmTime">
                            <el-date-picker v-model="formData.feeConfirmTime" type="datetime" placeholder="选择日期时间"
                                format="yyyy-MM-dd HH:mm:ss" value-format="yyyy-MM-dd HH:mm:ss"></el-date-picker>
                        </el-form-item>
                    </el-col>
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
                </el-row>

                <el-row :gutter="20">
                    <el-col :span="6" v-if="formData.policyType === '1'">
                        <el-form-item label="保单打印时间">
                            <el-date-picker v-model="formData.policyPrintTime" type="date" placeholder="选择日期时间"
                                format="yyyy-MM-dd" value-format="yyyy-MM-dd"></el-date-picker>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="投保确认码">
                            <el-input v-model="formData.insuranceConfirmCode" maxlength="50"
                                placeholder="请输入"></el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="保单验真码">
                            <el-input v-model="formData.policyVerificationCode" maxlength="50"
                                placeholder="请输入"></el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="银行流水号">
                            <el-input v-model="formData.bankTransactionNo" maxlength="50" placeholder="请输入"></el-input>
                        </el-form-item>
                    </el-col>
                </el-row>

                <el-divider></el-divider>

                <!-- 投保险种 section - 仅在商业险模式下显示 -->
                <template v-if="formData.policyType === '0'">
                    <div class="insurance-types-header">
                        <div class="section-header">投保险种</div>
                        <el-button v-if="!isViewMode" type="primary" size="small" @click="addInsuranceType"
                            class="add-insurance-btn">
                            <i class="el-icon-plus"></i> 添加险种
                        </el-button>
                    </div>

                    <div class="custom-insurance-table">
                        <!-- 表头 -->
                        <div class="table-header">
                            <div class="header-item">险种名称</div>
                            <div class="header-item">类型</div>
                            <div class="header-item">保额</div>
                            <div class="header-item">保费(元)</div>
                            <div class="header-item" v-if="!isViewMode">操作</div>
                        </div>

                        <!-- 表格内容 -->
                        <div v-for="(item, index) in formData.insuranceTypes" :key="index" class="table-row">
                            <div class="row-item">
                                <el-select v-model="item.name" placeholder="请选择" :disabled="isViewMode"
                                    @change="() => handleCvrgChange(index)">
                                    <el-option v-for="cvrgItem in availableCvrgList(index)" :key="cvrgItem.dicCode"
                                        :label="isInsuranceTypeSelected(cvrgItem.dicCode, index) ? cvrgItem.dicName + ' (已选)' : cvrgItem.dicName"
                                        :value="cvrgItem.dicCode"
                                        :disabled="isInsuranceTypeSelected(cvrgItem.dicCode, index)">
                                    </el-option>
                                </el-select>
                            </div>
                            <div class="row-item">{{ item.name ? getInsuranceTypeName(item.name) : '' }}</div>
                            <div class="row-item">
                                <el-form-item :prop="`insuranceTypes.${index}.amount`"
                                    :rules="insuranceTypeRules[`insuranceTypes.${index}.amount`]">
                                    <!-- 如果险种有子项选项，显示下拉框 -->
                                    <!-- <el-select v-if="hasCvrgChildren(item.name)" v-model="item.amount"
                                        placeholder="请选择保额" :disabled="isViewMode">
                                        <el-option v-for="child in getCvrgChildren(item.name)" :key="child.dicCode"
                                            :label="child.dicName" :value="child.dicCode">
                                        </el-option>
                                    </el-select> -->

                                    <!-- 否则显示普通输入框，添加输入验证 -->
                                    <el-input v-model="item.amount" placeholder="请输入保额" :disabled="isViewMode"
                                        maxlength="20" @input="() => handleAmountInput(index)">
                                    </el-input>
                                </el-form-item>
                            </div>
                            <div class="row-item">
                                <el-form-item :prop="`insuranceTypes.${index}.premium`"
                                    :rules="insuranceTypeRules[`insuranceTypes.${index}.premium`]">
                                    <el-input v-model="item.premium" placeholder="请输入保费" :disabled="isViewMode"
                                        @input="() => handlePremiumInput(index)">
                                    </el-input>
                                </el-form-item>
                            </div>
                            <div class="row-item" v-if="!isViewMode">
                                <el-button type="text" @click="removeInsuranceType(index)" class="delete-btn"
                                    :disabled="isViewMode">删除</el-button>
                            </div>
                        </div>

                        <!-- 没有数据时的提示 -->
                        <div v-if="formData.insuranceTypes.length === 0" class="no-data">
                            请添加险种
                        </div>
                    </div>
                </template>

                <el-divider></el-divider>

                <!-- 特别约定 section -->
                <div class="section-header">特别约定</div>
                <el-form-item>
                    <el-input type="textarea" v-model="formData.specialTerms" :rows="4" maxlength="500"
                        placeholder="请输入特别约定内容"></el-input>
                </el-form-item>

                <el-divider></el-divider>

                <!-- DMO字段信息 section -->
                <div class="section-header">DMO字段信息</div>
                <el-row :gutter="20">
                    <el-col :span="6">
                        <el-form-item label="实际跟进人">
                            <el-input v-model="formData.actualDriver" maxlength="50" placeholder="请输入"></el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="车辆ID">
                            <el-input v-model="formData.vehicleId" maxlength="50" placeholder="请输入"></el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="个人信息保护政策">
                            <el-select v-model="formData.personalInfoPolicy" placeholder="请选择">
                                <el-option v-for="item in dictionaryData.premiumPolicyList" :key="item.dicCode"
                                    :label="item.dicName" :value="item.dicCode">
                                </el-option>
                            </el-select>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="去年投保公司">
                            <el-input v-model="formData.lastYearInsurer" maxlength="50" placeholder="请输入"></el-input>
                        </el-form-item>
                    </el-col>
                </el-row>

                <el-row :gutter="20">
                    <!-- <el-col :span="6">
                        <el-form-item label="业务类别">
                            <el-select v-model="formData.businessType" placeholder="请选择">
                                <el-option v-for="item in dictionaryData.busTypeList" :key="item.dicCode"
                                    :label="item.dicName" :value="item.dicCode">
                                </el-option>
                            </el-select>
                        </el-form-item>
                    </el-col> -->
                    <el-col :span="12">
                        <el-form-item label="是否购买宝马事故维修补偿产品">
                            <el-radio-group v-model="formData.hasBMWAccidentRepairProduct">
                                <el-radio v-for="item in dictionaryData.bmwArcList" :key="item.dicCode"
                                    :label="item.dicCode">
                                    {{ item.dicName }}
                                </el-radio>
                            </el-radio-group>
                        </el-form-item>
                    </el-col>
                </el-row>
            </el-form>
        </div>

        <div slot="footer" class="dialog-footer">
    <div class="footer-content">
        <div class="remark-section">
            <div class="remark-label">保单备注</div>
            <el-input v-model="formData.policyRemark" placeholder="请输入备注信息" class="remark-input"
                :disabled="isViewMode" maxlength="200" show-word-limit></el-input>
        </div>
        <div class="button-section">
            <el-button @click="handleClose">{{ isViewMode ? '关闭' : '取消' }}</el-button>
            <el-button v-if="!isViewMode" type="primary" @click="savePolicy" :loading="saveLoading">保存</el-button>
        </div>
    </div>
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
        mode: {
            type: String,
            default: 'edit', // 'edit' or 'view'
            validator: (value) => ['edit', 'view'].includes(value)
        },
        policyData: {
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
                premiumPolicyList: [],
                busTypeList: [],
                bmwArcList: [],
                cvrgList: [],
                newEnergyCvrgList: []
            })
        }
    },
    data() {
        return {
            isNewCarWithoutPlate: false,
            startDatePickerOptions: {
                disabledDate: (time) => {
                    // 这里可以添加保险起期的限制规则（但是不用好像-存着万一呢）
                    return false; // 默认无限制
                }
            },
            endDatePickerOptions: {
                disabledDate: (time) => {
                    if (!this.formData.insuranceStartDate) {
                        return true; // 如果没有选择起期，禁用所有日期
                    }

                    // 获取起期的时间戳
                    const startDate = new Date(this.formData.insuranceStartDate);

                    // 当前日历上显示日期的时间戳
                    const currentDate = new Date(time);

                    // 比较时需要只比较日期部分
                    currentDate.setHours(0, 0, 0, 0);
                    const startDateOnly = new Date(startDate);
                    startDateOnly.setHours(0, 0, 0, 0);

                    // 禁用所有小于起期当天的日期
                    return currentDate.getTime() < startDateOnly.getTime();
                },
            },
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

                // 保单信息
                policyType: '交强险',
                insuranceCompany: '',
                policyNo: '',
                insuranceStartDate: '',
                insuranceEndDate: '',
                vehicleDamageAmount: '',
                thirdPartyAmount: '',
                totalPremium: '',
                travelTax: '',
                feeConfirmTime: '',
                policyCreateTime: '',
                insuranceConfirmTime: '',
                policyPrintTime: '',
                insuranceConfirmCode: '',
                policyVerificationCode: '',
                bankTransactionNo: '',
                remark: '',

                // 投保险种
                insuranceTypes: [],

                // 特别约定
                specialTerms: '',

                // DMO字段信息
                actualDriver: '',
                vehicleId: '',
                personalInfoPolicy: '',
                lastYearInsurer: '',
                businessType: '',
                hasBMWAccidentRepairProduct: '',
                policyRemark: ''
            },
            insuranceTypeRules: {}
        };
    },
    computed: {
        formRules() {
            // 基础规则，不会随保单类型变化的规则
            const baseRules = {
                licensePlate: [
                    {
                        required: true, message: '请输入车牌号', trigger: 'blur', validator: (rule, value, callback) => {
                            if (this.isNewCarWithoutPlate || value) {
                                callback(); // 勾选了新车未上牌或有输入值，验证通过
                            } else {
                                callback(new Error('请输入车牌号'));
                            }
                        }
                    }
                ],
                frameNo: [
                    { required: true, message: '请输入车架号', trigger: 'blur' }
                ],
                registrationDate: [
                    { required: true, message: '请选择初登日期', trigger: 'change' }
                ],
                policyType: [
                    { required: true, message: '请选择保单类型', trigger: 'change' }
                ],
                insuranceCompany: [
                    { required: true, message: '请选择保险公司名称', trigger: 'change' }
                ],
                policyNo: [
                    { required: true, message: '请输入保单号', trigger: 'blur' }
                ],
                insuranceStartDate: [
                    { required: true, message: '请选择保险起期', trigger: 'change' }
                ],
                insuranceEndDate: [
                    { required: true, message: '请选择保险止期', trigger: 'change' }
                ],
                totalPremium: [
                    { required: true, message: '请输入保费合计', trigger: 'blur' }
                ],
                feeConfirmTime: [
                    { required: true, message: '请选择收费确认时间', trigger: 'change' }
                ],
                personalInfoPolicy: [
                    { required: true, message: '请选择个人信息保费政策', trigger: 'change' }
                ],
            };

            // 根据保单类型决定车损险和三者险是否必填
            if (this.formData.policyType === '1') { // 交强险
                return {
                    ...baseRules,
                    // 交强险模式下这两个字段不是必填的，设置为空数组或非必填规则
                    vehicleDamageAmount: [],
                    thirdPartyAmount: []
                };
            } else { // 商业险
                return {
                    ...baseRules,
                    vehicleDamageAmount: [
                        { required: true, message: '请输入车损险保额', trigger: 'blur' }
                    ],
                    thirdPartyAmount: [
                        { required: true, message: '请输入三者险保额', trigger: 'blur' }
                    ]
                };
            }
        },
        isViewMode() {
            return this.mode === 'view';
        },
        thirdPartyAmountOptions() {
            const thirdPartyInsurance = this.dictionaryData.cvrgList.find(item => item.dicCode === "0301600");

            // 如果找到并且有children属性，返回其children，否则返回空数组
            return (thirdPartyInsurance && thirdPartyInsurance.children) ? thirdPartyInsurance.children : [];
        },
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
        },
        'formData.insuranceStartDate': function (newVal) {
            if (!newVal && this.formData.insuranceEndDate) {
                this.formData.insuranceEndDate = '';
            }
        },
        'formData.insuranceTypes': {
            handler() {
                this.$nextTick(() => {
                    this.updateInsuranceTypeRules();
                });
            },
            deep: true
        },
        'formData.policyType': function (newVal) {
            this.$nextTick(() => {
                if (this.$refs.entireForm) {
                    // 清除之前的验证结果
                    this.$refs.entireForm.clearValidate(['vehicleDamageAmount', 'thirdPartyAmount']);
                }
            });
        }
    },
    methods: {
        initFormData() {
            this.loading = true;

            // 使用传入的保单数据
            if (Object.keys(this.policyData || {}).length > 0) {
                // 深拷贝保单数据，避免直接修改props
                this.formData = JSON.parse(JSON.stringify(this.policyData));

                if (!this.formData.activeUrl && this.policyData.activeUrl) {
                    this.formData.activeUrl = this.policyData.activeUrl;
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
                // 编辑模式使用空数据
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

                // 保单信息
                policyType: '交强险',
                insuranceCompany: '',
                policyNo: '',
                insuranceStartDate: '',
                insuranceEndDate: '',
                vehicleDamageAmount: '',
                thirdPartyAmount: '',
                totalPremium: '',
                feeConfirmTime: '',
                travelTax: '',
                policyCreateTime: '',
                insuranceConfirmTime: '',
                policyPrintTime: '',
                insuranceConfirmCode: '',
                policyVerificationCode: '',
                bankTransactionNo: '',
                remark: '',

                // 投保险种
                insuranceTypes: [],

                // 特别约定
                specialTerms: '',

                // DMO字段信息
                actualDriver: '',
                vehicleId: '',
                personalInfoPolicy: '',
                lastYearInsurer: '',
                businessType: '',
                hasBMWAccidentRepairProduct: ''
            };
        },
        handleNewCarCheckboxChange(val) {
            if (val) {
                // 如果勾选了"新车未上牌"，清空车牌号
                this.formData.licensePlate = '';
                // 重新验证表单，确保清除错误提示
                this.$nextTick(() => {
                    this.$refs.entireForm.validateField('licensePlate');
                });
            }
        },
        // 处理车牌号输入 - 允许中文、大写字母和数字，强制转换小写字母为大写
        handleLicensePlateInput() {
            if (this.formData.licensePlate) {
                // 将小写字母转换为大写字母
                this.formData.licensePlate = this.formData.licensePlate.toUpperCase();

                // 过滤非法字符 (只保留中文、大写字母和数字)
                // this.formData.licensePlate = this.formData.licensePlate.replace(/[^A-Z0-9\u4e00-\u9fa5]/g, '');
            }
        },
        // 处理车架号输入 - 只允许大写字母和数字，强制转换小写字母为大写
        handleFrameNoInput() {
            if (this.formData.frameNo) {
                // 将小写字母转换为大写字母
                this.formData.frameNo = this.formData.frameNo.toUpperCase();

                // 过滤非法字符 (只保留大写字母和数字)
                // this.formData.frameNo = this.formData.frameNo.replace(/[^A-Z0-9]/g, '');
            }
        },
        // 处理发动机号输入 - 只允许大写字母和数字，强制转换小写字母为大写
        handleEngineNoInput() {
            if (this.formData.engineNo) {
                // 将小写字母转换为大写字母
                this.formData.engineNo = this.formData.engineNo.toUpperCase();

                // 过滤非法字符 (只保留大写字母和数字)
                // this.formData.engineNo = this.formData.engineNo.replace(/[^A-Z0-9]/g, '');
            }
        },
        // 处理核定载质量输入 - 只允许数字，大于等于0，最大99999
        handleApprovedMassInput() {
            if (this.formData.approvedMass === '') return;

            // 过滤非数字字符
            let value = this.formData.approvedMass.replace(/[^\d]/g, '');

            // 转为数字并限制范围
            let numValue = parseInt(value, 10);
            if (isNaN(numValue)) {
                numValue = '';
            } else if (numValue > 99999) {
                numValue = 99999;
            }

            this.formData.approvedMass = numValue.toString();
        },

        // 处理核定载客输入 - 只允许数字，大于等于0，最大100
        handleSeatingCapacityInput() {
            if (this.formData.seatingCapacity === '') return;

            // 过滤非数字字符
            let value = this.formData.seatingCapacity.replace(/[^\d]/g, '');

            // 转为数字并限制范围
            let numValue = parseInt(value, 10);
            if (isNaN(numValue)) {
                numValue = '';
            } else if (numValue > 100) {
                numValue = 100;
            }

            this.formData.seatingCapacity = numValue.toString();
        },

        // 处理排量输入 - 只允许数字和小数点，大于等于0，最大99999，最多5位小数
        handleDisplacementInput() {
            if (this.formData.displacement === '') return;

            // 允许数字和小数点，且只允许一个小数点
            let value = this.formData.displacement;
            if (!/^\d*\.?\d*$/.test(value)) {
                value = value.replace(/[^\d.]/g, '');
                // 确保只有一个小数点
                const parts = value.split('.');
                value = parts[0] + (parts.length > 1 ? '.' + parts.slice(1).join('') : '');
            }

            // 检查数值范围
            let numValue = parseFloat(value);
            if (!isNaN(numValue)) {
                if (numValue > 99999) {
                    numValue = 99999;
                    value = numValue.toString();
                }

                // 限制小数位数
                const parts = value.split('.');
                if (parts.length > 1 && parts[1].length > 5) {
                    parts[1] = parts[1].substring(0, 5);
                    value = parts[0] + '.' + parts[1];
                }
            }

            this.formData.displacement = value;
        },

        // 处理功率输入 - 只允许数字和小数点，大于等于0，最大99999，最多5位小数
        handlePowerInput() {
            if (this.formData.power === '') return;

            // 允许数字和小数点，且只允许一个小数点
            let value = this.formData.power;
            if (!/^\d*\.?\d*$/.test(value)) {
                value = value.replace(/[^\d.]/g, '');
                // 确保只有一个小数点
                const parts = value.split('.');
                value = parts[0] + (parts.length > 1 ? '.' + parts.slice(1).join('') : '');
            }

            // 检查数值范围
            let numValue = parseFloat(value);
            if (!isNaN(numValue)) {
                if (numValue > 99999) {
                    numValue = 99999;
                    value = numValue.toString();
                }

                // 限制小数位数
                const parts = value.split('.');
                if (parts.length > 1 && parts[1].length > 5) {
                    parts[1] = parts[1].substring(0, 5);
                    value = parts[0] + '.' + parts[1];
                }
            }

            this.formData.power = value;
        },
        // 处理车损险保额输入 - 只允许数字和小数点，大于等于0，最大999999，最多2位小数
        handleVehicleDamageAmountInput() {
            if (this.formData.vehicleDamageAmount === '') return;

            // 允许数字和小数点，且只允许一个小数点
            let value = this.formData.vehicleDamageAmount;
            if (!/^\d*\.?\d*$/.test(value)) {
                value = value.replace(/[^\d.]/g, '');
                // 确保只有一个小数点
                const parts = value.split('.');
                value = parts[0] + (parts.length > 1 ? '.' + parts.slice(1).join('') : '');
            }

            // 检查数值范围
            let numValue = parseFloat(value);
            if (!isNaN(numValue)) {
                if (numValue > 999999) {
                    numValue = 999999;
                    value = numValue.toString();
                }

                // 限制小数位数
                const parts = value.split('.');
                if (parts.length > 1 && parts[1].length > 2) {
                    parts[1] = parts[1].substring(0, 2);
                    value = parts[0] + '.' + parts[1];
                }
            }

            this.formData.vehicleDamageAmount = value;
        },

        // 处理三者险保额输入 - 只允许数字和小数点，大于等于0，最大999999，最多2位小数
        handleThirdPartyAmountInput() {
            if (this.formData.thirdPartyAmount === '') return;

            // 允许数字和小数点，且只允许一个小数点
            let value = this.formData.thirdPartyAmount;
            if (!/^\d*\.?\d*$/.test(value)) {
                value = value.replace(/[^\d.]/g, '');
                // 确保只有一个小数点
                const parts = value.split('.');
                value = parts[0] + (parts.length > 1 ? '.' + parts.slice(1).join('') : '');
            }

            // 检查数值范围
            let numValue = parseFloat(value);
            if (!isNaN(numValue)) {
                if (numValue > 999999) {
                    numValue = 999999;
                    value = numValue.toString();
                }

                // 限制小数位数
                const parts = value.split('.');
                if (parts.length > 1 && parts[1].length > 2) {
                    parts[1] = parts[1].substring(0, 2);
                    value = parts[0] + '.' + parts[1];
                }
            }

            this.formData.thirdPartyAmount = value;
        },

        handleAmountInput(index) {
            const item = this.formData.insuranceTypes[index];

            if (item.amount) {
                this.$set(item, 'amountError', false);
            }
        },

        // 处理保费输入 - 只允许数字和小数点，大于等于0，最大999999，最多2位小数
        handlePremiumInput(index) {
            const item = this.formData.insuranceTypes[index];
            if (item.premium === '') return;

            if (item.premium) {
                this.$set(item, 'premiumError', false);
            }

            // 允许数字和小数点，且只允许一个小数点
            let value = item.premium;
            if (!/^\d*\.?\d*$/.test(value)) {
                value = value.replace(/[^\d.]/g, '');
                // 确保只有一个小数点
                const parts = value.split('.');
                value = parts[0] + (parts.length > 1 ? '.' + parts.slice(1).join('') : '');
            }

            // 检查数值范围
            let numValue = parseFloat(value);
            if (!isNaN(numValue)) {
                if (numValue > 999999) {
                    numValue = 999999;
                    value = numValue.toString();
                }

                // 限制小数位数
                const parts = value.split('.');
                if (parts.length > 1 && parts[1].length > 2) {
                    parts[1] = parts[1].substring(0, 2);
                    value = parts[0] + '.' + parts[1];
                }
            }

            // 更新保费值
            this.$set(this.formData.insuranceTypes[index], 'premium', value);
        },
        // 处理保费合计输入 - 只允许数字和小数点，大于等于0，最大999999，最多2位小数
        handleTotalPremiumInput() {
            if (this.formData.totalPremium === '') return;

            // 允许数字和小数点，且只允许一个小数点
            let value = this.formData.totalPremium;
            if (!/^\d*\.?\d*$/.test(value)) {
                value = value.replace(/[^\d.]/g, '');
                // 确保只有一个小数点
                const parts = value.split('.');
                value = parts[0] + (parts.length > 1 ? '.' + parts.slice(1).join('') : '');
            }

            // 检查数值范围
            let numValue = parseFloat(value);
            if (!isNaN(numValue)) {
                if (numValue > 999999) {
                    numValue = 999999;
                    value = numValue.toString();
                }

                // 限制小数位数
                const parts = value.split('.');
                if (parts.length > 1 && parts[1].length > 2) {
                    parts[1] = parts[1].substring(0, 2);
                    value = parts[0] + '.' + parts[1];
                }
            }

            this.formData.totalPremium = value;
        },
        // 处理车船税输入 - 只允许数字和小数点，大于等于0，最大999999，最多2位小数
        handletravelTax() {
            if (this.formData.travelTax === '') return;

            // 允许数字和小数点，且只允许一个小数点
            let value = this.formData.travelTax;
            if (!/^\d*\.?\d*$/.test(value)) {
                value = value.replace(/[^\d.]/g, '');
                // 确保只有一个小数点
                const parts = value.split('.');
                value = parts[0] + (parts.length > 1 ? '.' + parts.slice(1).join('') : '');
            }

            // 检查数值范围
            let numValue = parseFloat(value);
            if (!isNaN(numValue)) {
                if (numValue > 999999) {
                    numValue = 999999;
                    value = numValue.toString();
                }

                // 限制小数位数
                const parts = value.split('.');
                if (parts.length > 1 && parts[1].length > 2) {
                    parts[1] = parts[1].substring(0, 2);
                    value = parts[0] + '.' + parts[1];
                }
            }

            this.formData.travelTax = value;
        },
        handleStartDateChange(val) {
            // 当起期改变时，更新止期的时间范围限制
            if (val) {
                const startDate = new Date(val);

                // 首先清空止期值
                if (this.formData.insuranceEndDate) {
                    const endDate = new Date(this.formData.insuranceEndDate);
                    if (endDate <= startDate) {
                        this.formData.insuranceEndDate = '';
                    }
                }

                // 更新止期的可选时间范围
                this.updateEndDateTimeRange(startDate);
            }
        },
        updateEndDateTimeRange(startDate) {
            // 提取起期的时间部分
            const hours = startDate.getHours();
            const minutes = startDate.getMinutes();
            const seconds = startDate.getSeconds();

            // 计算当天的可选时间范围
            const minTimeStr = `${hours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`;

            // 对于止期，如果选择的是起期当天，则时间必须晚于起期时间
            this.$set(this.endDatePickerOptions, 'selectableRange', `${minTimeStr} - 23:59:59`);
        },
        onEndDatePickerOpen() {
            if (this.formData.insuranceStartDate) {
                const startDate = new Date(this.formData.insuranceStartDate);
                const endDate = this.formData.insuranceEndDate ? new Date(this.formData.insuranceEndDate) : null;

                // 仅当选择的是同一天时应用时间范围限制
                if (endDate && this.isSameDay(startDate, endDate)) {
                    this.updateEndDateTimeRange(startDate);
                } else {
                    // 如果不是同一天，则不限制时间范围
                    this.$set(this.endDatePickerOptions, 'selectableRange', '00:00:00 - 23:59:59');
                }
            }
        },
        isSameDay(date1, date2) {
            return date1.getFullYear() === date2.getFullYear() &&
                date1.getMonth() === date2.getMonth() &&
                date1.getDate() === date2.getDate();
        },
        handleEndDateChange(val) {
            if (!val || !this.formData.insuranceStartDate) return;

            const startDate = new Date(this.formData.insuranceStartDate);
            const endDate = new Date(val);

            // 检查是否是同一天
            if (this.isSameDay(startDate, endDate)) {
                // 如果止期时间早于起期时间，自动调整止期时间
                if (endDate.getTime() <= startDate.getTime()) {
                    // 创建一个新的日期，设置为起期时间后1小时
                    const newEndDate = new Date(startDate.getTime());
                    newEndDate.setHours(startDate.getHours() + 1);

                    // 格式化为所需格式并更新formData
                    const year = newEndDate.getFullYear();
                    const month = (newEndDate.getMonth() + 1).toString().padStart(2, '0');
                    const day = newEndDate.getDate().toString().padStart(2, '0');
                    const hours = newEndDate.getHours().toString().padStart(2, '0');
                    const minutes = newEndDate.getMinutes().toString().padStart(2, '0');
                    const seconds = newEndDate.getSeconds().toString().padStart(2, '0');

                    // 更新表单数据
                    this.formData.insuranceEndDate = `${year}-${month}-${day} ${hours}:${minutes}:${seconds}`;

                    // 显示提示信息
                    this.$message.info('已自动调整止期时间为起期后1小时');
                }
            }
        },
        onEndDateSelect(val) {
            if (!val) return;

            // 如果只选择了日期，还没选择时间，这时val只包含日期部分
            if (val.indexOf(' ') === -1) {
                const startDate = new Date(this.formData.insuranceStartDate);
                const selectedDate = new Date(val);

                // 检查是否是同一天
                if (this.isSameDay(startDate, selectedDate)) {
                    // 获取起期的时间部分
                    const hours = startDate.getHours().toString().padStart(2, '0');
                    const minutes = startDate.getMinutes().toString().padStart(2, '0');
                    const seconds = startDate.getSeconds().toString().padStart(2, '0');

                    // 默认设置为起期后1小时
                    let newHours = parseInt(hours) + 1;
                    if (newHours > 23) newHours = 23;

                    // 更新止期为起期后1小时
                    this.formData.insuranceEndDate = `${val} ${newHours.toString().padStart(2, '0')}:${minutes}:${seconds}`;
                }
            }
        },
        hasCvrgChildren(cvrgCode) {
            if (!cvrgCode) return false;

            // 在普通险种列表中查找
            const normalCvrg = this.dictionaryData.cvrgList.find(item => item.dicCode === cvrgCode);
            if (normalCvrg && normalCvrg.children && normalCvrg.children.length > 0) {
                return true;
            }

            // 在新能源险种列表中查找
            const newEnergyCvrg = this.dictionaryData.newEnergyCvrgList.find(item => item.dicCode === cvrgCode);
            if (newEnergyCvrg && newEnergyCvrg.children && newEnergyCvrg.children.length > 0) {
                return true;
            }

            return false;
        },
        getCvrgChildren(cvrgCode) {
            if (!cvrgCode) return [];

            // 在普通险种列表中查找
            const normalCvrg = this.dictionaryData.cvrgList.find(item => item.dicCode === cvrgCode);
            if (normalCvrg && normalCvrg.children && normalCvrg.children.length > 0) {
                return normalCvrg.children;
            }

            // 在新能源险种列表中查找
            const newEnergyCvrg = this.dictionaryData.newEnergyCvrgList.find(item => item.dicCode === cvrgCode);
            if (newEnergyCvrg && newEnergyCvrg.children && newEnergyCvrg.children.length > 0) {
                return newEnergyCvrg.children;
            }

            return [];
        },
        // 处理险种选择变化
        handleCvrgChange(index) {
            const item = this.formData.insuranceTypes[index];
            // 更新险种类型
            item.type = item.name;

            // 如果选择的险种有子项，且当前没有设置保额，则清空保额
            if (this.hasCvrgChildren(item.name) && !this.getCvrgChildren(item.name).some(child => child.dicCode === item.amount)) {
                item.amount = '';
            }

            // 更新验证规则
            this.updateInsuranceTypeRules();
        },
        handleClose() {
            // 重置表单验证状态和字段值
            this.$refs.entireForm.clearValidate();
            // 关闭对话框
            // this.dialogVisible = false;
            this.$emit('cancel');
        },
        addInsuranceType() {
            if (this.formData.insuranceTypes.length >= 30) {
                this.$message.warning('最多只能添加30个险种');
                return;
            }
            // 添加新的险种选项
            this.formData.insuranceTypes.push({
                name: '', // 险种代码
                type: '', // 将自动根据name设置
                amount: '', // 保额
                premium: '' // 保费
            });
        },
        getCvrgDisplayText(cvrgItem, currentIndex) {
            const isSelected = this.isInsuranceTypeSelected(cvrgItem.dicCode, currentIndex);
            return cvrgItem.dicName + (isSelected ? ' (已选)' : '');
        },
        removeInsuranceType(index) {
            this.formData.insuranceTypes.splice(index, 1);
        },
        getInsuranceTypeName(code) {
            return insuranceTypeName(code);
        },

        isMainInsurance(code) {
            return isMainInsurance(code);
        },

        getInsuranceTypeText(type) {
            // 如果没有传入code或者传入的是主险/附加险字符串，使用默认处理
            if (!type || type === 'main' || type === 'additional') {
                const typeMap = {
                    main: '主险',
                    additional: '附加险'
                };
                return typeMap[type] || type;
            }

            return this.getInsuranceTypeName(type);
        },
        // 判断险种是否已被选中
        isInsuranceTypeSelected(code, currentIndex) {
            if (!code) return false;

            // 检查该险种是否在其他项中已被选择
            return this.formData.insuranceTypes.some((item, index) => {
                // 排除当前项自身
                return index !== currentIndex && item.name === code;
            });
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
        savePolicy() {
            // 过滤掉没有选择险种名称的行
            this.formData.insuranceTypes = this.formData.insuranceTypes.filter(item => item.name);

            // 更新险种验证规则
            this.updateInsuranceTypeRules();

            // 创建一个包含所有规则的对象
            const allRules = { ...this.rules, ...this.insuranceTypeRules };

            // 验证表单
            this.$refs.entireForm.validate(valid => {
                if (valid) {
                    this.saveLoading = true;
                    const apiData = this.prepareApiData();

                    this.$https('/activityPolicy/updateActivityPolicyInfo', {
                        body: apiData
                    }).then(response => {
                        if (response && response.header && response.header.code === '10000') {
                            this.$emit('save', this.formData);
                            this.dialogVisible = false;
                        } else if (!response) {
                            this.$message.error('保存失败');
                        }
                    }).catch(error => {
                        console.error('保存失败:', error);
                        this.$message.error('保存失败');
                    }).finally(() => {
                        // 无论成功或失败，都关闭loading状态
                        this.saveLoading = false;
                    });
                } else {
                    this.$message.error('请填写所有必填项');
                    return false;
                }
            });
        },

        validateInsuranceTypes() {
            if (this.formData.insuranceTypes.length === 0) {
                return true;
            }

            let isValid = true;

            for (let i = 0; i < this.formData.insuranceTypes.length; i++) {
                const item = this.formData.insuranceTypes[i];

                if (item.name) {
                    if (!item.amount) {
                        this.$set(item, 'amountError', true);
                        isValid = false;
                    } else {
                        this.$set(item, 'amountError', false);
                    }

                    if (!item.premium) {
                        this.$set(item, 'premiumError', true);
                        isValid = false;
                    } else {
                        this.$set(item, 'premiumError', false);
                    }
                }
            }

            return isValid;
        },

        updateInsuranceTypeRules() {
            // 重置规则
            this.insuranceTypeRules = {};

            // 为每个有名称的险种添加对应的验证规则
            this.formData.insuranceTypes.forEach((item, index) => {
                if (item.name) {
                    // 为保额添加验证规则
                    this.insuranceTypeRules[`insuranceTypes.${index}.amount`] = [
                        { required: true, message: '请输入保额', trigger: 'blur' }
                    ];

                    // 为保费添加验证规则
                    this.insuranceTypeRules[`insuranceTypes.${index}.premium`] = [
                        { required: true, message: '请输入保费', trigger: 'blur' }
                    ];
                }
            });
        },

        prepareApiData() {
            const policyId = this.policyData.policyId || '';

            // 根据保单类型决定是否提交险种数据
            const cvrgList = this.formData.policyType === '1' ? [] :
                this.formData.insuranceTypes.map(item => {
                    // 对于有children的险种，amount是dicCode，否则是用户输入的金额
                    const amount = this.hasCvrgChildren(item.name) ? item.amount : item.amount;

                    return {
                        code: item.name || '', // 险种代码
                        amount: amount || '', // 保额
                        cvrgPrem: item.premium || '' // 保费
                    };
                });

            return {
                policyId: policyId,
                vehicleInfo: {
                    frmNo: this.formData.frameNo || '',
                    vehicleName: this.formData.carModel || '',
                    modelName: this.formData.carModel || '',
                    engNo: this.formData.engineNo || '',
                    plateNo: this.isNewCarWithoutPlate ? "新车未上牌" : (this.formData.licensePlate || ''),
                    fstRegNo: this.formData.registrationDate || '',
                    seatNum: this.formData.seatingCapacity || '',
                    ton: this.formData.approvedMass || '',
                    insId: this.formData.insuranceCompany || '',
                    vhlUsageCode: this.formData.usageNature || '',
                    carFuelType: this.formData.energyType || '',
                    vhlPower: this.formData.power || '',
                    desplacement: this.formData.displacement || '',
                    carKindCode: this.formData.vehicleType || ''
                },
                policyInfo: {
                    policyId: policyId,
                    comenceTime: this.formData.insuranceStartDate || '',
                    terminateTime: this.formData.insuranceEndDate || '',
                    policyNo: this.formData.policyNo || '',
                    insId: this.formData.insuranceCompany || '',
                    vehicleDamageCoverage: this.formData.vehicleDamageAmount || '',
                    thirdPartyCoverage: this.formData.thirdPartyAmount || '',
                    premiumTotal: this.formData.totalPremium || '',
                    policyType: this.formData.policyType || '',
                    chargeConfirmTime: this.formData.feeConfirmTime || '',
                    travelTax: this.formData.policyType === '0' ? '' : (this.formData.travelTax || ''),
                    policyGenTime: this.formData.policyCreateTime || '',
                    policyConfirmTime: this.formData.insuranceConfirmTime || '',
                    policyPrintTime: this.formData.policyPrintTime || '',
                    policyVerCode: this.formData.policyVerificationCode || '',
                    policyConfirmCode: this.formData.insuranceConfirmCode || '',
                    bankTxnNo: this.formData.bankTransactionNo || '',
                    specialAgreement: this.formData.specialTerms || '',
                    policyRemark: this.formData.policyRemark || '',
                },
                cvrgList: cvrgList,
                dmoInfo: {
                    actualFollower: this.formData.actualDriver || '',
                    vehicleId: this.formData.vehicleId || '',
                    premiumPolicyStatus: this.formData.personalInfoPolicy || '',
                    insureIns: this.formData.lastYearInsurer || '',
                    busType: this.formData.businessType || '',
                    bmwArcStatus: this.formData.hasBMWAccidentRepairProduct || ''
                }
            };
        },
        viewElectronicForm() {
            if (this.formData.fileUrl) {
                console.log(this.formData.fileUrl);
                window.open(this.formData.fileUrl, '_blank');
            } else {
                this.$message.info('没有可查看的电子保单');
            }
        }
    }
};
</script>

<style scoped>
.insurance-policy-dialog {
    font-size: 14px;
}

/* .dialog-content-container {
  border: 1px solid #EBEEF5;
  padding: 0px 15px 15px 15px;
  margin-bottom: 10px;
  border-radius: 4px;
} */

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
    /* padding-bottom: 10px;
  border-bottom: 1px solid #ebeef5; */
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

/* 添加险种按钮 */
.add-insurance-btn {
    padding: 6px 10px;
    font-size: 12px;
    height: 28px;
    line-height: 1;
    background-color: #3488ff;
    border-color: #3488ff;
}

.custom-insurance-table {
    /* border: 1px solid #EBEEF5; */
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

.footer-content {
    width: 100%;
}

.remark-section {
    margin-bottom: 20px;
}

.remark-label {
    font-weight: bold;
    margin-bottom: 8px;
    text-align: left;
}

.remark-input {
    width: 100%;
}

.button-section {
    display: flex;
    justify-content: flex-end;
}

.delete-btn {
    /* color: #3488ff; */
}

.delete-btn:hover {
    /* color: #66b1ff; */
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