<template>
    <el-container class="device">
        <el-aside width="330px" ref="treeBox" style="height:100%">
            <el-card shadow="always" class="tree">
                <template slot="header">
                    <el-input :placeholder="$t('pc.device_type.please_enter_the_type_of_device')" size="small" clearable
                        v-model="deviceName" @clear="onFilter">
                        <el-button slot="append" icon="el-icon-search" @click="onFilter"></el-button>
                    </el-input>
                </template>
                <el-tree :data="treeData" @node-click="onNodeClick" :expand-on-click-node="false" ref="tree"
                    node-key="deviceId" :filter-node-method="filterNode" style="overflow-y:auto"
                    :default-expanded-keys="treeExpandData">
                    <div slot-scope="{ node }" class="tree-node">
                        <el-tooltip :content="node.label" :disabled="isShowTooltip" :open-delay="300" placement="top"
                            effect="dark">
                            <span class="over-ellipsis" @mouseover="mouseOver($event)">
                                {{ node.label }}
                            </span>
                        </el-tooltip>
                    </div>
                </el-tree>
            </el-card>
        </el-aside>
        <el-main class="table-container">
            <btn-bar :btns="btns" @btn-click="onBtnClick"></btn-bar>
            <el-divider></el-divider>
            <component :is="com.tag" v-bind="com" @cancel="onCancel" @save="onSave"></component>
        </el-main>
        <el-dialog :visible.sync="dialog.show" :title="dialog.title" :width="dialog.width" :top="dialog.top || '15vh'"
            destroy-on-close>
            <component v-if="dialog.show" :is="dialog.tag" v-bind="dialog" ref="panel" title=""></component>
            <template v-if="dialog.tag === 'check'">
                <span slot="footer" class="dialog-footer">
                    <el-button @click="dialog.show = false" :disabled="dialog.loading">{{
                        $t('pc.global.return')
                    }}</el-button>
                </span>
            </template>
            <template v-else-if="dialog.tag">
                <span slot="footer" class="dialog-footer">
                    <el-button @click="dialog.show = false" :disabled="dialog.loading">{{
                        $t('pc.global.cancel')
                    }}</el-button>
                    <el-button type="primary" @click="dialog.submit" :loading="dialog.loading">{{
                        dialog.submitText ||
                            $t('pc.global.confirm')
                    }}</el-button>
                </span>
            </template>
        </el-dialog>
        <el-dialog :visible.sync="dialog1.show" :title="dialog1.title" :width="dialog1.width"
            :top="dialog1.top || '15vh'" destroy-on-close>
            <component v-if="dialog1.show" :is="dialog1.tag" v-bind="dialog1" ref="panel"
                @on-success="onTemplateUploadOK" @on-error="onTemplateUploadFail" @noFile="noFile" title=""></component>
            <template v-if="dialog1.tag === 'detail' && dialog1.showConfirm !== false">
                <span slot="footer" class="dialog-footer">
                    <el-divider></el-divider>
                    <el-button @click="dialog1.show = false" :disabled="dialog1.loading">{{
                        $t('pc.global.return')
                    }}</el-button>
                </span>
            </template>
            <template v-else>
                <span slot="footer" class="dialog-footer">
                    <el-divider></el-divider>
                    <el-button v-if="dialog1.showCancel !== false" @click="dialog1.show = false"
                        :disabled="dialog1.loading">{{ dialog1.cancelText || $t('pc.global.cancel') }}</el-button>
                    <el-button v-if="dialog1.showConfirm !== false" type="primary" @click="dialog1.submit"
                        :loading="dialog1.loading">{{ dialog1.submitText || $t('pc.global.save') }}</el-button>
                </span>
            </template>
        </el-dialog>
    </el-container>
</template>

