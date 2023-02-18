<template>
    <div class="alarm">
        <wz-search-bar ref="searchBar" :items="items" has-reset @search="onSearch" @reset="onReset"></wz-search-bar>
        <wz-table :data="tableData" :heads="tableHead" ref="table" :loading="loading" @btn-click="onTableBtnClick">
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
import { wzTable, wzSearchBar } from 'wz-ui'
import { recordGetAlarmList } from '../../../api'
import dayjs from 'dayjs'
export default {
    components: {
        wzTable,
        wzSearchBar
    },
    props: {
        form: Object
    },
    watch: {
        form: {
            handler(newVal) {
                this.fetchData()
            },
            deep: true
        }
    },
    methods: {
        onSearch(form) {
            this.searchForm.head.current = 1
            Object.assign(this.searchForm.body, { ...form })
            this.fetchData()
        },
        onReset() {
            this.searchForm = { body: {}, head: { current: 1, size: 10, total: 0 } }
            this.fetchData()
        },
        handleSizeChange() {
            this.searchForm.head.current = 1
            this.fetchData()
        },
        handleCurrentChange() {
            this.fetchData()
        },
        onTableBtnClick({ event, row }) {
            const func = new Map([['check', this.onCheck]])
            func.has(event) && func.get(event).call(this, row)
        },
        onCheck({ id }) {
            this.$router.push({ name: '告警详情', params: { id } })
        },
        fetchData() {
            this.loading = true
            const { time = [], ...rest } = this.searchForm.body
            const params = { ...rest }
            if (time && time.length !== 0) {
                params.startDate = time[0]
                params.endDate = time[1]
            }
            // params.facCode = this.form.facCode
            params.facId = this.form.id
            Object.assign(this.searchForm.body, params)
            recordGetAlarmList(this.searchForm)
                .then(({ current, records, total, size }) => {
                    Object.assign(this.searchForm.head, { current, total, size })
                    this.tableData = records
                })
                .finally(() => (this.loading = false))
        }
    },
    data() {
        return {
            tableHead: [
                {
                    label: '事件名称',
                    prop: 'alarmName'
                },
                {
                    label: '告警等级',
                    prop: 'alarmLevel',
                    formatter: ({ alarmLevel }) => this.$enum('alarm_level', alarmLevel)
                },
                {
                    label: '告警状态',
                    prop: '告警状态',
                    formatter: ({ alarmStatus }) =>
                        this.$enum('alarm_status', alarmStatus)
                },
                {
                    label: '告警时间',
                    prop: 'alarmDate',
                    formatter: ({ alarmDate }) => {
                        return alarmDate
                            ? dayjs(alarmDate).format('YYYY-MM-DD HH:mm:ss')
                            : ''
                    }
                },
                {
                    label: this.$t('pc.global.operate'),
                    rawWidth: 150,
                    // fixed: 'right',
                    btns: ({ loginFlag, userStatus, userType }) => {
                        return [{ label: this.$t('pc.global.check'), event: 'check' }]
                    }
                }
            ].map((item, index) => {
                const key = ['event_name', 'alarm_level', 'state', 'creation_time']
                if (index < key.length) {
                    item.label = this.$t(`pc.device_alarm_configuration.${key[index]}`)
                }
                return item
            }),
            tableData: [],
            items: [
                {
                    label: '事件名称',
                    labelWidth: '110px',
                    tag: 'el-input',
                    placeholder: this.$t('pc.global.please_input', {
                        name: this.$t(
                            'pc.device_alarm_configuration.event_name'
                        ).toLocaleLowerCase()
                    }),
                    clearable: true,
                    prop: 'alarmName'
                },
                {
                    label: '告警等级',
                    labelWidth: '110px',
                    tag: 'el-select',
                    placeholder: '',
                    clearable: true,
                    prop: 'alarmLevel',
                    options: this.$enums('alarm_level')
                },
                {
                    label: '告警时间',
                    labelWidth: '80px',
                    prop: 'time',
                    tag: 'el-date-picker',
                    type: 'datetimerange',
                    startPlaceholder: this.$t('pc.global.start_date'),
                    endPlaceholder: this.$t('pc.global.end_date'),
                    valueFormat: 'timestamp'
                }
            ].map((item, index) => {
                const key = ['event_name', 'alarm_level', 'alarm_time', 'creation_time']
                if (index < key.length) {
                    item.label = this.$t(`pc.device_alarm_configuration.${key[index]}`)
                }
                return item
            }),
            loading: false,
            searchForm: { body: {}, head: { current: 1, size: 10, total: 0 } }
        }
    }
}
</script>

<style lang="scss" scoped>

</style>
