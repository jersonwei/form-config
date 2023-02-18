<template>
    <el-container class="space">
        <el-aside width="330px" ref="treeBox" style="height:100%">
            <el-card shadow="always" class="tree">
                <template slot="header">
                    <el-input :placeholder="$t('pc.space_type.please_enter_space_type')" size="small" clearable
                        v-model="deviceName" @clear="onFilter">
                        <el-button slot="append" icon="el-icon-search" @click="onFilter"></el-button>
                    </el-input>
                </template>
                <el-tree :data="treeData" @node-click="onNodeClick" :expand-on-click-node="false" ref="tree"
                    node-key="id" :filter-node-method="filterNode" style="overflow-y:auto"
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
            <component v-if="dialog.show" :is="dialog.tag" v-bind="dialog" ref="panel"></component>
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
                            $t('pc.global.submit')
                    }}</el-button>
                </span>
            </template>
        </el-dialog>
        <el-dialog :visible.sync="dialog1.show" :title="dialog1.title" :width="dialog1.width"
            :top="dialog1.top || '15vh'" destroy-on-close>
            <component v-if="dialog1.show" :is="dialog1.tag" v-bind="dialog1" ref="panel"
                @on-success="onTemplateUploadOK" @on-error="onTemplateUploadFail" @noFile="noFile"></component>
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
    getSpaceByCode,
    createSpace,
    delSpace,
    updateSpace,
    findSpaceTreeList
} from '../../../api'
import { tablePage } from '../../../mixins/mixins'
import { uploadFile } from '../../../mixins/baseMixins'
import detail from './Detail.vue'
import edit from './Edit.vue'
import dataImport from '../../../components/dataImport.vue'
export default {
    name: 'Space',
    // mixins: [treeHeight.call(this)],
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
            return data.label.includes(value)
        },
        // 点击树，获取基础信息
        onNodeClick(org) {
            this.org = org
            getSpaceByCode({ id: org.deviceId }).then(data => {
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
                findSpaceTreeList({ orgCode, orgId: value }).then(data => {
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
            findSpaceTreeList({ orgCode, orgId: value }).then(data => {
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
                ({ spaceName, spaceCode, level, children, parentId, id }) => {
                    const obj = {
                        [key]: spaceCode,
                        label: spaceName,
                        parentId,
                        deviceId: id,
                        level
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
            findSpaceTreeList().then(data => {
                if (!data) {
                    return
                }
                this.treeData = this.parseTreeNode(data)
                this.treeExpandData = [this.treeData[0].id]
                const id = this.treeData[0].deviceId
                this.org = this.treeData[0]
                if (id) {
                    getSpaceByCode({ id: id }).then(data => {
                        this.$set(this, 'com', {
                            tag: 'detail',
                            detail: data
                        })
                    })
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
                width: '566px',
                title: this.$t('pc.space_type.add_same_level_space_type'),
                id,
                deviceId,
                parentId: parentId || 0,
                type: 1,
                level,
                submitText: this.$t('pc.global.save'),
                submit: this.handleDeviceAdd
            })
        },
        // 新增同级保存
        handleDeviceAdd() {
            this.$refs.panel.validate().then(form => {
                this.dialog.loading = true
                createSpace(form)
                    .then(() => {
                        this.dialog.show = false
                        this.$message.success(
                            this.$t('pc.global.add_success', {
                                name: this.$t('pc.space_type.space_type')
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
                width: '566px',
                title: this.$t('pc.space_type.add_subordinate_space_type'),
                id,
                parentId: parentId || 0,
                deviceId,
                type: 3,
                level,
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
                width: '566px',
                title: this.$t('pc.space_type.edit_space_type'),
                id,
                parentId: parentId || 0,
                deviceId,
                type: 2,
                level,
                submitText: this.$t('pc.global.save'),
                submit: this.handleDeviceEdit
            })
        },
        // 编辑保存按钮
        handleDeviceEdit() {
            this.$refs.panel.validate().then(form => {
                this.dialog.loading = true
                form.id = this.org.deviceId
                updateSpace(form)
                    .then(() => {
                        this.dialog.show = false
                        this.$message.success(
                            this.$t('pc.global.edit_success', {
                                name: this.$t('pc.space_type.space_type')
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
            const { deviceId, label } = this.org
            const h = this.$createElement
            this.$msgbox({
                title: this.$t('pc.global.hint'),
                message: h('p', { class: 'flex-column' }, [
                    h('span', { style: 'color:#F56C6C' }, this.$t('pc.space_type.note3')),
                    h('span', null, this.$t('pc.global.delete_tip', { name: label }))
                ]),
                showCancelButton: true
            }).then(() => {
                delSpace({ id: deviceId }).then(() => {
                    this.$message.success(
                        this.$t('pc.global.delete_success', {
                            name: this.$t('pc.space_type.classification')
                        })
                    )
                    this.fetchData()
                    this.com.tag = ''
                })
            })
        },
        // 批量导入
        onImport() {
            this.$set(this, 'dialog1', {
                show: true,
                tag: 'data-import',
                title: this.$t('pc.global.batch_import'),
                width: '532px',
                loading: false,
                submit: this.handleImport,
                status: 1,
                submitText: this.$t('pc.global.import'),
                templateUrl: 'spaceTemplate',
                uploadUrl: '/assets/spcspace/import',
                fileName: this.$t('pc.global.templet_import', {
                    name: this.$t('pc.space_type.space_type')
                })
            })
        },
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
    created() {
        this.fetchData()
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
            return [
                {
                    content: this.$t('pc.global.batch_import'),
                    event: 'import',
                    disabled: !this.$checkPermission('kjlx_dr')
                },
                {
                    content: this.$t('pc.space_type.add_a_similar_level'),
                    event: 'brotherNodeAdd',
                    disabled: !this.$checkPermission('kjlx_xz')
                },
                {
                    content: this.$t('pc.space_type.add_subordinates'),
                    event: 'childrenNodeAdd',
                    disabled: !this.$checkPermission('kjlx_xz')
                },
                {
                    content: this.$t('pc.global.edit'),
                    event: 'edit',
                    disabled: !this.$checkPermission('kjlx_bj')
                },
                {
                    content: this.$t('pc.global.delete'),
                    event: 'delete',
                    disabled: !this.$checkPermission('kjlx_sc')
                }
            ]
        }
    },
    data() {
        return {
            com: { tag: '', tag1: '' },
            org: {},
            deviceName: '',
            treeData: [],
            treeExpandData: [],
            dialog: { submit: () => { } },
            dialog1: { submit: () => { } },
            isShowTooltip: false
        }
    }
}
</script>

<style lang="scss" scoped>
.space {
    height: 100%;
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
</style>
