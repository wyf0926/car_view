<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="queryList()">
      <el-form-item label="用户姓名" prop="key">
        <el-input v-model="dataForm.key" placeholder="请输入用户姓名" clearable></el-input>
      </el-form-item>
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
      <el-form-item>
        <el-button @click="queryList()">查询</el-button>
        <el-button v-if="isAuth('business:customer:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button v-if="isAuth('business:customer:delete')" type="danger" @click="deleteHandle()"
                   :disabled="dataListSelections.length <= 0">批量删除
        </el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      :header-cell-style="getRowClass"
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      @expand-change="expandChangeHandle"
      :expand-row-keys="expands"
      :row-key='getRowKeys'
      stripe
      style="width: 100%;">
      <el-table-column
        type="expand">
        <template slot-scope="props">
          <el-button v-show="isAuth('business:customer:delete')&&itemListSelections.length>=2" type="danger"
                     @click="deleteSeriesItemHandle()" :disabled="itemListSelections.length <= 0">批量删除车款
          </el-button>
          <el-table
            :data="itemList"
            v-loading="itemListLoading"
            stripe
            :header-cell-style="{background : 'lightblue' , color: '#fff'}"
            @selection-change="itemSelectionChangeHandle">
            <el-table-column
              type="selection"
              header-align="center"
              align="center"
              width="50">
            </el-table-column>
            <el-table-column
              type="index"
              width="50"
              label="序号">
            </el-table-column>
            <el-table-column
              prop="seriesName"
              header-align="center"
              align="center"
              label="车款名">
            </el-table-column>
            <el-table-column
              prop="carPlate"
              header-align="center"
              align="center"
              label="车牌号">
            </el-table-column>
            <el-table-column
              prop="vin"
              header-align="center"
              align="center"
              label="车架号">
            </el-table-column>
            <el-table-column
              prop="engineNo"
              header-align="center"
              align="center"
              label="发动机号">
            </el-table-column>
            <el-table-column
              fixed="right"
              header-align="center"
              align="center"
              width="200"
              label="操作">
              <template slot-scope="scope">
                <el-button type="text" @click="updateSeriesItemHandle(scope.row.relId)">修改</el-button>
                <el-button type="text" @click="deleteSeriesItemHandle(scope.row.relId)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </template>
      </el-table-column>
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50">
      </el-table-column>
      <el-table-column
        type="index"
        width="50">
      </el-table-column>
      <el-table-column
        prop="name"
        header-align="center"
        align="center"
        label="客户姓名">
      </el-table-column>
      <el-table-column
        prop="type"
        header-align="center"
        align="center"
        label="客户类型">
        <template slot-scope="scope">
          <span style="color: lime" v-if="scope.row.type == 1"><el-tag type="warning">个人</el-tag></span>
          <span style="color: deepskyblue" v-if="scope.row.type == 2"><el-tag type="success">企业</el-tag></span>
        </template>
      </el-table-column>
      <el-table-column
        prop="phone"
        header-align="center"
        align="center"
        label="联系方式">
      </el-table-column>
      <el-table-column
        prop="address"
        header-align="center"
        align="center"
        label="地址">
      </el-table-column>
      <el-table-column
        prop="cardNo"
        header-align="center"
        align="center"
        label="银行卡号">
      </el-table-column>
      <el-table-column
        prop="bank"
        header-align="center"
        align="center"
        label="银行卡开户行">
      </el-table-column>
      <el-table-column
        prop="total"
        header-align="center"
        align="center"
        label="历史消费金额">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="200"
        label="操作">
        <template slot-scope="scope">
          <el-button type="text" @click="addOrUpdateHandle(scope.row.customerId)">修改</el-button>
          <el-button type="text" @click="addSeriesItem(scope.row.customerId)">新增车辆</el-button>
          <el-button type="text" @click="deleteHandle(scope.row.customerId)">删除</el-button>
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
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
    <add-car-item v-if="addSeriesItemVisible" ref="addCarItem" @refreshItemList="refreshItemList"
                  @refreshDataList="getDataList" :customerId="customerId"></add-car-item>
  </div>
</template>

<script>
  import AddOrUpdate from './customer-add-or-update'
  import AddCarItem from './customercarseriesrel-add-or-update'

  export default {
    data() {
      return {
        dataForm: {
          key: '',
          type: ''
        },
        customerId: undefined,
        dataList: [],
        itemList: [],
        expands: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        itemListLoading: false,
        dataListSelections: [],
        itemListSelections: [],
        addOrUpdateVisible: false,
        addSeriesItemVisible: false,
        options: []
      }
    },
    components: {
      AddOrUpdate,
      AddCarItem
    },
    created() {
      this.getOptions()
    },
    activated() {
      this.getDataList()
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
      // 设置表头样式
      getRowClass({row, column, rowIndex, columnIndex}) {
        return 'background-color:#45c2b5;color: #fff;font-weight: 500;'
      },
      // 查询数据
      queryList() {
        this.pageIndex = 1
        this.getDataList()
      },
      // 获取数据列表
      getDataList() {
        this.dataListLoading = true
        if (this.dataForm.key !== '' || this.dataForm.type !== '') {
          this.pageIndex = 1
        }
        this.$http({
          url: this.$http.adornUrl('/business/customer/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'key': this.dataForm.key,
            'type': this.dataForm.type
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
      // 获取数据列表
      getItemList(customerId) {
        this.itemListLoading = true
        this.$http({
          url: this.$http.adornUrl('/business/customercarseriesrel/' + customerId),
          method: 'get'
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.itemList = data.itemList;
          } else {
            this.itemList = []
          }
          this.itemListLoading = false
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
      // 新增车辆
      addSeriesItem(id) {
        this.customerId = id
        this.addSeriesItemVisible = true
        if (this.expands.length != 0) {
          this.expands = []
          this.expands.push(id)
        } else {
          this.expands.push(id)
        }
        this.getItemList(id)
        this.$nextTick(() => {
          this.$refs.addCarItem.init()
        })
      },
      // 车款多选
      itemSelectionChangeHandle(val) {
        this.itemListSelections = val
      },
      // 修改车款
      updateSeriesItemHandle(id) {
        this.addSeriesItemVisible = true
        this.$nextTick(() => {
          this.$refs.addCarItem.init(id)
        })
      },
      // 展开行操作
      expandChangeHandle(row, expandedRows) {
        let that = this
        if (expandedRows.length) {
          that.expands = []
          if (row) {
            that.expands.push(row.customerId)
            that.customerId = row.customerId
            that.getItemList(row.customerId)
          }
        } else {//说明收起了
          that.expands = []
          that.customerId = undefined
        }
      },
      getRowKeys(row) {
        return row.customerId
      },
      // 刷新数据列表
      refreshItemList() {
        this.itemListLoading = true
        this.$http({
          url: this.$http.adornUrl('/business/customercarseriesrel/' + this.customerId),
          method: 'get'
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.itemList = data.itemList;
          } else {
            this.itemList = []
          }
          this.itemListLoading = false
        })
      },
      // 删除
      deleteHandle(id) {
        var ids = id ? [id] : this.dataListSelections.map(item => {
          return item.customerId
        })
        this.$confirm(`确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/business/customer/delete'),
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
      // 删除车款
      deleteSeriesItemHandle(id) {
        var ids = id ? [id] : this.itemListSelections.map(item => {
          return item.relId
        })
        this.$confirm(`确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/business/customercarseriesrel/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.getItemList(this.customerId)
                  this.getDataList()
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        })
      }
    }
  }
</script>
