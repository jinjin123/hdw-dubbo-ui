<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="登录名" prop="loginName">
      <el-input v-model="dataForm.loginName" placeholder="登录帐号"></el-input>
      </el-form-item>
      <el-form-item label="用户名" prop="name">
        <el-input v-model="dataForm.name" placeholder="登录帐号"></el-input>
      </el-form-item>
      <el-form-item label="密码" prop="password" :class="{ 'is-required': !dataForm.id }">
        <el-input v-model="dataForm.password" type="password" placeholder="密码"></el-input>
      </el-form-item>
      <el-form-item label="确认密码" prop="comfirmPassword" :class="{ 'is-required': !dataForm.id }">
        <el-input v-model="dataForm.comfirmPassword" type="password" placeholder="确认密码"></el-input>
      </el-form-item>
      <el-form-item label="邮箱" prop="email">
        <el-input v-model="dataForm.email" placeholder="邮箱"></el-input>
      </el-form-item>
      <el-form-item label="手机号" prop="mobile">
        <el-input v-model="dataForm.phone" placeholder="手机号"></el-input>
      </el-form-item>
      <el-form-item label="角色" size="mini" prop="roleIdList">
        <el-checkbox-group v-model="dataForm.roleIdList">
          <el-checkbox v-for="role in roleList" :key="role.id" :label="role.id">{{ role.name }}</el-checkbox>
        </el-checkbox-group>
      </el-form-item>
      <el-form-item label="性别" size="mini" prop="sex">
        <el-radio-group v-model="dataForm.sex">
          <el-radio :label="0">男</el-radio>
          <el-radio :label="1">女</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="状态" size="mini" prop="status">
        <el-radio-group v-model="dataForm.status">
          <el-radio :label="0">正常</el-radio>
          <el-radio :label="1">禁用</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="是否过期" size="mini" prop="expired">
        <el-radio-group v-model="dataForm.expired">
          <el-radio :label="0">是</el-radio>
          <el-radio :label="1">否</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="是否领导" size="mini" prop="isLeader">
        <el-radio-group v-model="dataForm.isLeader">
          <el-radio :label="0">是</el-radio>
          <el-radio :label="1">否</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="用户类别" prop="userType">
        <el-radio-group v-model="dataForm.userType" @change="changeUserTypeHandler">
          <el-radio v-for="(userType, index) in dataForm.typeList" :label="index" :key="index">{{ userType }}</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item v-if="dataForm.userType === 1" label="所属企业" prop="singleEnterpriseName">
        <el-popover
          ref="singleEnterpriseListPopover"
          placement="bottom-start"
          trigger="click">
          <el-tree
            :data="singleEnterpriseList"
            :props="singleEnterpriseListTreeProps"
            node-key="id"
            ref="singleEnterpriseListTree"
            @current-change="singleEnterpriseListTreeCurrentChangeHandle"
            :default-expand-all="false"
            :highlight-current="true"
            :expand-on-click-node="false">
          </el-tree>
        </el-popover>
        <el-input v-model="dataForm.singleEnterpriseName" v-popover:singleEnterpriseListPopover :readonly="true"
                  placeholder="点击选择所属企业" class="menu-list__input"></el-input>
      </el-form-item>
      <el-form-item v-if="dataForm.userType === 1" label="所属部门" prop="departmentName">
        <el-popover
          ref="deptListPopover"
          placement="bottom-start"
          trigger="click">
          <el-tree
            :data="deptList"
            :props="deptListTreeProps"
            node-key="id"
            ref="deptListTree"
            @current-change="deptListTreeCurrentChangeHandle"
            :default-expand-all="false"
            :highlight-current="true"
            :expand-on-click-node="false">
          </el-tree>
        </el-popover>
        <el-input v-model="dataForm.departmentName" v-popover:deptListPopover :readonly="true" placeholder="点击选择所属部门"
                  class="menu-list__input"></el-input>
      </el-form-item>
      <el-form-item v-if="dataForm.userType === 1" label="所属职务" prop="jobName">
        <el-popover
          ref="jobListPopover"
          placement="bottom-start"
          trigger="click">
          <el-tree
            :data="jobList"
            :props="jobListTreeProps"
            node-key="id"
            ref="jobListTree"
            @current-change="jobListTreeCurrentChangeHandle"
            :default-expand-all="false"
            :highlight-current="true"
            :expand-on-click-node="false">
          </el-tree>
        </el-popover>
        <el-input v-model="dataForm.jobName" v-popover:jobListPopover :readonly="true" placeholder="点击选择所属职务"
                  class="menu-list__input"></el-input>
      </el-form-item>
      <el-form-item size="mini" v-if="dataForm.userType === 2" label="企业授权">
        <el-tree
          :data="unitList"
          :props="unitListTreeProps"
          node-key="id"
          ref="unitListTree"
          :default-expand-all="false"
          show-checkbox>
        </el-tree>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  import { isEmail, isMobile } from '@/utils/validate'
  import {treeDataTranslate} from '@/utils'
  export default {
    data () {
      var validatePassword = (rule, value, callback) => {
        if (!this.dataForm.id && !/\S/.test(value)) {
          callback(new Error('密码不能为空'))
        } else {
          callback()
        }
      }
      var validateComfirmPassword = (rule, value, callback) => {
        if (!this.dataForm.id && !/\S/.test(value)) {
          callback(new Error('确认密码不能为空'))
        } else if (this.dataForm.password !== value) {
          callback(new Error('确认密码与密码输入不一致'))
        } else {
          callback()
        }
      }
      var validateEmail = (rule, value, callback) => {
        if (!isEmail(value)) {
          callback(new Error('邮箱格式错误'))
        } else {
          callback()
        }
      }
      var validateMobile = (rule, value, callback) => {
        if (!isMobile(value)) {
          callback(new Error('手机号格式错误'))
        } else {
          callback()
        }
      }
      return {
        visible: false,
        roleList: [],
        dataForm: {
          id: 0,
          loginName: '',
          name: '',
          password: '',
          comfirmPassword: '',
          sex: 0,
          email: '',
          mobile: '',
          roleIdList: [],
          status: 1,
          expired: 1,
          isLeader: 1,
          userType: 1,
          typeList: ['超级用户', '企业用户', '监管用户'],
          enterpriseId: '',
          departmentId: '',
          jobId: '',
          enterpriseName: '',
          singleEnterpriseName: '',
          departmentName: '',
          jobName: '',
          enterpriseNodeList: []
        },
        // 监管用户企业授权
        unitList: [],
        unitListTreeProps: {
          label: 'label',
          children: 'children'
        },
        // 企业用户绑定企业
        singleEnterpriseList: [],
        singleEnterpriseListTreeProps: {
          label: 'name',
          children: 'children'
        },
        // 企业用户绑定企业部门
        deptList: [],
        deptListTreeProps: {
          label: 'name',
          children: 'children'
        },
        // 企业用户绑定企业职务
        jobList: [],
        jobListTreeProps: {
          label: 'name',
          children: 'children'
        },
        tempKey: -666666, // 临时key, 用于解决tree半选中状态项不能传给后台接口问题. # 待优化
        dataRule: {
          loginName: [
            { required: true, message: '登录名不能为空', trigger: 'blur' }
          ],
          name: [
            { required: true, message: '用户名不能为空', trigger: 'blur' }
          ],
          password: [
            { validator: validatePassword, trigger: 'blur' }
          ],
          comfirmPassword: [
            { validator: validateComfirmPassword, trigger: 'blur' }
          ],
          email: [
            { required: true, message: '邮箱不能为空', trigger: 'blur' },
            { validator: validateEmail, trigger: 'blur' }
          ],
          phone: [
            { required: true, message: '手机号不能为空', trigger: 'blur' },
            { validator: validateMobile, trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id, enterpriseId, departmentId) {
        this.dataForm.id = id || 0
        this.dataForm.enterpriseId = enterpriseId || 0
        this.dataForm.departmentId = departmentId || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          this.$http({
            url: this.$http.adornUrl('/sys/role/select'),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            this.roleList = data && data.code === 0 ? data.list : []
          }).then(() => {
            // 企业用户绑定企业
            return this.$http({
              url: this.$http.adornUrl('/enterprise/selectTreeByAreaCode'),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              this.singleEnterpriseList = treeDataTranslate(data.list, 'id', 'parentId')
            })
          }).then(() => {
            // 企业用户绑定部门
            return this.$http({
              url: this.$http.adornUrl(`/enterprise/enterpriseDepartment/select/${this.dataForm.enterpriseId}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              this.deptList = treeDataTranslate(data.deptList, 'id', 'parentId')
            })
          }).then(() => {
            // 企业用户绑定职务
            return this.$http({
              url: this.$http.adornUrl(`/enterprise/enterpriseJob/select/${this.dataForm.departmentId}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              this.jobList = treeDataTranslate(data.jobList, 'id')
            })
          }).then(() => {
            // 监管用户企业授权
            if (this.dataForm.userType === 2) {
              this.$http({
                url: this.$http.adornUrl('/enterprise/selectTreeByAreaCode'),
                method: 'get',
                params: this.$http.adornParams()
              }).then(({data}) => {
                this.unitList = treeDataTranslate(data.list, 'id', 'parentId')
              }).then(() => {
                this.visible = true
                this.$nextTick(() => {
                  this.$refs.unitListTree.setCheckedKeys([])
                })
              })
            }
          }).then(() => {
            if (this.dataForm.id) {
              this.$http({
                url: this.$http.adornUrl(`/sys/user/info/${this.dataForm.id}`),
                method: 'get',
                params: this.$http.adornParams()
              }).then(({data}) => {
                if (data && data.code === 0) {
                  this.dataForm.loginName = data.user.loginName
                  this.dataForm.name = data.user.name
                  this.dataForm.sex = data.user.sex
                  this.dataForm.email = data.user.email
                  this.dataForm.phone = data.user.phone
                  this.dataForm.roleIdList = data.user.roleIdList
                  this.dataForm.status = data.user.status
                  this.dataForm.expired = data.user.expired
                  this.dataForm.userType = data.user.userType
                  this.dataForm.isLeader = data.user.isLeader
                  this.dataForm.enterpriseId = data.user.enterpriseId
                  this.dataForm.departmentId = data.user.departmentId
                  this.dataForm.jobId = data.user.jobId
                  if (this.dataForm.userType === 2) {
                    this.dataForm.enterpriseNodeList = data.user.enterpriseNodeList
                    this.initUnitList()
                  }
                  if (this.dataForm.userType === 1) {
                    this.singleEnterpriseListTreeSetCurrentNode()
                    this.deptListTreeSetCurrentNode()
                    this.jobListTreeSetCurrentNode()
                  }
                }
              })
            }
          })
        })
      },
      initUnitList () {
        // 监管用户企业授权
        return this.$http({
          url: this.$http.adornUrl('/enterprise/selectTreeNodeByAreaCode'),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          this.unitList = data.list
          console.log(this.unitList)
        }).then(() => {
          this.visible = true
          this.$nextTick(() => {
            this.$refs.unitListTree.setCheckedNodes(this.dataForm.enterpriseNodeList)
          })
        })
      },
      // 企业用户企业树选中
      singleEnterpriseListTreeCurrentChangeHandle (data, node) {
        this.dataForm.enterpriseId = data.id
        this.dataForm.singleEnterpriseName = data.name
        // 联动企业部门
        this.chageDeptList(this.dataForm.enterpriseId)
        // 联动企业职务
        this.chageJobList(this.dataForm.enterpriseId)
      },
      // 企业用户企业树设置当前选中节点
      singleEnterpriseListTreeSetCurrentNode () {
        this.$refs.singleEnterpriseListTree.setCurrentKey(this.dataForm.enterpriseId)
        this.dataForm.singleEnterpriseName = (this.$refs.singleEnterpriseListTree.getCurrentNode() || {})['name']
      },
      // 企业用户企业部门树选中
      deptListTreeCurrentChangeHandle (data, node) {
        this.dataForm.departmentId = data.id
        this.dataForm.departmentName = data.name
        this.$http({
          url: this.$http.adornUrl(`/enterprise/enterpriseJob/select/` + data.id),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          this.jobList = treeDataTranslate(data.jobList, 'id', 'parentId')
        })
      },
      // 企业用户企业部门树设置当前选中节点
      deptListTreeSetCurrentNode () {
        this.$refs.deptListTree.setCurrentKey(this.dataForm.departmentId)
        this.dataForm.departmentName = (this.$refs.deptListTree.getCurrentNode() || {})['name']
      },
      // 企业用户企业职务树选中
      jobListTreeCurrentChangeHandle (data, node) {
        this.dataForm.jobId = data.id
        this.dataForm.jobName = data.name
      },
      // 企业用户企业职务树设置当前选中节点
      jobListTreeSetCurrentNode () {
        this.$refs.jobListTree.setCurrentKey(this.dataForm.jobId)
        this.dataForm.jobName = (this.$refs.jobListTree.getCurrentNode() || {})['name']
      },
      // 用户类型改变事件
      changeUserTypeHandler (value) {
        if (value === 2) {
          this.$http({
            url: this.$http.adornUrl('/enterprise/selectTreeNodeByAreaCode'),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            this.unitList = data.list
          }).then(() => {
            this.visible = true
            this.$nextTick(() => {
              this.$refs.unitListTree.setCheckedNodes(this.dataForm.enterpriseNodeList)
            })
          })
        }
      },
      chageDeptList (enterpriseId) {
        this.$http({
          url: this.$http.adornUrl(`/enterprise/enterpriseDepartment/select/${enterpriseId}`),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          this.deptList = treeDataTranslate(data.deptList, 'id', 'parentId')
        })
      },
      chageJobList (departmentId) {
        this.$http({
          url: this.$http.adornUrl(`/enterprise/enterpriseJob/select/${departmentId}`),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          this.jobList = treeDataTranslate(data.jobList, 'id', 'parentId')
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/sys/user/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'loginName': this.dataForm.loginName,
                'name': this.dataForm.name,
                'password': this.dataForm.password,
                'salt': this.dataForm.salt,
                'email': this.dataForm.email,
                'phone': this.dataForm.phone,
                'status': this.dataForm.status,
                'roleIdList': this.dataForm.roleIdList,
                'expired': this.dataForm.expired,
                'isLeader': this.dataForm.isLeader,
                'userType': this.dataForm.userType,
                'enterpriseId': this.dataForm.enterpriseId,
                'departmentId': this.dataForm.departmentId,
                'jobId': this.dataForm.jobId,
                'enterpriseIdList': this.dataForm.userType === 2 ? [].concat(this.$refs.unitListTree.getCheckedKeys()) : []
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
