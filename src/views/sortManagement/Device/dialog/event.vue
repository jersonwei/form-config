<template>
    <div class="event">
        <div class="title">事件定义</div>
        <el-divider></el-divider>
        <el-form :model="form" :rules="rules" label-suffix="：" label-width="100px">
            <el-row>
                <template v-for="item in baseInfoItems">
                    <el-col :span="item.span || 12" :key="item.prop">
                        <el-form-item :prop="item.prop" :label="item.label">
                            <component :is="item.tag" v-bind="item" v-model="form[item.prop]"></component>
                        </el-form-item>
                    </el-col>
                </template>
            </el-row>
        </el-form>
        <div class="title">事件映射</div>
        <el-divider></el-divider>
        <custom-table :data="mapping" :heads="tableHead" ref="table" @btn-click="onTableBtnClick">
            <template v-slot:slot-1="scope">
                <el-input v-model="scope.row.dictEnValue" placeholder="请输入事件编码" clearable></el-input>
            </template>
            <template v-slot:slot-2="scope">
                <el-input v-model="scope.row.dictLabel" placeholder="请输入事件名称" clearable></el-input>
            </template>
        </custom-table>
        <div class="flex-row center addEvent">
            <span @click="(event, row, scope) => addEventMapping(event, row, scope)">+点击添加事件映射</span>
        </div>
    </div>
</template>

<script>
import { attributeGetOne } from '../../../../api'
import customTable from '../../../../components/table.vue'
export default {
    components: { customTable },
    props: { id: [Number, String] },
    methods: {
        onTableBtnClick({ event, row, scope }) {
            const func = new Map([['delete', row => this.onDelete(row, scope.$index)]])
            func.has(event) && func.get(event).call(this, row)
        },
        onDelete(row, index) {
            this.mapping.splice(index, 1)
        },
        addEventMapping() {
            this.mapping.push({ dictEnValue: '', dictLabel: '' })
        },
        validate() {
            return new Promise((resolve, reject) => {
                const { ...rest } = this.form
                resolve({
                    ...rest,
                    dictList: this.mapping
                })
            })
        }
    },
    created() {
        const { id } = this
        id &&
            attributeGetOne({ id }).then(data => {
                const { dictList, id, attrCode, attrName, attrType } = data
                this.form.attrCode = attrCode
                this.form.attrName = attrName
                this.form.id = id
                this.form.attrType = attrType
                this.mapping = dictList
            })
    },
    data() {
        return {
            form: {
                attrCode: '',
                attrName: ''
            },
            rules: {
                attrCode: [{ required: true, message: '请输入事件服务' }],
                attrName: [{ required: true, message: '请输入事件服务' }]
            },
            baseInfoItems: [
                {
                    label: '事件服务',
                    prop: 'attrCode',
                    placeholder: '请输入事件服务',
                    maxlength: 50
                },
                {
                    label: '中文名称',
                    prop: 'attrName',
                    placeholder: '请输入中文名称',
                    maxlength: 50
                }
            ].map(item => {
                if (!item.tag) {
                    item.tag = 'el-input'
                }
                item.clearable = true
                return item
            }),
            mapping: [],
            tableHead: [
                { label: '事件编码', prop: 'dictEnValue', slot: true, slotTag: 'slot-1' },
                { label: '事件名称', prop: 'dictLabel', slot: true, slotTag: 'slot-2' },
                {
                    label: '操作',
                    // fixed: 'right',
                    btns: () => {
                        return [
                            {
                                label: '删除',
                                event: 'delete'
                            }
                        ]
                    }
                }
            ]
        }
    }
}
</script>

<style lang="scss" scoped>
.event {
    .addEvent {
        margin: 16px 0;

        span {
            line-height: 22px;
            font-size: 14px;
            color: #007cc8;
            font-weight: normal;
            cursor: pointer;
        }
    }
}
</style>
