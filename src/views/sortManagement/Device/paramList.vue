<template>
    <div class="params-list">
        <btn-bar :btns="btns" @btn-click="onBtnClick"></btn-bar>
        <wz-table :data="tableData" :heads="realTableHead" ref="table" :loading="loading"
            @btn-click="onTableBtnClick"></wz-table>
        <el-pagination ref="pager" layout="sizes,prev,pager,next,jumper" :current-page.sync="searchForm.head.current"
            :page-size.sync="searchForm.head.size" :total="searchForm.head.total" @current-change="fetchData"
            @size-change="handleSizeChange" background></el-pagination>
        <!-- 弹窗部分 -->
        <el-dialog :visible.sync="dialog.show" :title="dialog.title" :width="dialog.width" :top="dialog.top || '15vh'"
            :close-on-click-modal="false" destroy-on-close>
            <!-- 表单部分 -->
            <div v-if="dialog.type !== 'form'">
                <h4>设备型号</h4>
                <el-divider></el-divider>
                <el-form :model="form" :rules="rules" label-suffix="：" label-width="100px" status-icon ref="baseInfo">
                    <el-row :gutter="20" class="base-info">
                        <el-col v-for="row in baseInfo" :span="row.span" :key="row.index">
                            <template v-for="item in row.list">
                                <el-form-item :key="item.prop" :label="item.label" :prop="item.prop">
                                    <span v-if="dialog.type !== 'list-add'">{{ form[item.prop]}}</span>
                                    <component v-else :is="item.tag" v-bind="item" v-model="form[item.prop]">
                                    </component>
                                </el-form-item>
                            </template>
                        </el-col>
                    </el-row>
                </el-form>
                <h4>参数设置</h4>
                <el-divider></el-divider>
            </div>

            <!-- 节点部分 -->
            <div class="form-container">
                <!-- 节点 -->
                <div class="form-title" v-if="dialog.type === 'form'">
                    <span>设备类型：{{ activeNodesName }}</span>
                    <el-button type="primary" @click="onAddField">{{
                        $t('pc.global.add')
                    }}</el-button>
                </div>


                <!-- 字段编辑 -->
                <el-form ref="fieldForm" :model="fieldDataCopy">
                    <el-table ref="fieldTable" :data="fieldDataCopy.data" border row-key="id" :max-height="500"
                        style="width: 100%">
                        <el-table-column type="index" width="40" label="#">
                        </el-table-column>
                        <el-table-column v-if="dialog.type === 'form'" width="50" label="排序">
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

                                        <!-- 默认值 -->
                                        <template v-if="item.prop === 'defaultValue' && row.customConfig">

                                            <!-- 文本 -->
                                            <span v-if="row.customConfig.tag === 'txt'">{{ row[item.prop] }}</span>
                                            <!-- 单选多选 -->
                                            <template
                                                v-else-if="row.customConfig.tag === 'radio' || row.customConfig.tag === 'checkbox'">
                                                <component :is="`el-${row.customConfig.tag}-group`"
                                                    v-model="row[item.prop]" :disabled="item.disabled">
                                                    <template v-for="radio in row.customConfig.options">
                                                        <component :is="`el-${row.customConfig.tag}`"
                                                            :label="radio.value" :key="radio.value">{{ radio.label }}
                                                        </component>
                                                    </template>
                                                </component>
                                            </template>

                                            <!-- 文件上传 -->
                                            <el-upload v-else-if="row.customConfig.tag === 'fileUpload'"
                                                :ref="row.fieldKey" class="upload-demo" :accept="accept"
                                                :file-list="row[item.prop]" action="" :headers="uploadHeader"
                                                :on-exceed="onExceed" :auto-upload="false" :limit="1">
                                                <el-button size="small" type="primary">点击上传</el-button>
                                                <div slot="tip" class="el-upload__tip">
                                                    文件格式:
                                                    dwg，psd，zip，rar，pdf，doc，rp，ppt，vsdx，pptx，docx，xls，xlsx，jpg，png，jpeg，gif；
                                                    <br />文件大小：≤50MB
                                                </div>
                                            </el-upload>
                                            <!-- 文件显示 -->
                                            <ul v-else-if="row.customConfig.tag === 'fileList'"
                                                class="el-upload-list el-upload-list--text">
                                                <li v-for="file in row[item.prop]" :key="file.name"
                                                    class="el-upload-list__item is-success" @click="down(file)">
                                                    <a class="el-upload-list__item-name">
                                                        <i class="el-icon-document"></i>{{ file.name }}
                                                    </a>
                                                </li>
                                            </ul>
                                            <!-- 其他 -->
                                            <div v-else class="row">
                                                <el-tooltip v-if="row.customConfig.tooltip" placement="top-start"
                                                    :content="row.customConfig.tooltip"><i
                                                        class="el-icon-question"></i></el-tooltip>
                                                <component :is="row.customConfig.tag" v-bind="row.customConfig"
                                                    :showAllLevels="row.customConfig.showAllLevels || false"
                                                    :disabled="row.customConfig.disabled" v-model="row[item.prop]">
                                                    <template v-if="row.customConfig.append" slot="append">{{
                                                        row.customConfig.append
                                                    }}</template>
                                                </component>
                                            </div>

                                        </template>
                                        <!-- 文本类型 -->
                                        <span v-else-if="!item.tag">{{ row[item.prop] }}</span>
                                        <component v-else :is="item.tag" v-bind="item" v-model="row[item.prop]"
                                            :placeholder="
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
                <el-button v-if="dialog.type !== 'list-detail'" type="primary" @click="onSubmit">{{
                    $t('pc.global.submit')
                }}</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
