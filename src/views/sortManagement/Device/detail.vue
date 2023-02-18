<template>
    <div class="detail">
        <div class="title">{{ $t('pc.global.basic_information') }}</div>
        <el-divider></el-divider>
        <el-form :model="detail" label-suffix="：" label-width="140px">
            <el-row>
                <template v-for="item in baseInfoItems">
                    <el-col :span="item.span || 8" :key="item.prop">
                        <el-form-item :label="item.label">
                            {{
                                item.formatter
                                    ? item.formatter(detail)
                                    : detail[item.prop] || $t('pc.global.nothing')
                            }}
                        </el-form-item>
                    </el-col>
                </template>
            </el-row>
        </el-form>
        <el-tabs v-if="detail.level === 4" v-model="activeName" type="card">
            <template>
                <el-tab-pane label="设备型号参数预设" name="third">
                    <params-list :data="detail"></params-list>
                </el-tab-pane>
                <el-tab-pane label="模型定义" name="first">
                    <div class="title row col-center row-between">
                        <span v-if="modelList.length !== 0" class="resynchronize" @click="onResynchronize">重新同步</span>
                    </div>
                    <el-divider></el-divider>
                    <el-collapse v-if="modelList.length !== 0" v-model="activeCollapse" accordion>
                        <custom-table :data="modelList" :heads="serviceTableHead" ref="table" :loading="loading"
                            @btn-click="(event, row, scope) => onTableBtnClick(event, row, scope)">
                        </custom-table>
                        <!-- </el-collapse-item> -->
                        <!-- </template> -->
                    </el-collapse>
                </el-tab-pane>

            </template>

            <template>
                <el-tab-pane label="事件定义" name="second">
                    <div class="title">
                        <el-divider></el-divider>
                        <custom-table :data="tableData" :heads="tableHead" ref="table" :loading="loading"
                            @btn-click="onTableBtnClick1"></custom-table>
                        <div class="flex-row center addEvent">
                            <span @click="addEvent()">+点击添加事件</span>
                        </div>
                    </div>
                </el-tab-pane>

            </template>


        </el-tabs>


        <el-dialog :close-on-click-modal="false" :visible.sync="dialog.show" :title="dialog.title" :width="dialog.width"
            :top="dialog.top || '15vh'" destroy-on-close>
            <component :is="dialog.tag" v-bind="dialog" ref="panel" title=""></component>
            <span slot="footer" class="dialog-footer">
                <el-divider></el-divider>
                <el-button v-if="dialog.showCancel !== false" @click="dialog.show = false" :disabled="dialog.loading">{{
                    dialog.cancelText || '取 消'
                }}</el-button>
                <el-button v-if="dialog.tag === 'resynchronize'" type="danger" @click="resynchronizeDelete"
                    :loading="dialog.loading">删除</el-button>
                <el-button v-if="dialog.showConfirm !== false" type="primary" @click="dialog.submit"
                    :loading="dialog.loading">{{ dialog.submitText || '保存' }}</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
