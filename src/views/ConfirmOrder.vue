<template>
    <Header></Header>
    <div class="confirmOrder">
      <div class="bgColor">
          <h1 class="main-shopcart">确认订单</h1>
      </div>
      <div class="main">
            <div class="info">
                <div class="head">商品信息</div>
                <div class="info-main" v-for="item in courses" :key='item.id'>
                    <div class="courseInfo">
                        <div class="coursebg">
                            <img :src="item.courseCover" alt="">
                        </div>
                        <div class="courseName">
                           {{item.courseName}}
                        </div>
                        <div class="coursePrice">
                            <span class="nowprice">￥{{item.discountPrice}}</span>
                            <span class="oldprice">￥{{item.salePrice}}</span>
                        </div>
                    </div>
                </div>
                <div class="choose">
                    <h3>支付方式 <span class="pay">{{payment.description}}</span></h3>
                    <div class="choosebg">
                        <span v-for="item in payModes" :key="item.code" @click="tabPayment(item)">{{item.description}}</span>
                    </div>
                </div>
                <ul class="foot">
                    <li class="foot-item">应付<span class="unique">￥{{totalPrice}}</span></li>
                    <li>
                        <button class="btn" @click="goPayment">确认订单</button>
                    </li>
                </ul>
            </div>
        </div>
        <!-- 二维码对话框 -->
        <el-dialog v-model='dialogVisible' class="pay-dialog" width="500px">
            <div class="dialogPrice">支付：<span class="prices">￥{{totalPrice}}</span></div>
            <div class="codeimg">
                <img :src="parUrl" />
            </div>
            <div class="alert">请您及时付款，已便订单尽快处理！</div>
            <div class="finish">
                <div class="error">支付遇到问题</div>
                <div class="success">我已支付</div>
            </div>
        </el-dialog>
    </div>
    <Foot></Foot>
</template>

<script setup>
//组件
import Header from '../components/common/Header.vue'
import Foot from '../components/common/Foot.vue'
//dialog
const dialogVisible = ref(true);
import {user} from '../store/user'
import { onBeforeMount, reactive } from '@vue/runtime-core';
import {createToken,settlement,wxcreateOrder,alicreateOrder,aliqueryOrder,wxqueryOrder,deleteShopCars} from '../api/index'
let store=user()
let router=useRouter()
//总价钱
let totalPrice=ref(0)
//结算的商品数据
let courses=ref([])
//当前的支付方式
let payment=reactive({
    description:'支付宝支付'
})
//支付方式列表
let payModes=ref([])
onBeforeMount(()=>{
    settlement(store.orderList).then(res=>{
        courses.value=res.data.courses
        totalPrice.value=res.data.totalPrice
        payModes.value=res.data.payModes
    })
})
//支付的二维码
let parUrl=ref('')
//支付的订单号
let orderNumber=ref('')
//选择支付的方式
const tabPayment=(item)=>{
    payment.description=item.description
    payment.code=item.code
    if(payment.code=='alipayment')
    {
        alicreateOrder({
            courses:store.orderList,
            payModes:item.code
        }).then(res=>
        {
            parUrl.value=res.data.payurl
            orderNumber.value=res.data.orderNumber
       })
    }
    else{
        wxcreateOrder({
            courses:store.orderList,
            payModes:item.code
        }).then(res=>
        {
            parUrl.value=res.data.payurl
            orderNumber.value=res.data.orderNumber
        })
    }  
}
//轮询定时器
let timer=ref('')
import {ElMessageBox} from 'element-plus'
import {ElMessage} from 'element-plus'
import { useRouter } from 'vue-router';
//查询订单状态:支付宝
const interPaymentAli=()=>{
    aliqueryOrder({orderNumber:orderNumber.value}).then(res=>{
        console.log(res);
       if(res.meta.code=='200')
       {
         clearInterval(timer.value)
         ElMessage.success({
			    message:'支付成功'
	     })
         //跳转页面
         router.push({name:'Home'})
         //删除购物车数据
         createToken().then(res=>{
            deleteShopCars(store.selectList,res.data.token)
         })
       }
    })
}
//查询订单状态:微信
const interPaymentWx=()=>{
    wxqueryOrder({orderNumber:orderNumber.value}).then(res=>{
       console.log(res);
       if(res.meta.code=='200')
       {
         clearInterval(timer.value)
         ElMessage.success({
			    message:'支付成功'
	    })
         //跳转页面
         router.push({name:'Home'})
         //删除购物车数据
         createToken().then(res=>{
            deleteShopCars(store.selectList,res.data.token)
        })
       }
    })
}
//去支付
const goPayment=()=>{
  if( payment.code =='alipayment' ){
    timer.value = setInterval( interPaymentAli , 2000 );
  }else{
    timer.value = setInterval( interPaymentWx , 2000 );
  }
  dialogVisible.value = true;
}
</script>

