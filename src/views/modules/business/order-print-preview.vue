<template>
  <el-dialog
    :close-on-click-modal="false"
    :visible.sync="visible"
    :append-to-body="true"
    width="60%">
    <div ref="print">
      <table cellspacing="0" align="center" width="100%" class="table">
        <tr align="center">
          <td style="font-size: 30px;font-weight: bold;height: 40px" colspan="7">
            江苏省机动车维修费用结算清单
          </td>
        </tr>
        <tr>
          <td align="right" colspan="7">
            <span style="font-weight: bold">维修单号:</span> {{dataForm.orderNo}}
          </td>
        </tr>
        <tr>
          <td rowspan="4" class="top">托修方</td>
          <td style="width:120px;height: 50px;text-align: center;font-weight: bold">单位名称</td>
          <td style="width:400px;height: 50px;padding-left: 10px" colspan="3">{{dataForm.customerName}}</td>
          <td style="width:120px;height: 50px;text-align: center;font-weight: bold">送修人</td>
          <td style="width:100px;height: 50px;padding-left: 10px">{{dataForm.driver}}</td>
        </tr>
        <tr>
          <td class="top_col_1">车牌号码</td>
          <td class="top_col_2">{{dataForm.carPlate}}</td>
          <td class="top_col_3">厂牌型号</td>
          <td class="top_col_4">{{dataForm.seriesName}}</td>
          <td class="top_col_5">发动机号</td>
          <td class="top_col_6">{{dataForm.engineNo}}</td>
        </tr>
        <tr>
          <td class="top_col_1">车架号</td>
          <td class="top_col_2">{{dataForm.vin}}</td>
          <td class="top_col_3">进厂日期</td>
          <td class="top_col_4">{{dataForm.incomingDate}}</td>
          <td class="top_col_5">结算日期</td>
          <td class="top_col_6">{{dataForm.payDate}}</td>
        </tr>
        <tr>
          <td class="top_col_1">出厂里程表读数</td>
          <td class="top_col_2">{{dataForm.mileageAfter}}公里</td>
          <td class="top_col_3">维修类别</td>
          <td class="top_col_4">{{dataForm.category}}</td>
          <td class="top_col_5">联系电话</td>
          <td class="top_col_6">{{dataForm.phone}}</td>
        </tr>
        <tr>
          <td rowspan="4" class="top">承修方</td>
          <td class="top_col_1">单位名称</td>
          <td colspan="3" style="width:400px;padding-left: 10px">{{dataForm.contractor.contractorName}}</td>
          <td class="top_col_5">联系电话</td>
          <td class="top_col_6">{{dataForm.contractor.phone}}</td>
        </tr>
        <tr>
          <td class="top_col_1">单位地址</td>
          <td colspan="5" class="top_col_7">{{dataForm.contractor.address}}</td>
        </tr>
        <tr>
          <td class="top_col_1">开户银行</td>
          <td colspan="5" class="top_col_7">{{dataForm.contractor.bank}}</td>
        </tr>
        <tr>
          <td class="top_col_1">银行账号</td>
          <td colspan="5" class="top_col_7">{{dataForm.contractor.cardNo}}</td>
        </tr>
      </table>
      <br>
      <span class="table_head">
        工时费
      </span>
      <table cellspacing="0" align="center" width="100%" class="table">
        <tr>
          <td style="width: 40px;text-align: center;font-weight: bold">序号</td>
          <td style="width: 300px;text-align: center;font-weight: bold">维修项目</td>
          <td style="width: 100px;text-align: center;font-weight: bold">结算工时</td>
          <td style="width: 100px;text-align: center;font-weight: bold">单价</td>
          <td style="width: 120px;text-align: center;font-weight: bold">金额</td>
          <td style="width: 150px;text-align: center;font-weight: bold">备注</td>
        </tr>
        <tr v-model="dataForm.maItemList[i]" v-for="(maItem,i) in dataForm.maItemList">
          <td style="width: 40px;text-align: center;">{{i+1}}</td>
          <td style="padding-left: 10px">{{maItem.maItem}}</td>
          <td style="text-align: center">{{maItem.manHour}}</td>
          <td style="text-align: center;">{{maItem.realPrice}}</td>
          <td style="text-align: center">{{maItem.totalAmount}}</td>
          <td style="text-align: center">{{maItem.comment}}</td>
        </tr>
        <tr>
          <td colspan="2" style="font-weight: bold;text-align: center">合计工时</td>
          <td style="text-align: center">{{totalSumAllHour}}</td>
          <td colspan="2" style="text-align: center;font-weight: bold">工时费用合计</td>
          <td style="text-align: center">{{totalSumAllHourFee}}</td>
        </tr>
      </table>
      <br>
      <span class="table_head">
        材料费
      </span>
      <table cellspacing="0" align="center" width="100%" class="table">
        <tr>
          <td style="width: 40px;text-align: center;font-weight: bold">序号</td>
          <td style="width: 300px;text-align: center;font-weight: bold">配件名称</td>
          <td style="width: 40px;text-align: center;font-weight: bold">单位</td>
          <td style="width: 60px;text-align: center;font-weight: bold">数量</td>
          <td style="width: 100px;text-align: center;font-weight: bold">单价</td>
          <td style="width: 120px;text-align: center;font-weight: bold">金额</td>
          <td style="width: 150px;text-align: center;font-weight: bold">备注</td>
        </tr>
        <tr v-model="dataForm.partList[i]" v-for="(part,i) in dataForm.partList">
            <td style="width: 40px;text-align: center;">{{i+1}}</td>
            <td style="padding-left: 10px">{{part.partName}}</td>
            <td style="width: 40px;text-align: center;">{{part.partUnit}}</td>
            <td style="width: 60px;text-align: center">{{part.usedQuantity}}</td>
            <td style="width: 100px;text-align: center">{{part.realPrice}}</td>
            <td style="width: 120px;text-align: center">{{part.totalAmount}}</td>
            <td style="width: 150px;text-align: center">{{part.comment}}</td>
        </tr>
        <tr>
          <td colspan="2" style="font-weight: bold;text-align: center">材料费合计</td>
          <td colspan="6" style="text-align: center">{{totalSumAllPart}}</td>
        </tr>
      </table>

      <br>
      <span class="table_head">
        合计金额
      </span>
      <table cellspacing="0" align="center" width="100%" class="table">
        <tr>
          <td class="middle_col_1">工时费</td>
          <td class="middle_col_2">{{totalSumAllHourFee}}</td>
        </tr>
        <tr>
          <td class="middle_col_1">材料费</td>
          <td class="middle_col_2">{{totalSumAllPart}}</td>
        </tr>
        <tr>
          <td class="middle_col_1">合计</td>
          <td class="middle_col_2">{{dataForm.totalAmount}}</td>
        </tr>
        <tr>
          <td class="middle_col_1">实收金额大写</td>
          <td class="middle_col_2">{{dataForm.uppercaseTotal}}</td>
        </tr>
      </table>

      <br>
      <span>1.是否有维修人支付费用更换的旧配件</span>
      <div>
        <table cellspacing="0" align="center" width="100%" class="table">
          <tr>
            <td v-for="item in oldPartList" width="35%">
              <label><input type="radio" disabled="true" v-bind:checked="dataForm.oldPart==item.itemValue"> {{item.itemText}}</label>
            </td>
          </tr>
        </table>
      </div>
      <span>2.结算清单项目及应付金额经双方核实，客户签字后生效</span>
      <br><br>
      <el-row>
        <el-col :span="8">
          <span>客户签字:</span><input type="text" name="name" style="border:none;border-bottom:1px solid #000;width: 100px"/>
        </el-col>
        <el-col :span="8">
          <span>承修方(盖章/签名):</span><input type="text" name="sign" style="border:none;border-bottom:1px solid #000; width: 100px"/>
        </el-col>
        <el-col :span="8" style="text-align: right;">
          <span>出单时间:</span><span> {{date}}</span>
        </el-col>
      </el-row>

    </div>
    <span slot="footer" class="dialog-footer">
    <el-button @click="handlePrint" type="primary">打印</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data() {
      return {
        visible: false,
        oldPartList: [],
        date: this.dayjs(new Date()).format('YYYY-MM-DD HH:mm:ss'),
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
          uppercaseTotal: '',
          contractor: {
            contractorId: '',
            contractorName: '',
            phone: '',
            address: '',
            bank: '',
            cardNo: ''
          },
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
      }
    },
    computed:{
      totalSumAllPart(){
        let totalSumAllPart = 0;
        this.dataForm.partList.map((item) => {if(!isNaN(item.totalAmount)) totalSumAllPart += item.totalAmount})
        if(isNaN(totalSumAllPart)){
          return 0
        }
        return totalSumAllPart
      },
      totalSumAllHour(){
        let totalSumAllHour = 0;
        this.dataForm.maItemList.map((item) => {if(!isNaN(item.manHour)) totalSumAllHour += item.manHour})
        if(isNaN(totalSumAllHour)){
          return 0
        }
        return totalSumAllHour
      },
      totalSumAllHourFee(){
        let totalSumAllHourFee = 0;
        this.dataForm.maItemList.map((item) => {if(!isNaN(item.totalAmount)) totalSumAllHourFee += item.totalAmount})
        if(isNaN(totalSumAllHourFee)){
          return 0
        }
        return totalSumAllHourFee
      }
    },
    created() {

    },
    mounted() {
      this.getOldPartList()
    },
    methods: {
      init(id) {
        this.visible = true
        this.$nextTick(() => {
          this.$http({
            url: this.$http.adornUrl(`/business/order/info/${id}`),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 0) {
              console.log(data)
              this.dataForm = data.order
              this.dataForm.oldPart = String(data.order.oldPart)
            }
          })
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
      // 打印操作
      handlePrint() {
        this.$print(this.$refs.print)
      },
    }
  }

</script>

<style>
  .el-dialog__body {
    padding-top: 10px;
  }

  @page {
    margin-top: 5mm;
    margin-bottom: 1mm;
  }

  .table {
    border-collapse:collapse;border-style: solid;border-width: 1px;
  }

  .table td {
    border-style: solid;border-width: 1px;

  }

  .table_head {
    font-weight: bold;
  }

  .top {
    width: 30px;
    margin: 0 auto;
    line-height: 20px;
    font-size: 20px;
    text-align: center;
    vertical-align: center;
    word-wrap: break-word;
    font-weight: bold;
  }

  .top_col_1 {
    width: 120px;
    text-align: center;
    font-weight: bold;
  }

  .top_col_2 {
    width: 100px;
    padding-left: 10px;
  }

  .top_col_3 {
    width: 80px;
    text-align: center;
    font-weight: bold;
  }

  .top_col_4 {
    width: 100px;
    padding-left: 10px;
  }

  .top_col_5 {
    width: 120px;
    text-align: center;
    font-weight: bold;
  }

  .top_col_6 {
    width: 100px;
    padding-left: 10px;
  }

  .top_col_7 {
    width: 620px;
    padding-left: 10px;
  }

  .middle_col_1 {
    width: 200px;
    font-weight: bold;
    text-align: center;
  }

  .middle_col_2 {
    text-align: center;
  }

</style>
