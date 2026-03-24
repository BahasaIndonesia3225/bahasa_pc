<template>
  <div class="forum">
    <div class="searchBar">
      <div class="searchForm">
        <div class="name">审核状态</div>
        <el-select
          style="width: 160px"
          clearable
          v-model="searchParams.state">
          <el-option label="未审批" value="0"/>
          <el-option label="已通过" value="1"/>
          <el-option label="未通过" value="2"/>
        </el-select>
        <el-button size="mini" type="primary" @click="handleSearch">查询</el-button>
      </div>
      <div class="operateForm"></div>
    </div>
    <el-table
      stripe
      size="mini"
      :data="tableData"
      style="height: 100%">
      <el-table-column type="index" width="50"></el-table-column>
      <el-table-column prop="memberName" label="作者"></el-table-column>
      <el-table-column prop="create" label="创建时间"></el-table-column>
      <el-table-column prop="ip" label="IP"></el-table-column>
      <el-table-column label="论坛内容" width="340">
        <template slot-scope="scope">
          <div>
            {{ scope.row.content }}
          </div>
          <div class="imageListContain">
            <el-image
              v-for="(url, index) in scope.row.picUrlList"
              :key="index"
              :src="url"
              :preview-src-list="scope.row.picUrlList"
              fit="fill">
            </el-image>
          </div>
        </template>
      </el-table-column>
      <el-table-column label="状态">
        <template slot-scope="scope">
          <el-tag type="info" v-if="scope.row.state === '0'">未审批</el-tag>
          <el-tag type="success" v-if="scope.row.state === '1'">已通过</el-tag>
          <el-tag type="warning" v-if="scope.row.state === '2'">未通过</el-tag>
        </template>
      </el-table-column>
      <el-table-column
        fixed="right"
        align="left"
        label="操作"
        width="160">
        <template #default="scope">
          <el-button size="small" @click="handleApproveComment(scope.row)">
            审核
          </el-button>
          <el-button size="small" type="danger" @click="handleDeleteComment(scope.row)">
            删除
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
        state: undefined,
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
      this.$ajax.get(process.env.VUE_APP_SERVER + '/dev-api/business/admin/forum/listForum', { params: this.searchParams }).then((response)=>{
        const { data } = response;
        const { content } = data;
        const { list, total } = content;
        this.tableData = list.map(item => {
          let picUrlList = [];
          const { pic } = item;
          if(pic) picUrlList = pic.split(',');
          return {
            ...item,
            picUrlList: picUrlList.map((url, index) => `http://`  + url),
          }
        });
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
    handleApproveComment(data) {
      this.$confirm(
        '同意发布该论坛吗？',
        '审核论坛',
        {
          distinguishCancelAndClose: true,
          confirmButtonText: '同意',
          cancelButtonText: '拒绝',
          type: 'info',
        }
      )
        .then(() => {
          this.handleApproveComment_('1', data)
        })
        .catch((action) => {
          if(action === "cancel") {
            this.handleApproveComment_('2', data)
          }
        })
    },
    handleApproveComment_(state, data) {
      const { id } = data;
      this.$ajax.post(process.env.VUE_APP_SERVER + '/dev-api/business/admin/forum/auditForum', { id, state }).then((response)=>{
        this.initTable();
        this.$message({
          message: '操作成功',
          type: 'success',
        })
      })
    },
    handleDeleteComment(data) {
      const { id } = data;
      this.$ajax.post(process.env.VUE_APP_SERVER + '/dev-api/business/admin/forum/deleteForum?id=' + id).then((response)=>{
        this.initTable();
        this.$message({
          message: '删除成功',
          type: 'success',
        })
      })
    }
  }
}
</script>
<style scoped>
div.forum {
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
