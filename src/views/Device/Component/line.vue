<template>
    <div class="line">
        <!-- <search-bar
      ref="searchBar"
      :items="items"
      has-reset
      @search="onSearch"
      @reset="onReset"
    ></search-bar> -->
        <div class="row row-between searchContainer">
            <div>
                <span>{{ $t('pc.device_ledger.detection_time') }}：</span>
                <template v-for="item in items">
                    <component :key="item.prop" :is="item.tag" v-model="searchForm[item.prop]" v-bind="item"
                        ref="facCateCodeRef"></component>
                </template>
                <span class="total" v-if="attrCode === 'ep'">{{ $t('pc.device_ledger.shared_electricity') }}：{{ total }}
                    kW·h</span>
            </div>

            <div>
                <el-button type="primary" @click="onSearch">{{ $t('pc.global.search') }}</el-button>
                <el-button @click="onReset">{{ $t('pc.global.reset') }}</el-button>
            </div>
        </div>
        <div ref="myChart" class="myChart" :style="{ height: `${height}px` }"></div>
    </div>
</template>

<script>
import * as echarts from 'echarts'
import dayjs from 'dayjs'
import { facilitiesGetFacAttrData } from '../../../api'
export default {
    components: {},
    props: {
        facLinkCode: String,
        height: Number,
        dataUnit: String,
        attrCode: String,
        attrName: String
    },
    created() {
        this.fetchData()
    },
    methods: {
        onSearch(form) {
            Object.assign(this.searchForm, { page: 1, ...form })
            this.fetchData()
        },
        onReset() {
            this.searchForm = {
                page: 1,
                rows: 10,
                total: 0,
                time: [dayjs().format('YYYY-MM-DD'), dayjs().format('YYYY-MM-DD')]
            }
            this.fetchData()
        },
        fetchData() {
            const { time = [] } = this.searchForm
            const params = {
                // deviceId: '4e0f48afffb744d995f6efff7a1d3d41',
                deviceId: this.facLinkCode,
                // attribute: 'currentTemperature'
                attribute: this.attrCode,
                startTime: dayjs(time[0] + ' 00:00:00').valueOf(),
                endTime: dayjs(time[1] + ' 23:59:59').valueOf()
            }
            // const { attrCode } = this
            facilitiesGetFacAttrData(params).then(res => {
                const data = res.map(item => {
                    // return [dayjs(item.uptime).format('YYYY-MM-DD HH:mm:ss'), item.currentTemperature]
                    return [dayjs(item.uptime).format('YYYY-MM-DD HH:mm:ss'), item[this.attrCode]]
                })
                if (res.length > 1) {
                    this.total = (
                        Number(res[res.length - 1].currentTemperature) -
                        Number(res[0].currentTemperature)
                    ).toFixed(2)
                }
                this.series = data

                this.drawRing()
            })
            // let base = +new Date(1988, 9, 3)
            // const oneDay = 24 * 3600 * 1000

            // const data = [[base, Math.random() * 300]]

            // for (var i = 1; i < 200; i++) {
            //   var now = new Date((base += oneDay))
            //   data.push([
            //     [now.getFullYear(), now.getMonth() + 1, now.getDate()].join('/'),
            //     Math.round((Math.random() - 0.5) * 20 + data[i - 1][1])
            //   ])
            // }
        },
        drawRing() {
            // 基于准备好的dom，初始化echarts实例
            if (this.myChart) {
                this.myChart.dispose()
            }
            this.myChart = echarts.init(this.$refs.myChart)
            // 绘制图表
            this.myChart.setOption({
                tooltip: {
                    trigger: 'axis',
                    // formatter: '{b} 车流量 {c} 辆',
                    formatter: res => {
                        console.log('res', res)
                        return `${res[0].name || ''} <br /> ${res[0].marker || ''}${this.attrName || ''} ${res[0].data[1] || ''
                            } ${this.dataUnit || ''}`
                    },
                    // position: ["10%", "50%"],
                    confine: true,
                    appendToBody: true
                },
                grid: {
                    // show: true,
                    left: 130,
                    top: 40,
                    right: 130,
                    bottom: 70
                },
                xAxis: {
                    // 设置x轴
                    type: 'category',
                    boundaryGap: false, // 坐标轴两边不留白
                    // data: this.trafficFlowxLabels,
                    axisLine: {
                        // 坐标轴轴线相关设置。
                        lineStyle: {
                            fontSize: 12,
                            color: '#00000050'
                        }
                    }
                },
                yAxis: {
                    name: `${this.$t('pc.global.unit') + ':' + this.dataUnit || ''}`,
                    nameTextStyle: {
                        color: '#8C8C8C',
                        align: 'left'
                    },
                    axisLine: {
                        lineStyle: {
                            fontSize: 12,
                            color: '#00000050'
                        }
                    },
                    splitLine: {
                        show: true,
                        lineStyle: {
                            type: 'dashed',
                            color: '#00000020'
                        }
                    },
                    type: 'value'
                },
                dataZoom: [
                    {
                        type: 'inside',
                        start: 0,
                        end: 20
                    },
                    {
                        start: 0,
                        end: 20
                    }
                ],
                series: [
                    {
                        data: this.series,
                        type: 'line', // 类型为折线图
                        lineStyle: {
                            // 线条样式 => 必须使用normal属性
                            normal: {
                                color: '#7DB9E3',
                                width: 1
                            }
                        },
                        areaStyle: {
                            color: {
                                type: 'linear',
                                x: 0,
                                y: 0,
                                x2: 0,
                                y2: 1,
                                colorStops: [
                                    {
                                        offset: 0,
                                        color: '#7DB9E3' // 0% 处的颜色
                                    },
                                    {
                                        offset: 1,
                                        color: '#7DB9E310' // 100% 处的颜色
                                    }
                                ],
                                global: false // 缺省为 false
                            }
                        },
                        itemStyle: {
                            normal: {
                                color: '#7DB9E3',
                                borderColor: '#FFFFFF' // 拐点边框颜色
                            }
                        },
                        // smooth: true,
                        symbol: 'circle',
                        symbolSize: 1,
                        showSymbol: false,
                        showAllSymbol: true
                    }
                ]
            })
        },
        doResize() {
            this.drawRing()
        }
    },
    beforeDestroy() {
        if (this.myChart) {
            this.myChart.dispose()
            this.myChart = null
        }
        window.removeEventListener('resize', this.doResize)
    },
    mounted() {
        // this.$nextTick(() => this.drawRing());
        this.drawRing()
        window.addEventListener('resize', this.doResize)
    },
    data() {
        var pickerOptions = {
            shortcuts: [
                {
                    text: this.$t('pc.global.today_date'),
                    onClick(picker) {
                        const end = new Date()
                        const start = new Date()
                        start.setTime(start.getTime())
                        picker.$emit('pick', [start, end])
                    }
                },
                {
                    text: this.$t('pc.global.this_week'),
                    onClick(picker) {
                        const num = dayjs(new Date()).day()
                        const end = dayjs()
                            .add(6 - num, 'day')
                            .format('YYYY-MM-DD')
                        const start = dayjs()
                            .subtract(num, 'day')
                            .format('YYYY-MM-DD')
                        // start.setTime(start.getTime() - 3600 * 1000 * 24 * 7)
                        picker.$emit('pick', [start, end])
                    }
                },
                {
                    text: this.$t('pc.global.this_month'),
                    onClick(picker) {
                        // const end = new Date()
                        // const start = new Date()
                        // start.setTime(start.getTime() - 3600 * 1000 * 24 * 30)
                        const end = dayjs()
                            .endOf('month')
                            .format('YYYY-MM-DD')
                        const start = dayjs()
                            .startOf('month')
                            .format('YYYY-MM-DD')
                        picker.$emit('pick', [start, end])
                    }
                }
            ]
        }
        return {
            total: 0,
            myChart: null,
            series: [],
            searchForm: {
                time: [dayjs().format('YYYY-MM-DD'), dayjs().format('YYYY-MM-DD')]
            },
            items: [
                {
                    label: '监测时间',
                    labelWidth: '80px',
                    prop: 'time',
                    tag: 'el-date-picker',
                    type: 'daterange',
                    startPlaceholder: '开始日期',
                    endPlaceholder: '结束日期',
                    pickerOptions: pickerOptions,
                    valueFormat: 'yyyy-MM-dd'
                }
            ]
        }
    }
}
</script>

<style lang="scss" scoped>
.line {
    .myChart {
        width: 100%;
        height: 100%;
    }
}

.searchContainer {
    width: 100%;

    .el-button {
        width: 80px;
        margin-left: 10px;
    }
}

.total {
    margin-left: 20px;
}
</style>
