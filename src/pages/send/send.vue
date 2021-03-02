<template>
  <div id="send">
    <div class="sendCont">
      <!-- <input type="text" name="uid" placeholder="用户的ID：" v-model="uid" /> -->
      <view class="container">
        <div class="contImage">
          <textarea
          name="cont"
          v-model="cont"
          cols="30"
          rows="10"
          :placeholder="placeholder"
          > </textarea>
          <span class="userImg" v-for="(item, index) in image" :key="index" >
            <img class="showimg" :src="item" @tap="preview" :data-src="item" alt="" />
            <img class="icon" src="../../static/del.png" title="删除图片" alt="删除图片" @click="delImg(index)">
          </span>
          <span class="videoImg" v-if="video !== ''">
            <video
              id="myVideo" 
              :src="video"
              @error="videoErrorCallback" 
              enable-danmu 
              show-fullscreen-btn='true'
              show-center-play-btn='false' 
              show-play-btn="true" 
              controls
              autoplay = 'true'
              picture-in-picture-mode="['push', 'pop']"
              @enterpictureinpicture='bindVideoEnterPictureInPicture'
              @leavepictureinpicture='bindVideoLeavePictureInPicture'
            ></video>
            <img class="videoicon" src="../../static/del.png" title="删除视频" alt="删除视频" @click="delVideo">
          </span>
          
        </div>

        <div class="img">
          <img
            src="../../static/phone.png"
            title="选择图片"
            @click="chooseimg"
            class="chooseImg"
          />
          <img src="../../static/video.png" class="chooseImg" title="视频" @click="videoImg" />
        </div>
        <view>
          <button type="warn" @tap="undo" class="clear">清空</button>
          <button class="disbutton" type="warn" open-type="getUserInfo" @click="send">
            发布
          </button>
        </view>
      </view>
    </div>
  </div>
</template>


