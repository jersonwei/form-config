<template>
    <div class="addDevice">
      <div class="returnBtn">
        <el-button type="primary" @click="onSave">{{
          id === 0 ? $t('pc.global.add') : $t('pc.global.save')
        }}</el-button>
        <el-button @click="$router.push({ name: '设备台账' })">返回</el-button>
      </div>
      <!-- 设备信息区域 -->
      <div class="customCard">
        <div class="title">{{ $t('pc.device_ledger.device_information') }}</div>
        <el-divider></el-divider>
        <div class="row col-top">
          <el-form
            ref="deviceInfo"
            :model="form"
            label-suffix="："
            label-width="200px"
            :rules="rules"
            status-icon
            class="deviceInfo"
          >
            <el-row>
              <template v-for="item in deviceInfoItems">
                <el-col :key="item.prop" :span="item.span ? item.span : 12">
                  <el-form-item
                    v-if="item.prop !== 'playAttr'"
                    :label="item.label"
                    :prop="item.prop"
                    :label-width="item.labelWidth"
                  >
                    <template v-if="item.tag === 'el-select'">
                      <el-select v-model="form[item.prop]" v-bind="item">
                        <template v-for="opt in item.options">
                          <el-option :key="opt.value" v-bind="opt"></el-option>
                        </template>
                      </el-select>
                    </template>
                    <template v-else-if="item.tag === 'el-cascader'">
                      <el-cascader
                        :ref="item.ref"
                        @change="e => change(e, item.prop)"
                        v-model="form[item.prop]"
                        v-bind="item"
                        :options="
                          item.optionsCode
                            ? options[item.optionsCode]
                            : item.option
                        "
                        :show-all-levels="item.showAllLevels || false"
                      ></el-cascader>
                    </template>
                    <template v-else-if="item.tag === 'el-autocomplete'">
                      <el-autocomplete
                        v-model="form[item.prop]"
                        :fetch-suggestions="querySearchAsync"
                        v-bind="item"
                        @select="handleSelect"
                      ></el-autocomplete>
                    </template>
                    <template v-else>
                      <component
                        :is="item.tag"
                        v-model="form[item.prop]"
                        v-bind="item"
                      ></component>
                    </template>
                  </el-form-item>
                  <el-form-item
                    v-if="
                      item.prop === 'playAttr' &&
                        facCateCodeIsInformationPublishingSystem
                    "
                    :label="item.label"
                    :prop="item.prop"
                    :label-width="item.labelWidth"
                  >
                    <el-radio-group v-model="form[item.prop]">
                      <el-radio :label="1">{{
                        $t('pc.device_ledger.cannot_play_audio_or_video')
                      }}</el-radio>
                      <el-radio :label="2">{{
                        $t('pc.device_ledger.audio_only')
                      }}</el-radio>
                      <el-radio :label="3">{{
                        $t('pc.device_ledger.video')
                      }}</el-radio>
                    </el-radio-group>
                  </el-form-item>
                </el-col>
              </template>
            </el-row>
          </el-form>
        </div>
      </div>
      <!-- 参数配置 目前只有设备为逆变器的时候有参数配置 -->
      <div class="customCard" v-if="isInverterType">
        <div class="title">技术参数</div>
        <el-divider></el-divider>
        <el-form
          :model="paramsData"
          label-suffix="："
          label-width="170px"
          status-icon
          ref="paramsInfo"
        >
          <el-row :gutter="20">
            <el-col
              v-for="row in parmasFormConfig"
              :span="row.span"
              :key="row.index"
            >
              <template v-for="item in row.list">
                <el-form-item
                  :key="item.prop"
                  :label="item.label"
                  :prop="item.prop"
                >
                  <template
                    v-if="item.tag === 'radio' || item.tag === 'checkbox'"
                  >
                    <component
                      :is="`el-${item.tag}-group`"
                      v-model="paramsData[item.prop]"
                      :disabled="item.disabled"
                    >
                      <template v-for="radio in item.options">
                        <component
                          :is="`el-${item.tag}`"
                          :label="radio.value"
                          :key="radio.value"
                          >{{ radio.label }}</component
                        >
                      </template>
                    </component>
                  </template>
                  <!-- 文件显示 -->
                  <ul
                    v-else-if="item.tag === 'fileList'"
                    class="el-upload-list el-upload-list--text"
                  >
                    <li
                      v-for="file in paramsData[item.prop]"
                      :key="file.name"
                      class="el-upload-list__item is-success"
                      @click="down(file)"
                    >
                      <a class="el-upload-list__item-name">
                        <i class="el-icon-document"></i>{{ file.name }}
                      </a>
                    </li>
                  </ul>
                  <el-upload
                    v-else-if="item.tag === 'fileUpload'"
                    :ref="item.prop"
                    class="upload-demo"
                    :accept="accept"
                    :file-list="paramsData[item.prop]"
                    action=""
                    :headers="uploadHeader"
                    :auto-upload="false"
                    :limit="1"
                  >
                    <el-button size="small" type="primary">点击上传</el-button>
                    <div slot="tip" class="el-upload__tip">
                      文件格式:
                      dwg，psd，zip，rar，pdf，doc，rp，ppt，vsdx，pptx，docx，xls，xlsx，jpg，png，jpeg，gif；
                      <br />文件大小：≤50MB
                    </div>
                  </el-upload>
                  <div v-else class="row">
                    <el-tooltip
                      v-if="item.tooltip"
                      placement="top-start"
                      :content="item.tooltip"
                      ><i class="el-icon-question"></i
                    ></el-tooltip>
                    <component
                      :is="item.tag"
                      v-bind="item"
                      :showAllLevels="item.showAllLevels || false"
                      :disabled="item.disabled"
                      v-model="paramsData[item.prop]"
                    >
                      <template v-if="item.append" slot="append">{{
                        item.append
                      }}</template>
                    </component>
                  </div>
                </el-form-item>
              </template>
            </el-col>
          </el-row>
        </el-form>
      </div>
      <!-- 采购信息展示区域 -->
      <!-- <div class="customCard">
        <div class="title">
          {{ $t('pc.device_ledger.purchasing_information') }}
        </div>
        <el-divider></el-divider>
        <el-form
          :model="form"
          label-suffix="："
          label-width="156px"
          :rules="rules"
          status-icon
        >
          <el-row>
            <template v-for="item in procurementInfoItems">
              <el-col :key="item.prop" :span="item.span ? item.span : 8">
                <el-form-item
                  :label="item.label"
                  :prop="item.prop"
                  :label-width="item.labelWidth"
                >
                  <template>
                    <component
                      :is="item.tag"
                      v-model="form[item.prop]"
                      v-bind="item"
                    ></component>
                  </template>
                </el-form-item>
              </el-col>
            </template>
          </el-row>
        </el-form>
      </div> -->
      <!-- 底部区域 -->
      <!-- <div class="footer row center">
        <el-button @click="onReturn">{{ $t('pc.global.cancel') }}</el-button>
        <el-button type="primary" @click="onSave">{{
          id === 0 ? $t('pc.global.add') : $t('pc.global.save')
        }}</el-button>
      </div> -->
    </div>
  </template>
  
  <script>
  import {
    findCategoryTree,
    facilitiesGetById,
    getBuildingFloorTree,
    facilitiesAdd,
    facilitiesUpdate,
    getFormrelationPage,
    getDeviceFormConfigByKey,
    getFormNodesConfig,
    getProjectAccountList,
    getStationAccountListAll
  } from '../../api'
  import objArr from 'wz-file/lib/api.js'
  import customUpload from '../../components/upload.vue'
  import { download } from '../../utils/func.js'
  export default {
    name: 'AddDevice',
    methods: {
      // 获取子组件表单数据
      getBimForm (value) {
        this.bimFormdata = value
        console.log(this.bimFormdata, 'this.bimFormdata')
      },
      onFileRemove () {
        this.fileList = []
        this.form.showFacImg = ''
        this.form.facImg = ''
        this.showUpload = true
      },
      async onFileListChange (file, fileList) {
        this.showUpload = false
        const { upload, getImgUrl } = objArr
        const type = file.name
          .split('.')
          .pop()
          .toLowerCase()
        const set = new Set(['jpg', 'png', 'gif', 'jpeg'])
        if (!set.has(type)) {
          this.$message.error(this.$t('pc.global.upload_warnming_tips1'))
          fileList.pop()
          this.fileList = []
          this.showUpload = true
          return
        }
        const {
          uploadFiles,
          uploadFiles: { length }
        } = this.$refs.upload.dom
        if (length > 0) {
          const [uploadFile] = uploadFiles
          const formData = new FormData()
          formData.append('file', uploadFile.raw)
          formData.append('classify', 'user')
          formData.append('bizPath', 'device')
          try {
            const { fid } = await upload(formData)
            this.form.facImg = fid
            await getImgUrl(fid).then(responseData => {
              const myBlob = new window.Blob([responseData], {
                type: 'image/jpeg'
              })
              const Url = window.URL.createObjectURL(myBlob)
              this.$set(this.form, 'showFacImg', Url)
            })
            // this.fileList = []
          } catch (e) {
            console.log(e)
          }
        }
      },
      change (e, prop) {
        if (prop === 'facCateCode') {
          const {
            id,
            facCateCode
          } = this.$refs.facCateCodeRef[0].getCheckedNodes()[0].data
          this.form.facCategoryId = id
          // 获取配置
          this.getDeviceFormConfig(facCateCode)
        }
      },
      async getDeviceFormConfig (facCateCode) {
        const { formConfigurationVos } = await getDeviceFormConfigByKey({
          formKey: facCateCode
        })
        if (formConfigurationVos) {
          this.formConfigurationVos = formConfigurationVos
          this.setFormConfig(formConfigurationVos)
          this.isInverterType = true
          this.isOtherType = null
        } else {
          this.isInverterType = false
        }
        this.setDeviceInfoItems(this.isInverterType)
      },
      onReturn () {
        this.$router.push({
          name: '设备台账',
          params: {
            parentIds: this.$route.params.parentIds,
            deviceId: this.$route.params.deviceId
          }
        })
        window.closeTags('新增设备')
        window.closeTags('编辑设备')
        window.closeTags('设备详情')
      },
      async onSave () {
        await this.$refs.deviceInfo.validate()
        const { id, tagType } = this
        console.log(id, tagType)
        const { building, facCateCode, openType, ...rest } = this.form
        console.log('进入下一步', building, facCateCode)
        // TODO: 接口完整后按要求整理入参数据
        const {
          facCamId,
          snapshot,
          airconditionLinkId,
          elevatorLinkId,
          powerLinkId,
          bimSnapshot
        } = this.bimFormdata
        const params = {
          ...rest,
          facCateCode: Array.isArray(facCateCode)
            ? facCateCode[facCateCode.length - 1]
            : facCateCode,
          spaceId: Array.isArray(building) ? building[2] : building,
          openType: Array.isArray(openType) ? openType.join(',') : null,
          facCamIds: Array.isArray(facCamId)
            ? facCamId.map(i => i[i.length - 1]).join(',')
            : facCamId,
          airconditionLinkSnapshot: snapshot,
          airconditionLinkIndoors: airconditionLinkId,
          elevatorLinkId: Array.isArray(elevatorLinkId)
            ? elevatorLinkId[elevatorLinkId.length - 1]
            : elevatorLinkId,
          powerLinkIds: Array.isArray(powerLinkId)
            ? powerLinkId.map(i => i[i.length - 1]).join(',')
            : powerLinkId,
          bimSnapshot: bimSnapshot
        }
        // 需要判断是否是逆变器类型  且 设备型号是默认型号other
        if (this.isOtherType === 'other') {
          // 参数配置是否有文件上传类型 以及是否需要上传文件
          if (this.uploadRefArr.length) {
            const fileUploadArr = []
            this.uploadRefArr.forEach(item => {
              const {
                uploadFiles,
                uploadFiles: { length }
              } = this.$refs[item][0]
  
              if (length > 0) {
                const [uploadFile] = uploadFiles
                const formData = new FormData()
                console.log(uploadFile)
                const docSize = uploadFile.size
  
                const arr = uploadFile.name.split('.')
                const acceptList = this.accept.split('，')
                const suffix = arr[arr.length - 1]
                if (!acceptList.includes(suffix)) {
                  this.$message.error(`文件限制格式: ${this.accept}！`)
                  return
                }
  
                if (docSize / 1024 / 1024 > 50) {
                  this.$message.error('文件大小不能超过50MB！')
                  return uploadRefArr
                }
  
                formData.append('file', uploadFile.raw)
                formData.append('classify', 'user')
                formData.append('bizPath', 'device')
                const { upload } = objArr
                fileUploadArr.push(upload(formData))
              }
            })
            if (fileUploadArr.length) {
              const res = await Promise.all(fileUploadArr)
              console.log(this.paramsData)
              this.uploadRefArr.forEach((item, index) => {
                const { fileName, url, fid } = res[index]
                this.paramsData[item] = JSON.stringify([
                  {
                    name: fileName,
                    fid
                  }
                ])
              })
            }
          }
          this.nodesConfig.forEach(({ fieldKey, fieldType }, index) => {
            const val = this.paramsData[fieldKey]
            if (val) {
              if (Array.isArray(val)) {
                if (fieldType === 'fileUpload') {
                  this.nodesConfig[index].defaultValue = JSON.stringify(val)
                } else {
                  this.nodesConfig[index].defaultValue = val.join(',')
                }
              } else {
                this.nodesConfig[index].defaultValue = val
              }
            }
          })
          // 如果是默认类型创建和编辑的时候需要设置 formVo
          const { modelName } = this.form
          params.formVo = {
            formKey: modelName,
            formName: modelName,
            formConfigurationVos: this.nodesConfig
          }
        }
  
        console.log('第一次赋值给params')
        // 如果添加时未选择信息系统或者修改时不是信息系统
        if (
          (id && this.form.systemCode !== 'information_publishing_system') ||
          (Array.isArray(facCateCode) &&
            facCateCode[0] !== 'information_publishing_system')
        ) {
          params.playAttr = null
        }
        if (tagType) {
          params.tagType = tagType
        }
        if (id === 0) {
          console.log('facilitiesAdd', params)
          facilitiesAdd(params).then(() => {
            this.$message.success(
              this.$t('pc.global.add_success', {
                name: this.$t('pc.device_ledger.device')
              })
            )
            window.closeTags('新增设备')
            this.$router.go(-1)
          })
        } else {
          params.id = id
          console.log('facilitiesUpdate', params)
          facilitiesUpdate(params).then(() => {
            this.$message.success(
              this.$t('pc.global.edit_success', {
                name: this.$t('pc.device_ledger.device')
              })
            )
            window.closeTags('编辑设备')
            this.$router.push({
              name: '设备台账',
              params: {
                parentIds: this.$route.params.parentIds,
                deviceId: this.$route.params.deviceId
              }
            })
          })
        }
      },
      // 解析数据
      parseTreeNode (node, key = 'id') {
        return node.map(({ children, level, ...rest }) => {
          const obj = {
            level,
            disabled: level !== 4,
            ...rest
          }
          if (Array.isArray(children)) {
            obj.children = this.parseTreeNode(children, key)
          }
          return obj
        })
      },
      // 根据路由id获取页面初始信息
      async getDetail () {
        const { id } = this.$route.params
        console.log('成功调取', id)
        // const { getImgUrl } = objArr
        id &&
          facilitiesGetById(id).then(async data => {
            const {
              spaceId,
              buildingId,
              floorId,
              facImg,
              openType,
              facCateCode,
              ...rest
            } = data
            console.log('facilitiesGetById', data)
            const obj = this.facCateCodeShow(
              facCateCode,
              this.options.categoryTree
            )
            this.form = { facImg, ...rest }
            this.form.facCateCode = obj.facCateCode
            this.form.playAttr = Number(this.form.playAttr)
            this.form.openType = openType?.split(',')
            this.form.assetStatus = Number(this.form.assetStatus)
            this.form.building = [
              Number(buildingId),
              Number(floorId),
              Number(spaceId)
            ]
            // if (facImg) {
            //   facImg && (this.fileList = [{ name: 'img', url: facImg }])
            //   this.showUpload = false
            // getImgUrl(facImg).then((responseData) => {
            //   const myBlob = new window.Blob([responseData], {type: 'image/jpeg'})
            //   const Url = window.URL.createObjectURL(myBlob)
            //   this.$set(this.form, 'showFacImg', Url)
            // })
            // }
            // 查询是否有设备类型模板
            await this.getDeviceFormConfig(facCateCode)
            // 查询设备型号
            this.getFormNodesByKey(data.modelName)
          })
      },
      async down ({ fid, name }) {
        const file = await objArr.download(fid)
        download(file, name, this)
      },
      // 根据关键字查询设备型号
      querySearchAsync (queryString, cb) {
        clearTimeout(this.timeout)
        this.timeout = setTimeout(() => {
          const params = {
            head: { current: 1, size: 100 },
            body: { formKeyLike: queryString }
          }
          getFormrelationPage(params).then(({ records }) => {
            if (records.length) {
              this.getFormNodesByKey()
            } else if (
              records.length === 1 &&
              records[0].formKey === queryString
            ) {
              this.getFormNodesByKey(records[0].formKey)
            }
            cb(records)
          })
        }, 333)
      },
      // 根据选择设备类型查询配置数据
      async getFormNodesByKey (formKey) {
        if (formKey) {
          const formConfigurationVos = await getFormNodesConfig({ formKey })
          this.isOtherType = null
          const valueArr = formConfigurationVos
          this.setFormConfig(this.formConfigurationVos, valueArr)
        } else {
          if (this.isOtherType !== 'other') {
            this.isOtherType = 'other'
            this.paramsData = {}
            this.setFormConfig(this.formConfigurationVos)
          }
        }
      },
      // 选中设备型号
      handleSelect (item) {
        const { formKey } = item
        this.getFormNodesByKey(formKey)
      },
      /**
       * @description: 设置配置
       * @param {*} dataArr 配置模板参数的数组
       * @param {*} valueArr 设备类型有默认值的数据数组
       * @return {*}
       */
      async setFormConfig (dataArr, valueArr) {
        const { config, data, uploadRefArr } = this.parseFormConfig(
          dataArr,
          valueArr
        )
        this.parmasFormConfig = config
        this.paramsData = data
        this.uploadRefArr = uploadRefArr
      },
      parseFormConfig (res, valueArr) {
        this.nodesConfig = res
        const config = []
        const span = 2
        for (let i = 0; i < span; i++) {
          config.push({
            index: i,
            span: 24 / span,
            list: []
          })
        }
        const data = {}
        const uploadRefArr = []
        res.forEach((item, index) => {
          // 自定义字段
          const { fieldName, fieldKey, fieldType } = item
          const customConfig = {
            label: this.$t(fieldName),
            prop: fieldKey,
            clearable: true,
            disabled: this.isOtherType !== 'other'
          }
          let defaultValue = null
          if (valueArr) {
            valueArr.forEach(subitem => {
              if (subitem.fieldKey === fieldKey) {
                defaultValue = subitem.defaultValue
              }
            })
          }
  
          if (defaultValue) data[fieldKey] = defaultValue
          switch (fieldType) {
            case 'textarea':
              customConfig.tag = 'el-input'
              customConfig.type = 'textarea'
              customConfig.maxlength = 200
              break
            case 'radio':
              customConfig.tag = 'radio'
              customConfig.options = item.contentSettings
                .split(',')
                .map(subItem => {
                  return { label: subItem, value: subItem }
                })
              break
            case 'checkbox':
              customConfig.tag = 'checkbox'
              customConfig.options = item.contentSettings
                .split(',')
                .map(subItem => {
                  return { label: subItem, value: subItem }
                })
              data[fieldKey] =
                this.isOtherType === 'other'
                  ? []
                  : defaultValue
                  ? defaultValue.split(',')
                  : []
              break
            case 'datepicker':
              customConfig.tag = 'el-date-picker'
              customConfig.type = 'date'
              customConfig.editable = false
              customConfig.format = 'yyyy-MM-dd'
              customConfig.valueFormat = 'yyyy-MM-dd'
              break
            case 'fileUpload':
              if (defaultValue) data[fieldKey] = JSON.parse(defaultValue)
              uploadRefArr.push(fieldKey)
              customConfig.tag =
                this.isOtherType === 'other' ? 'fileUpload' : 'fileList'
              break
            default:
              // 设置默认的控件为输入框
              customConfig.tag = 'el-input'
              customConfig.maxlength = 40
              break
          }
          config[index % span].list.push(customConfig)
        })
        return { config, data, uploadRefArr }
      },
  
      // isInverterType 是逆变器类型
      setDeviceInfoItems (isInverterType) {
        this.deviceInfoItems = [
          {
            label: '所属项目',
            prop: 'projectId',
            tag: 'el-select',
            loading: true,
            options: []
          },
          {
            label: '所属场站',
            prop: 'stationId',
            tag: 'el-select',
            options: []
          },
          {
            label: '设备名称',
            prop: 'facName',
            maxlength: 50
          },
          {
            label: '设备类型',
            prop: 'facCateCode',
            tag: 'el-cascader',
            optionsCode: 'categoryTree',
            ref: 'facCateCodeRef',
            props: {
              checkStrictly: true,
              label: 'facCateName',
              value: 'facCateCode'
            }
          },
          {
            label: '设备物联ID',
            prop: 'facLinkCode',
            maxlength: 100
          },
          {
            label: '设备型号',
            prop: 'modelName',
            tag: isInverterType ? 'el-autocomplete' : 'el-input',
            valueKey: 'formKey',
            value: 'formKey',
            placeholder: isInverterType ? '可输入关键字查询' : '',
            maxlength: 50
          },
          {
            label: '设备品牌',
            prop: 'brandName',
            maxlength: 50
          },
          {
            label: '生产厂家',
            prop: 'manufacturer',
            maxlength: 50
          },
          {
            label: '规格',
            prop: 'specification',
            maxlength: 50
          },
          {
            label: '尺寸信息(高X宽X深mm)',
            prop: 'sizeInfomation',
            maxlength: 50
          },
          {
            label: '重量(kg)',
            prop: 'weight',
            maxlength: 50
          }
        ].map((item, index) => {
          const key = [
            'device_name',
            'device_type',
            'device_play_attr',
            'device_number',
            'spatial_location',
            'asset_status',
            'opening_mode',
            'device_iot_id',
            'bim_component_id',
            'use_period',
            'importance',
            'device_brand',
            'manufacturer',
            'device_model',
            'original_code',
            'date_of_manufacture',
            'date_of_first_activation',
            'repair_deadline'
          ]
          if (!item.tag) {
            Object.assign(item, { tag: 'el-input', clearable: true })
          } else {
            Object.assign(item, { clearable: true })
          }
          return item
        })
      },
      // 设备类型级联回显
      facCateCodeShow (facCateCode, list) {
        if (facCateCode === null) {
          return null
        }
        const echoTreeArr = []
        const eachAry = [facCateCode]
        const itemAry = [] // 分类树组件，每一项的id数组
        const idArr = []
        // 递归分类数据
        const recursionCategory = list => {
          const len = list.length
          for (let i = 0; i < len; i++) {
            // 循环list参数，匹配回显的facCateCode
            itemAry.push(list[i].facCateCode) // 构建分类树数组项,入栈
            idArr.push(list[i].id) // 构建分类树数组项,入栈
            for (let j = 0; j < eachAry.length; j++) {
              // 遍历子节点分类id，拼凑成数组项facCateCode，并终止循环
              if (eachAry[j] === list[i].facCateCode) {
                // 匹配到子节点facCateCode
                echoTreeArr.push({
                  facCateCode: JSON.parse(JSON.stringify(itemAry)),
                  id: JSON.parse(JSON.stringify(idArr))
                }) // push进树分类数据
                eachAry.splice(j, 1) // 删除以匹配到的facCateCode
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
        return echoTreeArr[0]
      }
    },
    components: { customUpload },
    props: {
      id: {
        type: [Number, String],
        default: 0
      },
      tagType: {
        type: String
      }
    },
    computed: {
      facCateCodeIsInformationPublishingSystem () {
        return (
          (Array.isArray(this.form.facCateCode) &&
            this.form.facCateCode[0] === 'information_publishing_system') ||
          (this.id &&
            this.form.systemCode === 'information_publishing_system' &&
            typeof this.form.facCateCode === 'string')
        )
      },
      uploadHeader () {
        const token = localStorage.getItem('token')
        const userId = sessionStorage.getItem('userId')
        const propertyId = localStorage.getItem('propertyId')
        const loginType = sessionStorage.getItem('uis')
        const userName = sessionStorage.getItem('userName')
        const type = sessionStorage.getItem('type')
        if (loginType) {
          return {
            Authorization: JSON.stringify({
              token,
              userId,
              propertyId,
              loginType,
              userName,
              type
            })
          }
        } else {
          return {
            Authorization: JSON.stringify({
              token,
              userId,
              propertyId,
              userName,
              type
            })
          }
        }
      }
    },
    watch: {
      form: {
        deep: true,
        // immediate: true,
        handler (newVal, oldVal) {
          const projectId = this.form.projectId
          if (projectId) {
            this.deviceInfoItems.find(
              item => item.prop === 'stationId'
            ).disabled = false
            const id = this.deviceInfoItems
              .find(item => item.prop === 'projectId')
              .options.find(i => i.value === projectId).id
            getStationAccountListAll({ projectId: id }).then(res => {
              this.$set(
                this.deviceInfoItems.find(item => item.prop === 'stationId'),
                'options',
                res.map(({ id, stationId, stationName }) => {
                  return {
                    label: stationName,
                    value: stationId,
                    id
                  }
                })
              )
            })
          } else {
            this.deviceInfoItems.find(
              item => item.prop === 'stationId'
            ).disabled = true
          }
        }
      }
    },
    created () {
      // 初始化基础信息
      this.setDeviceInfoItems(false)
      if (!this.$route.params.id) this.mode === 'add'
      console.log('addDevice', this.$route.params)
      findCategoryTree().then(data => {
        console.log(this.parseTreeNode(data))
        this.$set(this.options, 'categoryTree', this.parseTreeNode(data))
        this.getDetail()
      })
      getBuildingFloorTree().then(data => {
        this.$set(this.options, 'buildingFloorTree', data)
      })
    },
    mounted () {
      if (!this.$route.params.id) {
        this.$set(
          this.deviceInfoItems.find(item => item.prop === 'stationId'),
          'disabled',
          true
        )
      }
      getProjectAccountList()
        .then(res => {
          const config = this.deviceInfoItems.find(
            item => item.prop === 'projectId'
          )
          config.loading = false
          config.options = res.map(({ id, projectId, projectName }) => {
            return {
              label: projectName,
              value: projectId,
              id
            }
          })
        })
        .catch(err => console.dir(err))
    },
    data () {
      const validatePurchasePrice = (rule, value, callback) => {
        if (value) {
          if (!/^([1-9]\d*)(\.\d{1,9})?$|^0\.\d{1,2}?$/.test(value)) {
            callback(new Error(this.$t('pc.device_ledger.note2')))
          } else {
            callback()
          }
        } else {
          callback()
        }
      }
      const validateServiceLife = (rule, value, callback) => {
        if (value) {
          if (!/^([1-9]\d*)(\.\d{1,9})?$|^0\.\d{1,2}?$/.test(value)) {
            callback(new Error(this.$t('pc.device_ledger.note3')))
          } else {
            callback()
          }
        } else {
          callback()
        }
      }
      const validateSupplierPhone = (rule, value, callback) => {
        if (value) {
          if (!/^[0-9]*$/.test(value)) {
            callback(new Error(this.$t('pc.device_ledger.note4')))
          } else {
            callback()
          }
        }
      }
      const validatePlayAttr = (rule, value, callback) => {
        if (this.facCateCodeIsInformationPublishingSystem && !value) {
          return callback(
            new Error(
              this.$t('pc.global.please_select', {
                name: this.$t('pc.play_equipment.playback_properties')
              })
            )
          )
        }
        callback()
      }
      return {
        options: {},
        form: {},
        bimFormdata: {},
        mode: this.$route.params.id ? 'edit' : 'add',
        activeName: 'baseInfo',
        showUpload: true,
        form: {
          // 资产状态: 1
          // src: require('../../assets/imgs/defaultImg.png')
          showFacImg: ''
        },
        fileList: [],
        rules: {
          facName: [
            {
              required: true,
              message: this.$t('pc.global.please_input', {
                name: this.$t('pc.device_ledger.device_name')
              })
            }
          ],
          projectId: [
            {
              required: true,
              message: '请选择所属项目',
              trigger: 'blur'
            }
          ],
          stationId: [
            {
              required: true,
              message: '请选择所属场站',
              trigger: 'blur'
            }
          ],
          facCateCode: [
            {
              required: true,
              message: this.$t('pc.global.please_select', {
                name: this.$t('pc.device_ledger.device_type')
              })
            }
          ],
          playAttr: [
            { required: true, validator: validatePlayAttr, trigger: 'change' }
          ],
          facCode: [
            {
              required: true,
              message: this.$t('pc.global.please_input', {
                name: this.$t('pc.device_ledger.device_number')
              })
            }
          ],
          building: [
            {
              required: true,
              message: this.$t('pc.global.please_select', {
                name: this.$t('pc.device_ledger.spatial_location')
              })
            }
          ],
          assetStatus: [
            {
              required: true,
              message: this.$t('pc.global.please_select', {
                name: this.$t('pc.device_ledger.asset_status')
              })
            }
          ],
          purchasePrice: [{ validator: validatePurchasePrice, trigger: 'blur' }],
          serviceLife: [{ validator: validateServiceLife, trigger: 'blur' }],
          supplierPhone: [{ validator: validateSupplierPhone, trigger: 'blur' }]
        },
        deviceInfoItems: [],
        // settingItems自定义初始数据
        // settingItems: [
        //   // {label:'设备ip',prop:'facIp'},
        //   // {label:'端口号',prop:'facPort'},
        //   // {label:'设备账号',prop:'facLOginName'},
        //   // {label:'设备密码',prop:'facLoginPwd'}
        //   {
        //     label: '设备绑定摄像头',
        //     prop: 'facCamId',
        //     tag: 'el-cascader',
        //     placeholder: '请选择设备绑定摄像头',
        //     showAllLevels: false,
        //     props: {
        //       checkStrictly: false,
        //       label: 'name',
        //       value: 'id',
        //       multiple: true
        //     },
        //     tootip: '用于三维中设备详情页的视频查看',
        //     options: []
        //   },
        //   {
        //     label: '室外机空调绑定',
        //     prop: 'airconditionLink',
        //     showAllLevels: false,
        //     props: {
        //       checkStrictly: false,
        //       label: 'name',
        //       value: 'id',
        //       multiple: true
        //     },
        //     tootip: '用于三位中空调室内外机的联动',
        //     options: []
        //   },
        //   {
        //     label: '电梯绑定梯控设备',
        //     prop: 'elevatorLinkId',
        //     tag: 'el-cascader',
        //     placeholder: '请选择电梯绑定梯控设备',
        //     showAllLevels: true,
        //     props: {
        //       checkStrictly: false,
        //       label: 'name',
        //       value: 'id'
        //     },
        //     tootip: '用于三维中电梯内梯控设备的绑定',
        //     options: []
        //   },
        //   {
        //     label: '配电柜绑定抽屉号',
        //     prop: 'powerLinkId',
        //     tag: 'el-cascader',
        //     placeholder: '请选择配电柜绑定抽屉号',
        //     showAllLevels: false,
        //     props: {
        //       checkStrictly: false,
        //       label: 'name',
        //       value: 'id',
        //       multiple: true
        //     },
        //     tootip: '用于三维中抽屉柜的绑定联动展示',
        //     options: []
        //   },
        //   {
        //     label: '三维联动快照',
        //     prop: 'bimSnapshot',
        //     placeholder: '请输入三维联动快照',
        //     tootip: '用于三维中点击设备列表后的设备位置的定位跳转',
        //     maxlength: 50
        //   }
        // ].map(item => {
        //   if (!item.tag) {
        //     Object.assign(item, { tag: 'el-input', clearable: true })
        //   }
        //   item.clearable = true
        //   return item
        // }),
        procurementInfoItems: [
          {
            label: '效率(%)',
            prop: 'effi'
          },
          {
            label: '额定输入电压(V)',
            prop: 'ratedInVoltage'
          },
          {
            label: '直流输入电压(V)',
            prop: 'dcInputVoltage'
          },
          {
            label: '输入电压频率(Hz)',
            prop: 'inputVoltageFreq'
          },
          {
            label: 'MPPT输入电流(A)',
            prop: 'inputCurrent'
          },
          {
            label: '并网等级',
            prop: 'gridGrade'
          },
          {
            label: '短路电流(A)',
            prop: 'shortCircuitCurrent'
          },
          {
            label: '冷却方式(A)',
            prop: 'modeCooling'
          },
          {
            label: '启动电压(V)',
            prop: 'startVoltage'
          },
          {
            label: '防护等级',
            prop: 'classProtection'
          },
          {
            label: 'MPPT电压范围(V)',
            prop: 'rangeVoltage'
          },
          {
            label: '夜间自耗电',
            prop: 'electricityNight'
          },
          {
            label: 'MPPT数量',
            prop: 'mpptNum'
          },
          {
            label: '噪音',
            prop: 'noise'
          }
        ].map((item, index) => {
          // const key = [
          //   'supplier_name',
          //   'contract_number',
          //   'contact',
          //   'contact_information',
          //   'purchase_price'
          // ]
          // if (index < key.length) {
          //   item.label = this.$t(`pc.device_ledger.${key[index]}`)
          // }
          if (!item.tag) {
            Object.assign(item, { tag: 'el-input', clearable: true })
          } else {
            Object.assign(item, { clearable: true })
          }
          return item
        }),
        // 参数配置相关
        isInverterType: false, // 是其他类型
        isOtherType: null, // 是其他类型
        parmasFormConfig: [],
        paramsData: {},
        accept:
          'dwg，psd，zip，rar，pdf，doc，rp，ppt，vsdx，pptx，docx，xls，xlsx，jpg，png，jpeg，gif'
      }
    }
  }
  </script>
  
  <style lang="scss" scoped>
  .addDevice {
    position: relative;
    .footer {
      margin: 0;
      height: 200px;
    }
  }
  
  .customCard {
    background-color: #fff;
    padding: 11px 24px 24px;
    .uoloadBox {
      margin-top: 40px;
      width: 148px;
      overflow: hidden;
      white-space: nowrap;
      .no-show-upload /deep/ {
        .el-upload--picture-card {
          display: none;
        }
      }
      /deep/ img {
        width: 180px;
        height: 180px;
      }
    }
    .deviceInfo {
      margin-top: 38px;
    }
    .el-form /deep/ {
      flex: 1;
      padding-left: 2px;
      padding-right: 36px;
      .el-input,
      .el-select,
      .el-autocomplete,
      .el-cascader {
        width: 100%;
        line-height: 33px;
      }
      .el-select .el-input,
      .el-cascader .el-input {
        width: 100%;
      }
    }
  }
  </style>
  