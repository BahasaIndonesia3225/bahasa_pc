<template>
  <div class="notice">
    <el-form
      ref="form"
      :model="form"
      :rules="rules"
      label-width="80px">
      <el-form-item label="公告" prop="remark">
        <el-input
          :rows="6"
          type="textarea"
          v-model="form.remark">
        </el-input>
      </el-form-item>
      <el-form-item>
        <el-button size="small" type="primary" @click="onSubmit">更新</el-button>
        <el-button size="small" @click="initData">重置</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>
<script>
export default {
  data() {
    return {
      form: {
        remark: ""
      },
      rules: {
        remark: [
          { required: true, message: '请输入公告', trigger: 'change' }
        ],
      }
    }
  },
  created() {
    this.initData();
  },
  methods: {
    initData() {
      this.$ajax.get(process.env.VUE_APP_SERVER + '/dev-api/business/admin/notice/all').then((response)=>{
        const { data } = response;
        const { content } = data;
        this.form.remark = content.remark;
      })
    },
    onSubmit() {
      this.$refs['form'].validate((valid) => {
        if (valid) {
          const data = { remark: this.form.remark };
          this.$ajax.post(process.env.VUE_APP_SERVER + '/dev-api/business/admin/notice/save', data).then((response)=>{
            Toast.success("更新成功！");
          })
        }
      });
    }
  }
}
</script>
<style>
div.notice {
  .el-form-item__content {
    height: unset !important;
  }
}
</style>
