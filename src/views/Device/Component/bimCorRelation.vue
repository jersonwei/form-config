<template>
    <!-- BIM展示关联信息区域 -->
    <div class="bimCard">
        <div class="title">BIM展示关联信息区域</div>
        <el-divider></el-divider>
        <!-- 编辑和添加显示内容 -->
        <div class="positionInfo" v-if="mode !== 'check'">
            <el-form :model="form" ref="settingInfo" label-width="180px" label-suffix=":" status-icon>
                <!-- 栅格布局  -->
                <el-row>
                    <template v-for="item in settingItems">
                        <el-col :span="12" :key="item.prop">
                            <el-form-item :label="item.label" :prop="item.prop">
                                <!-- 条件显示区域一-->
                                <template v-if="item.prop === 'airconditionLink'">
                                    <div class="row aircondition-link">
                                        <el-input v-model.lazy="form.snapshot" placeholder="请输入快照名称" clearble
                                            :prop="item.prop"></el-input>
                                        <el-input v-model.lazy="form.airconditionLinkId" placeholder="请输入室内机组"
                                            :prop="item.prop" clearable></el-input>
                                        <el-tooltip placement="top-start" :content="item.tootip">
                                            <div>
                                                <i class="el-icon-question"></i>
                                            </div>
                                        </el-tooltip>
                                    </div>
                                </template>
                                <!-- 条件显示区域二 -->
                                <template v-else-if="item.tag === 'el-cascader'">
                                    <div class="row aircondition-link">
                                        <el-cascader :key="item.prop" :ref="item.ref" v-model="form[item.prop]"
                                            v-bind="item"></el-cascader>
                                        <el-tooltip placement="top-start" :content="item.tootip">
                                            <div>
                                                <i class="el-icon-question"></i>
                                            </div>
                                        </el-tooltip>
                                    </div>
                                </template>
                                <!-- 条件显示区域三 -->
                                <template v-else>
                                    <!-- <h1>条件显示区域三</h1> -->
                                    <div class="row aircondition-link">
                                        <!-- 动态组件  -->
                                        <component :is="item.tag" v-model.lazy="form[item.prop]" v-bind="item"
                                            :prop="item.prop"></component>
                                        <el-tooltip placement="top-start" :content="item.tootip">
                                            <div>
                                                <i class="el-icon-question"></i>
                                            </div>
                                        </el-tooltip>
                                    </div>
                                </template>
                            </el-form-item>
                        </el-col>
                    </template>
                </el-row>
            </el-form>
        </div>
        <div v-else class="positionInfo">
            <el-form class="procurementInfo" :model="form" label-suffix="：" label-width="146px" status-icon>
                <el-row>
                    <template v-for="item in settingItems">
                        <el-col :key="item.prop" :span="item.span ? item.span : 12">
                            <el-form-item :label="item.label" :prop="item.prop" :label-width="item.labelWidth">
                                <div class="value" v-if="item.prop === 'airconditionLinkId'">
                                    <span v-if="form.snapshot">{{ form.snapshot }}；</span>
                                    <span>{{ form[item.prop] }}</span>
                                </div>
                                <span class="value" v-else>{{ form[item.prop] }}</span>
                            </el-form-item>
                        </el-col>
                    </template>
                </el-row>
            </el-form>
        </div>
    </div>
    <!-- BIM展示关联信息区域底部 -->
