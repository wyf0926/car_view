<template>
  <el-dialog
    :title="!dataForm.maItemId ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
             label-width="80px">
      <el-row>
        <el-col :span="11">
          <el-form-item label="维修项目名" prop="itemName" label-width="100px">
            <el-input v-model="dataForm.itemName" placeholder="维修项目名" :disabled="dataForm.maItemId ? true : false"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="13">
          <el-form-item label="参考单价" prop="referPrice" label-width="80px">
            <el-input-number v-model="dataForm.referPrice" placeholder="参考单价" :min="0" :max="10000000000" :precision="2"
                             :step="100"></el-input-number> 元
          </el-form-item>
        </el-col>
      </el-row>
      <el-form-item label="备注" prop="comment" label-width="100px">
        <el-input v-model="dataForm.comment" placeholder="备注"></el-input>
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
    data() {
      return {
        visible: false,
        dataForm: {
          maItemId: 0,
          itemName: '',
          referPrice: '',
          comment: ''
        },
        dataRule: {
          itemName: [
            {required: true, message: '维修项目名不能为空', trigger: 'blur'}
          ]
        }
      }
    },
    methods: {
      init(id) {
        this.dataForm.maItemId = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.maItemId) {
            this.$http({
              url: this.$http.adornUrl(`/business/maitem/info/${this.dataForm.maItemId}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.itemName = data.maItem.itemName
                this.dataForm.referPrice = data.maItem.referPrice
                this.dataForm.comment = data.maItem.comment
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
              url: this.$http.adornUrl(`/business/maitem/${!this.dataForm.maItemId ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'maItemId': this.dataForm.maItemId || undefined,
                'itemName': this.dataForm.itemName,
                'referPrice': this.dataForm.referPrice,
                'comment': this.dataForm.comment
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
