<template>
    <div class="runParams">
        <!-- <search-bar
      ref="searchBar"
      :items="items"
      has-reset
      @search="onSearch"
      @reset="onReset"
    ></search-bar> -->
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
import { wzTable } from 'wz-ui'
// import searchBar from '@/components/searchBar.vue'
import { facAttributePages } from '../../../api'
import dayjs from 'dayjs'
export default {
    components: {
        wzTable
        // searchBar
    },
    props: {
        form: Object
    },
    watch: {
        form: {
            handler(newVal) {
                this.fetchData()
                // const { facCateCode, facLinkCode } = newVal
            },
            deep: true
        }
    },
    created() {
        this.$nextTick(() => {
            // console.log(this.form)
            // const { facCateCode } = this.form
            // console.log(facCateCode)
            // facAttributePages({ facCateCode }).then(data => {
            //   console.log(data)
            // })
        })
    },
    computed: {
        tableHead() {
            return [
                {
                    label: '属性',
                    prop: 'attrName'
                },
                {
                    label: '单位',
                    prop: 'dataUnit'
                },
                {
                    label: '参数',
                    prop: 'params'
                },
                {
                    label: '监测时间',
                    prop: 'lastUpdatedDate',
                    formatter: ({ lastUpdatedDate }) => {
                        return lastUpdatedDate
                            ? dayjs(lastUpdatedDate).format('YYYY-MM-DD HH:mm:ss')
                            : ''
                    }
                },
                {
                    label: '曲线图',
                    rawWidth: 150,
                    // fixed: 'right',
                    btns: ({ loginFlag, userStatus, userType }) => {
                        return [{ label: this.$t('pc.global.check'), event: 'check' }]
                    }
                }
            ].map((item, index) => {
                const key = [
                    'attribute',
                    'unit',
                    'parameter',
                    'monitoring_time',
                    'graphs'
                ]
                item.label = this.$t(`pc.device_ledger.${key[index]}`)
                return item
            })
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
        onCheck({ attrCode, attrName, dataUnit }) {
            const { facLinkCode } = this.form
            this.$emit('openLine', { facLinkCode, attrCode, attrName, dataUnit })
        },
        fetchData() {
            Object.assign(this.searchForm.body, {
                facCategoryId: this.form.facCategoryId,
                facLinkCode: this.form.facLinkCode,
                id: this.form.id
            })
            facAttributePages(this.searchForm)
                .then(({ records, current, size, total }) => {
                    this.tableData = records || []
                    Object.assign(this.searchForm.head, {
                        current: current || 1,
                        size: size || 10,
                        total: total || 0
                    })
                })
                .finally(() => (this.loading = false))
        }
    },
    data() {
        return {
            tableData: [],
            loading: false,
            searchForm: { body: {}, head: { current: 1, size: 10, total: 0 } }
        }
    }
}
</script>

<style lang="scss" scoped>

</style>
