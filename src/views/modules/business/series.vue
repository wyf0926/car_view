<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="queryList()">
      <el-form-item label="车系名称">
        <el-input v-model="dataForm.seriesName" placeholder="车系名称" clearable></el-input>
      </el-form-item>
      <el-form-item label="车型">
        <el-select v-model="dataForm.carType" multiple filterable clearable placeholder="请选择车型" @change="changeSelectList">
          <el-option
            v-for="item in options"
            :key="item.itemValue"
            :label="item.itemText"
            :value="item.itemText">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button @click="queryList()">查询</el-button>
        <el-button v-if="isAuth('business:series:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button v-if="isAuth('business:series:delete')" type="danger" @click="deleteHandle()"
                   :disabled="dataListSelections.length <= 0">批量删除车系
        </el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      @expand-change="expandChangeHandle"
      :expand-row-keys="expands"
      :row-key='getRowKeys'
      style="width: 100%;"
      stripe
      :header-cell-style="getRowClass">
      <el-table-column
        type="expand">
        <template slot-scope="props">
          <el-button v-show="isAuth('business:series:delete')&&itemListSelections.length>=2" type="danger"
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
              prop="specName"
              header-align="center"
              align="center"
              label="车款名">
            </el-table-column>
            <el-table-column
              prop="referPrice"
              header-align="center"
              align="center"
              sortable
              label="参考价格">
            </el-table-column>
            <el-table-column
              fixed="right"
              header-align="center"
              align="center"
              width="200"
              label="操作">
              <template slot-scope="scope">
                <el-button type="text" @click="updateSeriesItemHandle(scope.row.itemId)">修改</el-button>
                <el-button type="text" @click="deleteSeriesItemHandle(scope.row.itemId)">删除</el-button>
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
        prop="seriesName"
        header-align="center"
        align="center"
        label=" 车系名">
      </el-table-column>
      <el-table-column
        prop="levelName"
        header-align="center"
        align="center"
        label="车型">
      </el-table-column>
      <el-table-column
        prop="seriesFctMaxPrice"
        header-align="center"
        align="center"
        sortable
        label="车系最高价格">
      </el-table-column>
      <el-table-column
        prop="seriesFctMinPrice"
        header-align="center"
        align="center"
        sortable
        label="车系最低价格">
      </el-table-column>
      <el-table-column
        header-align="center"
        align="center"
        label="图片">
        <template slot-scope="scope">
          <!--          　<img :src="scope.row.seriesImg" :preview-src-list="scope.row.seriesImg" width="100" height="80" onerror="this.src='../../../../static/img/errorpic.jpeg'"/>-->
          <el-image style="width: 100px; height: 100px" :src="scope.row.seriesImg"
                    :preview-src-list="[scope.row.seriesImg]" fit="contain"
                    :error="require('@/assets/img/errorpic.jpeg')">
            <div slot="error" class="image-slot">
              <el-image style="width: 100px; height: 100px" :src="require('@/assets/img/errorpic.jpeg')"
                        :preview-src-list="[require('@/assets/img/errorpic.jpeg')]" fit="contain"/>
            </div>
          </el-image>
        </template>
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="200"
        label="操作">
        <template slot-scope="scope">
          <el-button type="text" @click="addOrUpdateHandle(scope.row.seriesId)">修改</el-button>
          <el-button type="text" @click="addSeriesItem(scope.row.seriesId)">新增车款</el-button>
          <el-button type="text" @click="deleteHandle(scope.row.seriesId)">删除</el-button>
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
    <add-series-item v-if="addSeriesItemVisible" ref="addSeriesItem" @refreshItemList="refreshItemList"
                     @refreshDataList="getDataList" :seriesId="seriesId"></add-series-item>
  </div>
</template>

<script>
  import AddOrUpdate from './series-add-or-update'
  import AddSeriesItem from './seriesitem-add-or-update'

  export default {
    data() {
      return {
        dataForm: {
          key: '',
          carType: []
        },
        seriesId: undefined,
        selection: '',
        options: [],
        expands: [],
        dataList: [],
        itemList: [],
        previewImg: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        itemListLoading: false,
        dataListSelections: [],
        itemListSelections: [],
        addOrUpdateVisible: false,
        addSeriesItemVisible: false
      }
    },
    components: {
      AddOrUpdate,
      AddSeriesItem
    },
    created() {
      this.getOptions()
    },
    activated() {
      this.getDataList()
    },
    methods: {
      changeSelectList(value) {
        this.selection = value.join(',')
      },
      // 查询数据
      queryList() {
        this.pageIndex = 1
        console.log(this.pageIndex);
        this.getDataList()
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
      // 设置表头样式
      getRowClass({row, column, rowIndex, columnIndex}) {
        return 'background-color:#45c2b5;color: #fff;font-weight: 500;'
      },
      // 获取数据列表
      getDataList() {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/business/series/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'seriesName': this.dataForm.seriesName,
            'carType': this.selection
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
      getItemList(seriesId) {
        this.itemListLoading = true
        this.$http({
          url: this.$http.adornUrl('/business/seriesitem/' + seriesId),
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
      // 刷新数据列表
      refreshItemList() {
        this.itemListLoading = true
        this.$http({
          url: this.$http.adornUrl('/business/seriesitem/' + this.seriesId),
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
      // 展开行操作
      expandChangeHandle(row, expandedRows) {
        let that = this
        if (expandedRows.length) {
          that.expands = []
          if (row) {
            that.expands.push(row.seriesId)
            that.seriesId = row.seriesId
            that.getItemList(row.seriesId)
          }
        } else {//说明收起了
          that.expands = []
          that.seriesId = undefined
        }
      },
      getRowKeys(row) {
        return row.seriesId
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
      // 车款多选
      itemSelectionChangeHandle(val) {
        this.itemListSelections = val
      },
      // 新增 / 修改
      addOrUpdateHandle(id) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(id)
        })
      },
      // 新增车款
      addSeriesItem(id) {
        this.seriesId = id
        this.addSeriesItemVisible = true
        if (this.expands.length != 0) {
          this.expands = []
          this.expands.push(id)
        } else {
          this.expands.push(id)
        }
        this.getItemList(id)
        this.$nextTick(() => {
          this.$refs.addSeriesItem.init()
        })
      },
      // 修改车款
      updateSeriesItemHandle(id) {
        this.addSeriesItemVisible = true
        this.$nextTick(() => {
          this.$refs.addSeriesItem.init(id)
        })
      },
      // 删除车款
      deleteSeriesItemHandle(id) {
        var ids = id ? [id] : this.itemListSelections.map(item => {
          return item.itemId
        })
        this.$confirm(`确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/business/seriesitem/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.getItemList(this.seriesId)
                  this.getDataList()
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        })
      },
      // 删除
      deleteHandle(id) {
        var ids = id ? [id] : this.dataListSelections.map(item => {
          return item.seriesId
        })
        this.$confirm(`确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/business/series/delete'),
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
      }
    }
  }
</script>
