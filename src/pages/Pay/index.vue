<template>
  <div class="pay-main">
    <div class="pay-container">
      <div class="checkout-tit">
        <h4 class="tit-txt">
          <span class="success-icon"></span>
          <span class="success-info"
            >订单提交成功，请您及时付款，以便尽快为您发货~~</span
          >
        </h4>
        <div class="paymark">
          <!-- 订单号 -->
          <span class="fl"
            >请您在提交订单<em class="orange time">4小时</em
            >之内完成支付，超时订单会自动取消。订单号：<em>{{
              orderId
            }}</em></span
          >
          <!-- 应付金额 -->
          <span class="fr"
            ><em class="lead">应付金额：</em
            ><em class="orange money">￥{{ totalFee }}</em></span
          >
        </div>
      </div>
      <div class="checkout-info">
        <h4>重要说明：</h4>
        <ol>
          <li>
            尚品汇商城支付平台目前支持<span class="zfb">支付宝</span>支付方式。
          </li>
          <li>其它支付渠道正在调试中，敬请期待。</li>
          <li>为了保证您的购物支付流程顺利完成，请保存以下支付宝信息。</li>
        </ol>
        <h4>
          支付宝账户信息：（很重要，<span class="save">请保存！！！</span>）
        </h4>
        <ul>
          <li>支付帐号：11111111</li>
          <li>密码：111111</li>
          <li>支付密码：111111</li>
        </ul>
      </div>
      <div class="checkout-steps">
        <div class="step-tit">
          <h5>支付平台</h5>
        </div>
        <div class="step-cont">
          <ul class="payType">
            <li><img src="./images/pay2.jpg" /></li>
            <li><img src="./images/pay3.jpg" /></li>
          </ul>
        </div>
        <div class="hr"></div>

        <div class="payshipInfo">
          <div class="step-tit">
            <h5>支付网银</h5>
          </div>
          <div class="step-cont">
            <ul class="payType">
              <li><img src="./images/pay10.jpg" /></li>
              <li><img src="./images/pay11.jpg" /></li>
              <li><img src="./images/pay12.jpg" /></li>
              <li><img src="./images/pay13.jpg" /></li>
              <li><img src="./images/pay14.jpg" /></li>
              <li><img src="./images/pay15.jpg" /></li>
              <li><img src="./images/pay16.jpg" /></li>
              <li><img src="./images/pay17.jpg" /></li>
              <li><img src="./images/pay18.jpg" /></li>
              <li><img src="./images/pay19.jpg" /></li>
              <li><img src="./images/pay20.jpg" /></li>
              <li><img src="./images/pay21.jpg" /></li>
              <li><img src="./images/pay22.jpg" /></li>
            </ul>
          </div>
        </div>
        <div class="hr"></div>

        <div class="submit">
          <a class="btn" @click="open">立即支付</a>
        </div>
        <div class="otherpay">
          <div class="step-tit">
            <h5>其他支付方式</h5>
          </div>
          <div class="step-cont">
            <span><a href="weixinpay.html" target="_blank">微信支付</a></span>
            <span>中国银联</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import QRCode from "qrcode";
