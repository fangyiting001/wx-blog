<template>
  <div id="info">
    <van-toast id="van-toast" />
    <view class="user">
      <navigator :url="'/pages/index/prosonalinfo?uid='+item.uid" hover-class="navigator-hover">
        <image class="img" :src="data.avatar"></image>
        <span class="uname">{{ data.uname }}</span>
        <span class="time">{{ data.time }}</span>
      </navigator>
    </view>
    <p class="text">{{ data.content }}</p>
    <div class="oneImg" v-if="data.imgs.length == 1">
      <img v-for="(itemimg, indexImg) in data.imgs" :key="indexImg" :src="itemimg" alt="" @tap="preview" :data-src="itemimg">
    </div>
    <div class="towImg" v-if="data.imgs.length == 2 || data.imgs.length == 4">
      <img v-for="(itemimg, indexImg) in data.imgs" :key="indexImg" :src="itemimg" alt="" @tap="preview" :data-src="itemimg">
    </div>
    <div class="thereImg" v-if="data.imgs.length == 3 || data.imgs.length == 5 || data.imgs.length > 7">
      <img v-for="(itemimg, indexImg) in data.imgs" :key="indexImg" :src="itemimg" alt="" @tap="preview" :data-src="itemimg">
    </div>
    <!-- <div class="image">
      <img v-for="(item, index) in image" :src="item.img" :key="index" alt="" 
      @tap="preview" :data-src="item.img" />
    </div> -->
    <div class="comment">
      <div class="discuss">
        <p>评论 {{common}}</p>
        <p v-if="data.flag == 1" @click="like(data.uid)">点赞 {{pot}}</p>
        <p v-if="data.flag == 2" class="red" @click="noLike(data.uid)">点赞 {{pot}}</p>
      </div>
      <div>
        <p class="send">
          <input type="text" placeholder="评论" v-model="value" class="input" />
          <button class="sendDiscuss" v-if="value == ''" disabled>发送</button>
          <button class="sendDiscuss" v-if="value !== ''" @click="sendDiscuss">
            发送
          </button>
        </p>
        <div class="discussCont">
          <div class="cont" v-for="(item, index) in discuss" :key="index">
            <img :src="item.avatar" alt="" />
            <span>{{ item.uname }}</span>
            <p>{{ item.discuss }}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
        data: {},
        uid: "",
        image: [],
        maxImg: [],
        discuss: [],
        common: 0,
        pot: 0,
        value: "",
        isActive: 0,
        list: [
            {
                val: "评论",
            },
            {
                val: "点赞",
                pot: 0
            },
        ],
            listArr: [],
        };
  },
  onLoad(options) {
    let uid = options.uid;
    let _this = this;
    wx.request({
      url: "https://wx.f1t.fun/index/index.php",
      success(res) {
        res.data.msg.forEach(item => {
          if (item.uid == uid) {
            _this.data = item;
          }
        })
      },
    });
    wx.request({
      url: "https://wx.f1t.fun/index/indexInfo.php",
      success(res) {
        _this.image = res.data.img;
        _this.image.forEach(item => {
          _this.maxImg.push(item.img);
        })
      },
    });
    wx.request({
      url: "https://wx.f1t.fun/index/indexInfo.php",
      success(res) {
        _this.discuss = res.data.discuss;
        _this.common = res.data.discuss.length;
      },
    });
    wx.request({
      url: "https://wx.f1t.fun/index/pot.php",
      success(res) {
        _this.pot = res.data.pot1.length;
      },
    });
    this.listArr.push(this.discuss[0]);
  },
  methods: {
    sendDiscuss() {
        this.value = "", 
        uni.showToast({
            title: "评论已发布",
            icon: "success",
        });
    },
    chooseClick(index) {
      this.listArr = [];
      this.isActive = index;
      this.listArr.push(this.discuss[index]);
    },
    // 点赞
    like(uid) {
      if (uid == this.data.uid) {
        if (this.data.flag == 1) {
          this.data.flag = 2;
          this.pot = this.pot + 1;
        }
      }
    },
    // 取消点赞
    noLike(uid) {
      if (uid == this.data.uid) {
        if (this.data.flag == 2) {
          this.data.flag = 1;
          this.pot = this.pot - 1;
        }
      }
    },
    // 图片放大
    preview(e) {
      let currentImg = e.target.dataset.src;
      wx.previewImage({
        current: currentImg, // 当前显示图片的http链接
        urls: this.maxImg
      })
    }
  },
};
</script>


<style>
#info {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}
.user {
  position: relative;
  float: left;
  height: 8vh;
  width: 94%;
  margin: 3% 3% 0 3%;
}
/* 发布头像 */
.img {
  position: absolute;
  top: 0;
  left: 0;
  width: 35px;
  height: 35px;
  border-radius: 17.5px;
}
/* 发布用户名 */
.uname {
  position: absolute;
  top: 2%;
  left: 12%;
  width: 90%;
  font-family: "楷体";
  /* margin-top: -2%;
        margin-left: -30%; */
}
/* 发布时间 */
.time {
  position: absolute;
  top: 40%;
  left: 12%;
  font-size: 13px;
}
/* 文本内容 */
.text {
  width: 94%;
  height: auto;
  line-height: 4vh;
  padding: 0 3%;
  text-align: left;
}
/* 文本图片 */
.towImg {
  width: 94%;
  margin: 3% 3% 0 3%;
}
.thereImg {
  width: 94%;
  text-align: left;
  margin: 3% 3% 0 3%;
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
.image {
  width: 94%;
  height: auto;
  margin: 3%;
}
.image img {
  width: 125px;
  height: 125px;
  margin: 0.5%;
}
/* 评论  点赞 */
.comment {
  width: 100%;
  height: auto;
}
.discuss {
  width: 94%;
  height: 6vh;
  margin: 1% 3%;
  text-align: center;
  color: #666;
  border-bottom: 1px solid #dfdfdf;
}
.discuss p {
  display: block;
  float: left;
  width: 47%;
  height: 6vh;
  line-height: 6vh;
}
/* 发送评论 */
.send {
  width: 94%;
  height: 6.6vh;
  margin: 2% 3%;
}
.input {
  display: block;
  float: left;
  width: 80%;
  height: 5.6vh;
  line-height: 5.6vh;
  padding-left: 2%;
  border: 1px solid rgb(216, 215, 215);
}
.sendDiscuss {
  float: left;
  width: 14%;
  height: 6vh;
  line-height: 6vh;
  text-align: center;
  background: #efb336;
  font-size: 13px;
  margin-left: 2%;
}
/* 评论的内容 */
.discussCont {
  width: 94%;
  height: auto;
  margin: 1% 3%;
}
.discussCont .cont img {
  display: block;
  float: left;
  width: 40px;
  height: 40px;
  border-radius: 20px;
}
.discussCont .cont span {
  display: block;
  float: left;
  width: 86%;
  font-size: 13px;
  color: #686868;
  padding-top: 0.5%;
  padding-left: 2%;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: inherit;
}
.discussCont .cont p {
  width: 87.8%;
  height: auto;
  padding-top: 0.5%;
  padding-bottom: 1.5%;
  margin-left: 12.2%;
  margin-bottom: 3%;
  color: #111;
  line-height: 3.6vh;
  font-size: 14.5px;
  border-bottom: 1px solid #dfdfdf;
}
.red {
  color: red;
}
</style>