<template>
  <div>
    <p>
      <el-input style="width: 200px; margin-right: 10px;" v-model="mobile" placeholder="请输入内容"></el-input>
      <button v-on:click="list(1)" class="btn btn-white btn-default btn-round" style="margin-right: 10px;">
        检索
      </button>
      <button class="btn btn-white btn-default btn-round" @click="addMember" style="margin-right: 10px;">新增会员</button>
      <button class="btn btn-white btn-default btn-round" @click="exportMember" style="margin-right: 10px;">导出会员</button>
      <span>超级用户不受单点登录的限制、不受习题的限制</span>
    </p>

    <el-dialog
      :title="dialogTitle"
      :visible.sync="dialogVisible"
      :close-on-click-modal="false"
      width="30%">
      <el-form ref="form" :model="form" :rules="rules" label-width="100px">
        <el-form-item label="用户名" prop="mobile">
          <el-input v-model="form.mobile" size="small"></el-input>
        </el-form-item>
        <el-form-item label="昵称" prop="name">
          <el-input v-model="form.name" size="small"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="form.password" size="small"></el-input>
        </el-form-item>
        <el-form-item label="角色" prop="role">
          <el-radio-group v-model="form.role" size="small">
            <el-radio :label="1">普通用户</el-radio>
            <el-radio :label="2">超级用户</el-radio>
          </el-radio-group>
        </el-form-item>
<!--        <el-form-item label="是否支付" prop="payStatus">-->
<!--          <el-radio-group v-model="form.payStatus" size="mini">-->
<!--            <el-radio :label="0">未支付</el-radio>-->
<!--            <el-radio :label="1">已支付</el-radio>-->
<!--          </el-radio-group>-->
<!--        </el-form-item>-->
        <el-form-item label="是否答题" prop="doQuestion">
          <el-radio-group v-model="form.doQuestion" size="mini">
            <el-radio :label="0">关闭</el-radio>
            <el-radio :label="1">开启</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="设备限制" prop="deviceLimitNum">
          <el-input-number
            size="small"
            v-model="form.deviceLimitNum"
            :min="1" :max="10"
          ></el-input-number>
        </el-form-item>
