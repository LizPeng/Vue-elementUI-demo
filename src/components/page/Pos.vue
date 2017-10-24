<template>
  <div class="pos">
   <div>
     <el-row>
       <el-col :span='7' id="order-list"  class="post-order">
        <el-tabs>
          <el-tab-pane label="点餐">
            <el-table :data="tableData" border style="width: 100%">
              <el-table-column prop="goodsName" label="商品"></el-table-column>
              <el-table-column prop="count" label="量" width="50"></el-table-column>
              <el-table-column prop="price" label="金额" width="70"></el-table-column>
              <el-table-column  fixed="right" label="操作"  width="110" >
                <template scope="scope">
                  <el-button type="text" size="small" @click="delSingleGoods(scope.row)">删除</el-button>
                  <el-button type="text" size="small" @click="addOrderList(scope.row)">增加</el-button>
                </template>
              </el-table-column>
            </el-table>
            <div class="totalDiv">
              <small>金额：</small>{{totalMoney}} 元  <small>数量：</small>{{totalCount}}
            </div>
            <div class="div-btn">
              <el-button type="warning">挂单</el-button>
              <el-button type="danger" @click="delAllGoods()">删除</el-button>
              <el-button type="primary">结账</el-button>
            </div>
          </el-tab-pane>
          <el-tab-pane label="挂单">挂单</el-tab-pane>
          <el-tab-pane label="外卖">外卖</el-tab-pane>
        </el-tabs>
       </el-col>
       <el-col :span='17'>
        <div class="often-goods">
          <div class="title">常用商品</div>
            <div class="often-goods-list">
              <ul>
                <li v-for="(goods, index) in oftenGoods" v-bind:key="index" @click="addOrderList(goods)">
                  <span>{{ goods.goodsName }}</span>
                  <span class="o-price">￥{{ goods.price}}</span>
                </li>
              </ul>
            </div>    
          </div>
          <div class="goods-type">
            <el-tabs>
              <el-tab-pane label="汉堡">
                <!--无序列表-->
                <div>
                  <ul class="cookList">
                    <li v-for="goods in type0Goods" v-bind:key="goods.goodsId" @click="addOrderList(goods)">
                      <span class="foodImg"><img v-bind:src="goods.goodsImg" width="100%"></span>
                      <span class="foodName">{{goods.goodsName}}</span>
                      <span class="foodPrice">￥{{ goods.price }}</span>
                    </li>
                  </ul>
                </div>
              </el-tab-pane>
              <el-tab-pane label="小食">
                <ul class="cookList">
                    <li v-for="goods in type1Goods" v-bind:key="goods.goodsId" @click="addOrderList(goods)">
                      <span class="foodImg"><img v-bind:src="goods.goodsImg" width="100%"></span>
                      <span class="foodName">{{goods.goodsName}}</span>
                      <span class="foodPrice">￥{{ goods.price }}</span>
                    </li>
                  </ul>
              </el-tab-pane>
              <el-tab-pane label="饮料">
                <ul class="cookList">
                    <li v-for="goods in type2Goods" v-bind:key="goods.goodsId" @click="addOrderList(goods)">
                      <span class="foodImg"><img v-bind:src="goods.goodsImg" width="100%"></span>
                      <span class="foodName">{{goods.goodsName}}</span>
                      <span class="foodPrice">￥{{ goods.price }}</span>
                    </li>
                  </ul>
              </el-tab-pane>
              <el-tab-pane label="汉堡">
                <ul class="cookList">
                    <li v-for="goods in type3Goods" v-bind:key="goods.goodsId" @click="addOrderList(goods)">
                      <span class="foodImg"><img v-bind:src="goods.goodsImg" width="100%"></span>
                      <span class="foodName">{{goods.goodsName}}</span>
                      <span class="foodPrice">￥{{ goods.price }}</span>
                    </li>
                  </ul>
              </el-tab-pane>
            </el-tabs>
        </div>
       </el-col>
     </el-row>
   </div>
  </div>
