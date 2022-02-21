<template>
  <el-dialog
    :title="!dataForm.orderId ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible"
    width="70%"
    v-if="visible"
    @close="handleClose">
    <h1 id="form_title">
      江苏省机动车维修费用结算清单
    </h1>
    <el-row type="flex" justify="end">
      <div>维修单号:<span id="orderNo">{{this.dataForm.orderNo}}</span></div>
    </el-row>
    <el-form :model="dataForm" ref="dataForm" label-width="80px">
      <el-collapse v-model="activeNames">
        <el-collapse-item title="托修方" name="1">
          <el-row type="flex" justify="space-around">
            <el-col :span="12">
            <el-form-item label="单位名称(车主姓名)" prop="customerName" label-width="150px" :rules="dataRule.customerName">
                <el-select
                  clearable
                  v-model="dataForm.customerName"
                  filterable
                  remote
                  reserve-keyword
                  placeholder="请输入客户名称"
                  :remote-method="queryCustomers"
                  :loading="customerLoading"
                  value-key="customerId"
                  @change="handleCustomerSelect"
                  style="width: 100%;"
                  @clear="handleSelectClear">
                  <el-option
                    v-for="item in customerOptions"
                    :key="item.customerId"
                    :label="item.name"
                    :value="item">
                  </el-option>
                </el-select>
              </el-form-item>
            </el-col>
            <el-col :span="6">
              <el-form-item label="联系电话" prop="phone" label-width="80px">
                <el-input v-model="dataForm.phone" style="width: 100%;"></el-input>
              </el-form-item>
            </el-col>
            <el-col :span="6">
              <el-form-item label="送修人" prop="driver" label-width="80px">
                <el-input v-model="dataForm.driver"  style="width: 100%;"></el-input>
              </el-form-item>
            </el-col>
          </el-row>
          <el-row type="flex" justify="space-around">
            <el-col :span="6">
              <el-form-item label="车牌号码" prop="carPlate" label-width="100px" :rules="dataRule.carPlate">
                <el-select v-model="dataForm.carPlate" filterable clearable placeholder="请选择车牌号" style="width: 100%" value-key="relId" @change="handleCarSelect">
                  <el-option
                    v-for="item in carPlateOptions"
                    :key="item.relId"
                    :label="`${item.carPlate}\xa0\xa0\xa0\xa0${item.seriesName}`"
                    :value="item">
                  </el-option>
                </el-select>
              </el-form-item>
            </el-col>
            <el-col :span="6">
              <el-form-item label="厂牌型号" prop="seriesName" label-width="100px">
                <el-input v-model="dataForm.seriesName" style="width: 100%;"></el-input>
              </el-form-item>
            </el-col>
            <el-col :span="6">
              <el-form-item label="车架号" prop="vin" label-width="80px" :rules="dataRule.vin">
                <el-input v-model="dataForm.vin" style="width: 100%;"></el-input>
              </el-form-item>
            </el-col>
            <el-col :span="6">
              <el-form-item label="发动机号" prop="engineNo" label-width="80px" :rules="dataRule.engineNo">
                <el-input v-model="dataForm.engineNo" style="width: 100%;"></el-input>
              </el-form-item>
            </el-col>
          </el-row>
          <el-row type="flex">
            <el-col>
              <el-form-item label="维修类别" prop="category" label-width="100px" :rules="dataRule.category">
                <el-select v-model="dataForm.category" clearable placeholder="请选择维修类别" style="width: 100%">
                  <el-option
                    v-for="item in maOptions"
                    :key="item.itemValue"
                    :label="item.itemText"
                    :value="item.itemText">
                  </el-option>
                </el-select>
              </el-form-item>
            </el-col>
            <el-col>
              <el-form-item label="进厂日期" prop="incomingDate" label-width="100px">
                <el-date-picker
                  style="width: 100%"
                  v-model="dataForm.incomingDate"
                  align="right"
                  type="date"
                  placeholder="请选择进厂日期"
                  value-format="yyyy-MM-dd"
                  :picker-options="pickerOptions">
                </el-date-picker>
              </el-form-item>
            </el-col>
            <el-col :offset="1">
              <el-form-item label="出厂里程表读数(公里)" prop="mileageAfter" label-width="150px">
                <el-input-number :min="0" :max="100000000000" :step="1000" style="width: 200px"
                                 v-model="dataForm.mileageAfter"></el-input-number>
              </el-form-item>
            </el-col>
          </el-row>
        </el-collapse-item>
        <el-collapse-item title="承修方" name="2">
          <el-col :span="12">
            <el-form-item label="单位名称" prop="contractorId" label-width="100px" :rules="dataRule.contractorId">
              <el-select v-model="dataForm.contractorId" clearable placeholder="请选择承修单位名称" @change="handleSelectContractor">
                <el-option
                  v-for="item in contractors"
                  :key="item.contractorId"
                  :label="item.contractorName"
                  :value="item.contractorId">
                </el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="联系电话" prop="contractor" label-width="100px">
              <el-input disabled v-model="contractor.phone"></el-input>
            </el-form-item>
          </el-col>
          <el-col>
            <el-form-item label="单位地址" prop="contractor" label-width="100px">
              <el-input disabled v-model="contractor.address"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="开户银行" prop="contractor" label-width="100px">
              <el-input disabled v-model="contractor.bank"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="账号" prop="contractor" label-width="100px">
              <el-input disabled v-model="contractor.cardNo"></el-input>
            </el-form-item>
          </el-col>
        </el-collapse-item>
        <el-collapse-item title="工时费" name="3">
          <el-table
            :data="dataForm.maItemList"
            :header-row-style="{height: '30px'}"
            :header-cell-style="{padding:'0'}"
            :row-style="{height: '30px'}"
            :cell-style="{padding:'0'}"
            height="90"
            border>
            <el-table-column
              label="序号"
              type="index"
              header-align="center"
              align="center"
              width="50">
            </el-table-column>
            <el-table-column
              label="维修项目名"
              header-align="center"
              align="center"
              prop="maItem"
              width="200">
              <template slot-scope="scope">
                <el-autocomplete
                  size="mini"
                  v-model="scope.row.maItem"
                  :fetch-suggestions="queryMaItemSearchAsync"
                  placeholder="维修项目名"
                  @select="handleSelectMaItem($event, scope.$index)"
                  value-key="itemName"
                  :trigger-on-focus="false"
                  style="width: 160px"
                  ref="maItemInput"
                  popper-class="el-remote-search"
                ></el-autocomplete>
              </template>
            </el-table-column>
            <el-table-column
              size="mini"
              prop="manHour"
              label="时长"
              header-align="center"
              align="center">
              <template slot-scope="scope">
                <el-input-number v-model="scope.row.manHour" size="mini" style="width: 100px;"
                                 type="number" :max="100000" :min="0" :step="0.5" :precision="1"
                                 controls-position="right"></el-input-number>
              </template>
            </el-table-column>
            <el-table-column
              prop="realPrice"
              label="单价"
              header-align="center"
              align="center">
              <template slot-scope="scope">
                <el-input-number v-model="scope.row.realPrice" size="mini" style="width: 120px;"
                                 type="number" :max="100000000" :min="0" :step="100" :precision="2"
                                 controls-position="right"></el-input-number>
              </template>
            </el-table-column>
            <el-table-column
              label="金额"
              header-align="center"
              align="center"
              prop="totalAmount"
              width="100">
              <template slot-scope="scope">
                {{Number(scope.row.manHour * scope.row.realPrice).toFixed(2)}}
              </template>
            </el-table-column>
            <el-table-column
              label="备注"
              header-align="center"
              align="center"
              prop="comment">
              <template slot-scope="scope">
                <el-input v-model="scope.row.comment"
                          size="mini"
                          type="text"
                          maxlength="10"
                          show-word-limit></el-input>
              </template>
            </el-table-column>
            <el-table-column
              label="操作"
              header-align="center"
              align="center"
              width="50px">
              <template slot-scope="scope">
                <el-button size="mini" type="danger" icon="el-icon-delete" circle @click="handleDeleteMaItemRow(scope.$index)"/>
              </template>
            </el-table-column>
          </el-table>
          <el-row>
            <el-button type="primary" style="width: 100%" icon="el-icon-plus" size="mini" @click="addMaItemListRow">新增维修项目</el-button>
          </el-row>
        </el-collapse-item>
        <el-collapse-item title="材料费" name="4">
          <el-form-item label-width="0px" prop="partList" :rules="dataRule.partList">
          <el-table
            :data="dataForm.partList"
            border
            :header-row-style="{height: '30px'}"
            :header-cell-style="{padding:'0'}"
            :row-style="{height: '30px'}"
            :cell-style="{padding:'0'}"
            height="180"
            >
            <el-table-column
              label="序号"
              type="index"
              header-align="center"
              align="center"
              width="50">
            </el-table-column>
            <el-table-column
              label="配件名称"
              header-align="center"
              align="center"
              prop="partName"
              width="200">
              <template slot-scope="scope">
              <el-autocomplete
                size="mini"
                v-model="scope.row.partName"
                :fetch-suggestions="queryPartSearchAsync"
                placeholder="配件名"
                @select="handleSelectPart($event,scope.$index)"
                value-key="name"
                :trigger-on-focus="false"
                ref="partInput"
                popper-class="el-remote-search"
                style="width: 150px"
              ></el-autocomplete>
              </template>
            </el-table-column>
            <el-table-column
              prop="usedQuantity"
              label="数量"
              header-align="center"
              align="center">
              <template slot-scope="scope">
                <el-input-number v-model="scope.row.usedQuantity" size="mini"
                                 type="number" :max="100000000" :min="0" :step="5"
                                 controls-position="right" style="width: 80%;"></el-input-number>
              </template>
            </el-table-column>
            <el-table-column
              prop="partUnit"
              label="单位"
              header-align="center"
              align="center"
              width="80%">
              <template slot-scope="scope">
                <el-input size="mini" v-model="scope.row.partUnit"></el-input>
              </template>
            </el-table-column>
            <el-table-column
              prop="realPrice"
              label="单价"
              header-align="center"
              align="center">
              <template slot-scope="scope">
                <el-input-number v-model="scope.row.realPrice" size="mini"
                                 type="number" :max="100000000" :min="0" :step="100" :precision="2"
                                 controls-position="right" style="width: 100%;"></el-input-number>
              </template>
            </el-table-column>
            <el-table-column
              label="金额"
              header-align="center"
              align="center"
              prop="totalAmount"
              width="100%">
              <template slot-scope="scope">
                {{Number(scope.row.usedQuantity * scope.row.realPrice).toFixed(2)}}
              </template>
            </el-table-column>
            <el-table-column
              label="备注"
              header-align="center"
              align="center"
              prop="comment">
              <template slot-scope="scope">
                <el-input v-model="scope.row.comment"
                          size="mini"
                          type="text"
                          maxlength="10"
                          show-word-limit></el-input>
              </template>
            </el-table-column>
            <el-table-column
              label="操作"
              header-align="center"
              align="center"
              width="50px">
              <template slot-scope="scope">
                <el-button size="mini" type="danger" icon="el-icon-delete" circle @click="handleDeletePartRow(scope.$index)"/>
              </template>
            </el-table-column>
          </el-table>
          <el-row>
            <el-button type="primary" style="width: 100%" icon="el-icon-plus" size="mini" @click="addPartListRow">新增配件</el-button>
          </el-row>
          </el-form-item>
        </el-collapse-item>
        <el-collapse-item title="杂项" name="5">
          <el-row>
            <span style="font-size: 20px; color: #00b7ee">是否有维修人支付费用更换的旧配件</span>
          </el-row>
          <el-row>
            <el-form-item prop="oldPart" :rules="dataRule.oldPart" label-width="0px">
            <el-radio-group v-model="dataForm.oldPart">
              <el-radio border v-for="item in oldPartList" :key="item.itemValue" :label=item.itemValue>{{item.itemText}}</el-radio>
            </el-radio-group>
            </el-form-item>
          </el-row>
            <el-divider></el-divider>
          <el-row>
          <el-col :span="12">
          <el-form-item label="结算日期" prop="payDate" label-width="100px" :rules="dataRule.payDate">
            <el-date-picker
              v-model="dataForm.payDate"
              align="right"
              type="date"
              placeholder="请选择结算日期"
              value-format="yyyy-MM-dd"
              :picker-options="pickerOptions">
            </el-date-picker>
          </el-form-item>
          </el-col>
          <el-col :span="12">
          <el-form-item label="支付方式" prop="payType" label-width="100px" :rules="dataRule.payType">
            <el-select v-model="dataForm.payType" clearable placeholder="请选择支付方式">
              <el-option
                v-for="item in payOptions"
                :key="item.itemValue"
                :label="item.itemText"
                :value="item.itemValue">
              </el-option>
            </el-select>
          </el-form-item>
          </el-col>
          </el-row>
          <el-row>
            <el-form-item label="备注" prop="comment" label-width="100px">
              <el-input v-model="dataForm.comment" style="width: 80%;"></el-input>
            </el-form-item>
          </el-row>
        </el-collapse-item>
      </el-collapse>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="handleClose" >取消</el-button>
