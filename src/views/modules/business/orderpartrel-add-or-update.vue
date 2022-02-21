<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
             label-width="80px">
      <el-form-item label="订单id" prop="orderId">
        <el-input v-model="dataForm.orderId" placeholder="订单id"></el-input>
      </el-form-item>
      <el-form-item label="配件id" prop="partId">
        <el-input v-model="dataForm.partId" placeholder="配件id"></el-input>
      </el-form-item>
      <el-form-item label="实际单价" prop="realPrice">
        <el-input v-model="dataForm.realPrice" placeholder="实际单价"></el-input>
      </el-form-item>
      <el-form-item label="使用数量" prop="usedQuantity">
        <el-input v-model="dataForm.usedQuantity" placeholder="使用数量"></el-input>
      </el-form-item>
      <el-form-item label="配件总价" prop="totalAmount">
        <el-input v-model="dataForm.totalAmount" placeholder="配件总价"></el-input>
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
          relId: 0,
          orderId: '',
          partId: '',
          realPrice: '',
          usedQuantity: '',
          totalAmount: ''
        },
        dataRule: {
          orderId: [
            {required: true, message: '订单id不能为空', trigger: 'blur'}
          ],
          partId: [
            {required: true, message: '配件id不能为空', trigger: 'blur'}
          ],
          realPrice: [
            {required: true, message: '实际单价不能为空', trigger: 'blur'}
          ],
          usedQuantity: [
            {required: true, message: '使用数量不能为空', trigger: 'blur'}
          ],
          totalAmount: [
            {required: true, message: '配件总价不能为空', trigger: 'blur'}
          ]
        }
      }
    },
    methods: {
      init(id) {
        this.dataForm.relId = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.relId) {
            this.$http({
              url: this.$http.adornUrl(`/business/orderpartrel/info/${this.dataForm.relId}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.orderId = data.orderPartRel.orderId
                this.dataForm.partId = data.orderPartRel.partId
                this.dataForm.realPrice = data.orderPartRel.realPrice
                this.dataForm.usedQuantity = data.orderPartRel.usedQuantity
                this.dataForm.totalAmount = data.orderPartRel.totalAmount
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
              url: this.$http.adornUrl(`/business/orderpartrel/${!this.dataForm.relId ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'relId': this.dataForm.relId || undefined,
                'orderId': this.dataForm.orderId,
                'partId': this.dataForm.partId,
                'realPrice': this.dataForm.realPrice,
                'usedQuantity': this.dataForm.usedQuantity,
                'totalAmount': this.dataForm.totalAmount
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
