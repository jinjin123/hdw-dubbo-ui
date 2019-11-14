<template>
  <div>
    <div class="alert-info">
      <i class="el-icon-info" style="color: #1890ff;padding: 0 5px;"></i>共追踪到 {{count}} 条近期HTTP请求记录
      <el-divider direction="vertical"></el-divider>
      <span class="alert-info-click" @click="getDataList">立即更新</span>
    </div>
    <el-table :data="dataList">
      <el-table-column prop="requestTime" label="请求时间">
      </el-table-column>
      <el-table-column label="请求方法">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.method === 'post'" type="success">{{scope.row.method}}</el-tag>
          <el-tag v-if="scope.row.method === 'get'" type="warning">{{scope.row.method}}</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="url"
                       label="请求url">
      </el-table-column>
      <el-table-column label="响应状态">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.status === 200" type="success">{{scope.row.status}}</el-tag>
          <el-tag v-else="" type="warning">{{scope.row.status}}</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="请求耗时">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.useTime < 3000" type="success">{{scope.row.useTime}}ms</el-tag>
          <el-tag v-else="" type="warning">{{scope.row.useTime}}ms</el-tag>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 30, 40]"
      :page-size="pageSize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
  </div>
</template>

<script>
  export default {
    data () {
      return {
        total: 0,
        pageSize: 10,
        pageIndex: 1,
        dataList: [],
        count: ''
      }
    },
    mounted () {
      this.getDataList()
    },
    methods: {
      getDataList () {
        this.dataList = [
          {
            requestTime: '2019-11-11 15:44:56',
            method: 'post',
            url: 'http://boot.jeecg.com/jeecg-boot//sys/login',
            status: 200,
            useTime: '4'
          }, {
            requestTime: '2019-11-11 15:44:56',
            method: 'get',
            url: 'http://boot.jeecg.com/jeecg-boot//sys/permission/getUserPermissionByToken',
            status: 101,
            useTime: '1'
          }
        ]
        this.count = 99
      },
      handleSizeChange (val) {
        this.pageSize = val
        this.getDataList()
      },
      handleCurrentChange (val) {
        this.pageIndex = val
        this.getDataList()
      }
    }
  }
</script>

<style scoped>

</style>
