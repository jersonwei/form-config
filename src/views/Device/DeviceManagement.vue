<template>
    <el-container class="pageContent deviceManagement">
        <!-- <el-aside
      width="330px"
      ref="treeBox"
      style="height:100%; width: 330px!important"
    >
      <el-card shadow="always" class="tree">
        <el-tree
          :data="treeData"
          @node-click="onNodeClick"
          :expand-on-click-node="false"
          :highlight-current="true"
          ref="tree"
          node-key="deviceId"
          style="overflow-y:auto"
          :default-expanded-keys="treeExpandData"
          :current-node-key="currentNodeKey"
        >
          <div slot-scope="{ node }" class="tree-node">
            <el-tooltip
              :content="node.label"
              :disabled="isShowTooltip"
              :open-delay="300"
              placement="top"
              effect="dark"
            >
              <div class="tree-header row pace-between">
                <span class="over-ellipsis" @mouseover="mouseOver($event)">
                  {{ node.label }}
                </span>
              </div>
            </el-tooltip>
          </div>
        </el-tree>
      </el-card>
    </el-aside> -->
        <!-- <span
                  v-if="node.level === 1"
                  class="system-picture"
                  @click.stop="checkSystemPicture(node)"
                  >系统拓补图</span
                > -->
        <el-main class="table-container">
            <!-- <btn-bar :btns="btns" @btn-click="onBtnClick"></btn-bar>
      <el-divider></el-divider> -->
            <wz-search-bar ref="searchBar" :items="items" has-reset @search="onSearch" @reset="onReset"
                @change="onFormChange" :key="key" :options="options"></wz-search-bar>
            <wz-table hasIndex :data="tableData" :heads="realTableHead" ref="table" :loading="loading"
                @btn-click="onTableBtnClick">
                <template v-slot:slot-1="scope">
                    <div class="slot-1 row">
                        <!-- <div class="row"> -->
                        <!-- <img :src="scope.row.facImg" alt="" /> -->
                        <img src="../../assets/imgs/defaultImg.png" alt="" class="defaultImg"
                            v-if="!scope.row.showFacImg" />
                        <el-image v-else :src="scope.row.showFacImg" fit="fill" lazy></el-image>
                        <div class="flex-column col-top">
                            <span class="name">{{ scope.row.facName }}</span>
                            <span>{{ scope.row.facCode }}</span>
                            <span>{{ scope.row.facCateParentName }}</span>
                        </div>
                        <!-- </div> -->
                    </div>
                </template>
                <template v-slot:slot-2="scope">
                    <div class="slot-2 row center">
                        <div v-if="Number(scope.row.facStatus) === 1">
                            <span class="normal">{{ $enum('device_status', 1) }}</span>
                        </div>
                        <div class="flex-column center" v-else-if="Number(scope.row.facStatus) === 3">
                            <span class="abnormal">{{ $enum('device_status', 3) }}</span>
                            <span v-if="scope.row.workOrderNo">{{ $t('pc.device_ledger.transfer_work_order') }}：{{
                                scope.row.workOrderNo
                            }}</span>
                            <span v-if="scope.row.alarmId">{{
                                $t('pc.device_ledger.equipment_alarm')
                            }}</span>
                            <!-- <span v-if="scope.row.orderStatus"
                >【{{ scope.row.orderStatus }}】</span
              > -->
                        </div>
                        <div v-else-if="Number(scope.row.facStatus) === 2">
                            <span class="deviceAlarm">{{ $enum('device_status', 2) }}</span>
                        </div>
                    </div>
                </template>
                <template v-slot:slot-3="scope">
                    <span>{{ scope.row.openType }}</span>
                </template>
            </wz-table>
            <div class="pagination row row-between">
                <span class="total">{{
                    $t('pc.global.total_items', { total: searchForm.head.total
                })
                }}</span>
                <el-pagination ref="pager" layout="sizes,prev,pager,next,jumper"
                    :current-page.sync="searchForm.head.current" :page-size.sync="searchForm.head.size"
                    :total="searchForm.head.total" @current-change="getGetFacilitiesPageList"
                    @size-change="handleSizeChange" background></el-pagination>
            </div>
        </el-main>
        <el-dialog :visible.sync="dialog1.show" :title="dialog1.title" :width="dialog1.width"
            :top="dialog1.top || '15vh'" destroy-on-close>
            <component :is="dialog1.tag" v-bind="dialog1" ref="panel" @on-success="onTemplateUploadOK"
                @on-error="onTemplateUploadFail" @noFile="noFile" title=""></component>
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
import dayjs from 'dayjs'
import { tablePage } from '../../mixins/mixins'
import { uploadFile } from '../../mixins/baseMixins'
import { wzTable } from 'wz-ui'
import objArr from 'wz-file/lib/api.js'
import {
    findCategoryTree,
    getFacilitiesPageList,
    // getBuildingFloorTree,
    facilitiesDel,
    getProjectAccountList,
    getStationAccount,
    getStationAccountListAll,
    getDistrictTree
} from '../../api'
import dataImport from '../../components/dataImport.vue'
export default {
    name: 'DeviceManagement',
    components: {
        dataImport,
        wzTable
    },
    methods: {
        onFormChange(form) {
            if (form.projectId) {
                this.$set(
                    this.$refs.searchBar._props.items.find(i => i.prop === 'stationId'),
                    'disabled',
                    false
                )
                const id = this.items
                    .find(item => item.prop === 'projectId')
                    .options.find(i => i.value === form.projectId).id
                getStationAccountListAll({ projectId: id }).then(res => {
                    this.$set(
                        this.items.find(item => item.prop === 'stationId'),
                        'options',
                        res.map(({ id, stationId, stationName }) => {
                            return {
                                label: stationName,
                                value: id,
                                id
                            }
                        })
                    )
                })
            }
            if (!form.projectId) {
                this.$refs.searchBar._props.items.find(
                    i => i.prop === 'stationId'
                ).disabled = true
                if (form.stationId) delete form.stationId
            }
            console.log(form, this.$refs.searchBar._props)
        },
        parseTreeData(treeData) {
            return treeData.map(node => {
                const { level, ...rest } = node
                const obj = {
                    level,
                    ...rest,
                    disabled: level === 4
                }
                if (Array.isArray(node.children)) {
                    obj.children = this.parseTreeData(node.children)
                } else if (Array.isArray(node.children) && !node.children.length) {
                    node.children = undefined
                }
                return obj
            })
        },
        checkSystemPicture(node) {
            this.$router.push({ name: '系统拓扑图' })
        },
        onSearch(form) {
            this.searchForm.head.current = 1
            let { code } = form
            if (!code?.length) {
                delete form.city
                delete form.area
                delete form.code
                delete form.province
            } else if (code?.length === 1) {
                delete form.city
                delete form.area
                delete form.code
                form.province = code[0]
            } else if (code?.length === 2) {
                delete form.area
                delete form.code
                form.province = code[0]
                form.city = code[1]
            } else if (code?.length === 3) {
                delete form.code
                form.province = code[0]
                form.city = code[1]
                form.area = code[2]
            }
            this.searchForm.body = { ...form }
            console.log(this.searchForm, form, '搜索入参')
            getFacilitiesPageList(this.searchForm)
                .then(({ current, total, records, size }) => {
                    Object.assign(this.searchForm.head, { current, size, total })
                    this.tableData = records.map(({ province, city, ...rest }) => {
                        return {
                            innerArea: city ? province + '-' + city : province || '--',
                            ...rest
                        }
                    })
                })
                .catch(() => {
                    this.tableData = getFalseData()
                })
        },
        mouseOver(event) {
            this.isShowTooltip =
                event.currentTarget.scrollWidth <= event.currentTarget.clientWidth
        },
        onBtnClick(event) {
            const func = new Map([
                ['import', this.onImport],
                ['add', this.onDeviceAdd],
                ['turnOnLight', this.controlLight]
            ])
            func.has(event) && func.get(event).call(this)
        },
        onDeviceAdd() {
            this.$router.push({ name: '新增设备' })
        },
        reset() {
            this.searchForm = { body: {}, head: { current: 1, size: 10 } }
            this.fetchData()
        },
        handleSizeChange(size) {
            this.searchForm.head.size = size
            this.getGetFacilitiesPageList()
        },
        onTableBtnClick({ event, row }) {
            const func = new Map([
                ['edit', this.onEdit],
                ['check', this.onCheck],
                ['delete', this.onDelete]
            ])
            func.has(event) && func.get(event).call(this, row)
        },
        onEdit({ id }) {
            this.$router.push({
                name: '编辑设备',
                params: {
                    id,
                    parentIds: this.org.parentIds,
                    deviceId: this.org.deviceId
                }
            })
        },
        onCheck(row) {
            this.$router.push({
                name: '设备详情',
                params: { id: row.id }
            })
            sessionStorage.setItem('deviceRow', JSON.stringify(row))
        },
        onDelete({ id, facName }) {
            this.$confirm(
                this.$t('pc.global.delete_tip', { name: facName }),
                this.$t('pc.global.hint'),
                {
                    type: 'warning'
                }
            ).then(() => {
                facilitiesDel(id).then(() => {
                    this.$message.success(
                        this.$t('pc.global.delete_success', { name: facName })
                    )
                    this.searchForm.head.current = 1
                    this.searchForm.head.size = 10
                    this.getGetFacilitiesPageList()
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
                templateUrl: 'facDownTemp',
                uploadUrl: '/assets/facilities/import',
                fileName: this.$t('pc.global.templet_import', {
                    name: this.$t('pc.device_ledger.device_ledger')
                })
            })
        },
        getGetFacilitiesPageList() {
            const { building = [], ...rest } = this.searchForm.body
            const params = { ...rest }
            params.buildingId = building[0]
            params.buildingFloorId = building[1]
            params.buildingFloorSpaceId = building[2]
            if (this.org.level !== 4) {
                // params.parentIds = this.org.parentIds + this.org.deviceId
            } else {
                params.facCategoryId = this.org.deviceId
            }
            Object.assign(this.searchForm.body, params)
            getFacilitiesPageList(this.searchForm).then(
                ({ current, total, records, size }) => {
                    Object.assign(this.searchForm.head, { current, size, total })
                    this.tableData = records.map(({ province, city, ...rest }) => {
                        return {
                            innerArea: city ? province + '-' + city : province || '--',
                            ...rest
                        }
                    })
                    this.tableData.map((item, index) => {
                        const { facImg } = item
                        if (facImg) {
                            const { getImgUrl } = objArr
                            getImgUrl(facImg).then(responseData => {
                                const myBlob = new window.Blob([responseData], {
                                    type: 'image/jpeg'
                                })
                                const Url = window.URL.createObjectURL(myBlob)
                                this.$set(this.tableData[index], 'showFacImg', Url)
                            })
                        }
                    })
                }
            )
        },
        fetchData() {
            // 获取设备分类树
            findCategoryTree().then(data => {
                this.$set(this.options, 'facCateCode', data)
            })
            // 请求列表数据
            this.getGetFacilitiesPageList()
        }
    },
    mixins: [tablePage.call(this), uploadFile.call(this)],
    created() {
        // getBuildingFloorTree().then(data => {
        //   this.$set(this.options, 'buildingFloorTree', data)
        // })
        getDistrictTree()
            .then(res => {
                this.items[2].options = this.parseTreeData(res)
            })
            .catch(err => console.dir(err))
        // 项目
        getProjectAccountList().then(data => {
            this.$set(this.options, 'project', data)
        })
        // 站场
        getStationAccountListAll().then(data => {
            this.$set(this.options, 'station', data)
        })
    },
    computed: {
        currentNodeKey() {
            if (this.$route.params.deviceId) {
                return this.$route.params.deviceId
            }
            return this.treeData[0].children[0]?.deviceId
        },
        btns() {
            return [
                {
                    content: this.$t('pc.device_ledger.add_device'),
                    event: 'add',
                    disabled: !this.$checkPermission('sbtz_xz')
                },
                {
                    content: this.$t('pc.device_ledger.import_device'),
                    event: 'import',
                    disabled: !this.$checkPermission('sbtz_dr')
                }
            ]
        },
        // 非照明系统的表头数据
        realTableHead() {
            const arr = [
                {
                    label: '设备名称',
                    prop: 'facName',
                    width: 150,
                    formatter: ({ facName }) => {
                        return facName ? facName : '--'
                    }
                },
                {
                    label: '物联ID',
                    prop: 'facLinkCode',
                    width: 150,
                    formatter: ({ facLinkCode }) => {
                        return facLinkCode ? facLinkCode : '--'
                    }
                },
                {
                    label: '所属项目简称',
                    prop: 'projectName',
                    width: 150,
                    formatter: ({ projectName }) => {
                        return projectName ? projectName : '--'
                    }
                },
                {
                    label: '项目场站',
                    prop: 'stationName',
                    width: 150,
                    formatter: ({ stationName }) => {
                        return stationName ? stationName : '--'
                    }
                },
                {
                    label: '所属地区',
                    prop: 'innerArea',
                    width: 150
                },
                {
                    label: '设备类型',
                    prop: 'facType',
                    width: 150,
                    formatter: ({ facType }) => {
                        return facType ? facType : '--'
                    }
                },
                {
                    label: '品牌型号',
                    prop: 'brandName',
                    width: 150,
                    formatter: ({ brandName }) => {
                        return brandName ? brandName : '--'
                    }
                },
                {
                    label: '物联状态',
                    prop: 'facStatus',
                    width: 150,
                    formatter: ({ facStatus }) => {
                        if (!facStatus) {
                            return '--'
                        } else {
                            return [
                                {
                                    label: '停用',
                                    value: '0'
                                },
                                {
                                    label: '未激活',
                                    value: '1'
                                },
                                {
                                    label: '离线',
                                    value: '2'
                                },
                                {
                                    label: '在线 ',
                                    value: '3'
                                }
                            ].find(i => i.value == facStatus).label
                        }
                    }
                },
                // {
                //   label: '联网状态',
                //   slot: true,
                //   slotTag: 'slot-2'
                // },
                {
                    label: this.$t('pc.global.operate'),
                    width: 150,
                    btns: () => {
                        let btns = []
                        const arr = [
                            {
                                label: this.$t('pc.global.check'),
                                event: 'check',
                                code: 'sbtz_ck'
                            }
                            // {
                            //   label: this.$t('pc.global.edit'),
                            //   event: 'edit',
                            //   code: 'sbtz_bj'
                            // },
                            // {
                            //   label: this.$t('pc.global.delete'),
                            //   event: 'delete',
                            //   code: 'sbtz_sc'
                            // }
                        ]
                        btns = this.$tableLinkCheck(arr)
                        return btns
                    }
                }
            ]
            return arr
        }
    },
    mounted() {
        getProjectAccountList()
            .then(res => {
                this.items.find(item => item.prop === 'projectId').options = res.map(
                    ({ id, projectId, projectName }) => {
                        return {
                            label: projectName,
                            value: id,
                            id
                        }
                    }
                )
            })
            .catch(err => console.dir(err))
    },
    data() {
        return {
            options: {},
            searchForm: {},
            org: {},
            treeData: [{ label: '全部', children: [] }],
            treeExpandData: [],
            isShowTooltip: false,
            tableData: [],
            items: [
                {
                    // label: this.$t('pc.device_ledger.running_status'),
                    label: '所属项目',
                    labelWidth: '100px',
                    tag: 'el-select',
                    placeholder: '请选择项目',
                    clearable: true,
                    prop: 'projectId',
                    options: []
                },
                {
                    // label: this.$t('pc.device_ledger.running_status'),
                    label: '项目场站',
                    labelWidth: '100px',
                    tag: 'el-select',
                    placeholder: '请选择项目场站',
                    clearable: true,
                    disabled: true,
                    prop: 'stationId',
                    // options: this.$enums('device_status')
                    optionsCode: 'station'
                },
                {
                    label: '所在地区',
                    labelWidth: '80px',
                    tag: 'el-cascader',
                    placeholder: '请选择地区',
                    clearable: true,
                    props: {
                        label: 'name',
                        value: 'code',
                        children: 'children',
                        checkStrictly: true
                    },
                    prop: 'code',
                    options: []
                },
                {
                    label: '联网状态',
                    labelWidth: '130px',
                    tag: 'el-select',
                    placeholder: this.$t('pc.global.please_select', {
                        name: this.$t('pc.device_ledger.running_status')
                            .toLocaleLowerCase()
                            .toLocaleLowerCase()
                    }),
                    clearable: true,
                    prop: 'facStatus',
                    options: this.$enums('device_status')
                },
                {
                    label: '运行状态',
                    labelWidth: '100px',
                    tag: 'el-select',
                    placeholder: this.$t('pc.global.please_select', {
                        name: this.$t('pc.device_ledger.asset_status').toLocaleLowerCase()
                    }),
                    clearable: true,
                    prop: 'facStatus1',
                    options: this.$enums('asset_status').filter(
                        ({ value }) => value !== 2
                    )
                },
                {
                    label: '设备类型',
                    labelWidth: '100px',
                    tag: 'el-cascader',
                    placeholder: '请选择设备类型',
                    clearable: true,
                    props: {
                        checkStrictly: true,
                        value: 'facCateCode',
                        label: 'facCateName'
                    },
                    prop: 'facCateCode',
                    optionsCode: 'facCateCode'
                },
                {
                    // label: this.$t('pc.device_ledger.device_name'),
                    label: '搜索关键字',
                    labelWidth: '100px',
                    tag: 'el-input',
                    // placeholder: this.$t('pc.global.please_input', {
                    //   name: this.$t('pc.device_ledger.device_name').toLocaleLowerCase()
                    // }),
                    placeholder: '请输入项目名称或地址',
                    clearable: true,
                    prop: 'facName',
                    maxlength: 50
                }
            ],
            key: 0,
            options: {
                facCateCode: []
            }
        }
    }
}
</script>

<style lang="scss" scoped>
.deviceManagement {
    // background-color: #fff;
    height: 100%;
    padding: 0 !important;

    .tree {
        min-height: 100%;
        border: 1px solid #e4e4e4;

        /deep/ .el-card__body {
            height: calc(100% - 55px);
            padding: 0;
        }
    }

    .tree-header {
        width: 100%;
        padding-right: 10px;

        .system-picture {
            color: #007cc8;
        }
    }

    .table-container {
        background-color: #fff;
        border: 1px solid #e4e4e4;

        // margin-left: 16px;
        // padding: 16px 24px 24px;
        .search-bar {
            margin-top: 16px;
        }

        .slot-1 {
            width: 100%;

            .el-image,
            img {
                width: 56px;
                height: 56px;
                margin-right: 8px;
            }

            .name {
                font-size: 14px;
                color: #000;
                line-height: 22px;
            }

            span {
                font-size: 12px;
                color: #939393;
                line-height: 16px;
            }
        }

        .slot-2 {
            .normal {
                font-size: 14px;
                line-height: 26px;
                color: #008843;
            }

            .abnormal {
                font-size: 14px;
                line-height: 22px;
                color: #f4a34d;
            }

            .deviceAlarm {
                font-size: 14px;
                line-height: 22px;
                color: #d5341e;
            }

            span {
                color: #939393;
                font-size: 12px;
                line-height: 16px;
            }
        }
    }
}

/deep/ .el-aside {
    background-color: #ffffff;
}
</style>