<style scoped>
.finish{
  width: 170px;

  line-height: 30px;
  margin: 0 auto;
  display: flex;
}
.success{
  margin-left: 20px;
  width: 70px;
  font-size:12px ;
  background: rgba(54, 137, 255, 0.22);
  color: #3692ff;
  cursor: pointer;
  border-radius: 8px;
}
.error{
  width: 100px;
  font-size:12px ;
  background: linear-gradient(90deg, #fc7979 0%, #da4848 100%);
  color: #ffffff;
  cursor: pointer;
  border-radius: 8px;

}
.el-dialog {
    text-align: center !important;
    border-radius: 10px!important;
}
.dialogPrice{
    padding-bottom: 20px;
    color: #93999F;
}
.prices{
    color: #F01414;
}
.alert{
    padding: 20px 0;
    font-size: 14px;
    color: #93999F;

}
.confirmOrder{
    width: 100%;
    padding-bottom:60px;
}
.bgColor{
  width: 100%;
  height: 200px;
  background-color: red;
  background: linear-gradient(-45deg, #ee7752, #e73c7e, #23a6d5, #23d5ab);
  background-size: 400% 400%;
  animation: gradient 15s ease infinite;
}
.main-shopcart{
    width: 1200px;
    margin: 0 auto;
    height: 42px;
    font-size: 24px;
    font-family: Microsoft YaHei;
    font-weight: bold;
    line-height: 35px;
    color: #FFFFFF;
    padding: 30px 0;
    opacity: 1;
}
@keyframes gradient {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}
.main{
    width: 1200px;
    margin: -100px auto 0 auto;
  /*background-color: red;*/
}
.main h1{
    color: #FFFFFF;
    padding: 30px 0 30px 47px;

}
.info{
    width: 1200px;
    padding: 5px 0px 20px 0px;
    background: #F3F5F7;
    border-radius: 10px;
  box-shadow: 0px 2px 5px #888888;
}
.info-main{
    margin: 10px 20px;
    background: #FFFFFF;
    border-radius: 10px;
    padding: 20px;
}
.head{
    padding: 20px;
    font-size: 18px;
    color: #333333;
}
/* 课程信息 */
.courseInfo{
    display: flex;
    width: 100%;
    height: 160px;
    margin: 0 auto;
}
.coursebg{
    width: 280px;
    height: 160px;
}
.coursebg img{
    width: 100%;
    height: 100%;
}
.courseName{
    margin:0 20px;
    width: 400px;
    height: 160px;
    font-size: 16px;
    color: #07111B;
    line-height:160px ;
    word-break:keep-all;
    white-space:nowrap;
    overflow:hidden;
    text-overflow:ellipsis;
}
.coursePrice{
    margin-left: 100px;
    height: 160px;
    text-align: right;
    line-height: 160px;
}
.coursePrice .nowprice{
    font-size: 18px;
    font-weight: 600;
    color: #1C1F21;
}
.coursePrice .oldprice{
    padding-left: 10px;
    font-size: 14px;
    text-decoration: line-through;
    color: #93999F;

}
/* 支付开始 */
.choose{
    width: 1200px;
    margin: 0 auto;
}
.choose h3{
    color: #222;
    font-size: 16px;
    font-weight: 400;
    padding: 0 20px;
}
.choosebg{
    display: flex;
    margin: 20px;
}
.choosebg span{
  margin-top: 60px;
  margin-right:40px;  
}
.payment{
    width: 130px;
    height: 50px;
    border-radius: 10px;
    cursor: pointer;
    margin-right: 10px;
    line-height: 50px;
    display: flex;
    padding: 0 10px;
    align-items: center;
}
.payment i{
    font-size: 35px;
    margin-right: 10px;
}
.payment span{
    line-height: 50px;
    color: #222222;
    font-weight: bold;
}

.payStyle{
  background: url("/image/checkedVip.png") no-repeat;
  background-size: 220px 111px;
  background-position: -67px -59px;
  border: #ff470a solid 1px !important;
}
.alipayment{
  border: #bfbfbf solid 1px;
  color: #01a8eb;
}
.wxpayment{
  border: #bfbfbf solid 1px;
  color: #01af37;
}

/* 支付结束 */
/* 结算 */
.foot{
    display: flex;
    justify-content: flex-end;
    width: 100%;
    height: 50px;
    line-height: 50px;
    color: #333333;
    margin-bottom: 10px;
}
.foot-item{
    width: 200px;
    height: 50px;
    line-height: 50px;
    font-size: 14px;
    font-weight: 400;
    color: #333333;
}
.foot-item:first-child{
    margin-top: 3px;
}
.old{
    margin-left: 10px;
}
.unique{
    margin-left: 5px;
    font-size: 24px;
    color: #F01414;
}
.btn{
    margin-right: 20px;
    width: 150px;
    height: 50px;
    border: none;
    color: #FFFFFF;
    font-size: 20px;
    border-radius: 5px;
    background: #F01414;
    cursor: pointer;
    box-shadow: 0px 3px 5px 2px #ff727f;
}
.pay{
    font-size: 18px;
    padding-left: 10px;
    color:#F01414 ;
}
.codeimg{
    margin:0 auto;
    border: #d2d2d2 solid 1px;
    width: 150px;
    height: 150px;
}
.codeimg img{
    width: 100%;
    height: 100%;
}
</style>