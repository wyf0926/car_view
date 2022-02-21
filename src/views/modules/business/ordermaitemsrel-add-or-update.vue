<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
             label-width="80px">
      <el-form-item label="维修单id" prop="orderId">
        <el-input v-model="dataForm.orderId" placeholder="维修单id"></el-input>
      </el-form-item>
      <el-form-item label="维修项目id" prop="maItemId">
        <el-input v-model="dataForm.maItemId" placeholder="维修项目id"></el-input>
      </el-form-item>
      <el-form-item label="维修工时" prop="manHour">
        <el-input v-model="dataForm.manHour" placeholder="维修工时"></el-input>
      </el-form-item>
      <el-form-item label="总工时费" prop="totalAmount">
        <el-input v-model="dataForm.totalAmount" placeholder="总工时费"></el-input>
      </el-form-item>
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
          relId: 0,
          orderId: '',
          maItemId: '',
          manHour: '',
          totalAmount: '',
          comment: ''
        },
        dataRule: {
          orderId: [
            {required: true, message: '维修单id不能为空', trigger: 'blur'}
          ],
          maItemId: [
            {required: true, message: '维修项目id不能为空', trigger: 'blur'}
          ],
          manHour: [
            {required: true, message: '维修工时不能为空', trigger: 'blur'}
          ],
          totalAmount: [
            {required: true, message: '总工时费不能为空', trigger: 'blur'}
          ],
          comment: [
            {required: true, message: '备注不能为空', trigger: 'blur'}
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
              url: this.$http.adornUrl(`/business/ordermaitemsrel/info/${this.dataForm.relId}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.orderId = data.orderMaItemsRel.orderId
                this.dataForm.maItemId = data.orderMaItemsRel.maItemId
                this.dataForm.manHour = data.orderMaItemsRel.manHour
                this.dataForm.totalAmount = data.orderMaItemsRel.totalAmount
                this.dataForm.comment = data.orderMaItemsRel.comment
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
              url: this.$http.adornUrl(`/business/ordermaitemsrel/${!this.dataForm.relId ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'relId': this.dataForm.relId || undefined,
                'orderId': this.dataForm.orderId,
                'maItemId': this.dataForm.maItemId,
                'manHour': this.dataForm.manHour,
                'totalAmount': this.dataForm.totalAmount,
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
