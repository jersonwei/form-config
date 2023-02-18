<template>
    <!-- <div class="detail" :style="{ height: `${pageHeight}px` }"> -->
    <div class="detail">
        <!-- <el-tabs type="card">
      <el-tab-pane label="基础信息">
        <el-form :model="detail" label-suffix="：" label-width="140px">
          <el-row>
            <template v-for="item in baseInfoItems">
              <el-col :span="item.span || 12" :key="item.prop">
                <el-form-item :label="item.label">
                  {{
                    item.formatter
                      ? item.formatter(detail)
                      : detail[item.prop] || '无'
                  }}
                </el-form-item>
              </el-col>
            </template>
          </el-row>
        </el-form>
      </el-tab-pane>
    </el-tabs>
    <el-tabs type="card">
      <el-tab-pane label="操作日志"> </el-tab-pane>
    </el-tabs> -->
        <div class="title">{{ $t('pc.space_type.basic_information') }}</div>
        <el-divider></el-divider>
        <el-form :model="detail" label-suffix="：" label-width="140px">
            <el-row>
                <template v-for="item in baseInfoItems">
                    <el-col :span="item.span || 12" :key="item.prop">
                        <el-form-item :label="item.label">
                            {{
                                item.formatter
                                    ? item.formatter(detail)
                                    : detail[item.prop] || $t('pc.global.nothing')
                            }}
                        </el-form-item>
                    </el-col>
                </template>
            </el-row>
        </el-form>
        <div v-if="false">
            <div class="title">{{ $t('pc.space_type.log') }}</div>
            <el-divider></el-divider>
            <wz-table :data="tableData" :heads="tableHead" ref="table" :loading="loading"></wz-table>
            <div class="pagination row row-right">
                <!-- <span class="total">共{{ searchForm.total }}条数据</span> -->
                <el-pagination ref="pager" layout="sizes,prev,pager,next,jumper" :current-page.sync="searchForm.page"
                    :page-size.sync="searchForm.rows" :total="searchForm.total" @current-change="fetchData"
                    @size-change="handleSizeChange" background></el-pagination>
            </div>
        </div>
    </div>
</template>

<script>
import { tablePage } from '../../../mixins/mixins'
import { wzTable } from 'wz-ui'
export default {
    mixins: [tablePage.call(this)],
    components: {
        wzTable
    },
    methods: {
        // doLayout () {
        //   this.pageHeight = 0
        //   this.$nextTick(() => {
        //     const card = document.querySelector(
        //       '.table-container .el-card .el-card__body'
        //     )
        //     this.pageHeight = card.clientHeight - 40
        //   })
        // }
        fetchData() { },
        handleSizeChange() {
            this.searchForm.page = 1
            this.fetchData()
        }
    },
    props: { detail: Object },
    created() { },
    // mounted () {
    //   window.addEventListener('resize', this.doLayout)
    //   this.doLayout()
    // },
    // beforeDestroy () {
    //   window.removeEventListener('resize', this.doLayout)
    // },
    data() {
        return {
            pageHeight: 0,
            baseInfoItems: [
                { label: '空间名称', prop: 'spaceName' },
                {
                    label: '上级类型',
                    formatter: ({ parentName }) => parentName || this.$t(`pc.global.nothing`)
                },
                { label: '类型编号', prop: 'spaceCode' }
            ].map((item, index) => {
                const key = ['space_name', 'superior_type', 'type_number']
                item.label = this.$t(`pc.space_type.${key[index]}`)
                return item
            }),
            tableHead: [
                { label: '修改内容', prop: '修改内容' },
                { label: '操作人', prop: '操作人' },
                { label: '操作日期', prop: '操作日期' }
            ].map((item, index) => {
                const key = ['modify_content', 'handler', 'handle_date']
                item.label = this.$t(`pc.space_type.${key[index]}`)
                return item
            }),
            tableData: []
        }
    }
}
</script>

<style lang="scss" scoped>

</style>
