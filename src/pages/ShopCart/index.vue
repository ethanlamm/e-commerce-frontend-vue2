<template>
  <div>
    <div class="cart" v-if="cartInfoList.length > 0">
      <h4>全部商品</h4>
      <div class="cart-main">
        <div class="cart-th">
          <div class="cart-th1">全部</div>
          <div class="cart-th2">商品</div>
          <div class="cart-th3">单价（元）</div>
          <div class="cart-th4">数量</div>
          <div class="cart-th5">小计（元）</div>
          <div class="cart-th6">操作</div>
        </div>
        <!-- 列表 -->
        <div class="cart-body">
          <ul class="cart-list" v-for="cart in cartInfoList" :key="cart.id">
            <!-- 勾选框 -->
            <li class="cart-list-con1">
              <input
                type="checkbox"
                name="chk_list"
                :checked="cart.isChecked == 1"
                @change="changeIsChecked(cart, $event)"
              />
            </li>
            <!-- 商品 -->
            <li class="cart-list-con2">
              <img :src="cart.imgUrl" />
              <div class="item-msg">
                {{ cart.skuName }}
              </div>
            </li>
            <!-- 单价 -->
            <li class="cart-list-con4">
              <span class="price">{{ cart.skuPrice }}</span>
            </li>
            <!-- 数量加减 -->
            <li class="cart-list-con5">
              <a class="mins" @click="handler(-1, cart)">-</a>
              <input
                autocomplete="off"
                type="text"
                minnum="1"
                class="itxt"
                :value="cart.skuNum"
                @change="handler($event.target.value * 1, cart)"
              />
              <a class="plus" @click="handler(1, cart)">+</a>
            </li>
            <!-- 小记 -->
            <li class="cart-list-con6">
              <span class="sum">{{ cart.skuPrice * cart.skuNum }}</span>
            </li>
            <!-- 操作 -->
            <li class="cart-list-con7">
              <a class="sindelet" @click="deleteCart(cart)">删除</a>
              <br />
              <a href="#none">移到收藏</a>
            </li>
          </ul>
        </div>
      </div>
      <!-- 底部栏 -->
      <div class="cart-tool">
        <div class="select-all">
          <input
            class="chooseAll"
            type="checkbox"
            :checked="isAllChecked && cartInfoList.length > 0"
            @change="checkAll"
          />
          <span>全选</span>
        </div>
        <div class="option">
          <a @click="deleteCheckedCart">删除选中的商品</a>
          <a href="#none">移到我的关注</a>
          <a href="#none">清除下柜商品</a>
        </div>
        <div class="money-box">
          <div class="chosed">已选择 <span>0</span>件商品</div>
          <div class="sumprice">
            <em>总价（不含运费） ：</em>
            <i class="summoney">{{ totalPrice }}</i>
          </div>
          <div class="sumbtn">
            <router-link class="sum-btn" to="/trade">结算</router-link>
          </div>
        </div>
      </div>
    </div>
    <div v-else><h2>购物车空空如也(JD做法，有待优化)</h2></div>
  </div>
</template>

<script>
import { mapState, mapActions, mapMutations, mapGetters } from "vuex";
import throttle from "lodash/throttle";
export default {
  name: "ShopCart",
  mounted() {
    this.getData();
  },
  computed: {
    ...mapGetters("shopcartOptions", ["cartInfoList"]),
    totalPrice() {
      let sum = 0;
      this.cartInfoList.forEach((item) => {
        sum += item.skuPrice * item.skuNum;
      });
      return sum;
    },
    isAllChecked() {
      return this.cartInfoList.every((item) => item.isChecked == 1);
    },
  },
  methods: {
    // 渲染页面
    getData() {
      this.$store.dispatch("shopcartOptions/getShopCartList");
    },
    // 数量加减[节流]
    handler: throttle(async function (num, cart) {
      if (num == 1) {
        // 加
        num = 1;
      } else if (num == -1) {
        // 减
        num = cart.skuNum >= 2 ? -1 : 0;
      } else {
        // 输入
        if (isNaN(num) | (num < 1)) {
          // 非法输入字符
          num = 0;
        } else {
          num = parseInt(num) - cart.skuNum;
        }
      }
      try {
        await this.$store.dispatch("detailOptions/updateShopCart", {
          skuId: cart.skuId,
          skuNum: num,
        });
        this.getData();
      } catch (error) {
        console.log(error.message);
      }
    }, 1000),
    // 删除单个商品
    async deleteCart(cart) {
      try {
        await this.$store.dispatch(
          "shopcartOptions/deleteCartById",
          cart.skuId
        );
        // 重新渲染页面
        this.getData();
      } catch (error) {
        console.log(err.message);
      }
    },
    // 删除选中的商品(底部栏)
    async deleteCheckedCart() {
      try {
        await this.$store.dispatch("shopcartOptions/deleteIsCheckedCart");
        this.getData();
      } catch (error) {
        console.log(err.message);
      }
    },
    // 修改商品选中状态(单个)
    async changeIsChecked(cart, event) {
      try {
        await this.$store.dispatch("shopcartOptions/changeCartIsChecked", {
          skuId: cart.skuId,
          isChecked: event.target.checked ? "1" : "0",
        });
        // 重新渲染页面
        this.getData();
      } catch (error) {
        console.log(err.message);
      }
    },
    // 全选
    async checkAll(e) {
      try {
        let checkedAll = e.target.checked ? "1" : "0";
        await this.$store.dispatch(
          "shopcartOptions/updateCartChecked",
          checkedAll
        );
        this.getData();
      } catch (error) {
        console.log(error.message);
      }
    },
  },
};
</script>

