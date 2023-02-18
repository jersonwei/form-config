<template>
    <div class="baseInfo">
        <div class="info">
            <div class="title">{{ $t("pc.device_ledger.device_information") }}</div>
            <el-divider></el-divider>
            <div class="row col-top">
                <!-- <div class="uoloadBox flex-column">
          <div class="flex-column">
            <span class="deviceImgText">{{
              $t('pc.device_ledger.device_pictures')
            }}</span>
            <img
              src="../../../assets/imgs/defaultImg.png"
              alt=""
              class="defaultImg"
              v-if="!form.facImg"
            />
            <img :src="showFacImg" alt="" v-else />
          </div>
          <div>
            <span class="deviceImgText">{{
              $t('pc.device_ledger.qr_code')
            }}</span>
            <div
              class="qr-code"
              @mouseover="btnShow = true"
              @mouseout="btnShow = false"
            >
              <el-image
                class=""
                :src="qrCode"
                :key="qrCode"
                fit="contain"
                id="printTable"
              ></el-image>
              <div class="btn-group flex-row" v-show="btnShow">
                <span @click="onDownload(qrCode)">{{
                  $t('pc.global.download')
                }}</span>
                <el-divider direction="vertical"></el-divider>
                <span @click="onPrint(qrCode)">{{
                  $t('pc.global.print')
                }}</span>
              </div>
            </div>
          </div>
        </div> -->
                <el-form class="deviceInfo" ref="deviceInfo" :model="form" label-suffix="：" label-width="166px"
                    status-icon>
                    <el-row>
                        <template v-for="item in items">
                            <el-col :key="item.prop" :span="item.span ? item.span : 12">
                                <el-form-item :label="item.label" :prop="item.prop" :label-width="item.labelWidth" v-if="
                                    item.prop === 'playAttr' &&
                                    form.systemCode === 'information_publishing_system'
                                ">
                                    <span class="value">{{
                                        [
                                            $t("pc.device_ledger.cannot_play_audio_or_video"),
                                            $t("pc.device_ledger.audio_only"),
                                            $t("pc.device_ledger.video")
                                        ][Number(form[item.prop]) - 1]
                                    }}</span>
                                </el-form-item>
                                <el-form-item :label="item.label" :prop="item.prop" :label-width="item.labelWidth"
                                    v-if="item.prop !== 'playAttr'">
                                    <span class="value" v-if="item.isTime">{{
                                        formatTime(form[item.prop], item.isDateTime)
                                    }}</span>
                                    <span class="value" v-else-if="
                                        item.label === $t('pc.device_ledger.asset_status')
                                    ">{{ $enum("asset_status", Number(form[item.prop])) }}</span>
                                    <!-- 开门方式查看显示 -->
                                    <span class="value" v-else-if="
                                        item.label === $t(`pc.authorization_list.opening_mode`)
                                    ">
                                        {{ form[item.prop] | openTypeFilter }}
                                    </span>
                                    <!-- 设备IP查看显示 -->
                                    <span class="value" v-else-if="
                                        item.label === $t(`pc.device_ledger.equipment_ip`)
                                    ">
                                        {{ form[item.prop] || "暂无" }}
                                    </span>
                                    <span class="value" v-else>{{ form[item.prop] }}</span>
                                </el-form-item>
                            </el-col>
                        </template>
                    </el-row>
                </el-form>
            </div>
        </div>
        <div class="info">
            <div class="title">技术参数</div>
            <el-divider></el-divider>
            <div class="row col-top">
                <el-form class="deviceInfo" ref="deviceInfo" :model="form" label-suffix="：" label-width="166px"
                    status-icon>
                    <el-row>
                        <template v-for="item in paramsData">
                            <el-col :key="item.prop" :span="item.span ? item.span : 12">
                                <el-form-item :label="item.label" :prop="item.prop" :label-width="item.labelWidth">
                                    <template v-if="
                                        item.fieldType === 'fileUpload' &&
                                        Array.isArray(item.value)
                                    ">
                                        <!-- 文件显示 -->
                                        <ul class="el-upload-list el-upload-list--text">
                                            <li v-for="file in item.value" :key="file.fileName"
                                                class="el-upload-list__item is-success" @click="down(file)">
                                                <a class="el-upload-list__item-name">
                                                    <i class="el-icon-document"></i>{{ file.fileName }}
                                                </a>
                                            </li>
                                        </ul>
                                    </template>
                                    <span v-else class="value">{{ item.value }}</span>
                                </el-form-item>
                            </el-col>
                        </template>
                    </el-row>
                </el-form>
            </div>
        </div>
    </div>
</template>