</template>
<script>
import {
    getBuildingFloorTree,
    facilitiesGetById,
    getTreelistWithFac
} from '../../../api'
export default {
    name: 'bimCorRelation',
    props: {
        mode: {
            type: String,
            required: true
        }
    },
    data() {
        return {
            form: {},
            settingItems:
                this.mode !== 'check'
                    ? [
                        {
                            label: '设备绑定摄像头',
                            prop: 'facCamId',
                            tag: 'el-cascader',
                            placeholder: '请选择设备绑定摄像头',
                            showAllLevels: false,
                            props: {
                                checkStrictly: false,
                                label: 'name',
                                value: 'id',
                                multiple: true
                            },
                            tootip: '用于三维中设备详情页的视频查看',
                            options: []
                        },
                        {
                            label: '室外机空调绑定',
                            prop: 'airconditionLink',
                            showAllLevels: false,
                            props: {
                                checkStrictly: false,
                                label: 'name',
                                value: 'id',
                                multiple: true
                            },
                            tootip: '用于三维中空调室内外机的联动',
                            options: []
                        },
                        {
                            label: '电梯绑定梯控设备',
                            prop: 'elevatorLinkId',
                            tag: 'el-cascader',
                            placeholder: '请选择电梯绑定梯控设备',
                            showAllLevels: true,
                            props: {
                                checkStrictly: false,
                                label: 'name',
                                value: 'id'
                            },
                            tootip: '用于三维中电梯内梯控设备的绑定',
                            options: []
                        },
                        {
                            label: '配电柜绑定抽屉号',
                            prop: 'powerLinkId',
                            tag: 'el-cascader',
                            placeholder: '请选择配电柜绑定抽屉号',
                            showAllLevels: false,
                            props: {
                                checkStrictly: false,
                                label: 'name',
                                value: 'id',
                                multiple: true
                            },
                            tootip: '用于三维中抽屉柜的绑定联动展示',
                            options: []
                        },
                        {
                            label: '三维联动快照',
                            prop: 'bimSnapshot',
                            placeholder: '请输入三维联动快照',
                            tootip: '用于三维中点击设备列表后的设备位置的定位跳转',
                            maxlength: 50
                        }
                    ].map(item => {
                        if (!item.tag) {
                            Object.assign(item, { tag: 'el-input', clearable: true })
                        }
                        item.clearable = true
                        return item
                    })
                    : [
                        { label: '设备绑定摄像头', prop: 'facCamId' },
                        { label: '室外机空调绑定', prop: 'airconditionLinkId' },
                        { label: '电梯绑定梯控设备', prop: 'elevatorLinkId' },
                        { label: '配电柜绑定抽屉号', prop: 'powerLinkId' },
                        { label: '三维联动快照', prop: 'bimSnapshot' }
                    ]
        }
    },
    async created() {
        const { id } = this.$route.params
        console.log('created', id, this.mode)
        // 初始化数据
        await getTreelistWithFac({ systemCode: 'video_monitor_system' }).then(data => {
            this.settingItems[0].options = this.parseTreeOptions(data || [], 'bim')
        })
        await getTreelistWithFac({ systemCode: 'elevator_system' }).then(data => {
            this.settingItems[2].options = this.parseTreeOptions(data || [], 'bim')
        })
        await getTreelistWithFac({ systemCode: 'power_supply_and_distribution_system' }).then(
            data => {
                this.settingItems[3].options = this.parseTreeOptions(data || [], 'bim')
            }
        )
        id && this.getDetail(id)
    },
    mounted() {
    },
    watch: {
        form: {
            deep: true,
            handler(newVal, oldVal) {
                console.log(newVal, oldVal)
                this.$emit('bimForm', newVal)
            }
        }
    },
    methods: {
        // 详情级联多选回显
        cascaderMultipleShow(ids, list) {
            if (ids === null) {
                return null
            }
            const itemAry = []
            const echoTreeArr = []
            const eachAry = ids.includes(',') ? ids.split(',') : [ids]
            // 递归分类数据
            const recursionCategory = (list = []) => {
                const len = list.length
                for (let i = 0; i < len; i++) {
                    // 循环list参数，匹配回显的id
                    itemAry.push(list[i].name) // 构建分类树数组项,入栈
                    for (let j = 0; j < eachAry.length; j++) {
                        // 遍历子节点分类id，拼凑成数组项id，并终止循环
                        if (Number(eachAry[j]) === list[i].id) {
                            // 匹配到子节点id
                            echoTreeArr.push(JSON.parse(JSON.stringify(itemAry))) // push进树分类数据
                            eachAry.splice(j, 1) // 删除以匹配到的id
                            break
                        }
                    }
                    if (eachAry.length <= 0) {
                        // 所有回显id匹配完成后，跳出循环
                        break
                    } else if (list[i].children && list[i].children.length > 0) {
                        // 如果存在子分类，递归继续
                        recursionCategory(list[i].children)
                    }
                    itemAry.pop() // 出栈
                }
            }
            recursionCategory(list) // 调用递归
            return ids.includes(',') ? echoTreeArr : echoTreeArr[0]
        },
        // 通过ids换算成names
        idsTurnNames() {
            this.form.facCamId = this.form.facCamId.map(i => {
                const facCamId = i[i.length - 1] + ''
                return this.cascaderMultipleShow(
                    facCamId,
                    this.settingItems[0].options
                )
            }).map(i => i[[i.length - 1]]).join('，')


            this.form.powerLinkId = this.form.powerLinkId.map(i => {
                const facCamId = i[i.length - 1] + ''
                return this.cascaderMultipleShow(
                    facCamId,
                    this.settingItems[3].options
                )
            }).map(i => i[[i.length - 1]]).join('，')

            this.form.elevatorLinkId = this.cascaderMultipleShow(
                this.form.elevatorLinkId[this.form.elevatorLinkId.length - 1] + '',
                this.settingItems[2].options
            ).join('/')
        },
        // 级联回显
        industrialChinaShow(ids, list) {
            if (ids === null) {
                return null
            }
            const echoTreeArr = []
            const eachAry = ids.includes(',') ? ids.split(',') : [ids]
            const itemAry = [] // 分类树组件，每一项的id数组
            // 递归分类数据
            const recursionCategory = (list = []) => {
                const len = list.length
                for (let i = 0; i < len; i++) {
                    // 循环list参数，匹配回显的id
                    itemAry.push(list[i].id) // 构建分类树数组项,入栈
                    for (let j = 0; j < eachAry.length; j++) {
                        // 遍历子节点分类id，拼凑成数组项id，并终止循环
                        if (Number(eachAry[j]) === list[i].id) {
                            // 匹配到子节点id
                            echoTreeArr.push(JSON.parse(JSON.stringify(itemAry))) // push进树分类数据
                            eachAry.splice(j, 1) // 删除以匹配到的id
                            break
                        }
                    }
                    if (eachAry.length <= 0) {
                        // 所有回显id匹配完成后，跳出循环
                        break
                    } else if (list[i].children && list[i].children.length > 0) {
                        // 如果存在子分类，递归继续
                        recursionCategory(list[i].children)
                    }
                    itemAry.pop() // 出栈
                }
            }
            recursionCategory(list) // 调用递归
            return ids.includes(',') ? echoTreeArr : echoTreeArr[0]
        },
        // 添加id唯一标识
        parseTreeOptions(data, type) {
            return data.map(({ children, id, level, ...rest }) => {
                let obj = {}
                if (Array.isArray(children)) {
                    obj = {
                        id: `${level},${id}`,
                        level,
                        ...rest
                    }
                    obj.children = this.parseTreeOptions(children, type)
                } else {
                    obj = {
                        id,
                        disabled: this.id === id && type === 'bim',
                        level,
                        ...rest
                    }
                }
                return obj
            })
        },
        async getDetail(id) {
            const res = await facilitiesGetById(id)
            const {
                facCamIds,
                airconditionLinkSnapshot,
                airconditionLinkIndoors,
                elevatorLinkId,
                powerLinkIds,
                bimSnapshot
            } = res
            this.$set(this, 'form', {
                facCamId: facCamIds.split(',').map(i => this.industrialChinaShow(
                    i,
                    this.settingItems[0].options
                )),
                snapshot: airconditionLinkSnapshot,
                airconditionLinkId: airconditionLinkIndoors,
                elevatorLinkId: this.industrialChinaShow(
                    elevatorLinkId,
                    this.settingItems[2].options
                ),
                powerLinkId: powerLinkIds.split(',').map(i => this.industrialChinaShow(
                    i,
                    this.settingItems[3].options
                )),
                bimSnapshot: bimSnapshot
            })
            if (this.mode === 'check') {
                this.idsTurnNames()
            }
        }
    }
}
</script>
<style lang="scss" scoped>
.bimCard {
    padding: 12px 24px 6px;
    margin: 38px 0;
    background: #fff;

    .aircondition-link {

        .el-input,
        .el-cascader {
            margin-right: 8px;
            width: 100%;
        }
    }
}

.procurementInfo {
    padding: 12px 24px 6px;
    margin-top: 24px;
    background: #fff;
}
</style>
