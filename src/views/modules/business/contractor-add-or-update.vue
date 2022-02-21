<template>
  <el-dialog
    :title="!dataForm.contractorId ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" label-width="80px">
      <el-row :gutter="24">
        <el-col :span="12">
          <el-form-item label="单位名称" prop="contractorName">
            <el-input v-model="dataForm.contractorName" placeholder="单位名称"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="联系电话" prop="phone">
            <el-input v-model="dataForm.phone" placeholder="联系电话"></el-input>
          </el-form-item>
        </el-col>
      </el-row>
      <el-form-item label="单位地址" prop="address">
        <el-input v-model="dataForm.address" placeholder="单位地址"></el-input>
      </el-form-item>
      <el-form-item label="开户银行" prop="bank">
        <el-input v-model="dataForm.bank" placeholder="开户银行"></el-input>
      </el-form-item>
      <el-form-item label="银行账号" prop="cardNo">
        <el-input v-model="dataForm.cardNo" placeholder="银行账号"></el-input>
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
          contractorId: 0,
          contractorName: '',
          phone: '',
          address: '',
          bank: '',
          cardNo: '',
          delState: '',
          createTime: '',
          createUser: '',
          modifyTime: '',
          modifyUser: ''
        },
        dataRule: {
          contractorName: [
            {required: true, message: '单位名称不能为空', trigger: 'blur'}
          ]
        }
      }
    },
    methods: {
      init(id) {
        this.dataForm.contractorId = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.contractorId) {
            this.$http({
              url: this.$http.adornUrl(`/business/contractor/info/${this.dataForm.contractorId}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.contractorName = data.contractor.contractorName
                this.dataForm.phone = data.contractor.phone
                this.dataForm.address = data.contractor.address
                this.dataForm.bank = data.contractor.bank
                this.dataForm.cardNo = data.contractor.cardNo
                this.dataForm.delState = data.contractor.delState
                this.dataForm.createTime = data.contractor.createTime
                this.dataForm.createUser = data.contractor.createUser
                this.dataForm.modifyTime = data.contractor.modifyTime
                this.dataForm.modifyUser = data.contractor.modifyUser
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
              url: this.$http.adornUrl(`/business/contractor/${!this.dataForm.contractorId ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'contractorId': this.dataForm.contractorId || undefined,
                'contractorName': this.dataForm.contractorName,
                'phone': this.dataForm.phone,
                'address': this.dataForm.address,
                'bank': this.dataForm.bank,
                'cardNo': this.dataForm.cardNo,
                'delState': this.dataForm.delState,
                'createTime': this.dataForm.createTime,
                'createUser': this.dataForm.createUser,
                'modifyTime': this.dataForm.modifyTime,
                'modifyUser': this.dataForm.modifyUser
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