<!--        <el-form-item label="手机号码" prop="phone">-->
<!--          <el-input-->
<!--            readonly-->
<!--            v-model="form.phone"-->
<!--            size="small">-->
<!--          </el-input>-->
<!--        </el-form-item>-->
        <el-form-item label="开通进阶课" prop="userType">
          <el-radio-group v-model="form.userType" size="mini">
            <el-radio :label="0">关闭</el-radio>
            <el-radio :label="1">开启</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="从此节课开始" prop="choice">
          <el-select
            class="mySelect"
            style="width: 100%"
            v-model="form.choice"
            placeholder="请选择">
            <el-option
              v-for="item in chapterOption"
              :key="item.id"
              :label="item.title"
              :value="item.id">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="授权类型" prop="licenseType">
          <el-radio-group v-model="form.licenseType" size="small">
            <el-radio :label="1">个人授权</el-radio>
            <el-radio :label="2">企业授权</el-radio>
            <el-radio :label="3">未知</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="网页登录方式" prop="loginType">
          <el-radio-group v-model="form.loginType" size="small">
            <el-radio :label="1">无限制</el-radio>
            <el-radio :label="2">只能扫码登录PC</el-radio>
            <el-radio :label="3">无法登录</el-radio>
          </el-radio-group>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="onCancel">取 消</el-button>
        <el-button type="primary" @click="onSubmit">确 定</el-button>
      </span>
    </el-dialog>
    <el-table
      stripe
      border
      :data="members"
      style="width: 100%">
      <el-table-column type="index" width="50"></el-table-column>
      <el-table-column prop="id" label="ID"></el-table-column>
      <el-table-column prop="mobile" label="用户名"></el-table-column>
      <el-table-column prop="name" label="昵称"></el-table-column>
      <el-table-column label="手机号码">
        <template slot-scope="scope">
          <span>{{ scope.row.phone || '未绑定' }}</span>
        </template>
      </el-table-column>
      <el-table-column label="角色">
        <template slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.role === 1" type="info">普通用户</el-tag>
          <el-tag size="mini" v-if="scope.row.role === 2">超级用户</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="支付状态">
        <template slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.payStatus === 1">已支付</el-tag>
          <el-tag size="mini" v-else-if="scope.row.payStatus === 0" type="info">未支付</el-tag>
          <el-tag size="mini" v-else type="info">未知</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="需要答题">
        <template slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.doQuestion === 1">是</el-tag>
          <el-tag size="mini" v-else-if="scope.row.doQuestion === 0" type="info">否</el-tag>
          <el-tag size="mini" v-else type="info">未知</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="title" label="已完成习题章节" width="120"></el-table-column>
      <el-table-column label="开通进阶课">
        <template slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.userType === 1">开通</el-tag>
          <el-tag size="mini" v-else-if="scope.row.userType === 0" type="info">未开通</el-tag>
          <el-tag size="mini" v-else type="info">未知</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="deviceLimitNum" label="设备限制"></el-table-column>
      <el-table-column prop="ip" label="最后登录IP" width="140"></el-table-column>
      <el-table-column prop="registerTime" label="注册时间" width="160"></el-table-column>
      <el-table-column label="操作" width="220">
        <template slot-scope="scope">
          <el-button
            @click="toViewingRecord(scope.row)"
            type="text"
            size="small">
            观看记录
          </el-button>
          <el-button
            @click="toCheckDevice(scope.row)"
            type="text"
            size="small">
            登录设备
          </el-button>
          <el-button
            @click="toDeleteUser(scope.row)"
            type="text"
            size="small">
            删除
          </el-button>
          <el-button
            @click="toEditUser(scope.row)"
            type="text"
            size="small">
            编辑
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <div class="pagination-contain">
      <pagination
        ref="pagination"
        v-bind:list="list"
        v-bind:itemCount="8">
      </pagination>
    </div>

    <el-dialog
      title="登录设备信息"
      :visible.sync="deviceDialogVisible"
      width="800px">
      <el-table :data="deviceContent" style="width: 100%">
        <el-table-column prop="id" label="ID"></el-table-column>
        <el-table-column prop="memberId" label="会员ID"></el-table-column>
        <el-table-column prop="deviceId" label="设备ID" width="300"></el-table-column>
        <el-table-column label="设备系统" width="160">
          <template slot-scope="scope">
            <span>{{ deviceTypeOption[scope.row.deviceType] }}</span>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="100">
          <template slot-scope="scope">
            <el-button @click="handleDelete(scope.row)" type="text" size="small">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
      <span slot="footer" class="dialog-footer">
        <el-button @click="deviceDialogVisible = false;">取 消</el-button>
        <el-button type="primary" @click="deviceDialogVisible = false;">保 存</el-button>
      </span>
    </el-dialog>

    <el-dialog
      title="观看记录"
      :visible.sync="viewingRecordDialogVisible"
      width="800px">
      <el-table :data="viewingRecordList" max-height="600" style="width: 100%">
        <el-table-column
          width="500"
          prop="courseName"
          label="课程名称">
        </el-table-column>
        <el-table-column
          prop="creatorTime"
          label="观看时间">
        </el-table-column>
      </el-table>
      <span slot="footer" class="dialog-footer">
        <el-button @click="viewingRecordDialogVisible = false;">取 消</el-button>
        <el-button type="primary" @click="viewingRecordDialogVisible = false;">保 存</el-button>
      </span>
    </el-dialog>

  </div>
</template>

