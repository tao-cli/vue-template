<!--   
  *@name ChoosedGoods    选择礼品或优惠券
  *@author Sam
  *@date 2019/12/11
  *@desc  选择积分礼品或优惠券
  *@param goods  选中的礼品或优惠券
  *@param goodsType 类型：礼品或优惠券，礼品-1，优惠券-2
  *
  *@function getGoods  present  选择礼品或优惠券方法回调
-->
<template>
  <div  style="display:flex; align-items:center;">
    <div v-if="!$_.isEmpty(curGoods)" style="display:flex; align-items:center;">
      <div class="show_info" v-if="goodsType == 1">  
        <div v-if="isMall"  class="goods-box">
          <div>
            <Row style="display:flex;align-items:center;">
              <i-col span="6" style="display:flex"><img style="width:58px;height:58px" :src="curGoods.presentPictureUrl||curGoods.imgUrl" /></i-col>
              <i-col class="goods-header">
                <h3>{{curGoods.presentName||curGoods.name}}</h3>
                <p><label>商品货号：</label>{{curGoods.presentIdentify||curGoods.id}}</p>
              </i-col>
            </Row>
            <p><label>商品类目：</label>{{curGoods.goodsCategoryText||'-'}}</p>
            <p><label>商品品牌：</label>{{curGoods.goodsBrandText||'-'}}</p>
            <p><label>开票税率：</label>{{curGoods.goodsTaxText||'-'}}</p>
          </div>
          <Poptip
            confirm width="220"
            title="是否确认删除该商品？"
            @on-ok="()=>{this.curGoods = {}; this.$emit('removeGoods')}"
            @on-cancel="()=>console.log('cancel')">
            <a v-if="!isEdit"><Icon type="ios-trash" /></a>
          </Poptip>
        </div>   
        <div  v-else >
          <div class="show_top">
            <!--222-->
            <span style="width:20%">
              <img style="width:48px;height:48px" :src="curGoods.presentPictureUrl||curGoods.imgUrl" alt />
            </span>
            <div style="width:80%">
              <Row>
                <i-col
                  span="20"
                  style="text-align:left;font-size:16px;font-weight:bold"
                >{{curGoods.presentName||curGoods.name}}</i-col>
              </Row>
              <Row v-if="goodsType=='1'&&curGoods.worth">
                <i-col
                  span="20"
                  style="text-align:left;font-size:12px;color:#9AA8B8"
                >礼品价格</i-col>
              </Row>
              <Row v-else-if="goodsType=='2'">
                <i-col
                  span="20"
                  style="text-align:left;font-size:12px;color:#9AA8B8"
                >优惠内容</i-col>
              </Row>
              <Row v-if="goodsType=='2' &&curGoods.worth">
                <i-col
                  span="10"
                  style="text-align:left;font-size:24px;color:#3E4552"
                >{{curGoods.worth}}</i-col>
              </Row>
              <Row v-else-if="curGoods.worth">
                <i-col
                  span="10"
                  style="text-align:left;font-size:24px;color:#3E4552"
                >{{"￥"+curGoods.worth}}</i-col>
              </Row>
              <Row>
                <i-col
                  span="10"
                  style="text-align:left;font-size:12px;color:#9AA8B8"
                >{{"ID："+(curGoods.presentIdentify||curGoods.id)}}</i-col>
                <i-col v-if="curGoods.availableNum"
                  span="12"
                  style="text-align:left;font-size:12px;color:#9AA8B8"
                >{{"库存："+curGoods.availableNum}}</i-col>
              </Row>
              <Row v-if="goodsType=='1'">
                <i-col
                  span="20"
                  style="text-align:left;font-size:12px;color:#9AA8B8"
                >{{"日期："+$format(curGoods.createTime)}}</i-col>
              </Row>
              <Row v-if="goodsType=='2'&&curGoods.createTime">
                <i-col
                  span="20"
                  style="text-align:left;font-size:12px;color:#9AA8B8"
                >{{"日期："+$format(curGoods.createTime[0])+"-"+$format(curGoods.createTime[1])}}</i-col>
              </Row>
            </div>
          </div>
          <div class="show_bottom" @click="modifyType" v-if="!isEdit">变更{{title}}</div>
        </div> 
      </div>
      <div v-else>
        <CouponList
          :couponList="[curGoods]"
          :spinShow="false"
          :checkedIds="[]"
          origin="activity"
        >
          <!-- <template #editbottom="{item}">
            <div @click="modifyType" class="coup_tip">
              <a href="javascirpt:void(0);">变更优惠券</a>
            </div>
          </template> -->
        </CouponList>
      </div>
      <a class="ml-10" @click="modifyType" v-if="isMall&&!isEdit"><Icon type="ios-add-circle-outline" />变更{{title}}</a>
    </div>
    <div v-else>
      <a
          class="linBlock"
          @click="modifyType"
        >
          <Icon type="ios-add-circle-outline" style="font-size:16px" />{{goodsType==1?'添加'+title:'添加优惠券'}}
        </a>
    </div>
    <Modal
        v-model="modalVisible" class-name="modal-box"
        :scrollable="true"
        :closable="false"
        :mask-closable="false"
        :width="goodsType==1?800:1480" :footer-hide="true"
        :title="`选择${title}`"
    >
      <div v-if="goodsType == 1&&modalVisible">
        <!-- 商城商品 -->
        <ChooseMallGoods v-if="isMall" @cancelGoods="cancelPresent" @confirmGoods="confirmPresent" :present="curGoods" :selectSkus="curSpecies" /> 
        <ChoosePresent @cancelPresent="cancelPresent" @confirmPresent="confirmPresent" v-else
          :present="curGoods" :url="presentUrl" :iCols="presentCols" />
      </div>
      <ChooseCoupon v-else @cancelCoupon="cancelPresent" @confirmCoupon="confirmPresent" :present="curGoods" />
    </Modal>
  </div>
