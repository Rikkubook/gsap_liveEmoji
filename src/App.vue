<template>
  <div id="app">
    <div class="videoContainer">
      <video autoplay="true" muted ref="myvideo"></video>
    </div>
    <div class="liveLabel">
      <div class="red" ref="liveTag">LIVE</div>
      <div class="time">{{ timeLabel }}</div>
    </div>
    <ul class="floatingEmojiList">
      <transition-group v-on:enter="enter">
        <!-- for 使用 transition-group-->
        <!-- 使用 v-on -->
        <li
          class="floatingEmoji"
          v-for="(emoji, index) in currentEmojiList"
          :key="index"
          @click="addEmoji(emoji)"
        >
          {{ emoji }}
        </li>
      </transition-group>
    </ul>
    <ul class="emojiToolBar">
      <li
        class="emojiBtn"
        v-for="(emoji, index) in emojis"
        :key="index"
        @click="addEmoji(emoji)"
      >
        {{ emoji }}
      </li>
    </ul>
    <div class="contentArea">
      <div class="inputArea">
        <input type="text" v-model.trim="message" />
        <button @click="addComment()">新增留言</button>
      </div>
      <div class="comments">
        <transition-group name="fade">
          <!-- 使用class -->
          <div class="comment" v-for="comment in comments" :key="comment.name">
            <div class="head"></div>
            <div class="content">
              <div class="name">
                {{ comment.name }}
              </div>
              <div class="sentence">
                {{ comment.content }}
              </div>
            </div>
            <button @click="removeComment(comment)">
              <i class="fa fa-times" aria-hidden="true"></i>
            </button>
          </div>
        </transition-group>
      </div>
    </div>
  </div>
</template>

<script>
import { TweenMax, TimelineMax, Power0, Power4, Power1 } from "gsap";

let emojiList = "🥳,😁,😍,😉,🥰,😡";
let commentObject = {
  color: "#333",
  name: "Hello1",
  content:
    "Lorem Ipsum is simply dummy text of the printing and typesetting industry. "
};
export default {
  name: "App",
  data() {
    return {
      time: 50,
      emojis: emojiList.split(","),
      currentEmojiList: [],
      comments: [],
      message: ""
    };
  },
  mounted() {
    // 呼吸燈
    TweenMax.to(this.$refs.liveTag, 1, {
      css: {
        backgroundColor: "rgba(255,0,0,0.3)"
      },
      ease: Power0.easeNone, //動畫沒事
      repeat: -1, //循環重複
      yoyo: true
    });

    // 時間
    setInterval(() => {
      this.time++;
    }, 1000);
    // 抓影像
    var constraints = { audio: true, video: { width: 1280, height: 720 } };

    navigator.mediaDevices // 瀏覽器去詢問
      .getUserMedia(constraints)
      .then((mediaStream) => {
        // callbackeFunction
        var video = this.$refs.myvideo;
        video.srcObject = mediaStream;
        console.log(mediaStream);
        video.onloadedmetadata = function (e) {
          video.play();
        };
      })
      .catch(function (err) {
        console.log(err.name + ": " + err.message);
      }); // always check for errors at the end.
  },
  methods: {
    enter(el) {
      console.log(el); // 回傳參數
      // 在此會是已經知道哪個元素且新增了
      let tl = new TimelineMax();
      let _id = el; // 指定元素;

      tl.set(_id, {
        //初始化
        scale: 0.2,
        x: Math.random() * -100 + 20 //產生隨機
      })
        .to(_id, 1, {
          // 1秒
          y: -Math.random() * 100,
          scale: 1,
          ease: Power4.easeOut // 先快後慢
        })
        .to(_id, 3, {
          x: -400,
          scale: 0.5,
          ease: Power1.easeIn // 先慢後快
        });
    },
    addEmoji(emoji) {
      console.log(emoji);
      this.currentEmojiList.push(emoji);
    },
    addComment() {
      let objCopy = JSON.parse(JSON.stringify(commentObject));
      // 此時是淺拷貝 要轉為字串當深拷貝 再轉回
      if (this.message != "") {
        commentObject.name += "!!!";
        objCopy.content = this.message;
        // 將輸入的替換模組
        this.message = "";
        this.comments.unshift(objCopy); //往 array 前方丟入
      } else {
        return;
      }
    },
    removeComment(obj) {
      this.comments = this.comments.filter((el) => el !== obj);
    }
  },
  computed: {
    timeLabel() {
      let sec = this.time % 60;
      let min = Math.floor(this.time / 60) % 60;
      let hour = Math.floor(this.time / 3600) % 24;
      let pd = (num) => (num + "").padStart(2, "0");
      return `${pd(hour)}: ${pd(min)}: ${pd(sec)}`;
    }
  }
};
</script>

