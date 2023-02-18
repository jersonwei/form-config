<template>
    <div class="order-records col">
        <!-- 页面部分 -->
        <el-card shadow="always" class="table-container">
            <template slot="header">
                <div class="row row-between">
                    <span class="card-header__title">{{ $t('表单配置') }}</span>
                    <div class="row">
                        <el-button v-if="checkedButton" @click="onAdd">{{
                            $t('新增表单')
                        }}</el-button>
                    </div>
                </div>
            </template>
            <div class="row align-center" style="margin-bottom: 22px">
                <el-form :model="searchForm" inline label-suffix="：" ref="formRef">
                    <template v-for="item in items">
                        <el-form-item :key="item.prop" :label="item.label" :prop="item.prop" style="margin-bottom: 0">
                            <component :is="item.tag" v-bind="item" v-model="searchForm[item.prop]"></component>
                        </el-form-item>
                    </template>
                </el-form>
                <el-button @click="onReset">{{
                    $t('pc.work_order_records.reset')
                }}</el-button>
                <el-button type="primary" @click="onSearch">{{
                    $t('pc.work_order_records.search')
                }}</el-button>
            </div>
            <div class="containerTable" ref="containerTable">
                <wz-table :loading="loading" :heads="tableHead" :data="tableData" has-index
                    @btn-click="onBtnClick"></wz-table>
                <div class="pagination row row-between">
                    <span class="total">{{
                        $t('pc.global.total_items', { total: searchForm.totalCount
                    })
                    }}</span>
                    <el-pagination ref="pager" layout="prev,pager,next,sizes,jumper"
                        :current-page.sync="searchForm.page" :page-size.sync="searchForm.rows"
                        :total="searchForm.totalCount" @current-change="fetchData" @size-change="handleSizeChange"
                        background hide-on-single-page></el-pagination>
                </div>
            </div>
        </el-card>
        <!-- 弹窗部分 -->
        <el-dialog :visible.sync="dialog.show" :title="dialog.title" :width="dialog.width" :top="dialog.top || '15vh'"
            :close-on-click-modal="false" destroy-on-close>
            <!-- 表单部分 -->
            <el-form v-if="dialog.type === 'form'" :model="form" :rules="rules" label-suffix="："
                :label-width="Locale === 'en' ? '160px' : '160px'" status-icon ref="baseInfo">
                <el-row :gutter="20" class="base-info">
                    <el-col v-for="row in baseInfo" :span="row.span" :key="row.index">
                        <template v-for="item in row.list">
                            <el-form-item :key="item.prop" :label="item.label" :prop="item.prop">
                                <component :is="item.tag" v-bind="item" v-model="form[item.prop]"></component>
                            </el-form-item>
                        </template>
                    </el-col>
                </el-row>
            </el-form>
            <!-- 节点部分 -->
            <div v-if="dialog.type === 'nodes'" class="form-container">
                <!-- 节点 -->
                <el-tabs v-model="activeNodes" type="card" @tab-click="setactiveNodes">
                    <el-tab-pane v-for="item in nodes" :key="item.name" :label="item.label"
                        :name="item.name"></el-tab-pane>
                </el-tabs>
                <el-button type="primary" @click="onAddField">{{
                    $t('pc.global.add')
                }}</el-button>
                <!-- 字段编辑 -->
                <el-form ref="fieldForm" :model="fieldDataCopy">
                    <el-table ref="fieldTable" :data="fieldDataCopy.data" border row-key="id" style="width: 100%">
                        <el-table-column type="index" width="40" label="#">
                        </el-table-column>
                        <el-table-column width="50" label="排序">
                            <i class="el-icon-rank my-handle"></i>
                        </el-table-column>
                        <el-table-column v-for="item in fieldFormConfig" :key="item.id" :prop="item.prop"
                            :label="item.label" :width="item.colWidth || '180px'">
                            <!-- 自定义表头 -->
                            <template slot="header">
                                <span :class="{ 'theader-required': item.isRequired }">{{
                                    item.label
                                }}</span>
                            </template>
                            <!-- 自定义表列 -->
                            <template slot-scope="{ row, $index }">
                                <template v-if="isShowTbody(row, item)">
                                    <el-form-item :prop="`data.${$index}.${item.prop}`"
                                        :rules="fieldRules[item.prop] || null">
                                        <component :is="item.tag" v-bind="item" v-model="row[item.prop]" :placeholder="
                                            typeof item.placeholder === 'function'
                                                ? item.placeholder(row)
                                                : item.placeholder
                                        " @click="onTableClick(item, row, $index)">
                                            <template v-if="item.tag === 'el-select'">
                                                <el-option v-for="opt in item.options" :key="opt.value"
                                                    v-bind="opt"></el-option>
                                            </template>
                                        </component>
                                    </el-form-item>
                                    <!-- <span v-else>{{ item.format ? item.format(scope.row[item.prop]): scope.row[item.prop] }}</span> -->
                                </template>
                            </template>
                        </el-table-column>
                    </el-table>
                </el-form>
            </div>
            <!-- 按钮部分 -->
            <span slot="footer" class="dialog-footer">
                <el-button @click="dialog.show = false">{{
                    $t('pc.global.cancel')
                }}</el-button>
                <el-button type="primary" @click="onSubmit">{{
                    $t('pc.global.submit')
                }}</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
