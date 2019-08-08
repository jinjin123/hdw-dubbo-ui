<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
             label-width="120px">
      <el-form-item label="企业名称" prop="enterpriseName">
        <el-input v-model="dataForm.enterpriseName" placeholder="企业名称"></el-input>
      </el-form-item>
      <el-form-item label="企业ID前缀" prop="prefix">
        <el-input v-model="dataForm.prefix" placeholder="企业id前缀"></el-input>
      </el-form-item>
      <el-form-item label="企业注册码" prop="businessLicenseNumber">
        <el-input v-model="dataForm.businessLicenseNumber" placeholder="企业注册码(工商注册码-三证合一)"></el-input>
      </el-form-item>
      <el-form-item label="企业编号" prop="enterpriseCode">
        <el-input v-model="dataForm.enterpriseCode" placeholder="企业编号"></el-input>
      </el-form-item>
      <el-form-item label="所属行业" prop="industryCode">
        <el-select v-model="dataForm.industryCode" placeholder="请选择">
          <el-option
            v-for="item in industryList"
            :key="item.id"
            :label="item.name"
            :value="item.id">
            <span style="float: left">{{ item.name }}</span>
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="所属区域" prop="areaCode">
        <el-select v-model="dataForm.areaCode" placeholder="请选择">
          <el-option
            v-for="item in areaList"
            :key="item.id"
            :label="item.name"
            :value="item.id">
            <span style="float: left">{{ item.name }}</span>
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="企业类型)" prop="enterpriseType">
        <el-radio-group v-model="dataForm.enterpriseType">
          <el-radio :label="0">国企</el-radio>
          <el-radio :label="1">民企</el-radio>
          <el-radio :label="2">私企</el-radio>
          <el-radio :label="3">外企</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="企业联系电话" prop="telephone">
        <el-input v-model="dataForm.telephone" placeholder="企业联系电话"></el-input>
      </el-form-item>
      <el-form-item label="企业邮箱" prop="email">
        <el-input v-model="dataForm.email" placeholder="企业邮箱"></el-input>
      </el-form-item>
      <el-form-item label="邮政编码" prop="zipCode">
        <el-input v-model="dataForm.zipCode" placeholder="邮政编码"></el-input>
      </el-form-item>
      <el-form-item label="法人" prop="legalPerson">
        <el-input v-model="dataForm.legalPerson" placeholder="法人"></el-input>
      </el-form-item>
      <el-form-item label="企业负责人姓名" prop="mainPerson">
        <el-input v-model="dataForm.mainPerson" placeholder="企业负责人姓名"></el-input>
      </el-form-item>
      <el-form-item label="企业负责人移动电话号码" prop="mainPersonMobile">
        <el-input v-model="dataForm.mainPersonMobile" placeholder="企业负责人移动电话号码"></el-input>
      </el-form-item>
      <el-form-item label="企业负责人固定电话号码" prop="mainPersonTelephone">
        <el-input v-model="dataForm.mainPersonTelephone" placeholder="企业负责人固定电话号码"></el-input>
      </el-form-item>
      <el-form-item label="企业安全负责人姓名" prop="safePerson">
        <el-input v-model="dataForm.safePerson" placeholder="企业安全负责人姓名"></el-input>
      </el-form-item>
      <el-form-item label="企业安全负责人移动电话号码" prop="safePersonMobile">
        <el-input v-model="dataForm.safePersonMobile" placeholder="企业安全负责人移动电话号码"></el-input>
      </el-form-item>
      <el-form-item label="企业安全负责人固定电话号码" prop="safePersonTelephone">
        <el-input v-model="dataForm.safePersonTelephone" placeholder="企业安全负责人固定电话号码"></el-input>
      </el-form-item>
      <el-form-item label="x坐标" prop="mapX">
        <el-input v-model="dataForm.mapX" placeholder="x坐标"></el-input>
      </el-form-item>
      <el-form-item label="y坐标" prop="mapY">
        <el-input v-model="dataForm.mapY" placeholder="y坐标"></el-input>
      </el-form-item>
      <el-form-item label="z坐标" prop="mapZ">
        <el-input v-model="dataForm.mapZ" placeholder="z坐标"></el-input>
      </el-form-item>
      <el-form-item label="地址" prop="address">
        <el-input v-model="dataForm.address" placeholder="地址"></el-input>
      </el-form-item>
      <el-form-item label="数据是否同步" prop="isSync">
        <el-radio-group v-model="dataForm.isSync">
          <el-radio :label="0">是</el-radio>
          <el-radio :label="1">否</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="企业状态" prop="status">
        <el-radio-group v-model="dataForm.status">
          <el-radio :label="0">正常</el-radio>
          <el-radio :label="1">禁用</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="企业附件" prop="">
        <el-upload
          ref="upload"
          class="upload-recordDetail"
          :action="uploadUrl"
          name="file"
          :auto-upload="false"
          multiple
          :file-list="fileList"
          :on-remove="handleRemove"
          :on-preview="handlePreview"
          :on-change="handleChange">
          <el-button slot="trigger" size="small" type="primary">点击上传</el-button>
          <el-button style="margin-left: 10px;" size="small" type="success" @click="submitUpload">上传到服务器</el-button>
          <div slot="tip" class="el-upload__tip">只能上传不超过10M大小的文件</div>
        </el-upload>
        <el-carousel :interval="4000" type="card" height="200px" v-if="carouselVisible"
                     style="border:1px solid #fff">
          <el-carousel-item v-for="item in imgList" :key="item.name">
            <h3>{{ item.name }}</h3>
            <img :src="item.url" alt="picture" width="100%" height="80%">
          </el-carousel-item>
        </el-carousel>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
    export default {
      data () {
        return {
          visible: false,
          dataForm: {
            id: 0,
            prefix: '',
            businessLicenseNumber: '',
            enterpriseCode: '',
            enterpriseName: '',
            industryCode: '',
            areaCode: '',
            enterpriseType: 0,
            telephone: '',
            email: '',
            zipCode: '',
            legalPerson: '',
            mainPerson: '',
            mainPersonMobile: '',
            mainPersonTelephone: '',
            safePerson: '',
            safePersonMobile: '',
            safePersonTelephone: '',
            mapX: '',
            mapY: '',
            mapZ: '',
            address: '',
            isTrans: '',
            isSync: 0,
            status: 0
          },
          industryList: [],
          areaList: [],
          uploadUrl: '',
          fileList: [],
          imgList: [],
          carouselVisible: false,
          dataRule: {
            prefix: [
                        {required: true, message: '企业id前缀不能为空', trigger: 'blur'}
            ],
            businessLicenseNumber: [
                        {required: true, message: '企业注册码(工商注册码-三证合一)不能为空', trigger: 'blur'}
            ],
            enterpriseCode: [
                        {required: true, message: '企业编号不能为空', trigger: 'blur'}
            ],
            enterpriseName: [
                        {required: true, message: '企业名称不能为空', trigger: 'blur'}
            ],
            industryCode: [
                        {required: true, message: '所属行业不能为空', trigger: 'blur'}
            ],
            areaCode: [
                        {required: true, message: '所属区域不能为空', trigger: 'blur'}
            ],
            enterpriseType: [
                        {required: true, message: '企业类型不能为空', trigger: 'blur'}
            ],
            telephone: [
                        {required: true, message: '企业联系电话不能为空', trigger: 'blur'}
            ],
            email: [
                        {required: true, message: '企业邮箱不能为空', trigger: 'blur'}
            ],
            zipCode: [
                        {required: true, message: '邮政编码不能为空', trigger: 'blur'}
            ],
            legalPerson: [
                        {required: true, message: '法人不能为空', trigger: 'blur'}
            ],
            mainPerson: [
                        {required: true, message: '企业负责人姓名不能为空', trigger: 'blur'}
            ],
            mainPersonMobile: [
                        {required: true, message: '企业负责人移动电话号码不能为空', trigger: 'blur'}
            ],
            mainPersonTelephone: [
                        {required: true, message: '企业负责人固定电话号码不能为空', trigger: 'blur'}
            ],
            safePerson: [
                        {required: true, message: '企业安全负责人姓名不能为空', trigger: 'blur'}
            ],
            safePersonMobile: [
                        {required: true, message: '企业安全负责人移动电话号码不能为空', trigger: 'blur'}
            ],
            safePersonTelephone: [
                        {required: true, message: '企业安全负责人固定电话号码不能为空', trigger: 'blur'}
            ],
            mapX: [
                        {required: true, message: 'x坐标不能为空', trigger: 'blur'}
            ],
            mapY: [
                        {required: true, message: 'y坐标不能为空', trigger: 'blur'}
            ],
            mapZ: [
                        {required: true, message: 'z坐标不能为空', trigger: 'blur'}
            ],
            address: [
                        {required: true, message: '地址不能为空', trigger: 'blur'}
            ],
            isSync: [
                        {required: true, message: '数据是否同步不能为空', trigger: 'blur'}
            ],
            status: [
                        {required: true, message: '企业状态不能为空', trigger: 'blur'}
            ]
          }
        }
      },
      methods: {
        init (id) {
          this.dataForm.id = id || 0
          this.visible = true
          this.$nextTick(() => {
            this.$refs['dataForm'].resetFields()
            if (this.dataForm.id) {
              this.$http({
                url: this.$http.adornUrl(`/enterprise/info/${this.dataForm.id}`),
                method: 'get',
                params: this.$http.adornParams()
              }).then(({data}) => {
                if (data && data.code === 0) {
                  this.dataForm.prefix = data.enterprise.prefix
                  this.dataForm.businessLicenseNumber = data.enterprise.businessLicenseNumber
                  this.dataForm.enterpriseCode = data.enterprise.enterpriseCode
                  this.dataForm.enterpriseName = data.enterprise.enterpriseName
                  this.dataForm.industryCode = data.enterprise.industryCode
                  this.dataForm.areaCode = data.enterprise.areaCode
                  this.dataForm.enterpriseType = data.enterprise.enterpriseType
                  this.dataForm.telephone = data.enterprise.telephone
                  this.dataForm.email = data.enterprise.email
                  this.dataForm.zipCode = data.enterprise.zipCode
                  this.dataForm.legalPerson = data.enterprise.legalPerson
                  this.dataForm.mainPerson = data.enterprise.mainPerson
                  this.dataForm.mainPersonMobile = data.enterprise.mainPersonMobile
                  this.dataForm.mainPersonTelephone = data.enterprise.mainPersonTelephone
                  this.dataForm.safePerson = data.enterprise.safePerson
                  this.dataForm.safePersonMobile = data.enterprise.safePersonMobile
                  this.dataForm.safePersonTelephone = data.enterprise.safePersonTelephone
                  this.dataForm.mapX = data.enterprise.mapX
                  this.dataForm.mapY = data.enterprise.mapY
                  this.dataForm.mapZ = data.enterprise.mapZ
                  this.dataForm.address = data.enterprise.address
                  this.dataForm.isSync = data.enterprise.isSync
                  this.dataForm.status = data.enterprise.status
                  this.initIndustryList()
                  this.initAreaList()
                }
              })
            }
          })
          this.$nextTick(() => {
            this.$refs['dataForm'].resetFields()
            this.uploadUrl = this.$http.adornUrl(`/enterprise/uploadFile?token=` + this.$cookie.get('token'))
            this.fileList = []
            this.imgList = []
            this.carouselVisible = false
            if (this.dataForm.id) {
              this.$http({
                url: this.$http.adornUrl(`/enterprise/selectFile/${this.dataForm.id}`),
                method: 'get',
                params: this.$http.adornParams()
              }).then(({data}) => {
                if (data && data.code === 0) {
                  var files = data.fileList
                  var x
                  for (x in files) {
                    var localAddr = files[x].attachmentPath
                    var f = {name: files[x].attachmentName, url: localAddr, id: files[x].id}
                    this.fileList.push(f)
                    var fi = {name: files[x].attachmentName, url: localAddr}
                    if (/\.(gif|jpg|jpeg|png|GIF|JPG|PNG)$/.test(files[x].attachmentName)) {
                      this.imgList.push(fi)
                    }
                    if (this.imgList.length > 0) {
                      this.carouselVisible = true
                    }
                  }
                }
              })
            }
          })
        },
        initIndustryList () {
          this.industryList = []
          this.$http({
            url: this.$http.adornUrl('/sys/dic/select/9'),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            this.industryList = data.dicList
          })
        },
        initAreaList () {
          this.areaList = []
          this.$http({
            url: this.$http.adornUrl('/sys/dic/select/16'),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            this.areaList = data.dicList
          })
        },
        submitUpload () {
          this.$refs.upload.submit()
        },
        handleRemove (file, fileList) {
          var id = file.id
          var fileName = file.name
          var url = ''
          if (file.id) {
            url = '/enterprise/deleteFileById?id=' + id
          } else {
            url = '/enterprise/deleteFileByName?fileName=' + fileName
          }
          if (url) {
            this.$http({
              url: this.$http.adornUrl(url),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
          this.handleChange(file, fileList)
        },
        handlePreview (file, fileList) {
          window.open(file.url)
        },
        handleChange (file, fileList) {
          this.imgList = []
          var x
          for (x in fileList) {
            var fi = {name: fileList[x].name, url: fileList[x].url}
            if (/\.(gif|jpg|jpeg|png|GIF|JPG|PNG)$/.test(fileList[x].name)) {
              this.imgList.push(fi)
            }
          }
          if (this.imgList.length > 0) {
            this.carouselVisible = true
          } else {
            this.carouselVisible = false
          }
        },
            // 表单提交
        dataFormSubmit () {
          this.$refs['dataForm'].validate((valid) => {
            if (valid) {
              this.$http({
                url: this.$http.adornUrl(`/enterprise/${!this.dataForm.id ? 'save' : 'update'}`),
                method: 'post',
                data: this.$http.adornData({
                  'id': this.dataForm.id || undefined,
                  'prefix': this.dataForm.prefix,
                  'businessLicenseNumber': this.dataForm.businessLicenseNumber,
                  'enterpriseCode': this.dataForm.enterpriseCode,
                  'enterpriseName': this.dataForm.enterpriseName,
                  'industryCode': this.dataForm.industryCode,
                  'areaCode': this.dataForm.areaCode,
                  'enterpriseType': this.dataForm.enterpriseType,
                  'telephone': this.dataForm.telephone,
                  'email': this.dataForm.email,
                  'zipCode': this.dataForm.zipCode,
                  'legalPerson': this.dataForm.legalPerson,
                  'mainPerson': this.dataForm.mainPerson,
                  'mainPersonMobile': this.dataForm.mainPersonMobile,
                  'mainPersonTelephone': this.dataForm.mainPersonTelephone,
                  'safePerson': this.dataForm.safePerson,
                  'safePersonMobile': this.dataForm.safePersonMobile,
                  'safePersonTelephone': this.dataForm.safePersonTelephone,
                  'mapX': this.dataForm.mapX,
                  'mapY': this.dataForm.mapY,
                  'mapZ': this.dataForm.mapZ,
                  'address': this.dataForm.address,
                  'isSync': this.dataForm.isSync,
                  'status': this.dataForm.status
                })
              }).then(({data}) => {
                if (data && data.code === 0) {
                  this.$message({
                    message: '操作成功',
                    type: 'success',
                    duration: 1500,
                    onClose: () => {
                      this.visible = false
                      this.$emit('refreshDataList')
                    }
                  })
                } else {
                  this.$message.error(data.msg)
                }
              })
            }
          })
        }
      }
    }
</script>
