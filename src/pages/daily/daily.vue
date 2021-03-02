<template>
  <div id="daily">
    <div class="top">
      <div class="dailyInfo" v-if="userInfo == ''">
        <div class="left">
          <img @click="login" src="../../static/user.png" alt="" />
        </div>
        <div class="right">
          <p class="noLogin">未登录,请先登录</p>
          <button class="login" @click="login" open-type="getUserInfo">
            登录
          </button>
          <!-- <div class="shouquan" v-if="userInfo == ''">
            <button >授权</button>
          </div> -->
        </div>
      </div>
      <div class="top">
        <div class="dailyInfo" v-if="userInfo !== ''">
          <div class="left">
            <img :src="avatarUrl" alt="" />
          </div>
          <div class="right">
            <p class="userinfo">
              <span class="nickname">{{ nickName }}</span>
              <img v-if="gender == '1'" src="../../static/man.png" title="男" />
              <img
                v-if="gender == '2'"
                src="../../static/woman.png"
                title="女"
              />
            </p>
            <p class="from">{{ province }}&nbsp;&nbsp;{{ city }}</p>
          </div>
        </div>
      </div>
    </div>
    <div class="myartical" v-if="userInfo !== ''">
      <div class="span">
        <h3>
          <!-- <img src="../../static/luck.png" alt=""> -->
          我的小确幸
          <!-- <img src="../../static/luck.png" alt=""> -->
        </h3>
      </div>
      <div class="cont">
        <ul>
          <li v-for="(item, index) in data" :key="index">
            <p style="display: none">{{ item.uid }}</p>
            <view class="left">
              <image class="img" :src="avatarUrl"></image>
              <span class="uname">{{ nickName }}</span>
              <span class="time">{{ item.time }}</span>
              <img
                class="del"
                @click="del(item.uid)"
                src="../../static/delete.png"
                alt=""
              />
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
              <img v-for="(itemimg, indexImg) in item.imgs" :key="indexImg" :src="itemimg" alt="" @tap="preview" :data-src="itemimg" :data-uid="item.uid" >
            </div>
            <div class="thereImg" v-if="item.imgs.length == 3 || item.imgs.length == 5 || item.imgs.length > 7">
              <img v-for="(itemimg, indexImg) in item.imgs" :key="indexImg" :src="itemimg" alt="" @tap="preview" :data-src="itemimg" :data-uid="item.uid">
            </div>
            <div class="other">
              <navigator
                :url="'/pages/index/info?uid=' + item.uid"
                hover-class="navigator-hover"
              >
                <p>
                  <img src="../../static/pinglun.png" alt="" class="pinglun" />
                  <span class="pltext">评论 {{ discuss }}</span>
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
  onLoad() {
    let _this = this;
    wx.request({
      url: "https://wx.f1t.fun/me/index.php",
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
    if (wx.getStorageSync("userinfo")) {
      _this.userInfo = wx.getStorageSync("userinfo");
      let userMsg = JSON.parse(_this.userInfo);
      _this.nickName = userMsg.nickName;
      _this.avatarUrl = userMsg.avatarUrl;
      _this.gender = userMsg.gender; //性别 0：未知、1：男、2：女
      _this.province = userMsg.province;
      _this.city = userMsg.city;
      _this.country = userMsg.country;
    }
  },
  methods: {
    login() {
      let _this = this;
      if (!wx.getStorageSync("userinfo")) {
        wx.getUserInfo({
          success: function (res) {
            _this.userInfo = res.userInfo.nickName;
            wx.setStorageSync("userinfo", JSON.stringify(res.userInfo));
            _this.userInfo = wx.getStorageSync("userinfo");
            _this.userInfo = wx.getStorageSync("userinfo");
            let userMsg = JSON.parse(_this.userInfo);
            _this.nickName = userMsg.nickName;
            _this.avatarUrl = userMsg.avatarUrl;
            _this.gender = userMsg.gender; //性别 0：未知、1：男、2：女
            _this.province = userMsg.province;
            _this.city = userMsg.city;
            _this.country = userMsg.country;
          },
        });
      }
    },
    // 删除
    del(uid) {
      let _this = this;
      wx.showModal({
        title: "删除",
        content: "您是否确定删除吗？",
        success(res) {
          if (res.confirm) {
            wx.showToast({
              title: "删除成功",
              icon: "success",
            });
            for (let x = 0; x < _this.data.length; x++) {
              if (uid == _this.data[x].uid) {
                _this.data.splice(x, 1);
              }
            }
          } else if (res.cancel) {
            wx.showToast({
              title: "取消删除",
              icon: "none",
            });
          }
        },
      });
    },
    // 点赞
    like(uid) {
      this.data.forEach((item) => {
        if (uid == item.uid) {
          if (item.flag == 1) {
            item.flag = 2;
            item.pot = item.pot + 1;
          }
        }
      });
    },
    noLike(uid) {
      this.data.forEach((item) => {
        if (uid == item.uid) {
          if (item.flag == 2) {
            item.flag = 1;
            item.pot = item.pot - 1;
          }
        }
      });
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
  },
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
  height: 15vh;
  background: #efb336;
}
.dailyInfo {
  z-index: -1;
  position: absolute;
  top: 36%;
  left: 5%;
  width: 90%;
  height: 20vh;
  background: #edece8;
  margin: 0 auto;
  border-radius: 5%;
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
  margin-top: 27%;
}
.top .shouquan {
  float: left;
  width: 36%;
  line-height: 5vh;
  height: 5vh;
  border-radius: 3%;
  margin: -17% 0 0 30%;
}
.top .shouquan button {
  width: 100%;
  font-size: 14px;
}
.right {
  float: left;
  width: 52%;
}
.right .noLogin {
  width: 100%;
  margin: 23% 0 0 -10%;
  color: #333;
}
.right .login {
  display: block;
  width: 35%;
  margin: 2% 0 0 -10%;
  height: 5vh;
  line-height: 5vh;
  font-size: 14px;
  color: #fff;
  background: red;
  letter-spacing: 2%;
}
.right .userinfo {
  width: 100%;
  margin-left: -10%;
  margin-top: 20%;
}
.right .userinfo .nickname {
  font-family: "楷体";
  font-size: 20px;
  color: #666;
}
.right .userinfo img {
  width: 20px;
  height: 20px;
  margin-left: 1%;
}
.right .from {
  width: 100%;
  margin-left: -10%;
  color: #666;
  margin-top: 1%;
}

/* 我的文章 */
.myartical {
  width: 100%;
  height: auto;
  margin-top: 20%;
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
h3 img {
  width: 16px;
  height: 16px;
  padding-top: 1%;
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
.del {
  position: absolute;
  top: 0;
  right: 3%;
  width: 20px;
  height: 20px;
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