import { wzTable } from 'wz-ui'
import { mapGetters } from 'vuex'
import Sortable from 'sortablejs'
import {
    getFormConfigPage,
    getFormConfigById,
    addFormConfig,
    editFormConfig,
    deleteFormConfig,
    findDictListByCode,
    updateFormNodesConfig,
    getFormNodesConfig
} from '../../api'
const validateKey = (rule, value, callback) => {
    if (/^[A-Za-z0-9_\\-]+$/.test(value)) {
        callback()
    } else {
        callback(new Error('请输入大小写字母、-、下划线组成的key'))
    }
}
export default {
    name: 'ParamsConfig',
    computed: {
        ...mapGetters(['Locale']),
        //...mapGetters({ user: 'GET_USER' })
        user() {
            const getUser = this?.$store?.getters?.GET_USER
            return getUser || {}
        },
        getAuth() {
            const getAuth = this?.$store?.getters?.GET_AUTH
            return getAuth || {}
        },
        checkedButton() {
            return this.$checkPermission('gdjl_xz')
        }
    },
    // mixins: [auth.call(this)],
    components: {
        wzTable
    },
    methods: {
        // 页面部分 -----------------------------------------------
        handleSizeChange() {
            this.onSearch()
        },
        fetchData() {
            const { page, rows, formName } = this.searchForm
            this.loading = true
            const params = {
                head: { current: page, size: rows },
                body: { formName }
            }
            getFormConfigPage(params)
                .then(({ total, current, size, records }) => {
                    Object.assign(this.searchForm, {
                        totalCount: total,
                        page: current,
                        rows: size
                    })
                    this.tableData = records
                    this.$nextTick(() => this.doLayout())
                })
                .catch(() => {
                    this.tableData = [
                        {
                            formName: '自定义表单',
                            formKey: 'other'
                        }
                    ]
                })
                .finally(() => (this.loading = false))
        },
        fetchCountData() {
            findWorkOrderStateListCount().then(data => {
                const { finished, notFinished } = data
                Object.assign(this.count, { finished, notFinished })
            })
        },
        doLayout() {
            this.tableHeight = 0
            this.$nextTick(() => {
                const {
                    pager: { $el = null },
                    containerTable
                } = this.$refs
                this.tableHeight =
                    containerTable.clientHeight -
                    ($el instanceof Comment ? 0 : $el.clientHeight)
            })
        },
        onSearch() {
            this.searchForm.page = 1
            this.fetchData()
        },
        onReset() {
            this.searchForm = { rows: 10, page: 1, totalCount: 0 }
            this.$refs.formRef.resetFields()
            this.onSearch()
        },
        onBtnClick({ event, row }) {
            const func = new Map([
                ['edit', this.onEdit],
                ['editNodes', this.onNodesEdit],
                ['delete', this.onDelete]
            ])
            func.has(event) && func.get(event).call(this, row)
        },
        // 基础数据表单部分 -----------------------------------------
        setBaseInfo(disabled) {
            this.baseInfo = [
                {
                    index: 0,
                    span: 24,
                    list: [
                        {
                            label: this.$t('表单名称'),
                            prop: 'formName',
                            tag: 'el-input',
                            clearable: true,
                            maxlength: 40
                        },
                        {
                            label: this.$t('key'),
                            prop: 'formKey',
                            tag: 'el-input',
                            clearable: true,
                            maxlength: 40,
                            disabled
                        }
                    ]
                }
            ]
        },
        onAdd() {
            // this.$set(this.dialog, 'show', true)
            this.dialog = {
                show: true,
                title: '新增表单',
                top: '5vh',
                width: '550px',
                type: 'form',
                event: 'add'
            }
            this.form = this.$options.data.call(this).form
            this.setBaseInfo(false)
        },
        async onEdit({ id, state }) {
            const { formName, formKey } = await getFormConfigById(id)
            this.form = { formName, formKey, id }
            // this.$set(this.dialog, 'show', true)
            this.dialog = {
                show: true,
                title: '新增表单',
                top: '5vh',
                width: '550px',
                type: 'form',
                event: 'edit'
            }
            this.setBaseInfo(true)
        },
        onFormSubmit() {
            this.$refs.baseInfo.validate(res => {
                if (res) {
                    const { event } = this.dialog
                    let func, text
                    if (event === 'add') {
                        func = addFormConfig
                        text = '表单创建成功！'
                    } else if (event === 'edit') {
                        func = editFormConfig
                        text = '表单编辑成功！'
                    }
                    func(this.form).then(res => {
                        this.$message.success(text)
                        this.dialog.show = false
                        this.fetchData()
                    })
                }
            })
        },
        onDelete({ id, state }) {
            this.$confirm('确认删除？', '提示', {
                type: 'warning'
            }).then(() => {
                deleteFormConfig({ body: [id] }).then(() => {
                    this.$message.success('删除成功')
                    this.fetchData()
                })
            })
        },

        // 节点编辑部分 ------------------------------------------------------------
        setactiveNodes(activeNodes) {
            this.fieldDataCopy = { data: this.fieldData[this.activeNodes] }
            this.activeNodesName = activeNodes.label
        },
        isShowTbody(row, theaderItem) {
            const { fieldProperty, fieldType, ifShow } = row
            const { prop } = theaderItem
            if (fieldProperty) {
                // 自定义
                if (
                    prop === 'contentSettings' &&
                    fieldType !== 'radio' &&
                    fieldType !== 'checkbox'
                )
                    return false
                // if (prop === 'defaultValue') return false
                return true
            } else {
                // 默认
                if (prop === 'fieldType' || prop === 'contentSettings') return false
                if (prop === 'defaultValue' && ifShow) return false
                return true
            }
        },
        async onNodesEdit({ id }) {
            // 先获取formKey
            const formData = await getFormConfigById(id)
            const { formKey, formName } = formData
            this.chiocedForm = formData
            // 然后拿formKey 去数据字典获取值
            const data = await findDictListByCode({ dictCode: formKey })
            if (data && data.length) {
                // 有数据字典的走数据字典
                this.nodes = data.map(item => {
                    return {
                        label: item.dictLabel,
                        name: item.dictValue
                    }
                })
            } else {
                // 无数据字典 以表单值为节点
                this.nodes = [
                    {
                        label: formName,
                        name: formKey
                    }
                ]
            }
            this.nodes.forEach(({ name }) => {
                this.fieldData[name] = []
            })

            const nodesData = await getFormNodesConfig({ formKey })
            nodesData.forEach(item => {
                this.fieldData[item.parentNode].push(item)
            })
            this.activeNodes = this.nodes[0].name
            this.activeNodesName = this.nodes[0].label
            this.fieldDataCopy = { data: this.fieldData[this.activeNodes] }
            // 有数值的话表示可以继续后续操作
            this.dialog = {
                show: true,
                title: `${formName}[${formKey}]-表单节点`,
                top: '5vh',
                width: '90%',
                type: 'nodes'
            }
            this.$nextTick(() => {
                this.setSortable()
            })
        },
        onAddField() {
            this.fieldData[this.activeNodes].push({
                id: new Date().getTime(),
                fieldName: '',
                fieldKey: '',
                fieldProperty: 1,
                ifRequired: 1,
                ifShow: 1,
                fieldType: ''
            })
        },
        onTableClick({ tag }, row, index) {
            if (tag === 'el-button') this.fieldData[this.activeNodes].splice(index, 1)
        },
        setSortable() {
            //获取对象
            var el = this.$refs.fieldTable.$el.querySelector('tbody')
            //设置配置
            var ops = {
                animation: 1000,
                handle: '.my-handle',
                //拖动结束
                onEnd: evt => {
                    // console.log(evt.oldIndex) // 当前行的被拖拽前的顺序
                    // console.log(evt.newIndex) // 当前行的被拖拽后的顺序
                    const { oldIndex, newIndex } = evt
                    //获取拖动后的排序
                    const currRow = this.fieldDataCopy.data.splice(oldIndex, 1)[0]
                    this.fieldDataCopy.data.splice(newIndex, 0, currRow)
                }
            }
            //初始化
            var sortable = Sortable.create(el, ops)
        },
        onNodesSubmit() {
            this.$refs.fieldForm.validate(res => {
                if (res) {
                    const { formName, formKey } = this.chiocedForm
                    const params = this.fieldData[this.activeNodes].map((item, index) => {
                        const arr = [
                            'bizName',
                            'createdBy',
                            'createdDate',
                            'lastUpdatedBy',
                            'lastUpdatedDate',
                            'strId',
                            'id',
                            'sort'
                        ]
                        // 删除多余字段
                        arr.forEach(prop => {
                            delete item[prop]
                        })
                        return {
                            formName,
                            formKey,
                            parentNode: this.activeNodes,
                            sort: index + 1,
                            ...item
                        }
                    })
                    updateFormNodesConfig(params).then(res => {
                        this.$message.success(`${this.activeNodesName}保存成功`)
                    })
                }
            })
        },
        // 提交部分 --------------------------------------------------------------------
        onSubmit() {
            const { type } = this.dialog
            if (type === 'form') {
                this.onFormSubmit()
            } else if (type === 'nodes') {
                this.onNodesSubmit()
            }
        }
    },
    created() {
        this.fetchData()
    },
    mounted() {
        this.doLayout()
        window.addEventListener('resize', this.doLayout)
    },
    beforeDestroy() {
        window.removeEventListener('resize', this.doLayout)
    },
    data() {
        return {
            dialog: { submit: () => { } },
            loading: false,
            tableHeight: 0,
            searchForm: { rows: 10, page: 1, totalCount: 0 },
            tableData: [],
            tableHead: [
                { label: this.$t('表单名称'), prop: 'formName' },
                { label: this.$t('key'), prop: 'formKey' },
                {
                    label: this.$t('pc.work_order_records.operation'),
                    rawWidth: 260,
                    btns: () => {
                        const { id } = this.user
                        let btns = []
                        const map = [
                            {
                                label: this.$t('pc.global.edit'),
                                event: 'edit'
                                // code: 'gdjl_pj'
                            },
                            {
                                label: this.$t('编辑节点'),
                                event: 'editNodes'
                                // code: 'gdjl_pj'
                            },
                            {
                                label: this.$t('pc.global.delete'),
                                event: 'delete'
                            }
                        ]
                        // const arr = map.has(state) ? [map.get(state)] : []
                        // btns = this.$tableLinkCheck(arr)
                        return map
                    }
                }
            ],
            items: [
                {
                    label: this.$t('表单名称'),
                    prop: 'formName',
                    tag: 'el-input',
                    clearable: true
                }
            ],
            baseInfo: [],
            form: {
                formName: '',
                formKey: ''
            },
            rules: {
                formName: [
                    {
                        required: true,
                        message: this.$t('请输入表单名称')
                    }
                ],
                formKey: [
                    {
                        required: true,
                        validator: validateKey
                    }
                ]
            },
            nodes: [],
            activeNodes: '0',
            fieldFormConfig: [
                {
                    key: 0,
                    prop: 'fieldName',
                    label: '名称',
                    tag: 'el-input',
                    isRequired: true
                },
                {
                    key: 1,
                    prop: 'fieldKey',
                    label: '字段',
                    tag: 'el-input',
                    isRequired: true
                },
                // {
                //   key: 2,
                //   colWidth: 120,
                //   prop: 'fieldProperty',
                //   label: '字段类型',
                //   tag: 'el-select',
                //   options: [
                //     { value: 0, label: '默认' },
                //     { value: 1, label: '自定义' }
                //   ]
                // },
                {
                    key: 3,
                    colWidth: 120,
                    prop: 'ifRequired',
                    label: '是否必填',
                    tag: 'el-switch',
                    'active-value': 1,
                    'inactive-value': 0
                },
                {
                    key: 4,
                    colWidth: 120,
                    prop: 'ifShow',
                    label: '是否显示',
                    tag: 'el-switch',
                    'active-value': 1,
                    'inactive-value': 0
                },
                {
                    key: 5,
                    prop: 'defaultValue',
                    label: '默认值',
                    colWidth: 'none',
                    tag: 'el-input',
                    placeholder: row => {
                        if (row.fieldType === 'fileUpload') {
                            return '[{"fileName": "文件名", "url": "下载链接"}]'
                        }
                        return ''
                    }
                },
                {
                    key: 6,
                    prop: 'fieldType',
                    label: '控件类型',
                    tag: 'el-select',
                    options: [
                        { value: 'text', label: '单行文本框' },
                        { value: 'textarea', label: '多行文本框' },
                        { value: 'radio', label: '单选框' },
                        { value: 'checkbox', label: '多选框' },
                        { value: 'datepicker', label: '日期' },
                        { value: 'fileUpload', label: '文件上传' }
                    ],
                    isRequired: true
                },
                {
                    key: 7,
                    prop: 'contentSettings',
                    label: '选项设置',
                    tag: 'el-input',
                    colWidth: 'none',
                    isRequired: true
                },
                {
                    key: 9,
                    prop: 'button',
                    colWidth: 90,
                    label: '操作',
                    tag: 'el-button',
                    type: 'danger',
                    icon: 'el-icon-delete'
                }
            ],
            fieldRules: {
                fieldName: [{ required: true, message: '请输入名称', trigger: 'blur' }],
                fieldKey: [{ required: true, message: '请输入字段', trigger: 'blur' }],
                // defaultValue: [{ required: true, message: '请输入默认值', trigger: 'blur' }],
                contentSettings: [
                    { required: true, message: '请输入选项设置', trigger: 'blur' }
                ],
                fieldType: [
                    { required: true, message: '请选择控件类型', trigger: 'change' }
                ]
            },
            fieldData: {},
            fieldDataCopy: { data: [] }
        }
    }
}
</script>

