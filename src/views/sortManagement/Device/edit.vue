<template>
    <div>
        <!-- <el-card shadow="never">
      <template slot="header">
        <div class="header row row-between">
          <span class="title">基础信息</span>
        </div>
      </template> -->
        <div class="title">{{ $t('pc.global.basic_information') }}</div>
        <el-divider></el-divider>
        <div class="row">
            <el-form ref="baseInfo" :model="form" label-suffix="：" label-width="140px" :rules="rules" status-icon>
                <el-row>
                    <el-col :span="12">
                        <el-form-item :label="$t('pc.device_type.category_name')" prop="facCateName">
                            <!-- <div class="tooltipRemark"></div> -->
                            <el-input :maxLength="50" v-model="form.facCateName" clearable></el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="12" v-if="type === 3 || level !== 1">
                        <el-form-item :label="$t('pc.device_type.superior_type')" prop="id">
                            <el-cascader ref="cascaderAddr" placeholder="" :options="treeData" :props="{
                                checkStrictly: true,
                                label: 'facCateName',
                                value: 'facCateCode'
                            }" v-model="form.id" clearable :show-all-levels="false" @change="change"
                                :disabled="type == 2"></el-cascader>
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <!-- 设备级别 -->
                        <el-form-item :label="$t('pc.device_type.device_level')" prop="level">
                            <!-- <div class="tooltipRemark">
                  <el-tooltip
                    placement="top-start"
                    content="只有设备级别为设备类型时，才可以在该设备分类下添加具体设备。"
                    ><i class="el-icon-question"></i
                  ></el-tooltip>
                </div> -->
                            <el-select v-model="form.level" clearable :disabled="type == 2">
                                <template v-for="opt in levelList">
                                    <el-option :key="opt.value" v-bind="opt"></el-option>
                                </template>
                            </el-select>
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <!-- 类型编号 -->
                        <el-form-item :label="$t('pc.device_type.type_number')" prop="facCateCode">
                            <el-input v-model="form.facCateCode" clearable :disabled="type == 2"
                                :maxlength="50"></el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="12" v-if="form.level === 4">
                        <el-form-item label="设备来源" prop="facSource">
                            <el-select v-model="form.facSource" clearable>
                                <template v-for="opt in deviceSource">
                                    <el-option :key="opt.value" v-bind="opt"></el-option>
                                </template>
                            </el-select>
                        </el-form-item>
                    </el-col>
                    <el-col :span="12" v-if="form.level === 4 && form.facSource === 2">
                        <el-form-item label="IoT平台" prop="iotPlatform" key='iotPlatform'>
                            <el-select v-model="form.iotPlatform" clearable>
                                <template v-for="opt in iotPlatform">
                                    <el-option :key="opt.value" v-bind="opt"></el-option>
                                </template>
                            </el-select>
                        </el-form-item>
                    </el-col>
                    <el-col :span="12" v-if="form.level === 4 && form.facSource === 2">
                        <el-form-item label="物模型" prop="productId" key='productId'>
                            <el-select v-model="form.productId" clearable>
                                <template v-for="opt in physicalModel">
                                    <el-option :key="opt.value" v-bind="opt" :disabled="opt.disabled"></el-option>
                                </template>
                            </el-select>
                        </el-form-item>
                    </el-col>
                    <el-col :span="12" v-if="form.level === 4 && form.facSource === 3">
                        <el-form-item label="业务平台" prop="servicePlatform" key='servicePlatform'>
                            <el-input v-model="form.servicePlatform" placeholder="请输入业务平台" clearable
                                :maxlength="50"></el-input>
                        </el-form-item>
                    </el-col>
                    <!-- <el-col :span="12" v-if="form.level == 4">
              <el-form-item label="设备品牌" prop="facCateBrand">
                <div :class="type == 2 ? 'tooltipRemark' : ''"></div>
                <el-input v-model="form.facCateBrand" clearable></el-input>
              </el-form-item>
            </el-col>
            <el-col :span="12" v-if="form.level == 4">
              <el-form-item label="设备型号" prop="facCateModel">
                <div :class="type == 2 ? '' : 'tooltipRemark'"></div>
                <el-input v-model="form.facCateModel" clearable></el-input>
              </el-form-item>
            </el-col>
            <el-col :span="12" v-if="form.level == 4">
              <el-form-item label="重要级别" prop="facCateLevel">
                <div :class="type == 2 ? 'tooltipRemark' : ''"></div>
                <el-select v-model="form.facCateLevel" clearable>
                  <template v-for="opt in facCateLevelList">
                    <el-option :key="opt.value" v-bind="opt"></el-option>
                  </template>
                </el-select>
              </el-form-item>
            </el-col> -->
                </el-row>
            </el-form>
        </div>
        <!-- 暂时注释掉上次图片的功能 -->
        <div class="row col-top device-upload" v-show="false">
            <span class="upload-label">上传图片：</span>
            <custom-upload ref="upload" name="imgFile" uploadTip="图片格式：png，jpg，jif；图片大小：≤10MB" accept=".jpg,.png,.gif"
                :showFileList="true" :autoUpload="false" :action="''" :limit="1"
                :class="{ 'no-show-upload': !showUpload }" list-type="picture-card" @onChange="onFileListChange"
                @onRemove="onFileRemove" @onDelete="handleDownload" :file-list="fileList"></custom-upload>
        </div>
        <!-- </el-card> -->
    </div>
