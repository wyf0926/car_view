<template>
  <el-dialog
    :title="!dataForm.customerId ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" label-width="100px">
      <el-row :gutter="24">
        <el-col :span="12">
          <el-form-item label="客户姓名" prop="name">
            <el-input v-model="dataForm.name" placeholder="客户姓名"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="客户类型" prop="type">
            <el-select v-model="dataForm.type" placeholder="请选择客户类型" clearable>
              <el-option
                v-for="item in options"
                :key="item.itemValue"
                :label="item.itemText"
                :value="item.itemValue">
              </el-option>
            </el-select>
          </el-form-item>
        </el-col>
      </el-row>
      <el-row :gutter="24">
        <el-col :span="12">
          <el-form-item label="联系方式" prop="phone">
            <el-input v-model="dataForm.phone" placeholder="联系方式"></el-input>
          </el-form-item>
        </el-col>
      </el-row>
      <el-form-item label="地址" prop="address">
        <el-input v-model="dataForm.address" placeholder="地址"></el-input>
      </el-form-item>
      <el-form-item label="银行卡号" prop="cardNo">
        <el-input v-model="dataForm.cardNo" placeholder="银行卡号"></el-input>
      </el-form-item>
      <el-form-item label="银行卡开户行" prop="bank">
        <el-input v-model="dataForm.bank" placeholder="银行卡开户行"></el-input>
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
          customerId: 0,
          name: '',
          type: '',
          phone: '',
          address: '',
          cardNo: '',
          bank: '',
          total: '',
          delState: '',
          createTime: '',
          createUser: '',
          modifyTime: '',
          modifyUser: ''
        },
        options: [],
        dataRule: {
          name: [
            {required: true, message: '客户姓名不能为空', trigger: 'blur'}
          ],
          type: [
            {required: true, message: '客户类型不能为空', trigger: 'blur'}
          ]
        }
      }
    },
    created() {
      this.getOptions()
    },
    methods: {
      // 获取字典选项
      getOptions() {
        this.$http({
          url: this.$http.adornUrl('/sys/sysdictitem/itemlist'),
          method: 'get',
          params: {
            'dictCode': 'customer_type'
          }
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.options = data.dictList
          } else {
            this.options = []
          }
        })
      },
      init(id) {
        this.dataForm.customerId = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.customerId) {
            this.$http({
              url: this.$http.adornUrl(`/business/customer/info/${this.dataForm.customerId}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.name = data.customer.name
                this.dataForm.type = data.customer.type
                this.dataForm.phone = data.customer.phone
                this.dataForm.address = data.customer.address
                this.dataForm.cardNo = data.customer.cardNo
                this.dataForm.bank = data.customer.bank
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
              url: this.$http.adornUrl(`/business/customer/${!this.dataForm.customerId ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'customerId': this.dataForm.customerId || undefined,
                'name': this.dataForm.name,
                'type': this.dataForm.type,
                'phone': this.dataForm.phone,
                'address': this.dataForm.address,
                'cardNo': this.dataForm.cardNo,
                'bank': this.dataForm.bank
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