<script>
  import Pagination from "../../components/pagination";
  export default {
    components: {Pagination},
    name: "business-member",
    data: function() {
      return {
        mobile: '',  //用户查询
        member: {},
        members: [], //列表tableData
        dialogVisible: false,
        dialogTitle: "",
        form: {
          id: "",
          mobile: "",        //用户名
          name: "",          //昵称
          password: "",      //密码
          role: 1,           //角色
          payStatus: 0,      //支付状态
          doQuestion: 1,     //是否需要答题
          deviceLimitNum: 2, //设备限制数量
          phone: '',         //手机号码
          userType: 0,      //进阶课
          choice: '',        //从此节课开始
          licenseType: 3,    //授权类型
          loginType: 1,      //登录方式
        },
        rules: {
          mobile: [{ required: true, message: '请输入用户名', trigger: 'blur' }],
          name: [{ required: true, message: '请输入昵称', trigger: 'blur' }],
          password: [{ required: true, message: '请输入密码', trigger: 'blur' }],
        },
        deviceContent: [],
        deviceTypeOption: {
          "1": 'android',
          "2": 'ios',
          "3": 'windows',
          "4": 'h5',
        },
        deviceDialogVisible: false,
        chapterOption: [],

        viewingRecordDialogVisible: false,
        viewingRecordList: [],
      }
    },
    mounted: function() {
      let _this = this;
      _this.$refs.pagination.size = 15;
      _this.list(1);
      _this.getChapterOption()
      // sidebar激活样式方法一
      // this.$parent.activeSidebar("business-member-sidebar");

    },
    methods: {
      getChapterOption() {
        let _this = this;
        _this.$ajax.post(process.env.VUE_APP_SERVER + '/dev-api/business/admin/section/list', {
          "page":1,
          "size":1500,
          "courseId":"TYAIILon",
          "chapterId":"NmmbLYYl"
        }).then((response)=>{
          const { code, content } = response.data;
          _this.chapterOption = content.list;
        })
      },
      /**
       * 列表查询
       */
      list(page) {
        let _this = this;
        Loading.show();
        _this.$ajax.post(process.env.VUE_APP_SERVER + '/dev-api/business/admin/member/list', {
          page: page,
          size: _this.$refs.pagination.size,
          mobile: this.mobile,
        }).then((response)=>{
          Loading.hide();
          let resp = response.data;
          _this.members = resp.content.list;
          _this.$refs.pagination.render(page, resp.content.total);

        })
      },
      exportMember() {
        this.$ajax.post(process.env.VUE_APP_SERVER + '/dev-api/business/admin/member/export', {}, {
          headers: {
            'Content-Type': 'application/json; application/octet-stream'
          },
          responseType: 'arraybuffer'
        }).then((res)=>{
          const aLink = document.createElement('a')
          var blob = new Blob([res.data], { type: "application/vnd.ms-excel;charset=UTF-8" })
          var fileName = "会员信息"
          aLink.href = URL.createObjectURL(blob)
          aLink.setAttribute('download', fileName) // 设置下载文件名称
          document.body.appendChild(aLink)
          aLink.click()
          document.body.appendChild(aLink)
        })
      },
      addMember() {
        this.form = {
          id: "",
          mobile: "",        //用户名
          name: "",          //昵称
          password: "",      //密码
          role: 1,           //角色
          payStatus: 0,      //支付状态
          doQuestion: 1,     //是否需要答题
          deviceLimitNum: 2, //设备限制数量
          phone: '',         //手机号码
          userType: 0,       //进阶课
          choice: '',        //从此节课开始
          licenseType: 3,    //授权类型
          loginType: 1,      //登录方式
        }
        this.dialogVisible = true
        this.dialogTitle = "新增会员"
      },
      onCancel() {
        this.dialogVisible = false;
        this.$refs['form'].resetFields();
      },
      onSubmit() {
        this.$refs['form'].validate((valid) => {
          if (valid) {
            const params = this.form;
            this.$ajax.post(process.env.VUE_APP_SERVER + '/dev-api/business/admin/member/save', params).then((response)=>{
              const {success, message} = response.data;
              if(success) {
                this.list(1);
                this.onCancel();
                this.$message({
                  type: 'success',
                  message: '新增成功'
                });
              }else {
                this.$message({
                  type: 'error',
                  message: message
                });
              }
            })
          } else {
            console.log('error submit!!');
            return false;
          }
        });
      },
      toEditUser(data) {
        const { id, mobile, name, password, role, payStatus, doQuestion, deviceLimitNum, phone, userType, choice, licenseType, loginType } = data;
        this.form = { id, mobile, name, password, role, payStatus, doQuestion, deviceLimitNum, phone, userType, choice, licenseType, loginType };
        this.dialogVisible = true;
        this.dialogTitle = "编辑会员"
      },
      toDeleteUser(data) {
        this.$prompt('确定删除该用户？请输入安全码校验。', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          inputPattern: /^HATI-HATI$/,
          inputErrorMessage: '请输入正确格式的安全校验码！'
        }).then(({ value }) => {
          const { id } = data;
          this.$ajax.delete(process.env.VUE_APP_SERVER + '/dev-api/business/admin/member/delete/' + id).then((response) => {
            this.$refs.pagination.size = 15;
            this.list(1);
            this.$message({
              type: 'success',
              message: '删除成功!'
            });
          })
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          });
        });
      },
      toCheckDevice(data) {
        const { id } = data;
        this.deviceContent = [];
        this.$ajax.get(process.env.VUE_APP_SERVER + '/dev-api/business/admin/member/device-list/' + id).then((response)=>{
          const { data } = response;
          const { content } = data;
          this.deviceContent = content;
          this.deviceDialogVisible = true;
        })
      },
      handleDelete(data) {
        const { id } = data;
        this.deviceContent = this.deviceContent.filter(item => item.id !== id)
        this.$ajax.delete(process.env.VUE_APP_SERVER + '/dev-api/business/admin/member/device-delete/' + id);
      },
      toViewingRecord(data) {
        const { id } = data;
        this.$ajax.get(process.env.VUE_APP_SERVER + '/dev-api/business/admin/member/watchHistory?memberId=' + id).then((response)=>{
          const { data } = response;
          const { content } = data;
          this.viewingRecordList = content;
          this.viewingRecordDialogVisible = true;
        })
      }
    }
  }
</script>
<style>
  .mySelect .el-input--suffix .el-input__inner {
    background: #ffffff !important;
  }
  .pagination-contain {
    margin-top: 20px;
    display: flex;
    justify-content: end;
  }
</style>
