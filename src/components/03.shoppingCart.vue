<template>
    <div>
        <div class="section">
            <div class="location">
                <span>当前位置：</span>
                <a href="/index.html">首页</a> &gt;
                <a href="/cart.html">购物车</a>
            </div>
        </div>

        <div class="section">
            <div class="wrapper">
                <div class="bg-wrap">
                    <!--购物车头部-->
                    <div class="cart-head clearfix">
                        <h2>
                            <i class="iconfont icon-cart"></i>我的购物车</h2>
                        <div class="cart-setp">
                            <ul>
                                <li class="first active">
                                    <div class="progress">
                                        <span>1</span>
                                        放进购物车
                                    </div>
                                </li>
                                <li>
                                    <div class="progress">
                                        <span>2</span>
                                        填写订单信息
                                    </div>
                                </li>
                                <li class="last">
                                    <div class="progress">
                                        <span>3</span>
                                        支付/确认订单
                                    </div>
                                </li>
                            </ul>
                        </div>
                    </div>
                    <!--购物车头部-->
                    <!--商品列表-->
                    <div class="cart-box">
                        <input id="jsondata" name="jsondata" type="hidden">
                        <table width="100%" align="center" class="cart-table" border="0" cellspacing="0" cellpadding="8">
                            <tbody>
                                <tr>
                                    <th width="48" align="center">
                                        <a>选择</a>
                                    </th>
                                    <th align="left" colspan="2">商品信息</th>
                                    <th width="84" align="left">单价</th>
                                    <th width="104" align="center">数量</th>
                                    <th width="104" align="left">金额(元)</th>
                                    <th width="54" align="center">操作</th>
                                </tr>
                                <!-- 没有商品时显示 -->
                                <tr v-if="message.length==0">
                                    <td colspan="10">
                                        <div class="msg-tips">
                                            <div class="icon warning">
                                                <i class="iconfont icon-tip"></i>
                                            </div>
                                            <div class="info">
                                                <strong>购物车没有商品！</strong>
                                                <p>您的购物车为空，
                                                    <router-link to="/index">马上去购物</router-link>吧！
                                                </p>
                                            </div>
                                        </div>
                                    </td>
                                </tr>
                                <!-- 商品列表 -->
                                <tr v-for="(item, index) in message" :key="item.id">
                                    <td>
                                        <el-switch v-model="item.selected" active-color="black" inactive-color="hotpink">
                                        </el-switch>
                                    </td>
                                    <td>
                                        <img style="width:65px" :src="item.img_url" alt="">
                                    </td>
                                    <td>
                                        {{item.title}}
                                    </td>
                                    <td>{{item.sell_price}}</td>
                                    <td>
                                        <!-- 直接min='0' 解析的是 字符串 -->
                                        <!-- 如果 要保留 原始的形参 ,额外的增加一个自定义的实参
                                            $event 保留事件参数
                                                传送门
                                            在后面增加自定义的参数
                                         -->
                                        <el-input-number size="mini" :min='0' :max="item.max" v-model="item.buycount" @change="numChange($event,item.id)"></el-input-number>
                                    </td>
                                    <td>{{item.buycount*item.sell_price}}</td>
                                    <!-- 使用element-ui的按钮 传入 id -->
                                    <td>
                                        <el-button @click="delOne(item.id)" type="danger" icon="el-icon-delete" circle></el-button>
                                    </td>
                                </tr>
                                <tr>
                                    <th align="right" colspan="8">
                                        已选择商品
                                        <b class="red" id="totalQuantity">{{totalCount}}</b> 件 &nbsp;&nbsp;&nbsp; 商品总金额（不含运费）：
                                        <span class="red">￥</span>
                                        <b class="red" id="totalAmount">{{totalPrice}}</b>元
                                    </th>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                    <!--/商品列表-->
                    <!--购物车底部-->
                    <div class="cart-foot clearfix">
                        <div class="right-box">
                            <button class="button" onclick="javascript:location.href='/index.html';">继续购物</button>
                            <button class="submit" @click="toOrder">立即结算</button>
                        </div>
                    </div>
                    <!--购物车底部-->
                </div>
            </div>
        </div>

    </div>
</template>
<script>
export default {
  // 在开发者工具中看到组件的名字
  name: "shoppingCart",
  data: function() {
    return {
      message: []
    };
  },
  // 生命周期函数(钩子函数)
  created() {
    // 准备数据 id1,id2,id3
    // console.log(this.$store.state.cartDate);
    // 定义变量暂存一个
    let cartDate = this.$store.state.cartDate;
    // 定义拼接的数据
    let ids = "";
    for (const key in cartDate) {
      ids += key;
      ids += ",";
    }
    // 去掉最后多余的逗号 第二个-1的意思是 最后一个 不要
    ids = ids.slice(0, -1);
    // console.log(ids);
    // 调用接口获取数据
    this.$axios.get(`/site/comment/getshopcargoods/${ids}`).then(response => {
      // 因为服务器返回的数据 没有购买数量 他们也不知道
      // 我们自行拼接个数
      response.data.message.forEach(v => {
        // 获取 Vuex中的 id对应的值
        v.buycount = cartDate[v.id];
        // 设置是否被选中
        v.selected = true;
        // 可以获取到 当前这个商品的 id
        // 带着这个id 去调用接口 即可
        this.$axios.get(`site/goods/getgoodsinfo/${v.id}`).then(resSon=>{
            // console.log(resSon);
            // 把库存 保存到 当前循环的那个对象中即可
            v.max = resSon.data.message.goodsinfo.stock_quantity
        })
      });
      // 再赋值给 message即可(后台真懒!!)
      this.message = response.data.message;
    });
  },
  // 计算属性
  computed: {
    // 总金额
    totalPrice() {
      // 被选中的
      let totalPrice = 0;
      // 购买个数*价格
      this.message.forEach(v => {
        if (v.selected == true) {
          // 累加
          // 返回
          totalPrice += v.sell_price * v.buycount;
        }
      });
      return totalPrice;
    },
    // 总个数
    totalCount() {
      // 被选中的
      let totalCount = 0;
      // 购买个数*价格
      this.message.forEach(v => {
        if (v.selected == true) {
          // 累加
          // 返回
          // 变为了字符串拼接 转化为整数即可
          totalCount += parseInt(v.buycount);
        }
      });
      return totalCount;
    }
  },
  // 方法
  methods: {
    numChange(num, id) {
      //   console.log(num,id);
      // 调用仓库的方法 (提交载荷)
      // 提交载荷的时候 判断数量是否 超过了 最大值
      this.message.forEach(v=>{
          if(v.id==id){
              if(num>v.max){
                  num = v.max;
              }
          }
      })
      // 上面已经修正了 num的数量
      this.$store.commit("updateGoodsNum", {
        goodId: id,
        goodNum: num
      });
    },
    // 删除数据
    delOne(id) {
      //   console.log(id);
      // 提交载荷 这里是删除 Vuex中的
      this.$store.commit("deleteGoods", id);
      // 页面中的 并没有删除
      // this.message
      this.message.forEach((v, index) => {
        if (v.id == id) {
          this.message.splice(index, 1);
        }
      });
    },
    // 去订单页
    toOrder(){
        // 数据拼接
        let ids = '';
        this.message.forEach(v=>{
            if(v.selected==true){
                ids+=v.id;
                ids+=',';
            }
        })
        ids=ids.slice(0,-1);
        this.$router.push('/order/'+ids);
        // 判断让 导航守卫 (main.js中)
    }
  }
};
</script>
<style>
</style>


