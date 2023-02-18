<template>
    <div class="task">
        <wz-search-bar ref="searchBar" :items="items" has-reset :deviceTaskType="searchForm.body.deviceTaskType"
            @search="onSearch" @reset="onReset" @change="onSearchChange"></wz-search-bar>
        <wz-table :data="tableData" :heads="head" ref="table" :loading="loading" @btn-click="onTableBtnClick">
        </wz-table>
        <div class="pagination row row-between">
            <span class="total">{{
                $t('pc.global.total_items', { total: searchForm.head.total
            })
            }}</span>
            <el-pagination ref="pager" layout="sizes,prev,pager,next,jumper"
                :current-page.sync="searchForm.head.current" :page-size.sync="searchForm.head.size"
                :total="searchForm.head.total" @current-change="fetchData(form.id)" @size-change="handleSizeChange"
                background></el-pagination>
        </div>
    </div>
</template>

<script>
import dayjs from 'dayjs'
import { tablePage } from '../../../mixins/mixins'
import {
    taskFacpage,
    taskFindWorkOrderPageListByFacId,
    taskFindScheduleFacLogPageListByFacId,
    taskFindMaintenanceTaskPageListByFacId
} from '../../../api'
import { wzTable } from 'wz-ui'
export default {
    mixins: [tablePage.call(this)],
    props: {
        form: Object
    },
    components: { wzTable },
    watch: {
        form: {
            handler(newVal) {
                const { id } = newVal
                this.fetchData(id)
            },
            deep: true
        },
        handleSizeChange(value) {
            Object.assign(this.searchForm.head, {
                current: 0,
                total: 0,
                size: value
            })
            const { id } = this.form
            this.fetchData(id)
        },
        'searchForm.body.deviceTaskType': {
            handler(newVal) {
                const arr = [
                    [
                        {
                            label: '工单名称',
                            prop: 'orderName',
                            btns: ({ orderName }) => {
                                let btns = []
                                const arr = [
                                    {
                                        label: orderName,
                                        event: 'check',
                                        code: 'gdjl_ck'
                                    }
                                ]
                                btns = this.$tableLinkCheck(arr)
                                return btns
                            }
                        },
                        {
                            label: '工单状态',
                            prop: 'stateName'
                        },
                        {
                            label: '报单时间',
                            prop: 'createdDate',
                            formatter: ({ createdDate }) =>
                                dayjs(createdDate).format('YYYY-MM-DD HH:mm:ss')
                        },
                        {
                            label: '报单人',
                            prop: 'reporterUserName'
                        },
                        {
                            label: '接单人',
                            prop: 'pickUpPersonUserName'
                        }
                    ].map((item, index) => {
                        const key = [
                            'work_order_name',
                            'work_order_status',
                            'order_time',
                            'declarer',
                            'order_taker'
                        ]
                        if (index < key.length) {
                            item.label = this.$t(`pc.work_order_records.${key[index]}`)
                        }
                        return item
                    }),
                    [
                        {
                            label: this.$t('pc.menu.byjh'),
                            prop: 'maintenanceName'
                        },
                        {
                            label: this.$t('pc.maintenance_plan.implementation_year'),
                            prop: 'carryOutYear'
                        },
                        {
                            label: this.$t('pc.maintenance_plan.maintenance_cycle'),
                            formatter: ({ scheduleCycleName }) => {
                                return this.$t(`pc.dict_list.${scheduleCycleName}`)
                            }
                        },
                        {
                            label: this.$t('pc.device_ledger.task_execution_time'),
                            prop: 'termOfValidityStart',
                            formatter: ({ termOfValidityStart }) => {
                                return dayjs(termOfValidityStart).format('YYYY-MM-DD HH:mm:ss')
                            }
                        },
                        {
                            label: this.$t('pc.maintenance_tasks.task_status'),
                            formatter: ({ maintenanceTaskStatus }) => {
                                return this.$enum('maintenance_task_status', maintenanceTaskStatus)
                            }
                        },
                        {
                            label: this.$t('pc.maintenance_plan.responsible_department'),
                            prop: 'personInChangeOrgName'
                        }
                    ],
                    [
                        {
                            label: this.$t('pc.inspection_tasks.inspection_task_name'),
                            prop: 'inspectTaskName',
                            btns: ({ inspectTaskName }) => {
                                let btns = []
                                const arr = [
                                    {
                                        label: inspectTaskName,
                                        event: 'check',
                                        code: 'xjrw_ck'
                                    }
                                ]
                                btns = this.$tableLinkCheck(arr)
                                return btns
                            }
                        },
                        {
                            label: this.$t('pc.inspection_tasks.affiliated_inspection_plan'),
                            prop: 'inspectPlanName'
                        },
                        {
                            label: this.$t('pc.inspection_tasks.plan_start_and_end_time'),
                            formatter: ({ inspectPlanStart, inspectPlanEnd }) => {
                                return (
                                    dayjs(inspectPlanStart).format('YYYY-MM-DD HH:mm:ss') +
                                    '-' +
                                    dayjs(inspectPlanEnd).format('YYYY-MM-DD HH:mm:ss')
                                )
                            }
                        },
                        {
                            label: this.$t('pc.inspection_tasks.principal'),
                            prop: 'execUserName'
                        }
                    ],
                    [
                        {
                            label: this.$t('pc.inspection_tasks.inspection_task_name'),
                            prop: 'scheduleFacLogName'
                        },
                        {
                            label: this.$t('pc.inspection_tasks.affiliated_inspection_plan'),
                            prop: 'scheduleName'
                        },
                        {
                            label: this.$t('pc.inspection_tasks.actual_start_and_end_time'),
                            // prop: 'termOfValidityStart',
                            formatter: ({ termOfValidityStart }) => {
                                return dayjs(termOfValidityStart).format('YYYY-MM-DD HH:mm:ss')
                            }
                        },
                        {
                            label: this.$t('pc.inspection_tasks.principal'),
                            prop: 'personInChargeUserName'
                        }
                    ]
                ]
                this.head = newVal ? arr[Number(newVal) - 1] : arr[1]
                this.searchForm.head = { current: 1, size: 10, total: 0 }
            },
            deep: true
        }
    },
    methods: {
        onSearchChange(form) {
            this.tableData = [];
            const { deviceTaskType } = form
            this.$set(this.searchForm.body, 'deviceTaskType', deviceTaskType)
        },
        onTableBtnClick({ event, row }) {
            const func = new Map([['check', this.onCheck]])
            func.has(event) && func.get(event).call(this, row)
        },
        onCheck({ id, state }) {
            const { deviceTaskType } = this.searchForm.body
            if (deviceTaskType === '1') {
                this.$router.push({ name: '工单详情', params: { id } })
            } else if (deviceTaskType === '2') {
            } else if (deviceTaskType === '3') {
                this.$router.push({ name: '巡检任务详情', params: { id } })
            } else if (deviceTaskType === '4') {
            }
        },
        onSearch(form) {
            this.searchForm.head.current = 1
            Object.assign(this.searchForm.body, { ...form })
            const { id } = this.form
            this.fetchData(id)
        },
        onReset() {
            this.searchForm = {
                body: { deviceTaskType: '1' },
                head: { current: 1, size: 10, total: 0 }
            }
            const { id } = this.form
            this.fetchData(id)
        },
        fetchData(id) {
            if (!id) {
                return
            }
            this.loading = true
            const { time = [], deviceTaskType, ...rest } = this.searchForm.body
            const params = { ...rest }
            if (time && time.length !== 0) {
                params.startDate = dayjs(time[0] + ' 00:00:00').valueOf()
                params.endDate = dayjs(time[1] + ' 23:59:59').valueOf()
            }
            params.facId = id
            console.log(params)
            Object.assign(this.searchForm.body, params)
            const apis = [
                taskFindWorkOrderPageListByFacId,
                taskFindMaintenanceTaskPageListByFacId,
                taskFacpage,
                taskFindScheduleFacLogPageListByFacId
            ]
            const api = apis[Number(deviceTaskType) - 1]
            api
                .call(this, this.searchForm)
                .then(data => {
                    if (!data) {
                        Object.assign(this.searchForm.head, {
                            current: 0,
                            total: 0,
                            size: 10
                        })
                        this.tableData = []
                    } else {
                        const { current, records, total, size } = data
                        Object.assign(this.searchForm.head, { current, total, size })
                        this.tableData = records
                    }
                })
                .finally(() => (this.loading = false))
        }
    },
    data() {
        return {
            searchForm: {
                body: { deviceTaskType: 1 },
                head: { current: 1, size: 10, total: 0 }
            },
            tableData: [],
            head: [
                {
                    label: '工单名称',
                    prop: 'orderName',
                    btns: ({ orderName }) => {
                        let btns = []
                        const arr = [
                            { label: orderName, event: 'check', code: 'gdjl_ck' }
                        ]
                        btns = this.$tableLinkCheck(arr)
                        return btns
                    }
                },
                {
                    label: '工单状态',
                    formatter: ({ stateName }) => this.$t(`pc.dict_list.${stateName}`)
                },
                {
                    label: '报单时间',
                    prop: 'createdDate',
                    formatter: ({ createdDate }) =>
                        dayjs(createdDate).format('YYYY-MM-DD HH:mm:ss')
                },
                {
                    label: '报单人',
                    prop: 'reporterUserName'
                },
                {
                    label: '接单人',
                    prop: 'pickUpPersonUserName'
                }
            ].map((item, index) => {
                const key = [
                    'work_order_name',
                    'work_order_status',
                    'order_time',
                    'declarer',
                    'order_taker'
                ]
                if (index < key.length) {
                    item.label = this.$t(`pc.work_order_records.${key[index]}`)
                }
                return item
            }),
            items: [
                {
                    label: this.$t('pc.device_ledger.task_type'),
                    labelWidth: '80px',
                    tag: 'el-select',
                    placeholder: '',
                    clearable: true,
                    prop: 'deviceTaskType',
                    options: this.$enums('task_type')
                },
                {
                    label: this.$t('pc.device_ledger.task_time'),
                    labelWidth: '80px',
                    prop: 'time',
                    tag: 'el-date-picker',
                    type: 'datetimerange',
                    startPlaceholder: this.$t('pc.global.start_date'),
                    endPlaceholder: this.$t('pc.global.end_date'),
                    valueFormat: 'yyyy-MM-dd'
                }
            ]
        }
    }
}
</script>

<style lang="scss" scoped>

</style>
