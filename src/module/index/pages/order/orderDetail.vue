<template>
  <div class="receiveDetails"  >
    <el-breadcrumb separator=">" class="bread">
      <svg class="icon combinedShape" aria-hidden="true">   <use xlink:href="#icon-locate"></use> </svg>
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>我买入的订单</el-breadcrumb-item>
      <el-breadcrumb-item>订单详情</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
    <el-row class="el-row-header statePosition">
      <el-col class="detail_title_color sellerColor stateShow">
            <svg class="icon detailIcon" aria-hidden="true"><use xlink:href="#icon-zhuangtai"></use> </svg>订单当前状态：{{orderDetail.txDetail.operationRecordVoList[orderDetail.txDetail.operationRecordVoList.length-1].state | transactionStatus}}
            &nbsp;<el-button class="button_color" size="small" v-if="(state.isBuyer==='false')&&(orderDetail.txDetail.operationRecordVoList[orderDetail.txDetail.operationRecordVoList.length-1].state===constantData.UNCONFIRMED)" @click.native.prevent="confirmOrder(orderDetail.txDetail.orderId)">确认订单</el-button>
            <el-button size="small" v-if="(state.isBuyer==='false')&&(orderDetail.txDetail.operationRecordVoList[orderDetail.txDetail.operationRecordVoList.length-1].state===constantData.CONFIRMED)&&(orderDetail.receOver.receLatestStatus===constantData.NOMESSAGE)" @click.native.prevent="signBill(orderDetail.txDetail.orderId)">签发应收账款</el-button>
            <el-button size="small" v-if="(state.isBuyer==='false')&&(orderDetail.receOver.receLatestStatus===constantData.ACCEPTED)&&(orderDetail.wayBillOver.wayBillLatestStatus===constantData.FORSEND)" @click.native.prevent="sendGood(orderDetail.txDetail.orderId)">发货</el-button>
            <el-button size="small" v-if="(state.isBuyer==='true')&&(orderDetail.receOver.receLatestStatus===constantData.FORACCEPT)" @click.native.prevent="acceptBill(orderDetail.receOver.receNo)">签收账款</el-button>
      </el-col>
    </el-row>
      <el-row>
        <el-col :span="24">
          <el-card class="box-card mybox" style="width:100%">
            <div slot="header" class="clearfix el-row-header">
              <svg class="icon detailIcon" aria-hidden="true">   <use xlink:href="#icon-order_H"></use> </svg>
              <span class="keynote">订单详情</span>
            </div>
            <div class="box-card mycard1 detailContent">
              <el-row>
                <el-col :span="6" class="msgName keynote" v-if="state.isBuyer==='false'">买家信息：</el-col>
                <el-col :span="6" class="msgName keynote" v-else>卖家信息：</el-col>
              </el-row>
              <el-row class="">
                <el-col :span="6" class="msgName" v-if="state.isBuyer==='true'"> 商家：{{orderDetail.txDetail.payeeCompanyName}}</el-col>
                <el-col :span="6" class="msgName" v-else>商家：{{orderDetail.txDetail.payeeCompanyName}}</el-col>
                <el-col :span="6" class="msgName">支付银行：{{orderDetail.txDetail.payerBank}}</el-col>
                <el-col :span="6" class="msgName">付款账户：{{orderDetail.txDetail.payerAccount}}</el-col>
                <el-col :span="6" class="msgName">付款方式：{{orderDetail.txDetail.payingMethod | payingMethod}}</el-col>
              </el-row>
              <el-row style="margin-top: 10px;">
                <el-col :span="6" class="msgName keynote">订单信息：</el-col>
              </el-row>
              <el-row>
                <el-col :span="6" class="msgName">订单编号：{{orderDetail.txDetail.orderId}}</el-col>
                <el-col :span="6" class="msgName">发起时间：{{orderDetail.txDetail.operationRecordVoList[0].operateTime | timeTransfer}}</el-col>
              </el-row>
              <el-row>
                <el-col :span="6" class="msgName">货品名称：{{orderDetail.txDetail.productName}}</el-col>
                <el-col :span="6" class="msgName">货品数量：{{orderDetail.txDetail.productQuantity}}</el-col>
                <el-col :span="6" class="msgName">订单金额(元)：{{orderDetail.txDetail.productTotalPrice | numTransfer}}</el-col>
              </el-row>
              <el-row>
                <el-col :span="6" class="msgName">最新交易状态：{{orderDetail.txDetail.operationRecordVoList[orderDetail.txDetail.operationRecordVoList.length-1].state | transactionStatus}}</el-col>
                <el-col :span="6" class="collapseBtn"><i class="el-icon-caret-bottom" @click="orderCollapse" :class="{rotate:isOrderCollapse, rotate1:!isOrderCollapse}"></i></el-col>
              </el-row>
              <!--<el-row class="">-->
              <!--<el-col :span="8" v-if="(orderDetail.txDetail.orderState!==constantData.UNCONFIRMED)">交易确认：{{orderDetail.txDetail.orderConfirmTime | timeTransfer}}</el-col>-->
              <!--</el-row>-->
                <el-row v-show="isOrderCollapse" class="collapseTop">
                  <template v-for="(item,index) in orderDetail.txDetail.operationRecordVoList">
                    <el-row class="status-list" :class="{circleColor:index==(orderDetail.txDetail.operationRecordVoList.length-1)}">
                      <el-col :span="8" :class="{circleColor1:index==(orderDetail.txDetail.operationRecordVoList.length-1)}"><span>{{item.operateTime | timeTransfer}} {{item.state | transactionStatus}}</span></el-col>
                    </el-row>
                  </template>
                </el-row>
            </div>
          </el-card>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="24">
          <el-card class="box-card mybox" style="width:100%">
            <div slot="header" class="clearfix el-row-header">
              <svg class="icon detailIcon" aria-hidden="true">   <use xlink:href="#icon-yszk_hui"></use> </svg>
              <span class="keynote">应收账款详情</span><el-button size="mini" type="text" class="detailButton" v-if="(orderDetail.receOver.receLatestStatus!==constantData.NOMESSAGE)" @click="receDetailPage(orderDetail.receOver.receNo)">查看详情 ></el-button>
            </div>
            <div class="box-card mycard1 detailContent" v-if="orderDetail.receOver.receLatestStatus===constantData.NOMESSAGE">
              <span class="msgName">暂无应收账款信息!</span>
            </div>
            <div class="box-card mycrad1 detailContent" v-else-if="orderDetail.receOver.receLatestStatus===constantData.FORISSUE">
              <span class="msgName">应收账款待签发!</span>
            </div>
            <div class="box-card mycard1 detailContent" v-else>
              <el-row>
                <el-col :span="6" class="msgName keynote">业务编号：{{orderDetail.receOver.receNo}}</el-col>
                <el-col :span="6" class="msgName">签发时间：{{orderDetail.receOver.receGenerateTime | timeTransfer}}</el-col>
                <el-col :span="6" class="msgName">到期日：{{orderDetail.receOver.dueDate | timeTransfer}}</el-col>
              </el-row>
              <el-row>
                <el-col :span="6" class="msgName">收款账户：{{orderDetail.receOver.receivingSide}}</el-col>
                <el-col :span="6" class="msgName">付款账户：{{orderDetail.receOver.payingSide}}</el-col>
              </el-row>
              <el-row>
                <el-col :span="6" class="msgName">账款金额(元)：{{orderDetail.receOver.receAmount | numTransfer}}</el-col>
                <el-col :span="6" class="msgName">票面利息：{{orderDetail.receOver.coupon}}</el-col>
              </el-row>
              <el-row>
                <el-col :span="8" class="msgName">
                  最新应收账款状态：{{orderDetail.receOver.receLatestStatus | receStatus}}
                </el-col>
                <el-col :span="6" class="collapseBtn"><i class="el-icon-caret-bottom" @click="receCollapse" :class="{rotate:isReceCollapse, rotate1:!isReceCollapse}"></i></el-col>
              </el-row>

              <el-row v-show="isReceCollapse" class="collapseTop">
                <template v-for="(item,index) in receHistory">
                  <el-row class="status-list" :class="{circleColor:index==(receHistory.length-1)}">
                    <el-col :span="8" :class="{circleColor1:index==(receHistory.length-1)}"><span>{{item.time | timeTransfer}} {{item.receivableStatus | receStatus}}</span></el-col>
                  </el-row>
                </template>
              </el-row>
            </div>
          </el-card>
        </el-col>
      </el-row>
      <el-row>
      <el-col :span="24">
        <el-card class="box-card mybox" style="width:100%">
          <div slot="header" class="clearfix el-row-header">
            <svg class="icon detailIcon" aria-hidden="true">   <use xlink:href="#icon-wl_H"></use> </svg>
            <span class="keynote">物流信息</span><el-button size="mini" type="text" class="detailButton" @click="logisticsDetailPage(orderDetail.txDetail.orderId)" v-if="(orderDetail.wayBillOver.wayBillLatestStatus!==constantData.NOMESSAGE)&&(orderDetail.wayBillOver.wayBillLatestStatus!==constantData.FORSEND)">查看详情 ></el-button>
          </div>
          <div class="box-card mycard1 detailContent" v-if="orderDetail.wayBillOver.wayBillLatestStatus===constantData.NOMESSAGE">
            <span class="msgName">暂无物流信息！</span>
          </div>
          <div class="box-card mycard1 detailContent" v-else-if="orderDetail.wayBillOver.wayBillLatestStatus===constantData.FORSEND">
            <span class="msgName">待发货！</span>
          </div>
          <div class="box-card mycard1 detailContent" v-else>
            <el-row>
              <el-col :span="6" class="msgName keynote">运单号：{{wayBillHistory.wayBillNo}}</el-col>
              <el-col :span="6" class="msgName">下单时间：{{wayBillHistory.sendReqTime | timeTransfer}}</el-col>
            </el-row>
            <el-row class="">
              <el-col :span="6" class="msgName">物流公司：{{wayBillHistory.logisticsEnterpriseName}}</el-col>
            </el-row>
            <el-row>
              <el-col :span="8" class="msgName">
                物流当前状态：{{orderDetail.wayBillOver.wayBillLatestStatus | wayBillStatus}}
              </el-col>
              <el-col :span="6" class="collapseBtn"><i class="el-icon-caret-bottom" @click="wayBillCollapse" :class="{rotate:isWayBillCollapse, rotate1:!isWayBillCollapse}"></i></el-col>
            </el-row>
            <el-row v-show="isWayBillCollapse" class="collapseTop">
              <template v-for="(item,index) in wayBillHistory.operationRecordVo">
                <el-row class="status-list" :class="{circleColor:index==(wayBillHistory.operationRecordVo.length-1)}">
                  <el-col :span="8" :class="{circleColor1:index==(wayBillHistory.operationRecordVo.length-1)}"><span>{{item.operateTime | timeTransfer}} {{item.state | wayBillStatus}}</span></el-col>
                </el-row>
              </template>
            </el-row>
          </div>
        </el-card>
      </el-col>
    </el-row>
      <el-row>
      <el-col :span="24">
        <el-card class="box-card mybox" style="width:100%">
          <div slot="header" class="clearfix el-row-header" v-if="state.isBuyer==='true'">
            <svg class="icon detailIcon" aria-hidden="true">   <use xlink:href="#icon-cc_H"></use> </svg>
            <span class="keynote">仓储信息</span><el-button size="mini" type="text" class="detailButton" v-if="orderDetail.repoOver.payerRepoBusiState>=constantData.INFORRESPONSE"  @click="repoDetailPage(orderDetail.repoOver.payerRepoBusinessNo)">查看详情 ></el-button>
          </div>
          <div slot="header" class="clearfix el-row-header" v-else>
            <svg class="icon detailIcon" aria-hidden="true">   <use xlink:href="#icon-cc_H"></use> </svg>
            <span class="keynote">仓储信息</span><el-button size="mini" type="text" class="detailButton" v-if="orderDetail.repoOver.payeeRepoBusiState>=constantData.FOROUT" @click="repoDetailPage(orderDetail.repoOver.payeeRepoBusinessNo)">查看详情  ></el-button>
          </div>
          <div class="box-card mycard1 detailContent" v-if="state.isBuyer==='true'"><!--区分买家和买家-->
            <div v-if="orderDetail.repoOver.payerRepoBusiState<constantData.INFORRESPONSE"><!--买家和卖家状态字段不同-->
              <span class="msgName">暂无仓储信息！</span>
              <!--买家在入库待响应时开始有显示-->
            </div>
            <div v-else>
            <el-row>
              <el-col :span="6" class="msgName keynote">仓储流水号：{{orderDetail.repoOver.payerRepoBusinessNo}}</el-col>
              <el-col :span="6" class="msgName">入库申请时间：{{orderDetail.repoOver.inApplyTime | timeTransfer}}</el-col>
            </el-row>
            <el-row>
              <el-col :span="6" class="msgName">仓储公司：{{orderDetail.txDetail.payerRepo}}</el-col>
            </el-row>
            <el-row>
              <el-col :span="6" class="msgName">最新仓储状态：{{orderDetail.repoOver.payerRepoBusiState | repoStatus}}</el-col>
              <el-col :span="6" class="collapseBtn"><i class="el-icon-caret-bottom" @click="buyerRepoCollapse" :class="{rotate:isBuyerRepoCollapse, rotate1:!isBuyerRepoCollapse}"></i></el-col>
            </el-row>
              <el-row v-show="isBuyerRepoCollapse" class="collapseTop">
                <template v-for="(item,index) in buyerRepoHistory">
                  <el-row class="status-list" :class="{circleColor:index==(buyerRepoHistory.length-1)}">
                    <el-col :span="8" :class="{circleColor1:index==(buyerRepoHistory.length-1)}"><span>{{item.operateTime | timeTransfer}} {{item.state | repoStatus}}</span></el-col>
                  </el-row>
                </template>
              </el-row>
            </div>
          </div>
          <div class="box-card mycard1 detailContent" v-if="state.isBuyer==='false'">
            <div v-if="orderDetail.repoOver.payeeRepoBusiState<constantData.FOROUT">
              <span class="msgName">暂无仓储信息！</span>
              <!--卖家在待出库时开始有显示-->
            </div>
            <div v-else>
            <el-row>
              <el-col :span="6" class="msgName keynote">仓储流水号：{{orderDetail.repoOver.payeeRepoBusinessNo}}</el-col>
              <el-col :span="6" class="msgName">仓单编号：{{orderDetail.repoOver.payeeRepoCertNo}}</el-col>
              <el-col :span="6" class="msgName" v-if="orderDetail.repoOver.payeeRepoBusiState>=constantData.CONFIRMED">出库申请时间：{{orderDetail.repoOver.outApplyTime | timeTransfer}}</el-col>
            </el-row>
            <el-row>
              <el-col :span="6" class="msgName" v-if="orderDetail.txDetail.payeeRepo===''">暂无仓储信息</el-col>
              <el-col :span="6" class="msgName" v-else>仓储公司：{{orderDetail.txDetail.payeeRepo}}</el-col>
              <!--注意这里多了一个字段-->
              <el-col :span="6" class="msgName">持有人：{{orderDetail.txDetail.payeeCompanyName}}</el-col>
            </el-row>
            <el-row>
              <el-col :span="8" class="msgName">最新仓储状态：{{orderDetail.repoOver.payeeRepoBusiState | repoStatus}}</el-col>
              <el-col :span="6" class="collapseBtn"><i class="el-icon-caret-bottom" @click="buyeeRepoCollapse" :class="{rotate:isBuyeeRepoCollapse, rotate1:!isBuyeeRepoCollapse}"></i></el-col>
            </el-row>
              <el-row v-show="isBuyeeRepoCollapse" class="collapseTop">
                <template v-for="(item,index) in buyeeRepoHistory">
                  <el-row class="status-list" :class="{circleColor:index==(buyeeRepoHistory.length-1)}">
                    <el-col :span="8" :class="{circleColor1:index==(buyeeRepoHistory.length-1)}"><span>{{item.operateTime | timeTransfer}} {{item.state | repoStatus}}</span></el-col>
                  </el-row>
                </template>
              </el-row>
            </div>
          </div>
        </el-card>
      </el-col>
    </el-row>
    </el-card>
  </div>