<style lang="less" scoped>
h2 {
  text-align: center;
  margin: 50px;
}
.cart {
  width: 1200px;
  margin: 0 auto;

  h4 {
    margin: 9px 0;
    font-size: 14px;
    line-height: 21px;
  }

  .cart-main {
    .cart-th {
      background: #f5f5f5;
      border: 1px solid #ddd;
      padding: 10px;
      overflow: hidden;

      & > div {
        float: left;
      }

      .cart-th1 {
        width: 25%;

        input {
          vertical-align: middle;
        }

        span {
          vertical-align: middle;
        }
      }

      .cart-th2 {
        width: 25%;
      }

      .cart-th3,
      .cart-th4,
      .cart-th5,
      .cart-th6 {
        width: 12.5%;
      }
    }

    .cart-body {
      margin: 15px 0;
      border: 1px solid #ddd;

      .cart-list {
        padding: 10px;
        border-bottom: 1px solid #ddd;
        overflow: hidden;

        & > li {
          float: left;
        }

        .cart-list-con1 {
          width: 15%;
        }

        .cart-list-con2 {
          width: 35%;

          img {
            width: 82px;
            height: 82px;
            float: left;
          }

          .item-msg {
            float: left;
            width: 150px;
            margin: 0 10px;
            line-height: 18px;
          }
        }

        .cart-list-con4 {
          width: 10%;
        }

        .cart-list-con5 {
          width: 17%;

          .mins {
            border: 1px solid #ddd;
            border-right: 0;
            float: left;
            color: #666;
            width: 6px;
            text-align: center;
            padding: 8px;
          }

          input {
            border: 1px solid #ddd;
            width: 40px;
            height: 33px;
            float: left;
            text-align: center;
            font-size: 14px;
          }

          .plus {
            border: 1px solid #ddd;
            border-left: 0;
            float: left;
            color: #666;
            width: 6px;
            text-align: center;
            padding: 8px;
          }
        }

        .cart-list-con6 {
          width: 10%;

          .sum {
            font-size: 16px;
          }
        }

        .cart-list-con7 {
          width: 13%;

          a {
            color: #666;
          }
        }
      }
    }
  }

  .cart-tool {
    overflow: hidden;
    border: 1px solid #ddd;

    .select-all {
      padding: 10px;
      overflow: hidden;
      float: left;

      span {
        vertical-align: middle;
      }

      input {
        vertical-align: middle;
      }
    }

    .option {
      padding: 10px;
      overflow: hidden;
      float: left;

      a {
        float: left;
        padding: 0 10px;
        color: #666;
      }
    }

    .money-box {
      float: right;

      .chosed {
        line-height: 26px;
        float: left;
        padding: 0 10px;
      }

      .sumprice {
        width: 200px;
        line-height: 22px;
        float: left;
        padding: 0 10px;

        .summoney {
          color: #c81623;
          font-size: 16px;
        }
      }

      .sumbtn {
        float: right;

        a {
          display: block;
          position: relative;
          width: 96px;
          height: 52px;
          line-height: 52px;
          color: #fff;
          text-align: center;
          font-size: 18px;
          font-family: "Microsoft YaHei";
          background: #e1251b;
          overflow: hidden;
        }
      }
    }
  }
}
</style>