<style lang="scss">
/* * {
  outline: 1px solid red;
} */
body {
  background-color: #2c3e50;
  display: flex;
  justify-content: center;
  align-items: center;
}
.liveLabel {
  position: absolute;
  left: 50%;
  top: 50px;
  transform: translateX(-50%);
  display: flex;
  .red {
    background-color: red;
    color: #fff;
    padding: 5px 10px;
    font-weight: 600;
  }
  .time {
    background-color: rgba(black, 0.4);
    color: #fff;
    padding: 5px 10px;
  }
}
.videoContainer {
  height: 450px;
  overflow: hidden;
  display: flex;
  justify-content: center;
  align-items: center;
  position: relative;
  background: gray;
  video {
    height: 100%;
  }
}
.emojiToolBar {
  margin: 0px;
  display: flex;
  list-style: none;
  position: absolute;
  z-index: 10;
  right: 0px;
  bottom: 0px;
  background-color: rgba(white, 0.9);
  width: 100%;
  justify-content: flex-end;
  padding: 10px;
  .emojiBtn {
    font-size: 28px;
    width: 50px;
    cursor: pointer;
    transition: 0.5s;
    &:active {
      transform: scale(0.8);
      transition: 0s;
    }
  }
}
.floatingEmojiList {
  margin: 0px;
  list-style: none;
  position: relative;
  .floatingEmoji {
    position: absolute;
    right: 50px;
    top: 0px;
    font-size: 40px;
  }
}
.contentArea {
  padding: 20px;
}
.inputArea {
  display: flex;
  margin-bottom: 20px;
  input {
    box-sizing: border-box;
    border: 1px solid #2c3e50;
    height: 28px;
    line-height: 1;
    border-radius: 5px;
    flex: 1;
    margin-right: 8px;
    &:focus {
      outline: none;
      border: 1px solid #2c3e50;
      box-shadow: 0 0 0 3px rgba(#2c3e50, 0.25);
    }
  }
  button {
    box-sizing: border-box;
    display: inline-block;
    border: 1px solid #2c3e50;
    border-radius: 5px;
    font-size: 16px;
    height: 28px;
    background-color: #2c3e50;
    color: #fff;
    font-weight: 600;
  }
}
.comments {
  text-align: left;
  .comment {
    border-radius: 5px;
    display: flex;
    padding: 5px 10px;
    box-shadow: 0px 0px 5px rgba(black, 0.1);
    margin-bottom: 10px;
    padding: 20px;
    align-items: center;
    .head {
      width: 40px;
      height: 40px;
      background-color: #333;
      border-radius: 50%;
      flex-shrink: 0;
      margin-right: 20px;
    }
    .content {
      .name {
        font-weight: 600;
        padding-bottom: 5px;
      }
    }
    button {
      margin-left: auto;
      background-color: transparent;
      border: none;
      font-size: 20px;
      color: rgb(161, 161, 161);
      &:hover {
        color: rgb(202, 49, 49);
      }
    }
  }
}

.fade-enter-active,
.fade-leave-active {
  opacity: 1;
  transform: translateY(0px);
  transition: opacity 3s, transform 1s;
}
.fade-enter,
.fade-leave-to {
  opacity: 0;
  transform: translateY(-20px);
}

#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  position: relative;
  color: #2c3e50;
  width: 398px;
  height: 744px;
  margin-top: 60px;
  background-color: #fff;
  overflow: hidden;
}
</style>