<script>
export default {
  data() {
    return {
      placeholder: "分享我的小确幸...",
      cont: "",
      uid: "",
      userInfo: "",
      artical: [],
      file: "",
      image: [],
      video: ''
    };
  },
  methods: {
    // 清空
    undo() {
      this.cont = "";
    },
    // 发布
    send() {
      let _this = this;
      wx.getSetting({
        success(res) {
          if (res.authSetting["scope.userInfo"]) {
            wx.authorize({
              scope: "scope.userInfo",
              success() {
                let value = wx.getStorageSync("userInfo");
                if (!value) {
                  wx.getUserInfo({
                    success: function (resp) {
                      _this.userInfo = resp.userInfo;
                      console.log(_this.artical);
                      wx.setStorageSync(
                        "userinfo",
                        JSON.stringify(resp.userInfo)
                      );
                    },
                  });
                } else {
                  console.log("已经存在");
                }
              },
            });
          }
        },
      });
      if (this.cont == "") {
        uni.showToast({
          title: "内容不能为空",
          icon: "none",
        });
        return;
      }
      uni.showToast({
        title: "发布成功",
        icon: "success",
      });
      // _this.artical.push({
      //   uid: _this.uid,
      //   acatar: _this.userInfo.avatarUrl,
      //   uname: _this.userInfo.nickName,
      //   time: new Date(),
      //   content: _this.cont,
      // });
      // _this.artical = JSON.stringify(_this.artical);

      // wx.request({
      //   url: "http://192.168.0.102/project-third-php/send/index.php",
      //   method: "POST",
      //   data: {
      //     data: _this.artical,
      //   },
      //   header: {
      //     "content-type": "application/x-www-form-urlencoded", // 默认值
      //   },
        // success(res) {
        //   console.log(res);
        //   if (res.statusCode == 200) {
        //     uni.showToast({
        //       title: "发布成功",
        //       icon: "success",
        //     });
        //   }
        // },
      // });
    },
    // 选择照片
    chooseimg() {
      if(this.video !== '') {
        wx.showToast({
          title: '您已上传视频，不能再上传图片',
          icon: 'none'
        });
        return;
      }
      let _this = this;
      wx.chooseImage({
        count: 9,
        sourceType: ['album','camera'],
        success (res) {
          const tempFilePaths = res.tempFilePaths;
          // if(tempFilePaths.length >= 9) {
          //   wx.showToast({
          //     title: '照片不能上传超过9张',
          //     icon: 'none'
          //   });
          //   return;
          // }
          tempFilePaths.forEach(item => {
            wx.uploadFile({
              url: 'https://wx.f1t.fun/send/image.php', //仅为示例，非真实的接口地址
              filePath: item,
              name: 'file',
              formData: {
                'user': "upload"
              },
              success (res){
                let obj = JSON.parse(res.data);
                if (obj.code == 200) {
                  if(_this.image.length >= 9) {
                    wx.showToast({
                      title: '照片不能上传超过9张',
                      icon: 'none'
                    });
                    return;
                  }
                  _this.file = obj.msg;
                  _this.image.push(_this.file);
                }
                // if(obj.code == 401) {
                //   wx.showToast({
                //     title: '图片过大，请重新上传',
                //     icon: 'none'
                //   });
                //   return;
                // }
              }
            })
          })
        }
      })
    },
    // 视频
    videoImg() {
      if(this.image.length !== 0) {
        wx.showToast({
          title: '您已上传视频，不能再上传图片',
          icon: 'none'
        });
        return;
      }
      let _this = this;
      wx.chooseVideo({
        sourceType: ['album','camera'],
        maxDuration: 15,
          compressed: true,
        camera: 'back',
        success(res) {
          // console.log(res.tempFilePath);
          wx.uploadFile({
            url: 'https://wx.f1t.fun/send/video.php', //仅为示例，非真实的接口地址
            filePath: res.tempFilePath,
            name: 'file',
            formData: {
              'user': "videoLoad"
            },
            header: {
              'content-type': 'multipart/form-data'
            },
            success (res){
              let obj = JSON.parse(res.data);
              _this.video = obj.msg;
            },
            fail() {
              console.log('请求失败');
            }
          })
        }
      })
    },
    // 播放视频
    bindVideoEnterPictureInPicture() {
      console.log('进入小窗模式')
    },

    bindVideoLeavePictureInPicture() {
      console.log('退出小窗模式')
    },
    // 图片放大
    preview(e) {
      let currentImg = e.target.dataset.src
      wx.previewImage({
        current: currentImg, // 当前显示图片的http链接
        urls: this.image
      })
    },
    // 删除图片
    delImg(index) {
      for (let x = 0; x < this.image.length; x++) {
        if (index == x) {
          this.image.splice(x, 1);
        }
      }
    },
    // 删除视频
    delVideo(){
      this.video = '';
    }
  },
};
</script>


<style>
input {
  width: 96%;
  margin: 2%;
}
.contImage {
  width: 96%;
  height: auto;
  min-height: 78vh;
  margin: 2%;
}
textarea {
  width: 100%;
  height: 30vh;
}
.userImg {
  position: relative;
}
.contImage .showimg {
  width: 125px;
  height: 125px;
  margin: 0.8%;
}

.contImage .icon {
  position: absolute;
  top: -12%;
  right: 2.2%;
  width: 16px;
  height: 16px;
}
.videoImg {
  position: relative;
}
.videoImg .videoicon {
  position: absolute;
  right: 5%;
  top: -100%;
  width: 16px;
  height: 16px;
}
.sendCont button {
  float: left;
  width: 45%;
  margin-left: 3.5%;
  background: #EFB336;
}
video {
  width: 90%;
  margin: 5%;
}
.container .img {
  width: 92%;
  text-align: right;
}
.container .chooseImg {
  width: 32px;
  height: 32px;
  margin: 2%;
}
</style>