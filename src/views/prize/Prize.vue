<template>
  <div class="container">
    <div class="lucky-wheel">
      <div class="lucky-title"></div>
      <div class="wheel-main">
        <div class="wheel-pointer" @click="beginRotate()"></div>
        <div class="wheel-bg" :style="rotateStyle">
          <div class="prize-list">
            <div
              class="prize-item"
              v-for="(item, index) in prizeList"
              :key="index"
              :style="item.style"
            >
              <div class="prize-pic">
                <img :src="item.icon" />
              </div>
              <div class="prize-type">{{ item.name }}</div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="main">
      <div class="main-bg"></div>
      <div class="bg-p"></div>
      <div class="content">
        <div class="count">今日免费抽奖次数： {{ count }}</div>
      </div>
      <div class="tip">
        <div class="tip-title">活动规则</div>
        <div class="tip-content">
          <p>
            1.每日签到后，即可获得一次幸运大转盘的机会，仅限当天有效，过期作废。
            2.金币抽奖，每10个金豆可兑换一次大转盘机会。
          </p>
          <p>2.金币抽奖，每10个金豆可以兑换一次大转盘抽奖机会</p>
          <p>
            3.所中金豆或积分到【我的账户】中查询。累计达到100金豆及以上，可以兑换相应奖品
          </p>
        </div>
      </div>
    </div>
    <div class="toast" v-show="prize">
      <div class="toast-container">
        <img :src="toastIcon" class="toast-picture" />
        <div class="close" @click="closeToast()"></div>
        <div class="toast-title">{{ toastTitle }}</div>
        <div class="toast-btn">
          <div class="toast-cancel" @click="closeToast">关闭</div>
        </div>
      </div>
    </div>
    <div class="toast-mask" v-show="prize"></div>
  </div>
</template>
<script>
const CIRCLE_ANGLE = 360;