</template>

<script>
  import axios from 'axios'
  export default {
    name: 'Pos',
    data () {
      return {
        tableData: [],
        oftenGoods: [],
        type0Goods: [],
        type1Goods: [],
        type2Goods: [],
        type3Goods: [],
        totalMoney: 0,
        totalCount: 0
      }
    },
    mounted: function () {
      var orderHeight = document.body.clientHeight
      document.getElementById('order-list').style.height = orderHeight + 'px'
    },
    created: function () {
      axios.get('http://jspang.com/DemoApi/oftenGoods.php')
        .then(response => {
          // console.log(response)
          this.oftenGoods = response.data
        })
        .catch(error => {
          console.log(error)
          alert('网络错误')
        })
      // 读取分类商品列表
      axios.get('http://jspang.com/DemoApi/typeGoods.php')
        .then(response => {
          console.log(response)
          // this.oftenGoods=response.data;
          this.type0Goods = response.data[0]
          this.type1Goods = response.data[1]
          this.type2Goods = response.data[2]
          this.type3Goods = response.data[3]
        })
        .catch(error => {
          console.log(error)
        })
    },
    methods: {
      // 添加订单列表的方法
      addOrderList (goods) {
        let isHave = false
        // 判断这个商品是否存在于订单列表
        for (let i = 0; i < this.tableData.length; i++) {
          console.log(this.tableData[i].goodsId)
          if (this.tableData[i].goodsId === goods.goodsId) {
            isHave = true
          }
        }
        if (isHave) {
          let arr = this.tableData.filter(o => o.goodsId === goods.goodsId)
          arr[0].count++
        } else {
          let newGoods = {goodsId: goods.goodsId, goodsName: goods.goodsName, price: goods.price, count: 1}
          this.tableData.push(newGoods)
        }
        this.getAllMoney()
      },
      // 删除单个商品
      delSingleGoods (goods) {
        console.log(goods)
        this.tableData = this.tableData.filter(o => o.goodsId !== goods.goodsId)
        this.getAllMoney()
      },
      getAllMoney () {
        this.totalCount = 0
        this.totalMoney = 0
        if (this.tableData) {
          this.tableData.forEach((element) => {
            this.totalCount += element.count
            this.totalMoney = this.totalMoney + (element.price * element.count)
          })
        }
      },
      delAllGoods () {
        this.tableData = []
        this.totalCount = 0
        this.totalMoney = 0
      },
      checkout () {
      }
    }
  }
</script>

<style scoped>
  .post-order{
    background-color: #f9fafc;
    border-right: 1px solid #c0ccda;
  }
  .div-btn{
    margin-top: 10px;
  }
  .title{
       height: 20px;
       border-bottom:1px solid #D3DCE6;
       background-color: #F9FAFC;
       padding:10px;
   }
   .often-goods-list ul li{
      list-style: none;
      float:left;
      border:1px solid #E5E9F2;
      padding:10px;
      margin:5px;
      background-color:#fff;
      cursor: pointer;
   }
  .o-price{
      color:#58B7FF; 
   }
   .goods-type{
     clear: both;
   }
  .cookList li{
       list-style: none;
       width:23%;
       border:1px solid #E5E9F2;
       height: auto;
       overflow: hidden;
       background-color:#fff;
       padding: 2px;
       float:left;
       margin: 2px;
       cursor: pointer;
   }
   .cookList li span{
        display: block;
        float:left;
   }
   .foodImg{
       width: 40%;
   }
   .foodName{
       font-size: 18px;
       padding-left: 10px;
       color:brown;
   }
   .foodPrice{
       font-size: 16px;
       padding-left: 10px;
       padding-top:10px;
   }
   .totalDiv{
     background-color: #fff;
     padding:10px;
     border-bottom: 1px solid 
   }
</style>