</template>

<script>
import {
    findCategoryTree,
    getFacCategory,
    getFacCategoryById,
    categoryGetProducts
} from '../../../api'
import customUpload from "../../../components/upload.vue";
import fileApi from "wz-file/lib/api.js";
const uploadAllFile = fileApi.upload;
export default {
    components: { customUpload },
    computed: {
        levels() {
            return this.levelList.filter(({ value }) => {
                // 一级设备不能是'设备类型'的分类
                if (this.level === 1 && this.type === 1) {
                    return value !== 4
                }
                return true
            })
        }
    },
    watch: {
        'form.iotPlatform': {
            async handler(newVal) {
                console.log('newVal', newVal)
                const _this = this
                if (newVal) {
                    await categoryGetProducts()
                        .then(data => {
                            _this.physicalModel = data.map(item => {
                                return {
                                    label: item.name,
                                    value: item.id
                                }
                            })
                        })
                        .catch(err => {
                            console.log(err)
                        })
                }
            },
            deep: true
        },
        'form.facSource': {
            handler(newVal) {
                if (newVal === 1) {
                    this.form.iotPlatform = ''
                    this.form.productId = ''
                    this.form.servicePlatform = ''
                }
            }
        }
    },
    methods: {
        handleDownload(file) {
            this.$refs.upload.dom.handleRemove(file)
        },
        onFileListChange(file, fileList) {
            const { length = 0 } = fileList || []
            length === 1 && (this.showUpload = false)
        },
        onFileRemove() {
            this.iconPath = ''
            this.showUpload = true
        },
        validate() {
            return new Promise((resolve, reject) => {
                this.$refs.baseInfo.validate().then(async () => {
                    const {
                        facCateName,
                        parentId,
                        level,
                        facCateCode,
                        facCateBrand,
                        facCateModel,
                        facCateLevel,
                        facSource,
                        iotPlatform,
                        servicePlatform,
                        productId
                    } = this.form
                    let iconPath = this.iconPath
                    const {
                        uploadFiles,
                        uploadFiles: { length }
                    } = this.$refs.upload.dom
                    console.log(uploadFiles)
                    console.log(this.$refs)
                    console.log(iconPath)
                    if (length > 0 && iconPath === '') {
                        const [uploadFile] = uploadFiles
                        const formData = new FormData()
                        formData.append('file', uploadFile.raw)
                        formData.append("classify", "assets");
                        formData.append("bizPath", "property");

                        try {
                            const { fid = "" } = await uploadAllFile(formData)
                            iconPath = fid;
                        } catch (e) {
                            reject(e)
                        }
                    }
                    if (facCateName === '') {
                        reject(
                            new Error(
                                this.$t('pc.device_type.please_enter_the_type_of_device')
                            )
                        )
                    } else if (level === '') {
                        reject(
                            new Error(
                                this.$t('pc.global.please_select', {
                                    name: this.$t('pc.device_type.device_level')
                                })
                            )
                        )
                    } else {
                        resolve({
                            facCateName,
                            parentId,
                            level,
                            facCateCode,
                            facCateBrand,
                            facCateModel,
                            facCateLevel,
                            icon: iconPath,
                            facSource,
                            iotPlatform,
                            servicePlatform,
                            productId
                        })
                    }
                })
            })
        },
        change(e) {
            console.log(e)
            console.log(this.$refs.cascaderAddr.getCheckedNodes())
            console.log(this.$refs.cascaderAddr.getCheckedNodes()[0].level)
            const level = this.$refs.cascaderAddr.getCheckedNodes()[0].level
            if (level) {
                const levelList = [
                    {
                        value: 1,
                        label: this.$t('pc.device_type.primary_device_system')
                    },
                    {
                        value: 2,
                        label: this.$t('pc.device_type.second_device_system')
                    },
                    {
                        value: 3,
                        label: this.$t('pc.device_type.third_device_system')
                    },
                    {
                        value: 4,
                        label: this.$t('pc.role_permissions.device_type')
                    }
                ]
                if (this.type === 1 || this.type === 2) {
                    this.levelList = levelList.splice(level - 1)
                } else {
                    this.levelList = levelList.splice(level)
                }
            }
            if (e.length === 0) {
                return
            }
            if (this.type === 3) {
                this.form.parentId = this.$refs.cascaderAddr.getCheckedNodes()[0].data.id
            } else {
                this.form.parentId = this.$refs.cascaderAddr.getCheckedNodes()[0].data.parentId
            }
        },
        parseTreeData(treeData) {
            return treeData.map(node => {
                console.log(node)
                const { level, ...rest } = node
                const obj = {
                    level,
                    ...rest,
                    disabled: level === 4
                }
                if (Array.isArray(node.children)) {
                    obj.children = this.parseTreeData(node.children)
                }
                return obj
            })
        }
    },
    props: {
        id: String,
        parentId: {
            type: Number
        },
        type: Number,
        deviceId: Number,
        level: Number
    },
    async created() {
        // console.log(this.type, this.deviceId, this.level, this.id, this.parentId)
        const _this = this
        // await categoryGetProducts()
        //   .then(data => {
        //     console.log(data)
        //   })
        //   .catch(err => console.dir(err))
        await findCategoryTree()
            .then(data => {
                console.log(this.parseTreeData(data))
                this.treeData = this.parseTreeData(data)
            })
            .catch(err => {
                console.log(err)
            })
        if (this.treeData.length === 0) {
            this.rules.id = []
        } else {
            this.rules.id = [
                {
                    required: true,
                    message: this.$t('pc.global.please_select', {
                        name: this.$t('pc.device_type.superior_type')
                    })
                }
            ]
        }
        // this.form.parentId = this.parentId
        if (this.type === 1) { // 新增同级
            if (
                this.parentId !== 0 &&
                this.parentId !== null &&
                this.parentId !== undefined
            ) {
                await getFacCategory({ id: this.deviceId })
                    .then(data => {
                        // this.form.id = data.parentCode
                        // this.form.parentId = data.parentId
                        if (data) {
                            this.$set(this.form, 'id', data.parentCode)
                            this.$set(this.form, 'parentId', data.parentId)
                        }
                    })
                    .catch(err => {
                        console.log(err)
                    })
            }
        } else if (this.type === 2) { // 编辑
            await getFacCategoryById({ id: this.deviceId })
                .then(data => {
                    if (data.iotPlatform) {
                        categoryGetProducts()
                            .then(res => {
                                const physicalModelData = res.map(item => {
                                    return {
                                        label: item.name,
                                        value: item.id
                                    }
                                })
                                _this.$set(_this, 'physicalModel', [...physicalModelData])
                                console.log('physicalModel', _this.physicalModel)
                                Object.assign(this.form, {
                                    productId: Number(data.productId)
                                })
                                this.$nextTick(() => {
                                    this.$refs.baseInfo.clearValidate()
                                })
                            })
                            .catch(err => {
                                console.log(err)
                            })
                    }
                    // 初始化图片赋值
                    if (data.icon) {
                        this.fileList = [
                            {
                                url: data.icon,
                            },
                        ];
                        this.showUpload = false;
                    }
                    Object.assign(this.form, {
                        facCateName: data.facCateName,
                        parentId: data.parentId,
                        level: data.level,
                        facCateCode: data.facCateCode,
                        facCateBrand: data.facCateBrand,
                        facCateModel: data.facCateModel,
                        facCateLevel: data.facCateLevel,
                        id: data.parentCode,
                        facSource: data.facSource,
                        iotPlatform: data.iotPlatform,
                        servicePlatform: data.servicePlatform
                    })
                })
                .catch(err => {
                    console.log(err)
                })
        } else { // 新增下级
            // if (this.parentId !== 0) {
            this.form.id = this.id
            this.form.parentId = this.deviceId
            // }
        }
        // }
        const levelList = [
            {
                value: 1,
                label: this.$t('pc.device_type.primary_device_system')
            },
            {
                value: 2,
                label: this.$t('pc.device_type.second_device_system')
            },
            {
                value: 3,
                label: this.$t('pc.device_type.third_device_system')
            },
            {
                value: 4,
                label: this.$t('pc.role_permissions.device_type')
            }
        ]
        if (this.type === 1 || this.type === 2) {
            this.levelList = levelList.splice(this.level - 1)
        } else {
            this.levelList = levelList.splice(this.level)
        }
    },
    data() {
        return {
            iconPath: '',
            fileList: [],
            showUpload: true,
            treeData: [],
            level1: 0,
            levelList: [
                {
                    value: 1,
                    label: this.$t('pc.device_type.primary_device_system')
                },
                {
                    value: 2,
                    label: this.$t('pc.device_type.second_device_system')
                },
                {
                    value: 3,
                    label: this.$t('pc.device_type.third_device_system')
                },
                {
                    value: 4,
                    label: this.$t('pc.role_permissions.device_type')
                }
            ],
            deviceSource: [
                {
                    label: '系统分类',
                    value: 1
                },
                {
                    label: 'IoT物模型',
                    value: 2
                },
                {
                    label: '业务系统',
                    value: 3
                }
            ],
            iotPlatform: [
                {
                    label: '招商集团IoT',
                    value: 1
                }
            ],
            facCateLevelList: [
                {
                    value: 1,
                    label: 'A'
                },
                {
                    value: 2,
                    label: 'B'
                },
                {
                    value: 3,
                    label: 'C'
                }
            ],
            physicalModel: [],
            form: {
                id: '',
                facCateName: '',
                parentId: '',
                level: '',
                facCateCode: '',
                facCateBrand: '',
                facCateModel: '',
                facCateLevel: '',
                facSource: '',
                iotPlatform: '',
                servicePlatform: '',
                productId: ''
            },
            val: [],
            rules: {
                facCateName: [
                    {
                        required: true,
                        message: this.$t('pc.global.please_input', {
                            name: this.$t('pc.device_type.category_name')
                        })
                    }
                ],
                id: [
                    {
                        required: true,
                        message: this.$t('pc.global.please_select', {
                            name: this.$t('pc.device_type.superior_type')
                        })
                    }
                ],
                level: [
                    {
                        required: true,
                        message: this.$t('pc.global.please_select', {
                            name: this.$t('pc.device_type.device_level')
                        })
                    }
                ],
                facCateCode: [{ required: true, message: '请输入类型编码' }],
                facSource: [{ required: true, message: '请选择设备来源' }],
                iotPlatform: [{ required: true, message: '请选择IoT平台' }],
                servicePlatform: [{ required: true, message: '请输入业务平台' }],
                productId: [{ required: true, message: '请选择物模型' }]
            }
        }
    }
}
</script>

<style lang="scss" scoped>
// .el-input /deep/ {
//   width: auto;
// }
.el-input,
.el-select,
.el-cascader {
    height: 32px;
    width: 100%;
}

.tooltipRemark {
    display: inline-block;
    width: 20px;
}

.no-show-upload /deep/ {
    .el-upload--picture-card {
        display: none;
    }
}

.device-upload {
    .upload-label {
        color: #606266;
        font-size: 14px;
        text-align: right;
        width: 140px;
        padding-right: 12px;
    }
}
</style>
