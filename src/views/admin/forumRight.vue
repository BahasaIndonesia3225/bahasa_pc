<template>
  <div class="forumRight">
    <div class="searchBar">
      <div class="searchForm">
        <el-button size="mini" type="primary" @click="handleSearch">刷新</el-button>
      </div>
      <div class="operateForm"></div>
    </div>
    <el-table
      stripe
      size="mini"
      :data="tableData"
      style="height: 100%">
      <el-table-column type="index" width="50"></el-table-column>
      <el-table-column prop="memberName" label="用户名"></el-table-column>
      <el-table-column prop="banTime" label="封禁时间"></el-table-column>
      <el-table-column prop="reason" label="封禁原因"></el-table-column>
      <el-table-column label="封禁状态">
        <template slot-scope="scope">
          <el-tag type="danger" v-if="scope.row.state === '0'">已禁用</el-tag>
          <el-tag type="success" v-if="scope.row.state === '1'">正常</el-tag>
        </template>
      </el-table-column>
      <el-table-column
        fixed="right"
        align="left"
        label="操作"
        width="160">
        <template #default="scope">
          <el-button
            size="small"
            :disabled="scope.row.state === '1'"
            @click="handleBanComment(scope.row)">
            解禁
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <div class="pageContain">
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="searchParams.page"
        :page-size="searchParams.size"
        :page-sizes="[100, 200, 300, 400]"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      searchParams: {
        page: 0,
        size: 10,
      },
      tableData: [],
      total: 0,
    }
  },
  created() {
    this.initTable();
  },
  methods: {
    initTable() {
      this.$ajax.get(process.env.VUE_APP_SERVER + '/dev-api/business/admin/forum/listBanRecord', { params: this.searchParams }).then((response)=>{
        const { data } = response;
        const { content } = data;
        const { list, total } = content;
        this.tableData = list;
        this.total = total;
      })
    },
    handleSearch() {
      this.initTable();
    },
    handleSizeChange(v) {
      this.searchParams.page = 1;
      this.searchParams.size = v;
      this.initTable();
    },
    handleCurrentChange(v) {
      this.searchParams.page = v;
      this.initTable();
    },
    handleBanComment(data) {
      const { memberId } = data;
      this.$ajax.post(process.env.VUE_APP_SERVER + '/dev-api//business/admin/forum/unban?memberId=' + memberId).then((response)=>{
        this.initTable();
        this.$message({
          message: '更新成功',
          type: 'success',
        })
      })
    }
  }
}
</script>
<style scoped>
div.forumRight {
  height: 100%;
  display: flex;
  flex-direction: column;
  .el-input, .el-select {
    width: 180px;
  }
  div.searchBar {
    height: 36px;
    margin-bottom: 10px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    div.searchForm {
      display: flex;
      justify-content: start;
      align-items: center;
      div.name {
        width: 60px;
        margin-right: 5px;
        font-size: 14px;
        &:not(:first-child) {
          margin-left: 14px;
        }
      }
      .el-button {
        margin-left: 14px;
      }
    }
    div.operateForm {
      display: flex;
      justify-content: start;
      align-items: center;
    }
  }
  .el-table {
    flex: 1;
    margin-bottom: 20px;
  }
  div.imageListContain {
    width: 330px;
    display: flex;
    justify-content: start;
    .el-image {
      width: 50px;
      height: 50px;
      &:not(:last-child) {
        margin-right: 6px;
      }
    }
  }
  .pageContain {
    display: flex;
    justify-content: end;
  }
}
</style>
