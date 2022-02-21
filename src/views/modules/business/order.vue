<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="queryList()">
      <el-form-item label="客户名称" prop="customerName">
        <el-input v-model="dataForm.customerName" placeholder="请输入客户名称" clearable></el-input>
      </el-form-item>
      <el-form-item label="车牌号码" prop="carPlate">
        <el-input v-model="dataForm.carPlate" placeholder="请输入车牌号码" clearable></el-input>
      </el-form-item>
      <el-form-item label="厂牌型号" prop="seriesName">
        <el-input v-model="dataForm.seriesName" placeholder="请输入厂牌型号" clearable></el-input>
      </el-form-item>
      <el-form-item label="车架号" prop="vin">
        <el-input v-model="dataForm.vin" placeholder="请输入车架号" clearable></el-input>
      </el-form-item>
      <el-form-item label="发动机号" prop="engineNo">
        <el-input v-model="dataForm.engineNo" placeholder="请输入发动机号" clearable></el-input>
      </el-form-item>
      <el-form-item label="支付方式" prop="payType">
        <el-select v-model="dataForm.payType" clearable placeholder="请选择支付方式">
          <el-option
            v-for="item in payOptions"
            :key="item.itemValue"
            :label="item.itemText"
            :value="item.itemValue">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="维修类别" prop="category">
        <el-select v-model="dataForm.category" clearable placeholder="请选择维修类别" style="width: 100%">
          <el-option
            v-for="item in maOptions"
            :key="item.itemValue"
            :label="item.itemText"
            :value="item.itemText">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="结算日期" prop="payDateRange">
        <el-date-picker
          v-model="payDateRange"
          type="daterange"
          align="right"
          unlink-panels
          range-separator="至"
          start-placeholder="开始日期"
          end-placeholder="结束日期"
          :picker-options="pickerOptions"
          value-format="yyyy-MM-dd">
        </el-date-picker>
      </el-form-item>
      <el-form-item>
        <el-button @click="resetQueryParam()" type="warning">重置条件</el-button>
        <el-button @click="queryList()">查询</el-button>
        <el-button v-if="isAuth('business:order:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button v-if="isAuth('business:order:delete')" type="danger" @click="deleteHandle()"
                   :disabled="dataListSelections.length <= 0">批量删除
        </el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      :header-cell-style="getRowClass"
      v-loading="dataListLoading"
      stripe
      @selection-change="selectionChangeHandle"
      style="width: 100%;">
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50">
      </el-table-column>
      <el-table-column
        type="index"
        label="序号"
        width="50">
      </el-table-column>
      <el-table-column
        prop="orderNo"
        header-align="center"
        align="center"
        label="维修单号"
        width="200px">
      </el-table-column>
      <el-table-column
        prop="customerName"
        header-align="center"
        align="center"
        label="客户名称"
        width="300px"
        show-overflow-tooltip>
      </el-table-column>
      <el-table-column
        prop="carPlate"
        header-align="center"
        align="center"
        label="车牌号码"
        width="150px">
      </el-table-column>
      <el-table-column
        prop="seriesName"
        header-align="center"
        align="center"
        label="厂牌型号"
        width="150px">
      </el-table-column>
      <el-table-column
        prop="vin"
        header-align="center"
        align="center"
        label="车架号"
        width="200px">
      </el-table-column>
      <el-table-column
        prop="engineNo"
        header-align="center"
        align="center"
        label="发动机号"
        width="150px">
      </el-table-column>
      <el-table-column
        prop="payType"
        header-align="center"
        align="center"
        label="支付方式"
        width="120px">
        <template slot-scope="scope">
          <span style="color: gold" v-if="scope.row.payType == 0"><el-tag type="warning">现金</el-tag></span>
          <span style="color: deepskyblue" v-if="scope.row.payType == 1"><el-tag type="success">银行卡</el-tag></span>
          <span style="color: lime" v-if="scope.row.payType == 2"><el-tag type="info">移动支付</el-tag></span>
        </template>
      </el-table-column>
      <el-table-column
        prop="payDate"
        header-align="center"
        align="center"
        label="结算日期"
        sortable
        width="120px">
      </el-table-column>
      <el-table-column
        prop="totalAmount"
        header-align="center"
        align="center"
        label="总金额"
        sortable
        width="150px">
      </el-table-column>
      <el-table-column
        prop="category"
        header-align="center"
        align="center"
        label="维修类型"
        width="120px">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="200"
        label="操作">
        <template slot-scope="scope">
          <el-button type="text" @click="addOrUpdateHandle(scope.row.orderId)">修改</el-button>
          <el-button type="text" @click="deleteHandle(scope.row.orderId)">删除</el-button>
          <el-button type="text" @click="transferToPrint(scope.row.orderId)">打印预览</el-button>
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
    <order-print-preview v-if="orderPrintVisible" ref="orderPrintPreview"></order-print-preview>
  </div>
</template>

<script>
  import AddOrUpdate from './order-add-or-update'
  import OrderPrintPreview from './order-print-preview'

  export default {
    data() {
      return {
        payOptions: [],
        maOptions: [],
        payDateRange: [],
        dataForm: {
          customerName: '',
          carPlate: '',
          vin: '',
          seriesName: '',
          engineNo: '',
          payType: '',
          category: '',
          beginDate: '',
          endDate: ''
        },
        pickerOptions: {
          shortcuts: [{
            text: '最近一周',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
              picker.$emit('pick', [start, end]);
            }
          }, {
            text: '最近一个月',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 30);
              picker.$emit('pick', [start, end]);
            }
          }, {
            text: '最近三个月',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 90);
              picker.$emit('pick', [start, end]);
            }
          }]
        },
        orderPrintVisible: false,
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false
      }
    },
    components: {
      AddOrUpdate,
      OrderPrintPreview
    },
    activated() {
      this.getDataList()
    },
    mounted() {
      this.getPayOptions()
      this.getMaOptions()
    },
    methods: {
      // 获取维修类别字典
      getMaOptions() {
        this.$http({
          url: this.$http.adornUrl('/sys/sysdictitem/itemlist'),
          method: 'get',
          params: {
            'dictCode': 'ma_type'
          }
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.maOptions = data.dictList
          } else {
            this.maOptions = []
          }
        })
      },
      // 获取结算方式选项
      getPayOptions () {
        this.$http({
          url: this.$http.adornUrl('/sys/sysdictitem/itemlist'),
          method: 'get',
          params: {
            'dictCode': 'pay_type'
          }
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.payOptions = data.dictList
          } else {
            this.payOptions = []
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
        this.$http({
          url: this.$http.adornUrl('/business/order/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'customerName': this.dataForm.customerName,
            'carPlate': this.dataForm.carPlate,
            'vin': this.dataForm.vin,
            'engineNo': this.dataForm.engineNo,
            'payType': this.dataForm.payType,
            'category': this.dataForm.category,
            'beginDate': this.payDateRange[0],
            'endDate': this.payDateRange[1]
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
      transferToPrint (id) {
        this.orderPrintVisible = true
        this.$nextTick(() => {
          this.$refs.orderPrintPreview.init(id)
        })
      },
      resetQueryParam () {
        this.dataForm = {}
        this.payDateRange = []
      },
      // 删除
      deleteHandle(id) {
        var ids = id ? [id] : this.dataListSelections.map(item => {
          return item.orderId
        })
        this.$confirm(`确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/business/order/delete'),
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
