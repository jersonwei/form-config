<template>
    <div class="log">
        <wz-search-bar ref="searchBar" :items="items" has-reset @search="onSearch" @reset="onReset"></wz-search-bar>
        <wz-table :data="tableData" :heads="tableHead" ref="table" :loading="loading">
        </wz-table>
        <div class="pagination row row-between">
            <span class="total">{{
                $t('pc.global.total_items', { total: searchForm.head.total
            })
            }}</span>
            <el-pagination ref="pager" layout="sizes,prev,pager,next,jumper"
                :current-page.sync="searchForm.head.current" :page-size.sync="searchForm.head.size"
                :total="searchForm.head.total" @current-change="fetchData" @size-change="handleSizeChange"
                background></el-pagination>
        </div>
    </div>
</template>

<script>
import dayjs from 'dayjs'
import { tablePage } from '../../../mixins/mixins'
import { getFacOperateLogPageList } from '../../../api'
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
        }
    },
    methods: {
        onSearch(form) {
            this.searchForm.head.current = 1
            Object.assign(this.searchForm.body, { ...form })
            const { id } = this.form
            this.fetchData(id)
        },
        onReset() {
            this.searchForm = { body: {}, head: { current: 1, size: 10, total: 0 } }
            const { id } = this.form
            this.fetchData(id)
        },
        fetchData(id) {
            if (!id) {
                return
            }
            this.loading = true
            const { time = [], ...rest } = this.searchForm.body
            const params = { ...rest }
            if (time && time.length !== 0) {
                params.startDate = dayjs(time[0] + ' 00:00:00').valueOf()
                params.endDate = dayjs(time[1] + ' 23:59:59').valueOf()
            }
            params.facId = id
            Object.assign(this.searchForm.body, params)
            getFacOperateLogPageList(this.searchForm)
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
            tableHead: [
                {
                    label: this.$t('pc.log_management.operation_type'),
                    formatter: ({ operateType }) =>
                        this.$enum('operation_type', Number(operateType))
                },
                // {
                //   label: '操作对象',
                //   prop: '操作对象'
                // },
                {
                    label: this.$t('pc.log_management.operation_content'),
                    prop: 'content'
                },
                {
                    label: this.$t('pc.log_management.operator'),
                    prop: 'createdByName'
                },
                {
                    label: this.$t('pc.log_management.operation'),
                    prop: 'createdDate',
                    formatter: ({ createdDate }) => {
                        if (!createdDate) {
                            return '-'
                        } else {
                            return dayjs(createdDate).format('YYYY-MM-DD HH:mm:ss')
                        }
                    }
                }
            ],
            tableData: [],
            items: [
                {
                    label: this.$t('pc.log_management.operation_type'),
                    labelWidth: '110px',
                    tag: 'el-select',
                    placeholder: '',
                    clearable: true,
                    prop: 'operateType',
                    options: this.$enums('operation_type').filter(({ value }) =>
                        new Set([null, 1, 2]).has(value)
                    )
                },
                // {
                //   label: '操作对象',
                //   labelWidth: '80px',
                //   tag: 'el-input',
                //   placeholder: '请输入搜索关键字',
                //   clearable: true,
                //   prop: 'alarmName'
                // },
                {
                    label: this.$t('pc.log_management.operation'),
                    labelWidth: '90px',
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