<script>
import {
    findCategoryTree,
    getFacCategory,
    createFacCategory,
    delFacCategory,
    updateFacCategory
} from '../../../api'
// import { treeHeight } from '@/mixins/treeTable'
import { tablePage } from '../../../mixins/mixins'
import { uploadFile } from '../../../mixins/baseMixins'
import detail from './Detail.vue'
import edit from './Edit.vue'
import dataImport from '../../../components/dataImport.vue'
export default {
    name: 'DeviceManage',
    mixins: [tablePage.call(this), uploadFile.call(this)],
    methods: {
        // 超出显示
        mouseOver(event) {
            this.isShowTooltip =
                event.currentTarget.scrollWidth <= event.currentTarget.clientWidth
        },
        // 树过滤
        onFilter() {
            this.$refs.tree.filter(this.deviceName)
        },
        filterNode(value, data) {
            // if (!value) return true
            // return data.label.indexOf(value) !== -1
            return data.label.includes(value)
        },
        // 点击树，获取基础信息
        onNodeClick(org) {
            this.treeExpandData = [org.deviceId]
            this.$set(this, 'org', org)
            getFacCategory({ id: org.deviceId }).then(data => {
                this.$set(this, 'com', {
                    tag: 'detail',
                    detail: data
                })
            })
        },
        // 取消
        onCancel() {
            const { id } = this.com.detail
            if (id) {
                this.com.tag = 'detail'
            } else {
                const { value, orgCode } = this.org
                findCategoryTree({ orgCode, orgId: value }).then(data => {
                    this.$set(this, 'com', {
                        tag: 'detail',
                        detail: data
                    })
                })
            }
        },
        // 保存
        onSave() {
            const { value, orgCode } = this.org
            findCategoryTree({ orgCode, orgId: value }).then(data => {
                this.$set(this, 'com', {
                    tag: 'detail',
                    detail: data
                })
            })
            this.fetchOrgData()
        },
        // 解析数据
        parseTreeNode(node, key = 'id') {
            return node.map(
                ({ facCateName, facCateCode, children, level, parentId, id }) => {
                    const obj = {
                        [key]: facCateCode,
                        label: facCateName,
                        disabled: level < 3 && !Array.isArray(children),
                        level,
                        parentId,
                        deviceId: id
                    }
                    if (Array.isArray(children)) {
                        obj.children = this.parseTreeNode(children, key)
                    }
                    return obj
                }
            )
        },
        // 获取左侧树
        fetchData() {
            findCategoryTree().then(data => {
                if (data.length !== 0) {
                    this.treeData = this.parseTreeNode(data)
                    if (this.treeData.length !== 0) {
                        this.treeExpandData = [this.treeData[0].deviceId]
                        this.$set(this, 'org', this.treeData[0])
                        getFacCategory({ id: this.treeData[0].deviceId }).then(data => {
                            this.$set(this, 'com', {
                                tag: 'detail',
                                detail: data
                            })
                        })
                    }
                } else {
                    this.treeData = []
                    this.treeExpandData = []
                }
            })
        },
        // 新增同级
        onBrotherNodeAdd() {
            const { id, parentId, level, deviceId } = this.org
            this.$set(this, 'dialog', {
                tag: 'edit',
                show: true,
                loading: false,
                title: this.$t('pc.device_type.add_device_category'),
                id,
                deviceId,
                parentId,
                type: 1,
                width: '800px',
                level: level,
                submitText: this.$t('pc.global.save'),
                submit: this.handleDeviceAdd
            })
        },
        // 新增同级保存
        handleDeviceAdd() {
            this.$refs.panel.validate().then(form => {
                this.dialog.loading = true
                createFacCategory(form)
                    .then(() => {
                        this.dialog.show = false
                        this.$message.success(
                            this.$t('pc.global.add_success', {
                                name: this.$t('pc.device_type.device_category')
                            })
                        )
                        this.fetchData()
                        this.com.tag = ''
                    })
                    .finally(() => (this.dialog.loading = false))
            })
        },
        // 新增下级
        onChildrenNodeAdd() {
            const { id, parentId, deviceId, level } = this.org
            this.$set(this, 'dialog', {
                tag: 'edit',
                show: true,
                loading: false,
                title: this.$t('pc.global.add_something', {
                    name: this.$t('pc.device_type.device_type')
                }),
                id,
                parentId,
                deviceId,
                type: 3,
                width: '800px',
                level: level,
                submitText: this.$t('pc.global.save'),
                submit: this.handleDeviceAdd
            })
        },
        // 编辑
        onNodeEdit() {
            const { id, parentId, deviceId, level } = this.org
            this.$set(this, 'dialog', {
                tag: 'edit',
                show: true,
                loading: false,
                title: this.$t('pc.global.edit_something', {
                    name: this.$t('pc.device_type.device_category')
                }),
                id,
                parentId,
                deviceId,
                type: 2,
                width: '800px',
                level: level,
                submitText: this.$t('pc.global.save'),
                submit: this.handleDeviceEdit
            })
        },
        // 编辑保存按钮
        handleDeviceEdit() {
            this.$refs.panel.validate().then(form => {
                this.dialog.loading = true
                form.id = this.org.deviceId
                updateFacCategory(form)
                    .then(() => {
                        this.dialog.show = false
                        this.$message.success(
                            this.$t('pc.global.edit_success', {
                                name: this.$t('pc.device_type.device_category')
                            })
                        )
                        this.fetchData()
                        this.com.tag = ''
                    })
                    .finally(() => (this.dialog.loading = false))
            })
        },
        // 删除
        onNodeDelete() {
            this.treeExpandData = [this.org.parentId]
            const { deviceId } = this.org
            const h = this.$createElement
            this.$msgbox({
                title: this.$t('pc.global.hint'),
                message: h('p', { class: 'flex-column' }, [
                    h('span', { style: 'color:#F56C6C' }, this.$t('pc.device_type.note3'))
                    // h('span', null, '确定继续删除吗？')
                ]),
                showCancelButton: true
            }).then(() => {
                const { defaultFlag = 0 } = this.com.detail
                if (defaultFlag === 1) {
                    return this.$message.info('内置设备不可删除！')
                }

                delFacCategory({ id: deviceId }).then(() => {
                    this.$message.success(
                        this.$t('pc.global.delete_success', {
                            name: this.$t('pc.device_type.category')
                        })
                    )
                    this.fetchData()
                    this.com.tag = ''
                })
            })
        },
        // 批量导入
        onImport() {
            // this.$set(this, 'dialog1', {
            //   tag: 'import',
            //   title: '批量导入',
            //   show: true,
            //   width: '532px',
            //   type: 'facCate',
            //   loading: false,
            //   submit: this.handleImport
            // })
            this.$set(this, 'dialog1', {
                show: true,
                tag: 'data-import',
                title: this.$t('pc.global.batch_import'),
                width: '532px',
                loading: false,
                submit: this.handleImport,
                status: 1,
                submitText: this.$t('pc.global.import'),
                templateUrl: 'deviceTemplate',
                uploadUrl: '/assets/category/import',
                fileName: `${this.$t('pc.global.templet_import', {
                    name: this.$t('pc.device_type.device_category')
                })}`
            })
        },
        // // 批量导入确认按钮
        // handleImport () {
        //   this.dialog1.loading = true
        //   this.$refs.panel.uploadFile()
        // },
        // noFile () {
        //   this.dialog1.loading = false
        //   this.$message.error('请选择需要上传的文件！')
        // },
        // onTemplateUploadOK ({ code, data }) {
        //   Object.assign(this.dialog1, { show: false, loading: false })
        //   let text = ''
        //   for (let i = 0; i < data.length; i++) {
        //     text += '\n' + data[i]
        //   }
        //   code === 200
        //     ? this.$message.success('模板导入成功')
        //     : this.$message.error(text)
        //   // : this.$message.warning(data[0])
        //   this.fetchData()
        // },
        // onTemplateUploadFail () {
        //   this.dialog1.loading = false
        // },
        onBtnClick(event) {
            const func = new Map([
                ['import', this.onImport],
                ['brotherNodeAdd', this.onBrotherNodeAdd],
                ['childrenNodeAdd', this.onChildrenNodeAdd],
                ['edit', this.onNodeEdit],
                ['delete', this.onNodeDelete]
            ])
            func.has(event) && func.get(event).call(this)
        }
        // resize () {
        //   this.treeHeight = 0
        //   this.$nextTick(() => {
        //     const { treeBox } = this.$refs
        //     this.treeHeight =
        //       treeBox.$el.clientHeight -
        //       40 -
        //       document.querySelector('.tree .el-card__header').clientHeight
        //   })
        // }
    },
    components: {
        detail,
        edit,
        dataImport
    },
    computed: {
        hasParentNode() {
            const { parentId = false } = this.detail
            return parentId
        },
        btns() {
            const { level } = this.org
            const btns = [
                // {
                //   content: this.$t('pc.global.batch_import'),
                //   event: 'import',
                //   disabled: !this.$checkPermission('sblx_dr')
                // },
                {
                    content: this.$t('pc.device_type.add_a_similar_level'),
                    event: 'brotherNodeAdd',
                    disabled: !this.$checkPermission('sblx_xz')
                },
                {
                    content: this.$t('pc.device_type.add_subordinates'),
                    event: 'childrenNodeAdd',
                    disabled: !this.$checkPermission('sblx_xz')
                },
                {
                    content: this.$t('pc.global.edit'),
                    event: 'edit',
                    disabled: !this.$checkPermission('sblx_bj')
                },
                {
                    content: this.$t('pc.global.delete'),
                    event: 'delete',
                    disabled: !this.$checkPermission('sblx_sc')
                }
            ]
            if (level === 4) {
                return btns.filter(item => item.event !== 'childrenNodeAdd')
            }
            return btns
        },
        toolbar() {
            const { length } = Object.keys(this.org)
            const { tag } = this.com
            const btns = [
                {
                    content: this.$t('pc.device_type.add_a_similar_level'),
                    icon: 'el-icon-plus',
                    click: this.onBrotherNodeAdd
                },
                {
                    content: this.$t('pc.device_type.add_subordinates'),
                    icon: 'el-icon-plus',
                    click: this.onChildrenNodeAdd,
                    disabled: this.org.level !== 4
                },
                {
                    content: this.$t('pc.global.edit'),
                    icon: 'el-icon-edit',
                    click: this.onNodeEdit
                },
                {
                    content: this.$t('pc.global.delete'),
                    icon: 'el-icon-close',
                    click: this.onNodeDelete
                },
                {
                    content: this.$t('pc.global.batch_import'),
                    icon: 'el-icon-document-copy',
                    click: this.onImport
                }
            ].map(btn => {
                Object.assign(btn, { disabled: tag === 'edit' })
                return btn
            })

            if (!length) {
                return btns.slice(btns.length - 1)
            }
            if (this.org.level === 4) {
                btns[1].disabled = true
            }
            return btns
        }
    },
    data() {
        return {
            com: { tag: '', tag1: '' },
            org: {},
            deviceName: '',
            treeHeight: 0,
            treeData: [],
            isShowTooltip: false,
            treeExpandData: [],
            dialog: { submit: () => { } },
            dialog1: { submit: () => { } }
            // btns: [
            //   {
            //     content: '批量导入',
            //     event: 'import'
            //   },
            //   {
            //     content: '新增同级',
            //     event: 'brotherNodeAdd'
            //   },
            //   {
            //     content: '新增下级',
            //     event: 'childrenNodeAdd'
            //   },
            //   {
            //     content: '编辑',
            //     event: 'edit'
            //   },
            //   {
            //     content: '删除',
            //     event: 'delete'
            //   }
            // ]
        }
    }
}
</script>

<style lang="scss" scoped>
.device {
    height: 100%;
    padding: 0;

    .tree {
        min-height: 100%;
        border: 1px solid #e4e4e4;

        /deep/ .el-card__body {
            height: calc(100% - 55px);
            padding: 0;
        }
    }

    .table-container {
        background-color: #fff;
        border: 1px solid #e4e4e4;
        margin-left: 16px;
    }
}

// .el-card {
//   height: 100%;
// }

// .table-container /deep/ {
//   padding: 0 0 0 20px;
//   .el-card__body {
//     height: calc(100% - 55px);
//   }
// }
</style>
