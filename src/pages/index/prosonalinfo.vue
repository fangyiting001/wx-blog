<template>
  <div id="daily">
    <div class="top">
      <div class="dailyInfo">
        <div class="left">
          <img :src="prosonal.avatar" alt="" />
        </div>
        <div class="right">
          <p class="userinfo">
            <span class="nickname">{{prosonal.uname}}</span>
            <img src="../../static/woman.png" title="女" />
          </p>
          <p class="from">FUJIAN&nbsp;&nbsp;XIAMEN</p>
        </div>
      </div>
    </div>
    <div class="myartical">
      <div class="span">
        <h3>我的小确幸</h3>
      </div>
      <div class="cont">
        <ul>
          <li v-for="(item, index) in data" :key="index">
            <p style="display: none">{{ item.uid }}</p>
            <view class="left">
              <image class="img" :src="prosonal.avatar"></image>
              <span class="uname">{{prosonal.uname}}</span>
              <span class="time">{{ item.time }}</span>
            </view>
            <navigator
              :url="'/pages/index/info?uid=' + item.uid"
              hover-class="navigator-hover"
            >
              <p class="text">{{ item.content }}</p>
            </navigator>
            <div class="oneImg" v-if="item.imgs.length == 1">
              <img v-for="(itemimg, indexImg) in item.imgs" :key="indexImg" :src="itemimg" alt="" @tap="preview" :data-src="itemimg" :data-uid="item.uid">
            </div>
            <div class="towImg" v-if="item.imgs.length == 2 || item.imgs.length == 4">
              <img v-for="(itemimg, indexImg) in item.imgs" :key="indexImg" :src="itemimg" alt="" @tap="preview" :data-src="itemimg" :data-uid="item.uid">
            </div>
            <div class="thereImg" v-if="item.imgs.length == 3 || item.imgs.length == 5 || item.imgs.length > 7">
              <img v-for="(itemimg, indexImg) in item.imgs" :key="indexImg" :src="itemimg" alt="" @tap="preview" :data-src="itemimg" :data-uid="item.uid">
            </div>
            <div class="other">
              <navigator :url="'/pages/index/info?uid='+item.uid" hover-class="navigator-hover">
                <p>
                  <img src="../../static/pinglun.png" alt="" class="pinglun" />
                  <span class="pltext">评论 {{discuss}}</span>
                </p>
              </navigator>
              <p v-if="item.flag == 1" @click="like(item.uid)">
                <img src="../../static/pot.png" alt="" class="pinglun" />
                <span class="pltext">赞 {{ item.pot }}</span>
              </p>
              <p v-if="item.flag == 2" @click="noLike(item.uid)">
                <img src="../../static/pot2.png" alt="" class="pinglun" />
                <span class="pltext">赞 {{ item.pot }}</span>
              </p>
            </div>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      prosonal: {},
      data: [],
      pot: 0,
      discuss: 0,
      userInfo: "",
      nickName: "",
      avatarUrl: "",
      gender: "",
      province: "",
      city: "",
      country: "",
    };
  },
  onLoad(options) {
    let uid = options.uid;
    wx.request({
      url: "https://wx.f1t.fun/index/index.php",
      success(res) {
        res.data.msg.forEach(item => {
          if (item.uid == uid) {
            _this.prosonal = item;
          }
        })
      },
    });
    let _this = this;
    wx.request({
      url: "https://wx.f1t.fun/index/prosonal.php",
      success(res) {
        _this.data = res.data.msg;
      },
    });
    wx.request({
      url: "https://wx.f1t.fun/index/indexInfo.php",
      success(res) {
        _this.discuss = res.data.discuss.length;
      },
    });
    wx.request({
      url: "https://wx.f1t.fun/index/pot.php",
      success(res) {
        _this.pot = res.data.pot1.length;
      },
    });
  },
  methods: {
    like(uid) {
      this.data.forEach(item => {
        if (uid == item.uid) {
          if (item.flag == 1) {
            item.flag = 2;
            item.pot = item.pot + 1;
          }
        }
      })
    },
    noLike(uid) {
      this.data.forEach(item => {
        if (uid == item.uid) {
          if (item.flag == 2) {
            item.flag = 1;
            item.pot = item.pot - 1;
          }
        }
      })
    },
    // 图片放大
    preview(e) {
      let currentImg = e.target.dataset.src;
      let uid = e.target.dataset.uid;
      this.data.forEach(item => {
        if(item.imgs) {
          if (item.uid == uid) {
            this.maxImg = item.imgs;
          }
        };
      });
      wx.previewImage({
        current: currentImg, // 当前显示图片的http链接
        urls: this.maxImg
      })
    }
  }
};
</script>