<style lang="scss" scoped>
.table-container /deep/ {
    flex: 1;
    margin-top: 20px;

    .el-card__body {
        height: calc(100% - 55px);
    }
}

.el-form-item {
    margin-bottom: 20px !important;
}

.order-records {
    height: 100%;

    .steps {
        .row {
            cursor: pointer;
        }

        flex: 1;

        .status {
            width: 32px;
            height: 32px;
            border-radius: 50%;
            text-align: center;
            line-height: 32px;
            font-family: PingFangSC-Regular;
            font-size: 14px;
            border: 1px solid #cccccc;
            color: #666666;
        }

        .step--label {
            margin-left: 8px;
            font-weight: bold;
        }

        .step--value {
            font-family: PingFangSC-Regular;
            font-size: 30px;
            color: #666666;
            // margin-top: 8px;
        }

        .step+.el-divider {
            width: unset;
        }

        .el-divider {
            flex: 1;
            margin: 0 16px;
            margin-bottom: 12px;
            width: 90%;
        }
    }
}

.el-form {
    flex: 1;
}

.base-info {
    flex: 1;

    .el-date-editor {
        max-width: 230px;
    }

    .el-select {
        max-width: 225px;
    }

    .el-input {
        max-width: 225px;
    }
}

.row .el-form+.el-button {
    line-height: unset;
}

.containerTable {
    height: calc(100% - 80px);
    overflow-y: auto;
}

/deep/ .el-table__header-wrapper {
    height: auto;
}

.form-container {
    padding: 20px 30px;

    /deep/ .el-dialog__body {
        padding: 0 20px;
    }

    /deep/ .el-form-item {
        margin-bottom: 0 !important;
    }

    /deep/ .el-table .cell {
        overflow: visible;
    }
}

.theader-required::before {
    content: '*';
    color: #f56c6c;
    margin-right: 4px;
}

.my-handle {
    cursor: move;
    font-size: 20px;
}
</style>