export default {
  data() {
    return {
      duration: 4000, // 总旋转时间
      circle: 8, // 旋转圈数
      mode: "ease-in-out",
      rotateAngle: 360, // 旋转角度
      prizeList: [], // 奖品
      angleList: [], // 所有奖品角度
      count: 10, // 次数
      prize: null, // 弹窗
      index: null, // 奖品
      isRotating: false, // 是否旋转
    };
  },
  computed: {
    rotateStyle() {
      return `
            -webkit-transition: transform ${this.duration}ms ${this.mode};
            transition: transform ${this.duration}ms ${this.mode};
            -webkit-transform: rotate(${this.rotateAngle}deg);
                transform: rotate(${this.rotateAngle}deg);`;
    },
    toastIcon() {
      return this.prize && this.prize.isPrize === 1
        ? require("../assets/img/congratulation.png")
        : require("../assets/img/sorry.png");
    },
    toastTitle() {
      return this.prize && this.prize.isPrize === 1
        ? "恭喜您，获得" + this.prize.name
        : "未中奖";
    },
  },
  methods: {
    initPrizeList() {
      // 这里可以发起请求，从服务端获取奖品列表
      let data = [
        {
          icon: require("../assets/img/bean_500.png"), // 奖品图片
          name: "奖品1", // 奖品名称
          isPrize: 1, // 该奖项是否为奖品
        },
        {
          icon: require("../assets/img/bean_five.png"),
          name: "奖品2",
          isPrize: 1,
        },
        {
          icon: require("../assets/img/bean_one.png"),
          name: "奖品3",
          isPrize: 1,
        },
        {
          icon: require("../assets/img/point_five.png"),
          name: "奖品4",
          isPrize: 1,
        },
        {
          icon: require("../assets/img/point_ten.png"),
          name: "奖品5",
          isPrize: 1,
        },
        {
          icon: require("../assets/img/bean_500.png"),
          name: "奖品6",
          isPrize: 1,
        },
        {
          icon: require("../assets/img/give_up.png"),
          name: "奖品7",
          isPrize: 0,
        },
        {
          icon: require("../assets/img/bean_500.png"),
          name: "奖品8",
          isPrize: 1,
        },
      ];
      this.prizeList = this.formatPrizeList(data);
    },
    formatPrizeList(list) {
      // 记录每个奖的位置
      const angleList = [];
      const l = list.length;
      // 计算单个奖项所占的角度
      const average = CIRCLE_ANGLE / l; // 45度
      const half = average / 2;  // 22.5度

      // 循环计算给每个奖项添加style属性
      list.forEach((item, i) => {
        // 每个奖项旋转的位置为 当前 i * 平均值 + 平均值 / 2
        const angle = -(i * average + half); // -22.5 -67.5 -112.5 -157.5 -202.5 -247.5 -292.5 -337.5
        // // 增加 style
        item.style = `-webkit-transform: rotate(${angle}deg);
                      transform: rotate(${angle}deg);`;
        // // 记录每个奖项的角度范围
        angleList.push(i * average + half); // ~ -22.5 -67.5 -112.5 -157.5 -202.5 -247.5 -292.5 -337.5
      });
      this.angleList = angleList;
      return list;
    },
    beginRotate() {
      // 添加次数校验
      if(this.count === 0) return
       // 开始抽奖
      // 这里这里向服务端发起请求，得到要获得的奖
      // 可以返回下标，也可以返回奖品 id，通过查询 奖品列表，最终得到下标
      // 随机获取下标
      this.index = this.random(this.prizeList.length - 1);
      // 减少剩余抽奖次数
      this.count--
      // 开始旋转
      this.rotating()
    },
    rotating(){
      const { isRotating, angleList, rotateAngle, index, circle, duration  } = this
      if(isRotating) return
      this.isRotating = true
       // 计算角度
       const angle = rotateAngle + circle * CIRCLE_ANGLE + angleList[index] - (rotateAngle % CIRCLE_ANGLE)
      
       this.rotateAngle = angle
       // 旋转结束后，允许再次触发
       setTimeout(() => {
          
          this.rotateOver()
        }, duration + 1000)
    },
     random (max, min = 0) {
      return parseInt(Math.random() * (max - min + 1) + min)
    },
     rotateOver () {
      this.isRotating = false

      this.prize = this.prizeList[this.index]

      console.log(this.prize, this.index)
    },
    //关闭弹窗
    closeToast() {
      this.prize = null;
    },
  },
  created() {
    this.initPrizeList();
  },
};
</script>
<style scoped>
.container {
  width: 100%;
  height: 100vh;
}
.lucky-wheel {
  width: 100%;
  background: rgb(252, 207, 133) url("../assets/img/color_pillar.png") no-repeat
    center bottom;
  background-size: 100%;
  padding-top: 20px;
}
.lucky-title {
  width: 100%;
  height: 304px;
  background: url("../assets/img/lucky_title.png") no-repeat center top;
  background-size: 100%;
}
.wheel-main {
  margin: 0 auto;
  position: relative;
  width: 590px;
  height: 590px;
}
.wheel-pointer {
  position: absolute;
  top: 50%;
  left: 50%;
  z-index: 2;
  width: 170px;
  height: 170px;
  background: url("../assets/img/draw_btn.png") no-repeat center top;
  background-size: 100%;
  transform: translate3d(-50%, -50%, 0);
}
.wheel-bg {
  position: absolute;
  top: 0;
  left: 0;
  z-index: 1;
  width: 100%;
  height: 100%;
  background: url("../assets/img/draw_wheel.png") no-repeat center top;
  background-size: 100%;
  color: #fff;
}
.wheel-bg div {
  text-align: center;
}
.prize-list {
  width: 100%;
  height: 100%;
  position: relative;
}
.prize-list .prize-item {
  position: absolute;
  width: 190px;
  height: 300px;
  top: 0;
  left: 50%;
  margin-left: -90px;
  transform-origin: 50% 100%;
}
.prize-pic img {
  width: 152px;
  height: 93px;
  margin-top: 59px;
}
.prize-count {
  font-size: 0.875rem;
}
.prize-type {
  font-size: 28px;
}
.main {
  position: relative;
  width: 100%;
  min-height: 534px;
  background: rgb(243, 109, 86);
  padding-bottom: 63px;
}
.main-bg {
  width: 100%;
  height: 246px;
  position: absolute;
  top: -128px;
  left: 0;
  background: url("../assets/img/luck_bg.png") no-repeat center top;
  background-size: 100%;
}
.bg-p {
  width: 100%;
  height: 110px;
  background: rgb(252, 207, 133);
}
.content {
  position: absolute;
  top: 7px;
  left: 0;
  background: rgb(243, 109, 86);
  width: 100%;
  height: 195px;
  font-size: 42px;
  color: #ffeb39;
}
.content div {
  text-align: center;
}
.tip {
  position: relative;
  margin: 94px auto 0;
  width: 656px;
  border: 1px solid #fbc27f;
}
.tip-title {
  position: absolute;
  top: -37;
  left: 50%;
  transform: translate(-50%, 0);
  font-size: 37;
  color: #fccc6e;
  background: rgb(243, 109, 86);
  padding: 12px 12px;
}
.tip-content {
  padding: 58.594px 23.438px;
  font-size: 32.813px;
  color: #fff8c5;
  line-height: 1.5;
}
.toast-mask {
  position: fixed;
  top: 0;
  left: 0;
  background: rgba(0, 0, 0, 0.6);
  z-index: 10000;
  width: 100%;
  height: 100%;
}
.toast {
  position: fixed;
  top: 50%;
  left: 50%;
  z-index: 20000;
  transform: translate(-50%, -50%);
  width: 578.906px;
  background: #fff;
  border-radius: 11.719px;
  padding: 11.719px;
  background-color: rgb(252, 244, 224);
}
.toast-container {
  position: relative;
  width: 100%;
  height: 100%;
  border: 1px dotted #fccc6e;
}
.toast-picture {
  position: absolute;
  top: -243.75px;
  left: -56.25px;
  width: 703.125px;
  height: 321.094px;
}
.toast-pictrue-change {
  position: absolute;
  top: -56.25px;
  left: -51.563px;
  width: 656.25px;
  height: 117.188px;
}
.toast-title {
  padding: 105.469px 0;
  font-size: 18px;
  color: #fc7939;
  text-align: center;
}
.toast-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 35.156px;
}
.toast-btn div {
  background-image: -moz-linear-gradient(
    -18deg,
    rgb(242, 148, 85) 0%,
    rgb(252, 124, 88) 51%,
    rgb(252, 124, 88) 99%
  );
  background-image: -ms-linear-gradient(
    -18deg,
    rgb(242, 148, 85) 0%,
    rgb(252, 124, 88) 51%,
    rgb(252, 124, 88) 99%
  );
  background-image: -webkit-linear-gradient(
    -18deg,
    rgb(242, 148, 85) 0%,
    rgb(252, 124, 88) 51%,
    rgb(252, 124, 88) 99%
  );
  box-shadow: 0px 4px 0px 0px rgba(174, 34, 5, 0.7);
  width: 175.781px;
  height: 70.313px;
  border-radius: 70.313px;
  text-align: center;
  line-height: 70.313px;
  color: #fff;
}
.close {
  position: absolute;
  top: -35.156px;
  right: -35.156px;
  width: 75px;
  height: 75px;
  background: url("../assets/img/close_store.png") no-repeat center top;
  background-size: 100%;
}
</style>

