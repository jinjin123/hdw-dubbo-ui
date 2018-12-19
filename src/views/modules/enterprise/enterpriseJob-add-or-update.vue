<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="所属企业" prop="enterpriseName">
        <el-popover
          ref="enterpriseListPopover"
          placement="bottom-start"
          trigger="click">
          <el-tree
            :data="enterpriseList"
            :props="enterpriseListTreeProps"
            node-key="id"
            ref="enterpriseListTree"
            @current-change="enterpriseListTreeCurrentChangeHandle"
            :default-expand-all="false"
            :highlight-current="true"
            :expand-on-click-node="false">
          </el-tree>
        </el-popover>
        <el-input v-model="dataForm.enterpriseName" v-popover:enterpriseListPopover :readonly="true"
                  placeholder="点击选择所属企业" class="menu-list__input"></el-input>
      </el-form-item>
      <el-form-item label="所属部门" prop="departmentName">
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
      <el-form-item label="职务名称" prop="jobName">
        <el-input v-model="dataForm.jobName" placeholder="职务名称"></el-input>
      </el-form-item>
    <el-form-item label="职务代码" prop="jobCode">
      <el-input v-model="dataForm.jobCode" placeholder="职务代码"></el-input>
    </el-form-item>
    <el-form-item label="预留1" prop="parameter1">
      <el-input v-model="dataForm.parameter1" placeholder="预留1"></el-input>
    </el-form-item>
    <el-form-item label="预留2" prop="parameter2">
      <el-input v-model="dataForm.parameter2" placeholder="预留2"></el-input>
    </el-form-item>
    <el-form-item label="是否同步" prop="isSync">
      <el-radio-group v-model="dataForm.isSync">
        <el-radio :label="0">是</el-radio>
        <el-radio :label="1">否</el-radio>
      </el-radio-group>
    </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  import {treeDataTranslate} from '@/utils'
  export default {
    data () {
      return {
        visible: false,
        dataForm: {
          id: 0,
          enterpriseId: '',
          departmentId: '',
          jobCode: '',
          jobName: '',
          createTime: '',
          updateTime: '',
          createUser: '',
          updateUser: '',
          parameter1: '',
          parameter2: '',
          isSync: 0,
          enterpriseName: '',
          departmentName: ''
        },
        enterpriseList: [],
        enterpriseListTreeProps: {
          label: 'name',
          children: 'children'
        },
        deptList: [],
        deptListTreeProps: {
          label: 'name',
          children: 'children'
        },
        dataRule: {
          enterpriseName: [
            { required: true, message: '所属企业不能为空', trigger: 'blur' }
          ],
          departmentName: [
            { required: true, message: '所属部门不能为空', trigger: 'blur' }
          ],
          jobCode: [
            { required: true, message: '职务代码不能为空', trigger: 'blur' }
          ],
          jobName: [
            { required: true, message: '职务名称不能为空', trigger: 'blur' }
          ],
          parameter1: [
            { required: false, message: '预留1不能为空', trigger: 'blur' }
          ],
          parameter2: [
            { required: false, message: '预留2不能为空', trigger: 'blur' }
          ],
          isSync: [
            { required: true, message: '是否同步不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id, enterpriseId) {
        this.dataForm.id = id || 0
        this.dataForm.enterpriseId = enterpriseId || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          this.$http({
            url: this.$http.adornUrl('/enterprise/selectTreeByAreaCode'),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            this.enterpriseList = treeDataTranslate(data.list, 'id', 'parentId')
          }).then(() => {
            this.$http({
              url: this.$http.adornUrl(`/enterprise/enterpriseDepartment/select/${this.dataForm.enterpriseId}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              this.deptList = treeDataTranslate(data.deptList, 'id', 'parentId')
            }).then(() => {
              if (this.dataForm.id) {
                this.$http({
                  url: this.$http.adornUrl(`/enterprise/enterpriseJob/info/${this.dataForm.id}`),
                  method: 'get',
                  params: this.$http.adornParams()
                }).then(({data}) => {
                  if (data && data.code === 0) {
                    this.dataForm.departmentId = data.enterpriseJob.departmentId
                    this.dataForm.jobCode = data.enterpriseJob.jobCode
                    this.dataForm.jobName = data.enterpriseJob.jobName
                    this.dataForm.parameter1 = data.enterpriseJob.parameter1
                    this.dataForm.parameter2 = data.enterpriseJob.parameter2
                    this.dataForm.isSync = data.enterpriseJob.isSync
                    this.enterpriseListTreeSetCurrentNode()
                    this.deptListTreeSetCurrentNode()
                  }
                })
              }
            })
          })
        })
      },
      // 企业树选中
      enterpriseListTreeCurrentChangeHandle (data, node) {
        this.dataForm.enterpriseId = data.id
        this.dataForm.enterpriseName = data.name
        // 企业部门联动
        this.chageDeptList(this.dataForm.enterpriseId)
      },
      // 企业树设置当前选中节点
      enterpriseListTreeSetCurrentNode () {
        this.$refs.enterpriseListTree.setCurrentKey(this.dataForm.enterpriseId)
        this.dataForm.enterpriseName = (this.$refs.enterpriseListTree.getCurrentNode() || {})['name']
      },
      // 企业部门树选中
      deptListTreeCurrentChangeHandle (data, node) {
        this.dataForm.departmentId = data.id
        this.dataForm.departmentName = data.name
      },
      // 企业部门树设置当前选中节点
      deptListTreeSetCurrentNode () {
        this.$refs.deptListTree.setCurrentKey(this.dataForm.departmentId)
        this.dataForm.departmentName = (this.$refs.deptListTree.getCurrentNode() || {})['name']
      },
      chageDeptList (enterpriseId) {
        console.log('联动企业部门：' + enterpriseId)
        this.$http({
          url: this.$http.adornUrl(`/enterprise/enterpriseDepartment/select/${enterpriseId}`),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          console.log(data.deptList)
          this.deptList = treeDataTranslate(data.deptList, 'id', 'parentId')
          console.log(this.deptList)
          this.deptListTreeSetCurrentNode()
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/enterprise/enterpriseJob/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'departmentId': this.dataForm.departmentId,
                'jobCode': this.dataForm.jobCode,
                'jobName': this.dataForm.jobName,
                'parameter1': this.dataForm.parameter1,
                'parameter2': this.dataForm.parameter2,
                'isSync': this.dataForm.isSync
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