<script>
import dayjs from 'dayjs'
import objArr from 'wz-file/lib/api.js'
import QRCode from 'qrcode'
import printJS from 'print-js'
import { download } from '../../../utils/func.js'
export default {
    props: {
        id: {
            type: [Number, String],
            default: 0
        },
        form: Object,
        paramsData: Array,
        timestamp: [Number, String]
    },
    filters: {
        openTypeFilter(value) {
            console.log('过滤器')
            switch (value) {
                case '0':
                    value = '刷卡'
                    break
                case '1':
                    value = '人脸'
                    break
                case '2':
                    value = '二维码'
                case null:
                    value = '暂无'
                    break
                default:
                    break
            }
            console.log('过滤器', value)
            return value
        }
    },
    watch: {
        form() {
            this.getImg()
            this.getQrCode()
        }
    },
    data() {
        return {
            showFacImg: '',
            qrCode: '',
            btnShow: false,
            items: [
                {
                    label: '所属项目',
                    prop: 'projectName'
                },
                {
                    label: '所属场站',
                    prop: 'stationName'
                },
                {
                    label: '设备名称',
                    prop: 'facName'
                },
                {
                    label: '设备类型',
                    prop: 'facCateName'
                },
                {
                    label: '设备物联ID',
                    prop: 'facLinkCode'
                },
                {
                    label: '设备型号',
                    prop: 'modelName'
                },
                // {
                //   label: 'BIM构件ID',
                //   prop: 'facBimCode'
                // },
                {
                    label: '设备品牌',
                    prop: 'brandName'
                },
                {
                    label: '生产厂家',
                    prop: 'manufacturer'
                },
                {
                    label: '规格',
                    prop: 'specification'
                },
                {
                    label: '尺寸信息(高X宽X深mm)',
                    prop: 'sizeInfomation',
                    labelWidth: '200'
                },
                {
                    label: '重量(kg)',
                    prop: 'weight'
                }
            ]
        }
    },
    methods: {
        getImg() {
            const { facImg } = this.form
            if (facImg) {
                const { getImgUrl } = objArr
                getImgUrl(facImg).then(responseData => {
                    const myBlob = new window.Blob([responseData], { type: 'image/jpeg' })
                    const Url = window.URL.createObjectURL(myBlob)
                    this.$set(this, 'showFacImg', Url)
                })
            }
        },
        getQrCode() {
            const { facQrcode } = this.form
            if (facQrcode) {
                QRCode.toDataURL(facQrcode, (err, url) => {
                    this.qrCode = url
                    err && console.log(err)
                })
            }
        },
        formatTime(time, isDateTime) {
            if (!time) {
                return null
            } else {
                if (isDateTime) {
                    return dayjs(time).format('YYYY-MM-DD HH:mm:ss')
                } else {
                    return dayjs(time).format('YYYY-MM-DD')
                }
            }
        },
        onPrint(b64data) {
            // print({ printable: b64data, type: 'image' })
            printJS({
                printable: 'printTable',
                type: 'html',
                header: null,
                targetStyles: ['*'],
                style: '{margin:0 10mm}'
            })
        },
        onDownload(b64data) {
            const link = document.createElement('a')
            link.id = 'qrCode'
            link.href = b64data
            // this.$refs.card.$el.appendChild(link)
            link.setAttribute('download', 'qrcode')
            link.click()
            this.$message.success(this.$t('pc.global.download_success'))
        },
        async down({ fid, fileName }) {
            const file = await objArr.download(fid)
            download(file, fileName, this)
        }
    },
    created() {
        this.getImg()
        this.getQrCode()
    }
}
</script>

<style lang="scss" scoped>
.baseInfo {
    .info {
        padding: 20px;
        background-color: #fff;
        margin-bottom: 20px;
    }

    .uoloadBox {
        margin-top: 6px;

        .deviceImgText {
            font-size: 14px;
            color: #231815;
            line-height: 24px;
        }

        img {
            width: 160px;
            height: 160px;
            margin-top: 8px;
        }

        .el-image {
            width: 160px;
            height: 160px;
            margin-top: 8px;
        }
    }

    .deviceInfo {
        margin-top: 38px;
    }

    .el-form /deep/ {
        flex: 1;
        padding-left: 2px;
        padding-right: 36px;

        .el-form-item__label {
            color: #231815;
        }

        .value {
            font-size: 14px;
            color: #5f5d5d;
        }
    }

    .qr-code {
        position: relative;

        .btn-group {
            span {
                flex: 1;
                text-align: center;
                cursor: pointer;
            }

            .el-divider {
                height: 40px;
            }

            bottom: 0;
            left: 0;
            width: 100%;
            position: absolute;
            background-color: rgba($color: #000000, $alpha: 0.8);
            color: white;
        }
    }
}
</style>
