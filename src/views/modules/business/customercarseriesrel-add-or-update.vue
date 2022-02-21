<template>
  <el-dialog
    :title="!dataForm.relId ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible"
    width="500px"
    v-if="visible"
    @close="handleClose">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" label-width="80px">
      <el-form-item label="车系名" prop="seriesName">
        <el-autocomplete
          :disabled="dataForm.relId ? true : false"
          class="inline-input"
          v-model="dataForm.seriesName"
          :fetch-suggestions="querySearchAsync"
          placeholder="请输入内容"
          :trigger-on-focus="false"
          value-key="seriesName"
          @select="handleSelect"
        ></el-autocomplete>
      </el-form-item>
      <el-form-item label="车牌号码" prop="carPlate">
        <el-input v-model="dataForm.carPlate" placeholder="车牌号码"></el-input>
      </el-form-item>
      <el-form-item label="车架号" prop="vin">
        <el-input v-model="dataForm.vin" placeholder="车架号"></el-input>
      </el-form-item>
      <el-form-item label="发动机号" prop="engineNo">
        <el-input v-model="dataForm.engineNo" placeholder="发动机号"></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="handleClose">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    props: ['customerId'],
    data() {
      return {
        visible: false,
        dataForm: {
          relId: 0,
          customerId: '',
          seriesName: '',
          carPlate: '',
          seriesId: '',
          vin: '',
          engineNo: ''
        },
        carListLoading: false,
        cars: [],
        dataRule: {
          seriesName: [
            {required: true, message: '车系名不能为空', trigger: 'blur'}
          ],
          carPlate: [
            {required: true, message: '车牌号码不能为空', trigger: 'blur'},
            {min: 7, max: 8, message: '请输入正确的车牌号'}
          ],
          vin: [
            {min: 17, max: 17, message: '请输入正确的17位车架号'}
          ],
          engineNo: [
            {min: 6, max: 12, message: '请输入正确的发动机号'}
          ]
        }
      }
    },
    created() {
      this.cars = this.loadAll()
    },
    computed: {
      realCarPlate() {
        return this.dataForm.carPlate
      }
    },
    watch: {
      realCarPlate(val) {
        if (val) {
          this.dataForm.carPlate = val.toUpperCase()
        }
      }
    },
    methods: {
      querySearchAsync(queryString, cb) {
        const cars = this.cars
        const results = queryString ? cars.filter(this.createFilter(queryString)) : cars
        cb(results)
      },
      createFilter(queryString) {
        return (cars) => {
          return (cars.seriesName.toLowerCase().indexOf(queryString.toLowerCase()) !== -1)
        }
      },
      handleSelect(item) {
        this.dataForm.seriesId = item.seriesId
      },
      loadAll() {
        this.carListLoading = true
        this.$http({
          url: this.$http.adornUrl('/business/series/listAll'),
          method: 'get'
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.cars = data.list
          } else {
            this.cars = []
          }
          this.carListLoading = false
        })
      },
      handleClose() {
        this.cars = this.loadAll()
        this.visible = false
        this.$refs['dataForm'].resetFields()
        this.dataForm.seriesId = ''
      },
      init(id) {
        this.dataForm.relId = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          this.dataForm.seriesId = ''
          if (this.dataForm.relId) {
            this.$http({
              url: this.$http.adornUrl(`/business/customercarseriesrel/info/${this.dataForm.relId}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.seriesName = data.customerCarVo.seriesName
                this.dataForm.carPlate = data.customerCarVo.carPlate
                this.dataForm.vin = data.customerCarVo.vin
                this.dataForm.engineNo = data.customerCarVo.engineNo
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit() {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            if (!this.dataForm.relId && !this.dataForm.seriesId) {
              this.$confirm(`${this.dataForm.seriesName} 为新车辆,确认同时将添加该车至车库?`, '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
              }).then(() => {
                this.$http({
                  url: this.$http.adornUrl(`/business/customercarseriesrel/${!this.dataForm.relId ? 'save' : 'update'}`),
                  method: 'post',
                  data: this.$http.adornData({
                    'relId': this.dataForm.relId,
                    'customerId': this.customerId,
                    'seriesName': this.dataForm.seriesName,
                    'carPlate': this.dataForm.carPlate,
                    'vin': this.dataForm.vin,
                    'engineNo': this.dataForm.engineNo
                  })
                }).then(({data}) => {
                  if (data && data.code === 0) {
                    this.$message({
                      message: '操作成功',
                      type: 'success',
                      duration: 1500,
                      onClose: () => {
                        this.handleClose()
                        this.$emit('refreshDataList')
                        this.$emit('refreshItemList')
                      }
                    })
                  } else {
                    this.$message.error(data.msg)
                  }
                })
              })
            } else {
              this.$http({
                url: this.$http.adornUrl(`/business/customercarseriesrel/${!this.dataForm.relId ? 'save' : 'update'}`),
                method: 'post',
                data: this.$http.adornData({
                  'relId': this.dataForm.relId,
                  'customerId': this.customerId,
                  'seriesId': this.dataForm.seriesId,
                  'carPlate': this.dataForm.carPlate,
                  'vin': this.dataForm.vin,
                  'engineNo': this.dataForm.engineNo
                })
              }).then(({data}) => {
                if (data && data.code === 0) {
                  this.$message({
                    message: '操作成功',
                    type: 'success',
                    duration: 1500,
                    onClose: () => {
                      this.handleClose()
                      this.$emit('refreshDataList')
                      this.$emit('refreshItemList')
                    }
                  })
                } else {
                  this.$message.error(data.msg)
                }
              })
            }
          }
        })
      }
    }
  }
</script>
