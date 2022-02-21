<template>
  <el-drawer
    title="字典配置"
    :width="screenWidth"
    :close-on-click-modal="false"
    :visible.sync="visible"
    :modal-append-to-body='false'>
    <div :style="{
          padding:'20px',
          border: '1px solid #e9e9e9',
          background: '#fff',}">
      <el-button style="margin-bottom: 8px" v-if="isAuth('business:sysdict:save')" type="primary"
                 @click="addOrUpdateHandle()">新 增
      </el-button>
      <el-button v-if="isAuth('business:sysdict:delete')" type="danger" @click="deleteHandle()"
                 :disabled="dataListSelections.length <= 0">批量删除
      </el-button>
      <el-table
        :data="dataList"
        stripe
        @selection-change="selectionChangeHandle"
        style="width: 100%;"
        :row-style="{height: '50px'}"
        :cell-style="{padding:'0'}"
        :header-cell-style="getRowClass">
        <el-table-column
          type="selection"
          header-align="center"
          align="center"
          width="50">
        </el-table-column>
        <el-table-column
          prop="itemText"
          header-align="center"
          align="center"
          label="字典项">
        </el-table-column>
        <el-table-column
          prop="itemValue"
          header-align="center"
          align="center"
          label="数据值">
        </el-table-column>
        <el-table-column
          fixed="right"
          header-align="center"
          align="center"
          width="150"
          label="操作">
          <template slot-scope="scope">
            <el-button type="text" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
            <el-button type="text" @click="deleteHandle(scope.row.id)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        @size-change="sizeChangeHandle"
        @current-change="currentChangeHandle"
        :current-page="pageIndex"
        :page-sizes="[10, 20, 50, 100]"
        :page-size="pageSize"
        :total="totalPage"
        layout="total, sizes, prev, pager, next, jumper">
      </el-pagination>
    </div>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"
                   :dictId="dictId"></add-or-update>
  </el-drawer>
</template>

<script>
  import AddOrUpdate from './sysdictitem-add-or-update'

  export default {
    props: ['dictId'],
    data() {
      return {
        dataForm: {
          key: ''
        },
        screenWidth: 800,
        visible: false,
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListSelections: [],
        addOrUpdateVisible: false
      }
    },
    components: {
      AddOrUpdate
    },
    created() {
      // 当页面初始化时,根据屏幕大小来给抽屉设置宽度
      this.resetScreenSize()
    },
    methods: {
      // 设置表头样式
      getRowClass({row, column, rowIndex, columnIndex}) {
        return 'background-color:#45c2b5;color: #fff;font-weight: 500;'
      },
      // 获取数据列表
      getDataList() {
        this.visible = true
        this.$http({
          url: this.$http.adornUrl('/sys/sysdictitem'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'dictId': this.dictId
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.page.list
            this.totalPage = data.page.totalCount
          } else {
            this.dataList = []
            this.totalPage = 0
          }
          this.dataListLoading = false
        })
      },
      // 每页数
      sizeChangeHandle(val) {
        this.pageSize = val
        this.pageIndex = 1
        this.getDataList()
      },
      // 当前页
      currentChangeHandle(val) {
        this.pageIndex = val
        this.getDataList()
      },
      // 多选
      selectionChangeHandle(val) {
        this.dataListSelections = val
      },
      // 新增 / 修改
      addOrUpdateHandle(id) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(id)
        })
      },
      // 删除
      deleteHandle(id) {
        var ids = id ? [id] : this.dataListSelections.map(item => {
          return item.id
        })
        this.$confirm(`确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/sys/sysdictitem/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.getDataList()
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        })
      },
      // 抽屉的宽度随着屏幕大小来改变
      resetScreenSize() {
        let screenWidth = document.body.clientWidth
        if (screenWidth < 600) {
          this.screenWidth = screenWidth
        } else {
          this.screenWidth = 600
        }
      }
    }
  }
</script>

<style>
  #el-drawer__title span {
    font-size: large;
    font-weight: bold;
  }
</style>
