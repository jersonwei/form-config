<template>
    <div class="edit-attr-container">
        <div class="title">物模型定义</div>
        <el-divider></el-divider>
        <el-form :model="detail" label-suffix="：" label-width="100px">
            <el-row>
                <template v-for="item in baseInfoItems">
                    <el-col :span="item.span || 12" :key="item.prop">
                        <el-form-item :label="item.label">
                            <span v-if="!item.tag">
                                {{
                                    item.formatter
                                        ? item.formatter(detail)
                                        : detail[item.prop] || '--'
                                }}
                            </span>
                            <template v-else-if="item.tag === 'el-radio-group'">
                                <el-radio-group v-model="detail[item.prop]">
                                    <el-radio v-for="radio in item.radios" :key="radio.label" :label="radio.value">{{
                                        radio.label
                                    }}</el-radio>
                                </el-radio-group>
                            </template>
                            <template v-else>
                                <component :is="item.tag" v-model="detail[item.prop]" v-bind="item"></component>
                            </template>
                        </el-form-item>
                    </el-col>
                </template>
            </el-row>
        </el-form>
        <template v-if="detail.bindFlag === '1'">
            <div class="title">映射定义</div>
            <el-divider></el-divider>
            <custom-table :data="detail.dictList" :heads="tableHead" ref="table" @btn-click="onTableBtnClick">
                <template v-slot:slot-1="scope">
                    <el-input v-model="scope.row.dictEnValue"></el-input> </template><template v-slot:slot-2="scope">
                    <el-input v-model="scope.row.dictLabel"></el-input>
                </template>
            </custom-table>
            <div class="flex-row center addTextButton">
                <span @click="onAdd()">+点击添加映射</span>
            </div>
        </template>
    </div>
</template>

<script>
import { attributeGetOne } from '../../../api'
import customTable from '../../../components/table.vue'
export default {
    props: {
        attrCode: String,
        facCateCode: String,
        attrId: Number
    },
    components: {
        customTable
    },
    methods: {
        validate() {
            return new Promise((resolve, reject) => {
                const { bindFlag } = this.detail
                if (bindFlag === '0') {
                    const { dictList, ...rest } = this.detail
                    resolve({ ...rest, dictList: [] })
                } else {
                    resolve(this.detail)
                }
            })
        },
        // 表格操作
        onTableBtnClick({ event, row, scope }) {
            const func = new Map([
                ['delete', row => this.onDelete(row, scope.$index)]
            ])
            func.has(event) && func.get(event).call(this, row)
        },
        onDelete(row, index) {
            console.log(row, index)
            this.detail.dictList.splice(index, 1)
        },
        onAdd() {
            this.detail.dictList.push({})
        }
    },
    created() {
        const { attrId } = this
        if (attrId) {
            attributeGetOne({ id: attrId }).then(data => {
                data.dictList = data.dictList || []
                data.bindFlag = data.dictList.length > 0 ? '1' : '0'
                this.$set(this, 'detail', data)
            })
        }
    },
    data() {
        return {
            baseInfoItems: [
                { label: '属性标识', prop: 'attrCode' },
                {
                    label: '属性名称',
                    prop: 'attrName',
                    tag: 'el-input',
                    maxLength: 50,
                    placeholder: '请输入属性名称'
                },
                {
                    label: '访问方式',
                    prop: 'accessMode',
                    formatter: ({ accessMode }) => {
                        const map = new Map([
                            ['RO', '只读'],
                            ['RW', '读写'],
                            // [3, '只写']
                        ])
                        if (map.has(accessMode)) {
                            return map.get(accessMode)
                        } else {
                            return '--'
                        }
                    }
                },
                {
                    label: '数据类型',
                    prop: 'dataType'
                },
                { label: '长度', prop: 'length' },
                {
                    label: '取值范围', prop: 'range',
                    formatter: ({ leVal, geVal }) => {
                        return (leVal || 0) + ' - ' + (geVal || 0)
                    }
                },
                { label: '单位', prop: 'dataUnit' },
                { label: '步长', prop: 'step' },
                { label: '枚举值', prop: 'enumDictCode' },
                {
                    tag: 'el-radio-group',
                    label: '绑定映射',
                    prop: 'bindFlag',
                    radios: [
                        { label: '是', value: '1' },
                        { label: '否', value: '0' }
                    ]
                }
            ],
            tableHead: [
                { label: '编码', prop: 'dictEnValue', slot: true, slotTag: 'slot-1' },
                { label: '映射定义', prop: 'dictLabel', slot: true, slotTag: 'slot-2' },
                {
                    label: '操作',
                    // width: 200,
                    btns: () => {
                        return [{ label: '删除', event: 'delete', type: 'danger' }]
                    }
                }
            ],
            detail: {}
        }
    }
}
</script>

<style lang="scss" scoped>
.addTextButton {
    margin: 16px 0;

    span {
        line-height: 22px;
        font-size: 14px;
        color: #007cc8;
        cursor: pointer;
    }
}
</style>