export default {
  name: "Pay",
  data() {
    return {
      timer: null,
      code: 201,
      code_success: 200,
    };
  },
  // 组件内独享路由
  beforeRouteLeave(to, from, next) {
    if (to.name == "paysuccess") {
      let leave_code = 200;
      if (this.code_success == leave_code) {
        next();
      } else {
        alert("请先支付！");
        clearInterval(this.timer);
        this.timer = null;
        next(false);
      }
    } else {
      next();
    }
  },
  computed: {
    payInfo() {
      return {};
    },
    orderId() {
      return this.payInfo.orderId || "";
    },
    totalFee() {
      return this.payInfo.totalFee || "";
    },
  },
  mounted() {
    // this.getOrderInfo();
  },
  methods: {
    // 获取订单信息
    // async getOrderInfo() {
    //   let result = await this.$api.reqOrderInfo(this.orderId);
    //   this.payInfo = result.data;
    // },
    // 弹窗
    async open() {
      // let url = await QRCode.toDataURL(this.payInfo.codeUrl);
      let url1 = await QRCode.toDataURL("https://www.aconvert.com");

      this.$alert(`<img src="${url1}">`, "请您微信支付", {
        dangerouslyUseHTMLString: true,
        center: true, // 布局居中
        showClose: false, // 关闭右上角X按钮
        showCancelButton: true, // 显示取消按钮(增加按钮)
        cancelButtonText: "支付遇到问题",
        confirmButtonText: "支付成功",
        // 手动关闭弹窗要进行判断
        beforeClose: (type, instance, done) => {
          if (type == "cancel") {
            clearInterval(this.timer);
            console.log("清空定时器");
            done();
            console.log("关闭弹窗");
            this.$router.push("/help");
            console.log("路由跳转");
            this.timer = null;
            console.log("重置定时器");
          } else {
            if (this.code_success == 200) {
              clearInterval(this.timer);
              console.log("清空定时器");
              done();
              console.log("关闭弹窗");
              console.log("11111111");
              this.$router.push("/paysuccess");
              console.log("路由跳转");
              this.timer = null;
              console.log("重置定时器");
            }
          }
        },
      });

      if (!this.timer) {
        // 发请求查询支付状态
        this.timer = setInterval(async () => {
          // let result = await this.$api.reqPayInfo(this.orderId);
          // code==200--支付成功    code==205--支付中
          console.log("发请求");
          if (this.code == 200) {
            // 支付成功
            console.log("支付成功");
            clearInterval(this.timer);
            console.log("清空定时器");
            // 保存code
            // this.code = result.code;
            console.log("保存code");
            // 关闭弹窗
            this.$msgbox.close();
            console.log("关闭弹窗");
            // 路由跳转至成功页面
            // this.$router.push("/paysuccess");
            console.log("路由跳转");
            this.timer = null;
            console.log("重置定时器");
          }
        }, 2500);
      }
    },
  },
};
</script>

<style lang="less" scoped>
.pay-main {
  margin-bottom: 20px;

  .pay-container {
    margin: 0 auto;
    width: 1200px;

    a:hover {
      color: #4cb9fc;
    }

    .orange {
      color: #e1251b;
    }

    .money {
      font-size: 18px;
    }

    .zfb {
      color: #e1251b;
      font-weight: 700;
    }

    .checkout-tit {
      padding: 10px;

      .tit-txt {
        font-size: 14px;
        line-height: 21px;

        .success-icon {
          width: 30px;
          height: 30px;
          display: inline-block;
          background: url(./images/icon.png) no-repeat 0 0;
        }

        .success-info {
          padding: 0 8px;
          line-height: 30px;
          vertical-align: top;
        }
      }

      .paymark {
        overflow: hidden;
        line-height: 26px;
        text-indent: 38px;

        .fl {
          float: left;
        }

        .fr {
          float: right;

          .lead {
            margin-bottom: 18px;
            font-size: 15px;
            font-weight: 400;
            line-height: 22.5px;
          }
        }
      }
    }

    .checkout-info {
      padding-left: 25px;
      padding-bottom: 15px;
      margin-bottom: 10px;
      border: 2px solid #e1251b;

      h4 {
        margin: 9px 0;
        font-size: 14px;
        line-height: 21px;
        color: #e1251b;
      }

      ol {
        padding-left: 25px;
        list-style-type: decimal;
        line-height: 24px;
        font-size: 14px;
      }

      ul {
        padding-left: 25px;
        list-style-type: disc;
        line-height: 24px;
        font-size: 14px;
      }
    }

    .checkout-steps {
      border: 1px solid #ddd;
      padding: 25px;

      .hr {
        height: 1px;
        background-color: #ddd;
      }

      .step-tit {
        line-height: 36px;
        margin: 15px 0;
      }

      .step-cont {
        margin: 0 10px 12px 20px;

        ul {
          font-size: 0;

          li {
            margin: 2px;
            display: inline-block;
            padding: 5px 20px;
            border: 1px solid #ddd;
            cursor: pointer;
            line-height: 18px;
          }
        }
      }
    }

    .submit {
      text-align: center;

      .btn {
        display: inline-block;
        padding: 15px 45px;
        margin: 15px 0 10px;
        font: 18px "微软雅黑";
        font-weight: 700;
        border-radius: 0;
        background-color: #e1251b;
        border: 1px solid #e1251b;
        color: #fff;
        text-align: center;
        vertical-align: middle;
        cursor: pointer;
        user-select: none;
        text-decoration: none;
      }
    }
  }
}
</style>