import {
    dictAttribute,
    delFacCategory,
    updateFacCategory,
    facCateResynchronize,
    addFacAttributeEvent,
    facAttributeEventList,
    deleteFacAttributeEvent,
    editFacAttributeEvent
} from '../../../api'
import editAttr from './EditAttr.vue'
import resynchronize from './Resynchronize.vue'
import ParamsList from './ParamsList.vue'
import event from './Dialog/event.vue'
import { tablePage } from '../../../mixins/mixins'
import customTable from '../../../components/table.vue'
export default {
    name: 'DeviceDetail',
    components: {
        customTable,
        editAttr,
        resynchronize,
        event,
        ParamsList
    },
    mixins: [tablePage.call(this)],
    methods: {
        fetchData() { },
        handleSizeChange() {
            this.searchForm.head.current = 1
            this.fetchData()
        },
        onTableBtnClick({ event, row }) {
            const func = new Map([['edit', row => this.onEdit(row)]])
            func.has(event) && func.get(event).call(this, row)
        },
        onEdit(row) {
            this.$set(this, 'dialog', {
                show: true,
                tag: 'edit-attr',
                title: '编辑属性',
                width: '800px',
                loading: false,
                attrCode: row.identifier,
                facCateCode: this.detail.facCateCode,
                attrId: row.id,
                submit: this.handleEdit
            })
        },
        handleEdit() {
            this.$refs.panel.validate().then(form => {
                this.dialog.loading = true
                const params = {
                    facCateCode: this.detail.facCateCode,
                    facCateId: this.detail.id,
                    attrType: 'attr',
                    ...form
                }
                editFacAttributeEvent(params)
                    .then(() => {
                        this.dialog.show = false
                        this.$message.success('属性编辑成功！')
                        this.getFacCateGetProduct()
                    })
                    .finally(() => (this.dialog.loading = false))
            })
        },
        // 获取iot列表
        getFacCateGetProduct() {
            facAttributeEventList({
                facCateId: this.detail.id
            }).then(data => {
                this.$set(this, 'modelList', data)
            })
        },
        // 同步
        onResynchronize() {
            facCateResynchronize({
                productId: this.detail.productId,
                facCateId: this.detail.id,
                facCateCode: this.detail.facCateCode
            }).then(data => {
                if (data) {
                    this.$message.success('物模型信息同步成功！')
                    this.getFacCateGetProduct(this.detail.productId)
                } else {
                    this.$set(this, 'dialog', {
                        show: true,
                        tag: 'resynchronize',
                        title: '重新同步物模型',
                        width: '400px',
                        submitText: '变更来源',
                        loading: false,
                        submit: this.handleResynchronize
                    })
                }
            })
        },
        // 编辑, 变更设备来源
        handleResynchronize() {
            const {
                facCateName,
                parentId,
                level,
                facCateCode,
                facCateBrand,
                facCateModel,
                facCateLevel,
                facImg,
                id
            } = this.detail
            const params = {
                facCateName,
                parentId,
                level,
                facCateCode,
                facCateBrand,
                facCateModel,
                facCateLevel,
                facImg,
                facSource: 1,
                iotPlatform: '',
                productId: '',
                id
            }
            updateFacCategory(params)
                .then(() => {
                    this.dialog.show = false
                    this.$message.success('设备分类编辑成功')
                    this.$emit('onFetchData1')
                })
                .finally(() => (this.dialog.loading = false))
        },
        // 删除设备类型
        resynchronizeDelete() {
            const { facCateCode } = this.detail
            const h = this.$createElement
            this.$msgbox({
                title: '提示',
                message: h('p', { class: 'flex-column' }, [
                    h(
                        'span',
                        { style: 'color:#F56C6C' },
                        '确定删除后，会将分类存在的下级一同删除，且无法恢复。'
                    ),
                    h('span', null, '确定继续删除吗？')
                ]),
                showCancelButton: true
            }).then(() => {
                delFacCategory({ facCateCode }).then(() => {
                    this.$message.success('设备分类删除成功')
                    this.dialog.show = false
                    this.$emit('onFetchData')
                })
            })
        },
        onTableBtnClick1({ event, row }) {
            const func = new Map([
                ['edit', row => this.onEventEdit(row)],
                ['delete', row => this.onEventDelete(row)]
            ])
            func.has(event) && func.get(event).call(this, row)
        },
        // 删除事件
        onEventDelete({ id }) {
            deleteFacAttributeEvent({ id }).then(() => {
                this.$message.success('事件删除成功！')
                this.getFacAttributeEventList()
            })
        },
        // 编辑事件
        onEventEdit({ id }) {
            this.$set(this, 'dialog', {
                show: true,
                tag: 'event',
                title: '编辑事件',
                width: '800px',
                loading: false,
                submit: this.handleEventEdit,
                id
            })
        },
        handleEventEdit() {
            this.$refs.panel.validate().then(form => {
                this.dialog.loading = true
                const params = {
                    facCateCode: this.detail.facCateCode,
                    facCateId: this.detail.id,
                    attrType: 'event',
                    ...form
                }
                editFacAttributeEvent(params)
                    .then(() => {
                        this.dialog.show = false
                        this.$message.success('事件编辑成功！')
                        this.getFacAttributeEventList()
                    })
                    .finally(() => (this.dialog.loading = false))
            })
        },
        // 新增事件
        addEvent() {
            this.$set(this, 'dialog', {
                show: true,
                tag: 'event',
                title: '新增事件',
                width: '800px',
                loading: false,
                submit: this.handleEventAdd
            })
        },
        // 新增事件确认按钮
        handleEventAdd() {
            this.$refs.panel.validate().then(form => {
                this.dialog.loading = true
                const params = {
                    facCateCode: this.detail.facCateCode,
                    facCateId: this.detail.id,
                    attrType: 'event',
                    ...form
                }
                addFacAttributeEvent(params)
                    .then(() => {
                        this.dialog.show = false
                        this.$message.success('事件新增成功！')
                        this.getFacAttributeEventList()
                    })
                    .finally(() => (this.dialog.loading = false))
            })
        },
        // 获取事件列表
        getFacAttributeEventList() {
            facAttributeEventList({
                facCateCode: this.detail.facCateCode,
                facCateId: this.detail.id,
                attrType: 'event'
            }).then(
                data => {
                    this.$set(this, 'tableData', data)
                }
            )
        }
    },
    props: { detail: Object },
    created() {
        this.getFacCateGetProduct()
    },
    watch: {
        'detail.productId': {
            handler(newVal) {
                if (newVal) {
                    this.getFacCateGetProduct()
                } else {
                    this.modelList = []
                }
            },
            deep: true
        },
        detail: {
            handler(newVal) {
                this.getFacAttributeEventList()
            },
            deep: true
        }
    },
    computed: {
        baseInfoItems() {
            let baseInfoItems = ''
            if (this.detail.level === 4 && this.detail.facSource) {
                if (this.detail.facSource === 1) {
                    baseInfoItems = [
                        { label: '分类名称', prop: 'facCateName' },
                        {
                            label: '上级类型',
                            formatter: ({ facCateParentName }) => facCateParentName || '无'
                        },
                        {
                            label: '设备级别',
                            prop: 'level',
                            formatter: ({ level }) => {
                                if (level === 1) {
                                    return '一级设备系统'
                                } else if (level === 2) {
                                    return '二级设备系统'
                                } else if (level === 3) {
                                    return '三级设备系统'
                                } else if (level === 4) {
                                    return '设备类型'
                                }
                            }
                        },
                        { label: '类型编号', prop: 'facCateCode' },
                        {
                            label: '设备来源',
                            prop: 'facSource',
                            formatter: ({ facSource }) => {
                                if (facSource === 1) {
                                    return '系统分类'
                                } else if (facSource === 2) {
                                    return 'IoT物模型'
                                } else {
                                    return '--'
                                }
                            }
                        }
                    ]
                } else if (this.detail.facSource === 2) {
                    baseInfoItems = [
                        { label: '分类名称', prop: 'facCateName' },
                        {
                            label: '上级类型',
                            formatter: ({ facCateParentName }) => facCateParentName || '无'
                        },
                        {
                            label: '设备级别',
                            prop: 'level',
                            formatter: ({ level }) => {
                                if (level === 1) {
                                    return '一级设备系统'
                                } else if (level === 2) {
                                    return '二级设备系统'
                                } else if (level === 3) {
                                    return '三级设备系统'
                                } else if (level === 4) {
                                    return '设备类型'
                                }
                            }
                        },
                        { label: '类型编号', prop: 'facCateCode' },
                        {
                            label: '设备来源',
                            prop: 'facSource',
                            formatter: ({ facSource }) => {
                                if (facSource === 1) {
                                    return '系统分类'
                                } else if (facSource === 2) {
                                    return 'IoT物模型'
                                } else {
                                    return '--'
                                }
                            }
                        },
                        {
                            label: 'IoT云平台',
                            prop: 'iotPlatform',
                            formatter: ({ iotPlatform }) => {
                                if (iotPlatform === 1) {
                                    return '招商集团IoT'
                                } else {
                                    return '--'
                                }
                            }
                        },
                        { label: '物模型', prop: 'productName' }
                    ]
                } else if (this.detail.facSource === 3) {
                    baseInfoItems = [
                        { label: '分类名称', prop: 'facCateName' },
                        {
                            label: '上级类型',
                            formatter: ({ facCateParentName }) => facCateParentName || '无'
                        },
                        {
                            label: '设备级别',
                            prop: 'level',
                            formatter: ({ level }) => {
                                if (level === 1) {
                                    return '一级设备系统'
                                } else if (level === 2) {
                                    return '二级设备系统'
                                } else if (level === 3) {
                                    return '三级设备系统'
                                } else if (level === 4) {
                                    return '设备类型'
                                }
                            }
                        },
                        { label: '类型编号', prop: 'facCateCode' },
                        {
                            label: '设备来源',
                            prop: 'facSource',
                            formatter: ({ facSource }) => {
                                if (facSource === 1) {
                                    return '系统分类'
                                } else if (facSource === 2) {
                                    return 'IoT物模型'
                                } else if (facSource === 3) {
                                    return '业务系统'
                                } else {
                                    return '--'
                                }
                            }
                        },
                        {
                            label: '业务平台',
                            prop: 'servicePlatform'
                        }
                    ]
                }
            } else {
                baseInfoItems = [
                    { label: '分类名称', prop: 'facCateName' },
                    {
                        label: '上级类型',
                        formatter: ({ facCateParentName }) => facCateParentName || '无'
                    },
                    {
                        label: '设备级别',
                        prop: 'level',
                        formatter: ({ level }) => {
                            if (level === 1) {
                                return '一级设备系统'
                            } else if (level === 2) {
                                return '二级设备系统'
                            } else if (level === 3) {
                                return '三级设备系统'
                            } else if (level === 4) {
                                return '设备类型'
                            }
                        }
                    },
                    { label: '类型编号', prop: 'facCateCode' }
                ]
            }
            return baseInfoItems
        }
    },
    data() {
        return {
            pageHeight: 0,
            loading: false,
            tableHead: [
                { label: '事件服务', prop: 'attrCode' },
                { label: '中文名称', prop: 'attrName' },
                {
                    label: '事件定义',
                    formatter: ({ dictList = [] }) => {
                        return `${dictList.length}条记录`
                    }
                },
                {
                    label: '操作',
                    // width: 200,
                    // fixed: 'right',
                    btns: () => {
                        return [
                            {
                                label: '编辑',
                                event: 'edit'
                            },
                            {
                                label: '删除',
                                event: 'delete'
                            }
                        ]
                    }
                }
            ],
            serviceTableHead: [
                { label: '属性标识', prop: 'attrCode' },
                { label: '属性名称', prop: 'attrName' },
                {
                    label: '数据类型',
                    prop: 'dataType'
                },
                // { label: '数据定义', prop: 'num' },
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
                    label: '操作',
                    btns: () => {
                        return [
                            {
                                label: '编辑',
                                event: 'edit'
                            }
                        ]
                    }
                }
            ],
            tableData: [],
            activeCollapse: ['1'],
            modelList: [
                // {
                //   serviceID: 'hk',
                //   serviceType: '门禁',
                //   serviceDescribe: '--',
                //   list: [
                //     {
                //       id: 1,
                //       attributeName: 'ConnectionState',
                //       name: '在线状态',
                //       type: 'int(整型)',
                //       num: '1条记录',
                //       way: '可读'
                //     },
                //     {
                //       id: 2,
                //       attributeName: 'ConnectionState',
                //       name: '在线状态',
                //       type: 'int(整型)',
                //       num: '1条记录',
                //       way: '可读'
                //     }
                //   ]
                // }
            ],
            dialog: { show: false, sumbit: () => { } },
            activeName: 'third'
        }
    }
}
</script>

<style lang="scss" scoped>
.collapseHeader {
    .item {
        margin-left: 16px;
        margin-right: 20px;
        font-size: 14px;

        .label {
            color: #606266;
        }

        .value {
            margin-left: 8px;
        }
    }
}

.resynchronize {
    font-weight: 500;
    color: #007cc8;
    cursor: pointer;
    margin-right: 16px;
}

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
</style>
