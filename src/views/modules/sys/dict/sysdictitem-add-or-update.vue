<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible"
    :append-to-body="true"
    width="500px">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" label-width="80px">
      <el-form-item label="字典项" prop="itemText">
        <el-input v-model="dataForm.itemText" placeholder="字典项文本"></el-input>
      </el-form-item>
      <el-form-item label="数据值" prop="itemValue">
        <el-input v-model="dataForm.itemValue" placeholder="字典项值"></el-input>
      </el-form-item>
      <el-form-item label="描述" prop="description">
        <el-input v-model="dataForm.description" placeholder="描述" type="textarea"
                  :autosize="{ minRows: 2, maxRows: 4}"></el-input>
      </el-form-item>
      <el-form-item label="排序" prop="sortOrder">
        <el-input-number v-model="dataForm.sortOrder" controls-position="right"></el-input-number>
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
    props: ['dictId'],
    data() {
      return {
        visible: false,
        dataForm: {
          id: 0,
          dictId: '',
          itemText: '',
          itemValue: '',
          description: '',
          sortOrder: 0,
          status: '',
          createBy: '',
          createTime: '',
          updateBy: '',
          updateTime: ''
        },
        dataRule: {
          itemText: [
            {required: true, message: '字典项文本不能为空', trigger: 'blur'}
          ],
          itemValue: [
            {required: true, message: '字典项值不能为空', trigger: 'blur'}
          ]
        }
      }
    },
    methods: {
      init(id) {
        this.dataForm.id = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/sys/sysdictitem/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.dictId = data.sysDictItem.dictId
                this.dataForm.itemText = data.sysDictItem.itemText
                this.dataForm.itemValue = data.sysDictItem.itemValue
                this.dataForm.description = data.sysDictItem.description
                this.dataForm.sortOrder = data.sysDictItem.sortOrder
                this.dataForm.status = data.sysDictItem.status
                this.dataForm.createBy = data.sysDictItem.createBy
                this.dataForm.createTime = data.sysDictItem.createTime
                this.dataForm.updateBy = data.sysDictItem.updateBy
                this.dataForm.updateTime = data.sysDictItem.updateTime
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
              url: this.$http.adornUrl(`/sys/sysdictitem/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'dictId': this.dictId,
                'itemText': this.dataForm.itemText,
                'itemValue': this.dataForm.itemValue,
                'description': this.dataForm.description,
                'sortOrder': this.dataForm.sortOrder
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