</template>
<script>
  import store from '../../vuex/store'
  import constantData from '../../../../common/const'
  import userInfo from '../../../../common/store'
  export default {
    name: 'index',
    data () {
      return {
        orderDetail: {
          txDetail: {
            operationRecordVoList:[
              {state:'',operateTime:''}
            ],
          },
          receOver: {},
          repoOver: {},
          wayBillOver: {}
        },
        receHistory:[
        ],
        wayBillHistory:{
          operationRecordVo:[],
          sendReqTime:''
        },
        buyerRepoHistory:[
        ],
        buyeeRepoHistory:[
        ],
        isOrderCollapse:false,
        isReceCollapse:false,
        isWayBillCollapse:false,
        isBuyerRepoCollapse:false,
        isBuyeeRepoCollapse:false,
      }
    },
    computed: {
      state () {
        return store.state;
      },
      constantData () {
          return constantData;
      }
    },
    methods: {
      confirmOrder (checkId) {
        console.log("确认订单！");
        store.commit('setCheckIdOrder',checkId);
        this.$router.push("/order/confirmOrder");
      },
      signBill (checkId) {
        console.log("签发应收账款");
        store.commit('setCheckIdOrder',checkId);
        this.$router.push("/allAccounts/signout/signout");
      },
      sendGood (checkId) {
        console.log("发货");
        store.commit('setCheckIdOrder',checkId);
        this.$router.push("/logistics/deliver");
      },
      acceptBill (checkId) {
        console.log("承兑签收");
        store.commit('setCheckIdRece',checkId);
        this.$router.push("/allAccounts/accept/accept");
      },
      orderCollapse () {
          if(this.isOrderCollapse){
            this.isOrderCollapse=false;
          }
          else {
            this.isOrderCollapse=true;
            this.isReceCollapse=false;
            this.isWayBillCollapse=false;
            this.isBuyerRepoCollapse=false;
            this.isBuyeeRepoCollapse=false;
            console.log("请求订单详情接口，不需要");
          }
      },
      receCollapse () {
        if(this.isReceCollapse){
          this.isReceCollapse=false;
        }
        else {
          this.isOrderCollapse=false;
          this.isReceCollapse=true;
          this.isWayBillCollapse=false;
          this.isBuyerRepoCollapse=false;
          this.isBuyeeRepoCollapse=false;
          console.log("请求应收详情接口");

          /*this.$http.get("http://localhost/server_test/getData4.php").then(function(res){
              this.receHistory=res.body.data;
          },
          function(err){
             console.log(err);
          });*/
        }
      },
      wayBillCollapse () {
        if(this.isWayBillCollapse){
          this.isWayBillCollapse=false;
        }
        else {
          this.isOrderCollapse=false;
          this.isReceCollapse=false;
          this.isWayBillCollapse=true;
          this.isBuyerRepoCollapse=false;
          this.isBuyeeRepoCollapse=false;
          console.log("请求物流详情接口");
        }
      },
      buyerRepoCollapse () {
        if(this.isBuyerRepoCollapse){
          this.isBuyerRepoCollapse=false;
        }
        else {
          this.isOrderCollapse=false;
          this.isReceCollapse=false;
          this.isWayBillCollapse=false;
          this.isBuyerRepoCollapse=true;
          this.isBuyeeRepoCollapse=false;
          console.log("请求仓储详情接口");
        }
      },
      buyeeRepoCollapse () {
        if(this.isBuyeeRepoCollapse){
          this.isBuyeeRepoCollapse=false;
        }
        else {
          this.isOrderCollapse=false;
          this.isReceCollapse=false;
          this.isWayBillCollapse=false;
          this.isBuyerRepoCollapse=false;
          this.isBuyeeRepoCollapse=true;
          console.log("请求仓储详情接口");
        }
      },
      receDetailPage (checkId) {
          store.commit('setCheckIdRece',checkId);
        this.$router.push("/allAccounts/detail/detail");
      },
      logisticsDetailPage(checkId){
          store.commit('setCheckIdOrder',checkId);
        this.$router.push("/logistics/wayBillDetails");
      },
      repoDetailPage(checkId){
        store.commit('setCheckIdRepo',checkId);
        this.$router.push("/warehousing/repoDetails");
      }
    },
    mounted() {
      document.body.scrollTop = 0;
      document.documentElement.scrollTop = 0;
      console.log("the state checkIdOrder is:" + store.state.checkIdOrder);
      this.$http.get("../v1/order/detail?orderNo=" + store.state.checkIdOrder).then(
        function (res) {
          // 处理成功的结果
          if(res.body.code != 0){
            this.$message.error(res.body.message);
            return;
          }
          this.orderDetail = res.body.data;
//          根据应收款编号查询应收款详情
          if(this.orderDetail.receOver.receNo!==''){
              var receTemp={};
              receTemp.receivableNo=this.orderDetail.receOver.receNo;
              receTemp.operatorAcctId=userInfo.fetchUserInfo().acctIds;//操作人账号，从localStorage中取得
              this.$http.post("../v1/receivable/receivableInfoWithSerial",receTemp,{emulateJSON:true}).then(function(res){
                if(res.body.code != 0){
                  this.$message.error(res.body.message);
                  return;
                }
                console.log("根据应收款编号查询应收款详情："+res.body.data);
                this.receHistory=res.body.data.serialVoList;//获取流水信息列表
              },function(err){
                  console.log(err);
              });
          }
//          根据仓储业务编号查询仓储详情
          if(store.state.isBuyer==='true'){
          if(this.orderDetail.repoOver.payerRepoBusinessNo!==''){
              this.$http.get("../v1/repository/getRepoBusiHistoryList?repoBusinessNo="+this.orderDetail.repoOver.payerRepoBusinessNo).then(function(res){
                if(res.body.code != 0){
                  this.$message.error(res.body.message);
                  return;
                }
                  console.log("根据买家仓储业务编号查询仓储详情："+res.body.data);
                  this.buyerRepoHistory=res.body.data.operationRecordVoList;
              },function (err) {
                console.log(err);
              });
          }
          }
          else {
              if(this.orderDetail.repoOver.payeeRepoBusinessNo!==''){
                this.$http.get("../v1/repository/getRepoBusiHistoryList?repoBusinessNo="+this.orderDetail.repoOver.payeeRepoBusinessNo).then(function(res){
                  if(res.body.code != 0){
                    this.$message.error(res.body.message);
                    return;
                  }
                  console.log("根据卖家仓储业务编号查询仓储详情："+res.body.data);
                  this.buyeeRepoHistory=res.body.data.operationRecordVoList;
                },function (err) {
                  console.log(err);
                });
              }
          }
//          根据订单号查询运单详情
          if(this.orderDetail.wayBillOver.wayBillLatestStatus>0){
              this.$http.get("../v1/waybill/wayBillDetail?orderNo="+this.orderDetail.txDetail.orderId).then(function(res){
                if(res.body.code != 0){
                  this.$message.error(res.body.message);
                  return;
                }
                console.log("根据订单号查询运单详情："+res.body.data);
                this.wayBillHistory=res.body.data;
                this.wayBillHistory.sendReqTime='';
                for(var item in this.wayBillHistory.operationRecordVo){
                  var temp=this.wayBillHistory.operationRecordVo[item];
                  if(temp.state===constantData.SENDFORRESPONSE){/*筛选申请发货时间，即发货待响应时间*/
                    this.wayBillHistory.sendReqTime=temp.operateTime;
                    break;
                  }
                }
              },function(err){
                  console.log(err);
              });
          }
        }, function (res) {
          // 处理失败的结果
          console.log(res);
        }
      );
    }
  }
</script>
<style>

</style>