<!--      <el-button @click="transferToPrint">打印预览</el-button>-->
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
    <order-print-preview v-if="orderPrintVisible" ref="orderPrintPreview"></order-print-preview>
  </el-dialog>
</template>

<script>
  import OrderPrintPreview from './order-print-preview'

  export default {
    data() {
      return {
        orderPrintVisible: false,
        visible: false,
        activeNames: ['1', '2', '3', '4', '5'],
        parts: [],
        maItems: [],
        partListLoading: false,
        maItemsListLoading: false,
        contractors: [],
        carPlateOptions: [],
        maOptions: [],
        payOptions: [],
        customerOptions: [],
        customerValue: [],
        customerList: [],
        customerLoading: false,
        customers: [],
        defaultDataForm: {},
        oldPartList: [],
        defaultContractor: [],
        contractor: {
          contractorId: '',
          contractorName: '',
          phone: '',
          address: '',
          bank: '',
          cardNo: ''
        },
        dataForm: {
          orderNo: '',
          orderId: 0,
          driver: '',
          carPlate: '',
          seriesName: '',
          vin: '',
          category: '',
          engineNo: '',
          payType: '',
          totalAmount: '',
          comment: '',
          fee: '',
          mileageAfter: '',
          incomingDate: '',
          payDate: '',
          oldPart: '',
          customerId: '',
          customerName: '',
          phone: '',
          contractorId: '',
          partList: [
            {
              partId: '',
              partName: '',
              partUnit: '',
              usedQuantity: '',
              realPrice: '',
              comment: '',
              totalAmount: ''
            }
          ],
          maItemList: [
            {
              maItemId: '',
              maItem: '',
              manHour: '',
              realPrice: '',
              totalAmount: '',
              comment: ''
            }
          ]
        },
        pickerOptions: {
          disabledDate(time) {
            return time.getTime() > Date.now();
          },
          shortcuts: [{
            text: '今天',
            onClick(picker) {
              picker.$emit('pick', new Date());
            }
          }, {
            text: '昨天',
            onClick(picker) {
              const date = new Date();
              date.setTime(date.getTime() - 3600 * 1000 * 24);
              picker.$emit('pick', date);
            }
          }, {
            text: '前天',
            onClick(picker) {
              const date = new Date();
              date.setTime(date.getTime() - 3600 * 1000 * 24 * 2);
              picker.$emit('pick', date);
            }
          }, {
            text: '一周前',
            onClick(picker) {
              const date = new Date();
              date.setTime(date.getTime() - 3600 * 1000 * 24 * 7);
              picker.$emit('pick', date);
            }
          }]
        },
        dataRule: {
          customerName: [
            {required: true, message: '请选择客户', trigger: 'blur'}
          ],
          contractorId: [
            {required: true, message: '请选择承修方', trigger: 'blur'}
          ],
          carPlate: [
            {required: true, message: '请选择车牌号', trigger: 'blur'}
          ],
          vin: [
            {min: 17, max: 17, message: '请输入正确的17位车架号'}
          ],
          engineNo: [
            {min: 6, max: 12, message: '请输入正确的发动机号'}
          ],
          category: [
            {required: true, message: '请选择维修类别', trigger: 'blur'}
          ],
          oldPart: [
            {required: true, message: '请选择旧配件处理方式', trigger: 'blur'}
          ],
          payDate: [
            {required: true, message: '请选择结算日期', trigger: 'blur'}
          ],
          payType: [
            {required: true, message: '请选择结算方式', trigger: 'blur'}
          ],
          usedQuantity :[
            {required: true, message: '请选择数量', trigger: 'blur'}
          ],
          partList: [
            { type: 'array', required: true, message: '请至少添加一个配件', trigger: 'change' }
          ]
        }
      }
    },
    components: {
      OrderPrintPreview
    },
    created() {
      this.defaultDataForm = JSON.parse(JSON.stringify(this.dataForm))
      this.contractor = JSON.parse(JSON.stringify(this.contractor))
      this.parts = this.loadAllParts()
      this.maItems = this.loadAllMaItems()
    },
    mounted() {
      this.getContractors()
      this.getMaOptions()
      this.getOldPartList()
      this.getPayOptions()
    },
    watch: {
      'dataForm.customerId': {
        handler(newName, oldName) {
          if(newName){
            this.getCarPlateOptions(newName)
          }
        },
        immediate: true,
        deep: true
      }
    },
    methods: {
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
      // 获取旧配件单选框选项
      getOldPartList () {
        this.$http({
          url: this.$http.adornUrl('/sys/sysdictitem/itemlist'),
          method: 'get',
          params: {
            'dictCode': 'old_part_type'
          }
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.oldPartList = data.dictList
          } else {
            this.oldPartList = []
          }
        })
      },
      // 维修项目名远程搜索
      queryMaItemSearchAsync(queryString, cb) {
        const maItems = this.maItems
        const results = queryString ? maItems.filter(this.createFilterForMaItem(queryString)) : maItems
        cb(results)
      },
      createFilterForMaItem(queryString) {
        return (maItems) => {
          return (maItems.itemName.toLowerCase().indexOf(queryString.toLowerCase()) !== -1)
        }
      },
      loadAllMaItems() {
        this.maItemsListLoading = true
        this.$http({
          url: this.$http.adornUrl('/business/maitem/listAll'),
          method: 'get'
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.maItems = data.list
          } else {
            this.maItems = []
          }
          this.maItemsListLoading = false
        })
      },
      // 配件名远程搜索
      queryPartSearchAsync(queryString, cb) {
        const parts = this.parts
        const results = queryString ? parts.filter(this.createFilterForPart(queryString)) : parts
        cb(results)
      },
      createFilterForPart(queryString) {
        return (parts) => {
          return (parts.name.toLowerCase().indexOf(queryString.toLowerCase()) !== -1)
        }
      },
      loadAllParts() {
        this.partListLoading= true
        this.$http({
          url: this.$http.adornUrl('/business/part/listAll'),
          method: 'get'
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.parts = data.list
          } else {
            this.parts = []
          }
          this.partListLoading = false
        })
      },
      // 新增配件行
      addPartListRow() {
        this.dataForm.partList.push({
          partName: '',
          partUnit: '',
          usedQuantity: '',
          realPrice: '',
          comment: '',
          totalAmount: ''
        })
        this.$nextTick(() => {
          this.$refs.partInput.focus()
        })
      },
      // 新增维修项目行
      addMaItemListRow() {
        this.dataForm.maItemList.push({
          maItem: '',
          manHour: '',
          realPrice: '',
          totalAmount: '',
          comment: ''
        })
        this.$nextTick(() => {
          this.$refs.maItemInput.focus()
        })
      },
      // 选中配件名操作
      handleSelectPart (item,index) {
        this.dataForm.partList[index].partId = item.partId
        this.dataForm.partList[index].partUnit = item.unit
        this.dataForm.partList[index].realPrice = item.unitPrice
      },
      // 选中维修项目名操作
      handleSelectMaItem (item, index) {
        this.dataForm.maItemList[index].maItemId = item.maItemId
        this.dataForm.maItemList[index].realPrice = item.referPrice
      },
      // 获取客户车辆列表
      getCarPlateOptions(customerId) {
        this.$http({
          url: this.$http.adornUrl('/business/customercarseriesrel/' + customerId),
          method: 'get'
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.carPlateOptions = data.itemList;
          } else {
            this.carPlateOptions = []
          }
        })
      },
      // 选中车辆的操作
      handleCarSelect(val) {
        this.dataForm.carPlate = val.carPlate
        this.dataForm.seriesName = val.seriesName
        this.dataForm.vin = val.vin
        this.dataForm.engineNo = val.engineNo
      },
      // 选中客户操作
      handleCustomerSelect(val) {
        this.dataForm.customerName = val.name
        this.dataForm.phone = val.phone
        this.dataForm.customerId = val.customerId
      },
      // 选中承修方操作
      handleSelectContractor(val) {
        let contractor = this.contractors.filter(item => item.contractorId == val)
        this.dataForm.contractorId = val
        this.contractor = contractor[0]
      },
      // 客户名远程搜索
      queryCustomers(queryString) {
        if (queryString !== '') {
          this.customerLoading = true;
        }
        this.$http({
          url: this.$http.adornUrl('/business/customer/list'),
          method: 'get',
          params: {
            'page': '1',
            'limit': '15',
            'key': queryString
          }
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.customers = data.page.list
            setTimeout(() => {
              this.customerLoading = false;
              this.customerOptions = this.customers.filter(item => {
                return item.name.toLowerCase()
                  .indexOf(queryString.toLowerCase()) > -1
              });
            }, 100);
          } else {
            this.customers = []
          }
        });
      },
      // 删除行数据操作
      handleDeletePartRow(index) {
        this.dataForm.partList.splice(index,1)
      },
      handleDeleteMaItemRow(index) {
        this.dataForm.maItemList.splice(index,1)
      },
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
      // 获取可选承修方列表
      getContractors() {
        this.$http({
          url: this.$http.adornUrl('/business/contractor/listAll'),
          method: 'get'
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.contractors = data.list
          } else {
            this.contractors = []
          }
        })
      },
      init(id) {
        this.dataForm.orderId = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.activeNames = ['1', '2', '3', '4', '5']
          if (this.dataForm.orderId) {
            this.$http({
              url: this.$http.adornUrl(`/business/order/info/${this.dataForm.orderId}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                console.log(data)
                this.dataForm.orderNo = data.order.orderNo
                this.dataForm.customerId = data.order.customerId
                this.dataForm.customerName = data.order.customerName
                this.dataForm.phone = data.order.phone
                this.dataForm.driver = data.order.driver
                this.dataForm.carPlate = data.order.carPlate
                this.dataForm.seriesName = data.order.seriesName
                this.dataForm.vin = data.order.vin
                this.dataForm.engineNo = data.order.engineNo
                this.dataForm.category = data.order.category
                this.dataForm.incomingDate = data.order.incomingDate
                this.dataForm.mileageAfter = data.order.mileageAfter
                this.dataForm.contractorId = data.order.contractorId
                this.contractor = data.order.contractor
                this.dataForm.partList = data.order.partList
                this.dataForm.maItemList = data.order.maItemList
                this.dataForm.oldPart = String(data.order.oldPart)
                this.dataForm.payDate = data.order.payDate
                this.dataForm.payType = String(data.order.payType)
                this.dataForm.comment = data.order.comment
              }
            })
          } else {
            this.resetDataForm()
            this.$refs['dataForm'].resetFields()
            this.$http({
              url: this.$http.adornUrl(`/business/order/serial`),
              method: 'get'
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.orderNo = data.orderNo
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
              url: this.$http.adornUrl(`/business/order/${!this.dataForm.orderId ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'orderId': this.dataForm.orderId || undefined,
                'orderNo': this.dataForm.orderNo,
                'category': this.dataForm.category,
                'customerId': this.dataForm.customerId,
                'customerName': this.dataForm.customerName,
                'driver': this.dataForm.driver,
                'incomingDate': this.dataForm.incomingDate,
                'mileageAfter': this.dataForm.mileageAfter,
                'oldPart': this.dataForm.oldPart,
                'phone': this.dataForm.phone,
                'carPlate': this.dataForm.carPlate,
                'engineNo': this.dataForm.engineNo,
                'vin': this.dataForm.vin,
                'payDate': this.dataForm.payDate,
                'payType': this.dataForm.payType,
                'seriesName': this.dataForm.seriesName,
                'contractorId': this.dataForm.contractorId,
                'maItemList': this.dataForm.maItemList.filter(item => item.maItem !== '' && item.maItem !== undefined && item.maItem.trim() !== ''),
                'partList':this.dataForm.partList.filter(item => item.partName !== '' && item.partName !== undefined && item.partName.trim() !== ''),
                'comment': this.dataForm.comment,
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
                  }
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      },
      // 关闭弹窗
      handleClose () {
        this.resetDataForm()
        this.$refs['dataForm'].resetFields()
        this.customerOptions = []
        this.visible = false
        this.parts = this.loadAllParts()
        this.maItems = this.loadAllMaItems()
        this.carPlateOptions = []
      },
      handleSelectClear () {
        this.customerOptions = []
      },
      // transferToPrint () {
      //   this.orderPrintVisible = true
      //   this.$nextTick(() => {
      //     this.$refs.orderPrintPreview.init()
      //   })
      // },
      // 重置表单
      resetDataForm () {
        this.dataForm = JSON.parse(JSON.stringify(this.defaultDataForm))
        this.contractor = JSON.parse(JSON.stringify(this.defaultContractor))
      }
    }
  }
</script>
<style>
  .el-dialog__body {
    padding-top: inherit;
  }

  #form_title {
    text-align: center;
    font-weight: bold;
    font-size: x-large;
    color: gray;
    margin: auto;
  }

  #orderNo {
    color: #0066cc;
  }

  .el-remote-search div {

      width: 200%;
      background-color: white;

  }
</style>