<style>
#daily {
  width: 100%;
  height: auto;
}
.top {
  z-index: -2;
  position: relative;
  width: 100%;
  height: 28vh;
  /* background: #efb336; */
  background: url("data:image/jpeg;base64,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") no-repeat;
  background-size: 100%;
}
.top .left {
  float: left;
  width: 30%;
  margin-left: 6%;
}
.top .left img {
  width: 66px;
  height: 66px;
  border-radius: 33px;
  margin-top: 48%;
  margin-left: 45%;
}
.right {
  float: left;
  width: 52%;
  margin: 6%;
}
.right .userinfo {
  width: 100%;
  margin-left: -10%;
  margin-top: 20%;
}
.right .userinfo .nickname {
  font-family: "楷体";
  font-size: 20px;
  color: #333;
}
.right .userinfo img {
  width: 20px;
  height: 20px;
  margin-left: 1%;
}
.right .from {
  width: 100%;
  margin-left: -10%;
  color: #333;
  margin-top: 1%;
}

/* 我的文章 */
.myartical {
  width: 100%;
  height: auto;
  padding-top: 5%;
  background: #f5f3f3;
}
.span {
  width: 94%;
  height: 5vh;
  margin-left: 3%;
  border-bottom: 1px solid #999;
}
.span h3 {
  font-family: "楷体";
  font-weight: 700;
}
.cont {
  z-index: 2;
  width: 100%;
  height: auto;
  /* margin-top: 54px; */
  background: #f5f3f3;
  /* overflow: hidden; */
}

ul {
  width: 100%;
  overflow: hidden;
  text-align: center;
}

li {
  position: relative;
  list-style: none;
  width: 94%;
  height: auto;
  background: #fff;
  /* border-bottom: 1px solid rgb(230, 227, 227); */
  margin: 3% 0;
  padding: 3% 3% 0 3%;
  text-align: center;
}

.left {
  position: relative;
  float: left;
  height: 5vh;
  width: 100%;
}
.img {
  position: absolute;
  top: 0;
  left: 0;
  width: 35px;
  height: 35px;
  border-radius: 17.5px;
}
.uname {
  position: absolute;
  top: 0;
  left: 12%;
  width: 100%;
  text-align: left;
  font-family: "楷体";
  /* margin-top: -2%;
	margin-left: -30%; */
}
.time {
  position: absolute;
  top: 50%;
  left: 12%;
  font-size: 13px;
}
.text {
  width: 100%;
  height: 8vh;
  line-height: 4.4vh;
  margin-bottom: 2.5%;
  padding-top: 2%;
  text-align: left;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: inherit;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
}
.thereImg {
  width: 100%;
  text-align: left;
}
.towImg img {
  width: 187px;
  height: 186px;
  margin: 0.8%;
}
.thereImg img {
  width: 122.9px;
  height: 122.9px;
  margin: 0.8%;
}
/* 评论 赞 */
.other {
  width: 94%;
  padding: 2%;
  height: 3vh;
  border-top: 1px solid #e6e6e6;
}
.other p {
  width: 50%;
  display: block;
  float: left;
}
.other p img {
  display: block;
  float: left;
  width: 25px;
  height: 25px;
  margin-left: 40%;
  margin-top: -2%;
}
.other p span {
  display: block;
  float: left;
  font-size: 12px;
  margin-top: 0.5%;
}
</style>