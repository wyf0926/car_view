<template>
  <el-dialog
    :title="!dataForm.itemId ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible"
    width="500px">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" label-width="80px">
      <el-form-item label="车款名" prop="specName">
        <el-input v-model="dataForm.specName" placeholder="车款名"></el-input>
      </el-form-item>
      <el-form-item label="参考价格" prop="referPrice">
        <el-input-number v-model="dataForm.referPrice" :precision="2" :step="5000" placeholder="参考价格" :max="10000000000"
                         :min="0"></el-input-number> 元
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
    props: ['seriesId'],
    data() {
      return {
        visible: false,
        dataForm: {
          itemId: 0,
          seriesId: '',
          specName: '',
          referPrice: '',
        },
        dataRule: {
          specName: [
            {required: true, message: '车款名不能为空', trigger: 'blur'}
          ]
        }
      }
    },
    methods: {
      init(id) {
        this.dataForm.itemId = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.itemId) {
            this.$http({
              url: this.$http.adornUrl(`/business/seriesitem/info/${this.dataForm.itemId}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.specName = data.seriesItem.specName
                this.dataForm.referPrice = data.seriesItem.referPrice
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit() {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/business/seriesitem/${!this.dataForm.itemId ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'seriesId': this.seriesId,
                'itemId': this.dataForm.itemId,
                'specName': this.dataForm.specName,
                'referPrice': this.dataForm.referPrice ? this.dataForm.referPrice : 0
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
                    this.$emit('refreshItemList')
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
