<template>
    <div class="procurementInfo">
        <div class="title">技术参数</div>
        <el-divider></el-divider>
        <el-form class="procurementInfo" :model="form" label-suffix="：" label-width="156px" status-icon>
            <el-row>
                <template v-for="item in procurementItems">
                    <el-col :key="item.prop" :span="item.span ? item.span : 8">
                        <el-form-item :label="item.label" :prop="item.prop" :label-width="item.labelWidth">
                            <span class="value">{{ form[item.prop] || '--' }}</span>
                        </el-form-item>
                    </el-col>
                </template>
            </el-row>
        </el-form>
    </div>
</template>

<script>
import { getFacfacilitiesparametersList } from '../../../api'
export default {
    props: {
        form: Object
    },
    created() {
        const id = this.$route.params.id
        getFacfacilitiesparametersList(id).then(res => {
            console.log(res)
            const {
                effi,
                ratedInVoltage,
                dcInputVoltage,
                inputVoltageFreq,
                inputCurrent,
                gridGrade,
                shortCircuitCurrent,
                modeCooling,
                startVoltage,
                classProtection,
                rangeVoltage,
                electricityNight,
                mpptNum,
                noise
            } = res[0]
            this.$set(this, 'form', {
                effi,
                ratedInVoltage,
                dcInputVoltage,
                inputVoltageFreq,
                inputCurrent,
                gridGrade,
                shortCircuitCurrent,
                modeCooling,
                startVoltage,
                classProtection,
                rangeVoltage,
                electricityNight,
                mpptNum,
                noise
            })
        })
    },
    data() {
        return {
            form: {},
            // form: {
            //   供应商名称: '供应商名称',
            //   合同编号: '合同编号',
            //   联系人: '联系人',
            //   联系方式: '联系方式',
            //   采购价: '采购价'
            // },
            procurementItems: [
                {
                    label: '效率(%)',
                    prop: 'effi'
                },
                {
                    label: '额定输入电压(V)',
                    prop: 'ratedInVoltage'
                },
                {
                    label: '直流输入电压(V)',
                    prop: 'dcInputVoltage'
                },
                {
                    label: '输入电压频率(Hz)',
                    prop: 'inputVoltageFreq'
                },
                {
                    label: 'MPPT输入电流(A)',
                    prop: 'inputCurrent'
                },
                {
                    label: '并网等级',
                    prop: 'gridGrade'
                },
                {
                    label: '短路电流(A)',
                    prop: 'shortCircuitCurrent'
                },
                {
                    label: '冷却方式(A)',
                    prop: 'modeCooling'
                },
                {
                    label: '启动电压(V)',
                    prop: 'startVoltage'
                },
                {
                    label: '防护等级',
                    prop: 'classProtection'
                },
                {
                    label: 'MPPT电压范围(V)',
                    prop: 'rangeVoltage'
                },
                {
                    label: '夜间自耗电',
                    prop: 'electricityNight'
                },
                {
                    label: 'MPPT数量',
                    prop: 'mpptNum'
                },
                {
                    label: '噪音',
                    prop: 'noise'
                }
            ]
        }
    }
}
</script>

<style lang="scss" scoped>
.procurementInfo {
    padding: 12px 24px 6px;
    margin-top: 24px;
    background: #fff;
}
</style>
