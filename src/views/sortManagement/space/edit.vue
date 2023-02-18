<template>
    <div>
        <!-- <el-card shadow="never">
      <template slot="header">
        <div class="header row row-between">
          <span class="title">基础信息</span>
        </div>
      </template> -->
        <div class="title">{{ $t('pc.space_type.basic_information') }}</div>
        <el-divider></el-divider>
        <div class="row">
            <el-form ref="baseInfo" :model="form" label-suffix="：" label-width="130px" :rules="rules" status-icon>
                <el-row>
                    <el-col :span="12">
                        <el-form-item :label="$t('pc.space_type.space_type')" prop="spaceName">
                            <el-input v-model="form.spaceName" clearable :maxlength="50"></el-input>
                            <!-- :disabled="type == 2" -->
                        </el-form-item>
                    </el-col>
                    <el-col :span="12" v-if="
                        type === 3 ||
                        (level !== 1 && type !== 1) ||
                        (level !== 1 && type === 1)
                    ">
                        <el-form-item v-if="parentId !== 0 || type === 3" :label="$t('pc.space_type.superior_type')"
                            prop="id">
                            <el-cascader ref="cascaderAddr" placeholder="" :options="treeData" :props="{
                                checkStrictly: true,
                                label: 'spaceName',
                                value: 'spaceCode'
                            }" v-model="form.id" clearable :show-all-levels="false" @change="change"
                                :disabled="type == 2"></el-cascader>
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item :label="$t('pc.space_type.type_number')" prop="spaceCode">
                            <el-input v-model="form.spaceCode" clearable :maxlength="50"></el-input>
                        </el-form-item>
                    </el-col>
                </el-row>
            </el-form>
        </div>
        <!-- </el-card> -->
    </div>
</template>

<script>
import { findSpaceTreeList, getSpace, getSpaceByCode } from '../../../api'
export default {
    methods: {
        validate() {
            return new Promise((resolve, reject) => {
                this.$refs.baseInfo.validate().then(() => {
                    const { spaceName, parentId, spaceCode } = this.form
                    if (spaceName === '') {
                        reject(new Error(this.$t('pc.space_type.please_enter_space_type')))
                    } else if (spaceCode === '') {
                        reject(
                            new Error(
                                this.$t('pc.global.plase_input', {
                                    name: this.$t('pc.space_type.type_number')
                                })
                            )
                        )
                    } else {
                        resolve({
                            spaceName,
                            parentId,
                            spaceCode
                        })
                    }
                })
            })
        },
        change(e) {
            if (this.type === 3) {
                this.form.parentId = this.$refs.cascaderAddr.getCheckedNodes()[0].data.id
            } else {
                this.form.parentId = this.$refs.cascaderAddr.getCheckedNodes()[0].data.parentId
            }
        }
    },
    props: {
        id: String,
        parentId: Number,
        type: Number,
        deviceId: Number,
        level: Number
    },
    created() {
        findSpaceTreeList().then(data => {
            // this.treeData = data
            this.$set(this, 'treeData', data)
            if (this.treeData.length === 0) {
                this.rules.id = []
            } else {
                this.rules.id = [
                    {
                        required: true,
                        message: this.$t('pc.global.please_select', {
                            name: this.$t('pc.space_type.superior_type')
                        })
                    }
                ]
            }
            if (this.type === 1) {
                if (this.parentId !== 0 && this.parentId !== null) {
                    // this.form.id = this.id
                    getSpaceByCode({ id: this.deviceId }).then(data => {
                        if (data) {
                            // this.form.id = data.parentName
                            this.$set(this.form, 'id', data.parentCode)
                            this.form.parentId = data.parentId
                        }
                    })
                } else {
                    // this.form.parentId = this.deviceId
                }
            } else if (this.type === 2) {
                getSpace({ id: this.deviceId }).then(data => {
                    Object.assign(this.form, {
                        spaceName: data.spaceName,
                        parentId: data.parentId,
                        spaceCode: data.spaceCode,
                        id: data.parentCode
                    })
                })
            } else {
                this.form.id = this.id
                this.form.parentId = this.deviceId
            }
        })

        // }
    },
    data() {
        return {
            treeData: [],
            form: {
                spaceName: '',
                id: '',
                spaceCode: ''
            },
            val: []
            // rules: {
            //   spaceName: [{ required: true, message: '请输入空间名称' }],
            //   id: [{ required: true, message: '请选择上级类型' }],
            //   spaceCode: [{ required: true, message: '请输入类型编号' }]
            // }
        }
    },
    computed: {
        rules() {
            let obj = {
                spaceName: [
                    { required: true, message: this.$t('pc.space_type.space_type_name') }
                ],
                spaceCode: [
                    {
                        required: true,
                        message: this.$t('pc.global.please_input', {
                            name: this.$t('pc.space_type.type_number')
                        })
                    }
                ]
            }
            if (this.parentId !== 0 || this.type === 3) {
                obj.id = [
                    {
                        required: true,
                        message: this.$t('pc.space_type.please_select', {
                            name: this.$t('pc.space_type.superior_type')
                        })
                    }
                ]
            }
            return obj
        }
    }
}
</script>

<style lang="scss" scoped>
.el-input /deep/ {
    width: auto;
}
</style>
