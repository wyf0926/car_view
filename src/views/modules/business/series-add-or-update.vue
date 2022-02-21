<template>
  <el-dialog
    :title="!dataForm.seriesId ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" label-width="80px">
      <el-row :gutter="24">
        <el-col :span="12">
          <el-form-item label=" 车系名" prop="seriesName">
            <el-input v-model="dataForm.seriesName" placeholder=" 车系名"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="车型" prop="levelName">
            <el-select v-model="dataForm.levelName" placeholder="请选择车型" @change="changeOption" value-key="itemValue">
              <el-option
                v-for="item in options"
                :key="item.itemValue"
                :label="item.itemText"
                :value="item">
              </el-option>
            </el-select>
          </el-form-item>
        </el-col>
      </el-row>
      <el-form-item label="图片地址" prop="seriesImg">
        <el-input v-model="dataForm.seriesImg" placeholder="车系图片地址"></el-input>
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
        options: [],
        dataForm: {
          seriesId: 0,
          seriesName: '',
          levelName: '',
          levelId: '',
          seriesImg: ''
        },
        dataRule: {
          seriesName: [
            {required: true, message: ' 车系名不能为空', trigger: 'blur'}
          ]
        }
      }
    },
    created() {
      this.getOptions()
    },
    methods: {
      init(id) {
        this.dataForm.seriesId = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.seriesId) {
            this.$http({
              url: this.$http.adornUrl(`/business/series/info/${this.dataForm.seriesId}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.seriesName = data.series.seriesName
                this.dataForm.levelName = data.series.levelName
                this.dataForm.seriesImg = data.series.seriesImg
              }
            })
          }
        })
      },
      // 获取字典选项
      getOptions() {
        this.$http({
          url: this.$http.adornUrl('/sys/sysdictitem/itemlist'),
          method: 'get',
          params: {
            'dictCode': 'car_type'
          }
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.options = data.dictList
          } else {
            this.options = []
          }
        })
      },
      changeOption(value) {
        this.dataForm.levelName = value.itemText
        this.dataForm.levelId = value.itemValue
      },
      // 表单提交
      dataFormSubmit() {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/business/series/${!this.dataForm.seriesId ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'seriesId': this.dataForm.seriesId || undefined,
                'seriesName': this.dataForm.seriesName,
                'levelName': this.dataForm.levelName,
                'levelId': this.dataForm.levelId,
                'seriesImg': this.dataForm.seriesImg
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