</template>
<script>
import CouponList from "@/components/commons/Coupon/ViewCouponList";
import ChoosePresent from "./ChoosePresent"
import ChooseCoupon from "./ChooseCoupon"
import ChooseMallGoods from "./ChooseMallGoods"

export default {
  name:'goods-panel',
  components:{
    ChoosePresent,ChooseCoupon,CouponList,ChooseMallGoods
  },
  props:{
    goods:{type:[Object],required:true},
    isMall:Boolean,    // 是否新增商城商品
    mallSpecies:{type:Array,default:()=>[]} ,  // 商城商品对应规格
    goodsType:{type:[String,Number],required:true},
    title:{type:String,default:"拼团商品"},
    presentUrl:{type:String,default:'/mall-server/mall/present/list'},    //如果要拉取其他商品列表，可定义 api 接口
    presentCols:{type:Array,default:()=>[]},     // 可自定义表格 title信息
    isEdit:[Number,String],     // 是否为编辑，编辑不能更换商品类型和变更商品
  },
  data(){
    return {
      curGoods:this.goods,modalVisible:false,
      curSpecies:this.mallSpecies
    }
  },
  watch:{
    goods(nval){this.curGoods = nval}
  },
  methods:{
    modifyType(){
      this.modalVisible = true;
    },
    cancelPresent(){this.modalVisible = false;},
    confirmPresent(present,species){
      this.modalVisible = false;
      this.curGoods = present;
      this.curSpecies = species || [];    // 如果是商城商品，会有规格值
      this.$emit("getGoods",present,species);
    },
  }
}
</script>
<style lang="scss">
  // .modal-box{z-index: 99999 !important;}
  .show_info {
    width: 358px;
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    background: rgba(255, 255, 255, 1);
    border-radius: 4px;
    border: 1px solid rgba(225, 230, 235, 1);
    .show_top {
      width: 100%;
      height: 100%;
      display: flex;
      padding: 20px;
      justify-content: space-around;
      background: rgba(255, 255, 255, 1);
      border-radius: 4px;
      border: 1px solid rgba(225, 230, 235, 1);
    }
    .show_bottom {
      line-height: 40px;
      background: rgba(247, 249, 250, 1);
      border-radius: 0px 0px 2px 2px;
      border: 1px solid rgba(225, 230, 235, 1);
      text-align: center;
      color: #3d71ff;
      cursor: pointer;
    }
    .goods-box{
      padding:14px 16px; position: relative;
      .goods-header{
        h3{
          font-size:16px;
          font-family:PingFangSC-Medium,PingFang SC;
          font-weight:600;
          color:rgba(62,69,82,1);
        }
      }
      .ivu-poptip-confirm{position: absolute; right:10px; top:10px;i{font-size:20px;}}
      p{
        font-size:12px;
        font-family:PingFangSC-Regular,PingFang SC;
        font-weight:400;
        color:rgba(62,69,82,1);
      }
      p label{
        font-size:10px;
        font-family:PingFangSC-Regular,PingFang SC;
        font-weight:400;
        color:rgba(154,168,184,1);
      }
    }
  }  
</style>

<style lang="scss">
  
  
  .btn-box{
    text-align: right; border-top:1px solid #e8eaec;
    height: 60px; line-height: 60px; margin-top:10px;
    button{margin-left:20px;}
  }
</style>