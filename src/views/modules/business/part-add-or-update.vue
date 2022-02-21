<template>
  <el-dialog
    :title="!dataForm.partId ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" label-width="80px">
      <el-row :gutter="24">
        <el-col :span="12">
          <el-form-item label="配件名" prop="name">
            <el-input v-model="dataForm.name" :disabled="dataForm.partId ? true : false" placeholder="配件名"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="单位" prop="unit">
            <el-input v-model="dataForm.unit" placeholder="单位"></el-input>
          </el-form-item>
        </el-col>
      </el-row>
      <el-row :gutter="24">
        <el-col :span="12">
          <el-form-item label="库存数量" prop="amount">
            <el-input-number v-model="dataForm.amount" placeholder="库存数量" :min="0" :max="100000" :step="100"></el-input-number>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="参考单价" prop="unitPrice">
            <el-input-number v-model="dataForm.unitPrice" placeholder="参考单价" :precision="2" :min="0" :max="100000000" :step="100"></el-input-number> 元
          </el-form-item>
        </el-col>
      </el-row>
      <el-form-item label="备注" prop="comment">
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
          partId: 0,
          name: '',
          unit: '',
          amount: '',
          unitPrice: '',
          comment: '',
          delState: '',
          createTime: '',
          createUser: '',
          modifyTime: '',
          modifyUser: ''
        },
        dataRule: {
          name: [
            {required: true, message: '配件名不能为空', trigger: 'blur'}
          ]
        }
      }
    },
    methods: {
      init(id) {
        this.dataForm.partId = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.partId) {
            this.$http({
              url: this.$http.adornUrl(`/business/part/info/${this.dataForm.partId}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.name = data.part.name
                this.dataForm.unit = data.part.unit
                this.dataForm.amount = data.part.amount
                this.dataForm.unitPrice = data.part.unitPrice
                this.dataForm.comment = data.part.comment
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
              url: this.$http.adornUrl(`/business/part/${!this.dataForm.partId ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'partId': this.dataForm.partId || undefined,
                'name': this.dataForm.name,
                'unitPrice': this.dataForm.unitPrice,
                'amount': this.dataForm.amount,
                'price': this.dataForm.price,
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
