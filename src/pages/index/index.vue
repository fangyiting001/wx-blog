<template>
  <view class="content">
    <!-- <div class="seach">
      <van-search
        :value="value"
        shape="round"
        background="#EFB336"
        placeholder="请输入搜索关键词"
      />
    </div> -->
    <!-- <navigator url="/pages/index/info">
	<button>点击</button>
	</navigator>
	 -->
   <!-- <view class="page-section page-section-spacing swiper padbor">
      <swiper
        :indicator-dots="indicatorDots"
        :autoplay="autoplay"
        :circular="circular"
        :vertical="vertical"
        :interval="interval"
        :duration="duration"
        :previous-margin="previousMargin"
        :next-margin="nextMargin" 
        style="width:100%;height:28vh"
      >
        <block v-for="(item, index) in banners" :key="index">
          <swiper-item>
            <img :src="item.bimg" style="width:100%;height:28vh">
          </swiper-item>
        </block>
      </swiper>
    </view> -->
    <div class="cont">
        <ul>
            <li v-for="(item, index) in data" :key="index">
            
            <p style="display: none">{{ item.uid }}</p>
            <navigator
              :url="'/pages/index/prosonalinfo?uid=' + item.uid"
              hover-class="navigator-hover"
            >
              <view class="left">
                <image class="img" :src="item.avatar"></image>
                <span class="uname">{{ item.uname }}</span>
                <span class="time">{{ item.time }}</span>
              </view>
            </navigator>
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
  </view>
</template>

<script>
export default {
  data() {
    return {
      // indicatorDots: true,
      // vertical: false,
      // autoplay: true,
      // circular: false,
      // interval: 2000,
      // duration: 500,
      // previousMargin: 0,
      // nextMargin: 0,
      value: "",
      data: [],
      uid: "",
      pot: 0,
      discuss: 0,
      scrollTop: 0,
      maxImg: [],
      video: []
    };
  },
  onLoad() {
    let _this = this;
    wx.request({
      url: "https://wx.f1t.fun/index/index.php",
      success(res) {
        _this.data = res.data.msg;
        // _this.data.forEach(item => {
        //   if(item.imgs) {
        //     item.imgs.forEach(imgItem => {
        //       imgItem.forEach(img => {
        //       _this.maxImg.push(img);
        //       }) 
        //     })
        //   };
        //   console.log(_this.maxImg);
        // })
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
  // 下拉刷新
  onPullDownRefresh(){
    let _this = this;
    setTimeout(() => {
        wx.request({
          url: "https://wx.f1t.fun/index/freshen.php",
          success(res) {
            _this.data = res.data.msg;
          },
        });
      }, 1000)
  },
  // 上拉加载更多
  onReachBottom() {
    let _this = this;
    wx.request({
      url: "https://wx.f1t.fun/index/freshen.php",
      success(res) {
        res.data.msg.forEach(item => {
          _this.data.push(item);
        })
      },
    });
  },
  methods: {
    info() {
      wx.navigateTo({
        url: "/pages/index/info",
      });
    },
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
    // img(uid) {
    //   this.data.forEach(item => {
    //     if(item.imgs) {
    //       if (item.uid == uid) {
    //         this.maxImg = item.imgs;
    //       }
    //     };
    //   });
    // },
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
.content {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.cont {
  z-index: 2;
  width: 100%;
  height: auto;
  /* margin-top: 54px; */
  background: #f5f3f3;
  /* overflow: hidden; */
}
.scroll-Y {
  height: 0;
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
  text-align: left;
  width: 100%;
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
/* .logo {
  height: 200rpx;
  width: 200rpx;
  margin: 200rpx auto 50rpx auto;
}

.text-area {
  display: flex;
  justify-content: center;
}

.title {
  font-size: 36rpx;
  color: #8f8f94;
} */
</style>