import { wzTable } from 'wz-ui'
import objArr from 'wz-file/lib/api.js'
import btnBar from '../../../components/btn.vue'
import Sortable from 'sortablejs'
import {
    getFormrelationPage,
    addFormrelation,
    editFormrelation,
    deleteFormrelation,
    getFormrelationById,
    getDeviceFormConfigByKey,
    updateDeviceFormConfig
} from '../../../api'
import dayjs from 'dayjs'
export default {
    name: 'ParamsList',
    components: {
        wzTable,
        btnBar
    },
    props: {
        data: {
            type: Object,
            default: () => { }
        }
    },
    data() {
        return {
            loading: false,
            btns: [
                {
                    content: '添加设备型号',
                    event: 'add'
                },
                {
                    content: '同步',
                    event: 'sync'
                },
                {
                    content: '配置自定义表单',
                    event: 'config'
                }
            ],
            realTableHead: [

                {
                    label: '设备型号',
                    prop: 'formKey'
                },
                {
                    label: '更新时间',
                    prop: 'lastUpdatedDate',
                    formatter: ({ lastUpdatedDate }) => {
                        return lastUpdatedDate && new dayjs(lastUpdatedDate).format('YYYY-MM-DD HH:mm:ss')
                    }
                },
                {
                    label: '更新来源',
                    prop: 'dataSource',
                    formatter: ({ dataSource }) => {
                        return dataSource == 1 ? '手动配置' : '自动配置'
                    }
                },


                {
                    // label: this.$t('pc.global.operate'),
                    label: '操作',
                    width: 150,
                    btns: [
                        {
                            label: '查看',
                            event: 'detail',
                            // code: 'sbtz_ck'
                        },
                        {
                            label: this.$t('pc.global.edit'),
                            event: 'edit',
                            // code: 'sbtz_bj'
                        },
                        {
                            label: this.$t('pc.global.delete'),
                            event: 'delete',
                            // code: 'sbtz_sc'
                        }
                    ]
                }

            ],
            tableData: [],
            searchForm: {
                head: {
                    current: 1,
                    size: 10,
                    total: 0
                }
            },
            dialog: { submit: () => { } },

            // 节点配置相关数据
            uploadRefArr: [],
            activeNodesName: '',
            fieldFormConfig: [],
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
            fieldDataCopy: { data: [] },

            accept: 'dwg，psd，zip，rar，pdf，doc，rp，ppt，vsdx，pptx，docx，xls，xlsx，jpg，png，jpeg，gif',

            form: {
                formKey: ''
            },
            baseInfo: [
                {
                    index: 0,
                    span: 12,
                    list: [
                        {
                            label: '设备型号',
                            prop: 'formKey',
                            tag: 'el-input',
                            clearable: true,
                            maxlength: 40
                        }
                    ]
                }
            ],
            rules: {
                formKey: [
                    {
                        required: true,
                        message: this.$t('请输入设备型号')
                    }
                ]
            },
        }
    },
    watch: {
        data: {
            handler() {
                this.fetchData()
            },
            immediate: true
        }
    },
    methods: {
        // 列表增删改查 部分 -------------------------------------------
        onBtnClick(event) {
            const func = new Map([
                ['add', this.onParamsAdd],
                ['sync', this.onSync],
                ['config', this.onConfig]
            ])
            func.has(event) && func.get(event).call(this)
        },
        async onParamsAdd() {
            this.form.formKey = ''
            this.dialog = {
                show: false,
                title: "添加型号",
                top: '5vh',
                width: '50%',
                type: 'list-add'
            }
            await this.initFormConfig()
            this.dialog.show = true
        },
        async initFormConfig(list) {
            this.fieldFormConfig = [
                {
                    key: 0,
                    prop: 'fieldName',
                    label: '名称',
                    colWidth: 'none',
                    isRequired: true
                },
                {
                    key: 5,
                    prop: 'defaultValue',
                    label: '预设值',
                    colWidth: 'none'
                }
            ]
            await this.setFormConfig()
            const formConfig = this.fieldDataCopy.data
            this.uploadRefArr = []

            formConfig.forEach((item, index) => {
                // 自定义字段
                const { fieldName, fieldKey, fieldType } = item
                // 用保存的默认值 覆盖 模板中空的默认值
                if (list) {
                    list.forEach(subitem => {
                        if (subitem.fieldKey === fieldKey) {
                            item.defaultValue = subitem.defaultValue
                        }
                    })
                }
                const { defaultValue } = item
                const customConfig = {
                    label: fieldName,
                    prop: fieldKey,
                    clearable: true,
                    // disabled: formKey !== 'other'
                }

                switch (fieldType) {
                    case 'textarea':
                        customConfig.tag = this.dialog.type === 'list-detail' ? 'txt' : 'el-input'
                        customConfig.type = 'textarea'
                        customConfig.maxlength = 200
                        break;
                    case 'radio':
                        customConfig.tag = this.dialog.type === 'list-detail' ? 'txt' : 'radio'
                        customConfig.options = item.contentSettings.split(',').map((subItem) => {
                            return { label: subItem, value: subItem }
                        })
                        break;
                    case 'checkbox':
                        customConfig.tag = this.dialog.type === 'list-detail' ? 'txt' : 'checkbox'
                        customConfig.options = item.contentSettings.split(',').map((subItem) => {
                            return { label: subItem, value: subItem }
                        })
                        item.defaultValue = item.defaultValue ? item.defaultValue.split(',') : []
                        break;
                    case 'datepicker':
                        customConfig.tag = this.dialog.type === 'list-detail' ? 'txt' : 'el-date-picker'
                        customConfig.type = 'date'
                        customConfig.editable = false
                        customConfig.format = 'yyyy-MM-dd'
                        customConfig.valueFormat = 'yyyy-MM-dd'
                        break;
                    case 'fileUpload':
                        if (defaultValue) {
                            item.defaultValue = JSON.parse(defaultValue)
                        } else {
                            item.defaultValue = []
                        }
                        this.uploadRefArr.push(fieldKey)
                        customConfig.tag = this.dialog.type === 'list-detail' ? 'fileList' : 'fileUpload'
                        break;
                    default:
                        // 设置默认的控件为输入框
                        customConfig.tag = this.dialog.type === 'list-detail' ? 'txt' : 'el-input'
                        customConfig.maxlength = 40
                        break;
                }
                item.customConfig = customConfig
            })
        },
        onSync() {

        },
        async onConfig() {
            this.fieldFormConfig = [
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
                    key: 6,
                    colWidth: 200,
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
                    isRequired: true,
                    placeholder: '选项用英文逗号分隔'
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
            ]
            await this.setFormConfig()
            // 有数值的话表示可以继续后续操作
            this.dialog = {
                show: true,
                title: "设备类型自定义表单",
                top: '5vh',
                width: '70%',
                type: 'form'
            }
            this.$nextTick(() => {
                this.setSortable()
            })
        },
        onTableBtnClick({ event, row }) {
            const func = new Map([
                ['detail', this.onDetail],
                ['edit', this.onEdit],
                ['delete', this.onDelete]
            ])
            func.has(event) && func.get(event).call(this, row)
        },
        async onDetail(row) {
            const res = await getFormrelationById(row.id)
            this.selectData = res
            const { formVo: { formKey, formConfigurationVos } } = res
            this.form = { formKey }
            this.dialog.type = 'list-detail'
            this.dialog = {
                show: false,
                title: "查看型号",
                top: '5vh',
                width: '50%',
                type: 'list-detail'
            }
            await this.initFormConfig(formConfigurationVos)
            this.dialog.show = true
        },
        async onEdit(row) {
            const res = await getFormrelationById(row.id)
            this.selectData = res
            const { formVo: { formKey, formConfigurationVos } } = res
            this.form = { formKey }
            this.dialog = {
                show: false,
                title: "编辑型号",
                top: '5vh',
                width: '50%',
                type: 'list-edit'
            }
            this.initFormConfig(formConfigurationVos)
            this.dialog.show = true
        },
        onDelete({ id, formKey }) {
            this.$confirm(`是否删除${formKey}`, {
                type: 'warning'
            })
                .then(() => {
                    deleteFormrelation(id).then(() => {
                        this.$message.success(`删除${formKey}成功`)
                        this.fetchData()
                    })
                })
                .catch(err => console.dir(err))
        },
        handleSizeChange() {
            this.searchForm.head.size = 1
            this.fetchData()
        },
        fetchData() {
            const { facCateCode } = this.data
            const body = {
                relationId: facCateCode,
                relationType: 'fac_cate'
            }
            const { head } = this.searchForm
            getFormrelationPage({ body, head })
                .then(({ records, current, size, total }) => {
                    Object.assign(this.searchForm.head, { current, size, total })
                    this.tableData = records
                })
                .catch(err => console.dir(err))
        },

        // 节点编辑部分 ------------------------------------------------------------
        async setFormConfig() {
            this.fieldDataCopy.data = []
            const { facCateName: formName, facCateCode: formKey } = this.data
            this.nodes = [
                {
                    label: formName,
                    name: formKey
                }
            ]
            this.nodes.forEach(({ name }) => {
                this.fieldData[name] = []
            })
            const { formConfigurationVos } = await getDeviceFormConfigByKey({ formKey })
            if (formConfigurationVos) {
                formConfigurationVos.forEach(item => {
                    this.fieldData[item.parentNode].push(item)
                })
            }
            this.activeNodes = this.nodes[0].name
            this.activeNodesName = this.nodes[0].label
            this.fieldDataCopy = { data: this.fieldData[this.activeNodes] }
            return formConfigurationVos
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
        // 自定义表单提交
        onNodesSubmit() {
            this.$refs.fieldForm.validate(res => {
                if (res) {
                    const { label: formName, name: formKey } = this.nodes[0]
                    const formConfigurationVos = this.fieldData[this.activeNodes].map((item, index) => {
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
                    const params = {
                        formName,
                        formKey,
                        formConfigurationVos
                    }
                    updateDeviceFormConfig(params).then(res => {
                        this.$message.success(`${this.activeNodesName}保存成功`)
                        this.dialog.show = false
                    })
                }
            })
        },
        //  添加设备型号
        async onAddOrEditSubmit(isEdit) {
            // 参数配置是否有文件上传类型 以及是否需要上传文件
            const res = await this.$refs.baseInfo.validate()
            if (!res) return
            const { formKey } = this.form
            if (this.uploadRefArr.length) {
                const fileUploadArr = []
                this.uploadRefArr.forEach((item) => {
                    const {
                        uploadFiles,
                        uploadFiles: { length }
                    } = this.$refs[item][0]
                    if (length > 0) {
                        uploadFiles.forEach((uploadFile) => {
                            const formData = new FormData()
                            const docSize = uploadFile.size

                            const arr = uploadFile.name.split('.')
                            const acceptList = this.accept.split('，')
                            const suffix = arr[arr.length - 1]
                            if (!acceptList.includes(suffix)) {
                                this.$message.error(`文件限制格式: ${this.accept}！`)
                                return
                            }

                            if (docSize / 1024 / 1024 > 50) {
                                this.$message.error('文件大小不能超过50MB！')
                                return uploadRefArr
                            }

                            if (!uploadFile.fid) {
                                formData.append('file', uploadFile.raw)
                                formData.append('classify', 'user')
                                formData.append('bizPath', 'device')
                                const { upload } = objArr
                                fileUploadArr.push(upload(formData))
                            }
                        })
                    }
                })
                if (fileUploadArr.length) {
                    const res = await Promise.all(fileUploadArr)
                    this.uploadRefArr.forEach((item, index) => {
                        const { fileName, url, fid } = res[index]
                        const i = this.fieldDataCopy.data.findIndex(subitem => subitem.fieldKey === item)
                        this.fieldDataCopy.data[i].defaultValue = JSON.stringify([{
                            name: fileName,
                            fid
                        }])
                    })
                }
            }
            // 处理其他参数
            this.fieldDataCopy.data.forEach(item => {
                if (Array.isArray(item.defaultValue)) {
                    if (item.fieldType === 'fileUpload') {
                        item.defaultValue = JSON.stringify(item.defaultValue)
                    } else {
                        item.defaultValue = item.defaultValue.join(',')
                    }
                }

                item.formKey = formKey
                item.formName = formKey
                item.parentNode = formKey
                delete item.customConfig
            })

            const { facCateCode } = this.data
            if (isEdit) {
                const { id, formId } = this.selectData
                editFormrelation({
                    id,
                    formId,
                    // 1 手动 2 自动
                    dataSource: 1,
                    formKey,
                    relationId: facCateCode,
                    relationType: 'fac_cate',
                    formVo: {
                        formConfigurationVos: this.fieldDataCopy.data,
                        formKey,
                        formName: formKey
                    }
                }).then(() => {
                    this.$message.success(`保存成功`)
                    this.dialog.show = false
                    this.fetchData()
                })
            } else {
                addFormrelation({
                    // 1 手动 2 自动
                    dataSource: 1,
                    formKey,
                    relationId: facCateCode,
                    relationType: 'fac_cate',
                    formVo: {
                        formConfigurationVos: this.fieldDataCopy.data,
                        formKey,
                        formName: formKey
                    }
                }).then(() => {
                    this.$message.success(`保存成功`)
                    this.dialog.show = false
                    this.fetchData()
                })
            }

        },

        onSubmit() {
            const { type } = this.dialog
            if (type === 'form') {
                this.onNodesSubmit()
            } else if (type === 'list-add') {
                this.onAddOrEditSubmit()
            } else if (type === 'list-edit') {
                this.onAddOrEditSubmit(true)
            }
        },
        // 文件相关
        async down({ fid, name }) {
            const file = await objArr.download(fid)
            download(file, name, this)
        },
        onExceed() {
            this.$message.warning('仅能上传一个文件,清删除后再上传')
        }

    },
    computed: {
        uploadHeader() {
            const token = localStorage.getItem('token')
            const userId = sessionStorage.getItem('userId')
            const propertyId = localStorage.getItem('propertyId')
            const loginType = sessionStorage.getItem('uis')
            const userName = sessionStorage.getItem('userName')
            const type = sessionStorage.getItem('type')
            if (loginType) {
                return {
                    Authorization: JSON.stringify({
                        token,
                        userId,
                        propertyId,
                        loginType,
                        userName,
                        type
                    })
                }
            } else {
                return {
                    Authorization: JSON.stringify({
                        token,
                        userId,
                        propertyId,
                        userName,
                        type
                    })
                }
            }
        }

    }
}
</script>

<style lang="scss" scoped>
.form-title {
    display: flex;
    justify-content: space-between;
}

.form-container {
    /deep/ .el-form-item {
        margin-bottom: 10 !important;
    }

    /deep/ .el-table__cell {
        padding-bottom: 0 !important;
    }
}
</style>
