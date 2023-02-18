<template>
    <div class="deviceDetail">
        <div class="returnBtn">
            <el-button @click="$router.push({ name: '设备台账' })">返回</el-button>
        </div>
        <baseInfo :timestamp="timestamp" :id="id" ref="panel" title="" :form="form" :paramsData="paramsData"
            @openLine="openLine"></baseInfo>
    </div>
</template>

<script>
import dayjs from 'dayjs'
import { wzDialog } from 'wz-ui'
import { facilitiesGetById, getFormNodesConfig } from '../../api'
import lineChart from './Component/line.vue'
import baseInfo from './Component/baseInfo.vue'
import runParams from './Component/runParams.vue'
import alarm from './Component/alarm.vue'
import log from './Component/log.vue'
import task from './Component/task.vue'
import procurementInfo from './Component/procurementInfo.vue'
import bimCorRelation from './Component/bimCorRelation.vue'
export default {
    name: 'DeviceDetail',
    props: {
        id: {
            type: [String, Number],
            default: 0
        },
        tagType: {
            type: String
        }
    },
    components: {
        baseInfo,
        runParams,
        alarm,
        log,
        task,
        procurementInfo,
        wzDialog,
        lineChart,
        bimCorRelation
    },
    watch: {
        id: {
            handler(newVal, oldVal) {
                facilitiesGetById(newVal).then(data => {
                    const {
                        buildingName,
                        floorName,
                        buildingFloorSpaceName,
                        ...rest
                    } = data
                    let building = ''
                    if (!buildingName) {
                        building = ''
                    } else if (!floorName) {
                        building = `${buildingName}`
                    } else if (!buildingFloorSpaceName) {
                        building = `${buildingName}-${floorName}`
                    } else if (floorName && buildingFloorSpaceName) {
                        building = `${buildingName}-${floorName}-${buildingFloorSpaceName}`
                    }
                    this.form = { building, ...rest }
                })
            }
        }
    },
    methods: {
        openLine({ facLinkCode, attrCode, attrName, dataUnit }) {
            this.$set(this, 'dialog', {
                title: this.$t('pc.device_ledger.historical_parameters'),
                tag: 'lineChart',
                show: true,
                width: '1039px',
                loading: false,
                top: '5vh',
                showConfirm: false,
                showCancel: false,
                facLinkCode,
                height: 350,
                attrCode,
                attrName,
                dataUnit
            })
        },
        async getFormConfig(formKey) {
            const res = await getFormNodesConfig({ formKey })
            this.paramsData = res.map((item) => {
                // 自定义字段
                let { fieldName, fieldKey, fieldType, defaultValue } = item
                if (fieldType === 'fileUpload') {
                    defaultValue = JSON.parse(defaultValue).map(subitem => {
                        const { name, fid } = subitem
                        return {
                            fileName: name,
                            fid
                        }
                    })
                }
                return {
                    label: fieldName,
                    prop: fieldKey,
                    fieldType,
                    value: defaultValue
                }
            })
        },
    },
    created() {
        this.$nextTick(() => {
            this.timestamp = dayjs().valueOf()
        })
        const { id } = this
        id &&
            facilitiesGetById(id).then(data => {
                const {
                    buildingName,
                    floorName,
                    buildingFloorSpaceName,
                    openType = '',
                    modelName,
                    ...rest
                } = data
                let building = ''
                if (!buildingName) {
                    building = ''
                } else if (!floorName) {
                    building = `${buildingName}`
                } else if (!buildingFloorSpaceName) {
                    building = `${buildingName}-${floorName}`
                } else if (floorName && buildingFloorSpaceName) {
                    building = `${buildingName}-${floorName}-${buildingFloorSpaceName}`
                }
                const _openType = openType ? (openType.split(',').map(i => {
                    const arr = ['刷卡', '人脸', '二维码']
                    return arr[i]
                }).join()) : '-'
                this.form = { building, openType: _openType, modelName, ...rest }
                console.log('created', this.form)
                // 判断是否逆变器类型  是的话加载
                this.getFormConfig(modelName)
            })

        if (this.$route.query.from === 'spaceTenseDetail') {
            this.activeName = 'runParams'
        }
    },
    data() {
        const validatePurchasePrice = (rule, value, callback) => {
            if (value) {
                if (!/^\d+(\.\d+)?$/.test(value)) {
                    callback(new Error('请输入大于0的正数'))
                } else {
                    callback()
                }
            } else {
                callback()
            }
        }
        const validateServiceLife = (rule, value, callback) => {
            if (value) {
                if (!/^[0-9]*[1-9][0-9]*$/.test(value)) {
                    callback(new Error('请输入正确年份'))
                } else {
                    callback()
                }
            } else {
                callback()
            }
        }
        return {
            dialog: { submit: () => { } },
            activeName: 'baseInfo',
            mode: 'check',
            timestamp: 0,
            tabList: [
                { label: this.$t('pc.global.basic_information'), tag: 'baseInfo' },
                {
                    label: this.$t('pc.device_ledger.operational_factor'),
                    tag: 'runParams'
                },
                { label: this.$t('pc.device_ledger.alarm_events'), tag: 'alarm' },
                { label: this.$t('pc.device_ledger.operation_log'), tag: 'log' },
                { label: this.$t('pc.device_ledger.task_record'), tag: 'task' }
            ],
            form: {},
            items: [
                {
                    label: '设备名称',
                    prop: 'facName'
                },
                {
                    label: '设备类型',
                    prop: 'facCateName'
                },
                {
                    label: '运行状态',
                    prop: 'facStatus'
                },
                {
                    label: '资产状态',
                    prop: 'assetStatus'
                }
            ].map((item, index) => {
                const key = [
                    'device_name',
                    'device_type',
                    'running_status',
                    'asset_status'
                ]
                item.label = this.$t(`pc.device_ledger.${key[index]}`)
                return item
            }),
            rules: {
                facName: [
                    {
                        required: true,
                        message: this.$t('pc.global.please_input', {
                            name: this.$t('pc.device_ledger.device_name')
                        })
                    }
                ],
                facCateCode: [
                    {
                        required: true,
                        message: this.$t('pc.global.please_select', {
                            name: this.$t('pc.device_ledger.device_type')
                        })
                    }
                ],
                // playAttr: [
                //   { required: true, validator: validatePlayAttr, trigger: 'change' },
                // ],
                facCode: [
                    {
                        required: true,
                        message: this.$t('pc.global.please_input', {
                            name: this.$t('pc.device_ledger.device_number')
                        })
                    }
                ],
                building: [
                    {
                        required: true,
                        message: this.$t('pc.global.please_select', {
                            name: this.$t('pc.device_ledger.spatial_location')
                        })
                    }
                ],
                assetStatus: [
                    {
                        required: true,
                        message: this.$t('pc.global.please_select', {
                            name: this.$t('pc.device_ledger.asset_status')
                        })
                    }
                ],
                purchasePrice: [{ validator: validatePurchasePrice, trigger: 'blur' }],
                serviceLife: [{ validator: validateServiceLife, trigger: 'blur' }]
                // supplierPhone: [{ validator: validateSupplierPhone, trigger: 'blur' }],
            },
            settingItems: [
                // { label: '设备ip', prop: 'facIp' },
                // { label: '端口号', prop: 'facPort' },
                // { label: '设备账号', prop: 'facLoginName' },
                // { label: '设备密码', prop: 'facLoginPwd' }
                {
                    label: '设备绑定摄像头',
                    prop: 'facCamId',
                    tag: 'el-cascader',
                    placeholder: '请选择设备绑定摄像头',
                    showAllLevels: false,
                    props: {
                        checkStrictly: false,
                        label: 'name',
                        value: 'id',
                        multiple: true
                    },
                    tootip: '用于三维中设备详情页的视频查看',
                    options: []
                },
                {
                    label: '室外机空调绑定',
                    prop: 'airconditionLink',
                    showAllLevels: false,
                    props: {
                        checkStrictly: false,
                        label: 'name',
                        value: 'id',
                        multiple: true
                    },
                    tootip: '用于三维中空调室内外机的联动',
                    options: []
                },
                {
                    label: '电梯绑定梯控设备',
                    prop: 'elevatorLinkId',
                    tag: 'el-cascader',
                    placeholder: '请选择电梯绑定梯控设备',
                    showAllLevels: true,
                    props: {
                        checkStrictly: false,
                        label: 'name',
                        value: 'id'
                    },
                    tootip: '用于三维中电梯内梯控设备的绑定',
                    options: []
                },
                {
                    label: '配电柜绑定抽屉号',
                    prop: 'powerLinkId',
                    tag: 'el-cascader',
                    placeholder: '请选择配电柜绑定抽屉号',
                    showAllLevels: false,
                    props: {
                        checkStrictly: false,
                        label: 'name',
                        value: 'id',
                        multiple: true
                    },
                    tootip: '用于三维中抽屉柜的绑定联动展示',
                    options: []
                },
                {
                    label: '三维联动快照',
                    prop: 'bimSnapshot',
                    placeholder: '请输入三维联动快照',
                    tootip: '用于三维中点击设备列表后的设备位置的定位跳转',
                    maxlength: 50
                }
            ].map(item => {
                if (!item.tag) {
                    Object.assign(item, { tag: 'el-input', clearable: true })
                }
                item.clearable = true
                return item
            }),
            paramsData: []
        }
    }
}
</script>

<style lang="scss" scoped>
.deviceDetail {
    position: relative;
    height: 100%;

    .customCard {
        // min-height: 100%;
        background: #fff;
        padding: 0 24px 24px;
    }

    .header {
        padding: 16px 0 6px;

        .label {
            color: #231815;
            font-size: 14px;
            line-height: 24px;
            min-width: 70px;
        }

        .value {
            font-size: 14px;
            color: #5f5d5d;
            line-height: 24px;
            margin-left: 8px;
        }

        .normal {
            color: #008843;
        }

        .abnormal {
            color: #d5341e;
        }
    }

    .aircondition-link {

        .el-input,
        .el-cascader {
            margin-right: 8px;
        }
    }
}
</style>
