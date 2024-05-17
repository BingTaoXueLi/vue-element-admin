<template>
  <div>
    <p class="title">聊天室</p>
    <!-- 聊天区 -->
    <div id="chatContainer" class="chatContainer">
      <!-- 导航区域 -->
      <!-- 某一个片段 -->
      <div v-for="(item, key) in dataList" :key="key" class="contentDiv">
        <p class="time">{{ getFormat(key) }}</p>
        <div v-for="(itemDetail, index) in item" :key="index">
          <div v-if="itemDetail.type === 0" class="shop content">
            <img :src="itemDetail.imgSrc">
            <div>
              <span>{{ itemDetail.shopName }}</span>
              <p>{{ itemDetail.content }}</p>
            </div>
          </div>
          <div v-if="itemDetail.type === 1" class="user content">
            <img :src="itemDetail.imgSrc">
            <div>
              <span>{{ itemDetail.nickName }}</span>
              <p>{{ itemDetail.content }}</p>
            </div>
          </div>
        </div>
      </div>
    </div>
    <!-- 发送内容区域 -->
    <div class="sendDiv">
      <input v-model="sendMsg" placeholder="请输入您的问题">
      <button @click="send">发送</button>
    </div>
  </div>
</template>

<script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.6.1/jquery.js"></script>
<script src="https://cdn.jsdelivr.net/npm/vant@2.9/lib/vant.min.js"></script>
<script>
let LinkStatus = {
    CLOSE: 0,
    SUCCESS: 1,
    ERROR: 2,
  }
  let UserType = {
    SHOP: 0,
    USER: 1,
  }
export default {
  name: 'chatRoom',
    data: function () {
      return {
        dataList: {},
        sendMsg: '',
        lastTimeStamp: 1664258797000,
        webSocket: null,
        linkStatus: LinkStatus.CLOSE, // 0 关闭，
      }
    },
    methods: {
      send() {
        if (this.linkStatus !== LinkStatus.SUCCESS) {
          return vant.Notify('失去链接，请刷新页面');
        }
        let time = new Date().getTime()
        let sendMsg = this.sendMsg
        this.sendMsg = ''
        let obj = {
          type: UserType.USER, //用户
          content: sendMsg,
          nickName: '小球2海绵宝宝',
          imgSrc: './img/user.jpg',
        }
        if (time - this.lastTimeStamp < 2 * 60 * 1000) {
          this.dataList[this.lastTimeStamp].push(obj)
        } else {
          this.lastTimeStamp = time
          this.$set(this.dataList, time, [obj])
        }
        this.handleScroll()
        setTimeout(() => {
          this.webSocket.send(sendMsg)
        }, 800)
      },
      initWebSocket() {
        this.webSocket = new WebSocket('ws://http://localhost:8000/')
        this.webSocket.onopen = () => {
          this.linkStatus = LinkStatus.SUCCESS
          this.webSocket.send('link')
        }
        this.webSocket.onerror = () => {
          this.linkStatus = LinkStatus.ERROR
          console.log('连接失败...')
        }
        this.webSocket.onmessage = (event) => {
          let msg = event.data.split('>>')
          let time = new Date().getTime()
          let obj = {
            type: UserType.SHOP, //商家
            content: msg[1],
            shopName: msg[0],
            imgSrc: './img/shop.jpg',
          }
          if (time - this.lastTimeStamp < 2 * 60 * 1000) {
            this.dataList[this.lastTimeStamp].push(obj)
          } else {
            this.lastTimeStamp = time
            this.$set(this.dataList, time, [obj])
          }
          this.handleScroll()
        }
        this.webSocket.onclose = () => {
          this.linkStatus = LinkStatus.CLOSE
          console.log('连接已关闭...')
        }
      },
      handleScroll() {
        let screenHeight = window.screen.height
        let clientHeight = document.getElementById('chatContainer').clientHeight
        if(clientHeight + 150 > screenHeight){
          // 滚动
          $('#chatContainer').animate({'scrollTop': clientHeight + 150}, 300)
        }
      },
      getFormat(d) {
        var now = new Date(parseFloat(d))
        var year = now.getFullYear()
        var month = now.getMonth() + 1
        var date = now.getDate()
        if (month >= 1 && month <= 9) {
          month = '0' + month
        }
        if (date >= 0 && date <= 9) {
          date = '0' + date
        }
        var hour = now.getHours()
        var minute = now.getMinutes()
        var second = now.getSeconds()
        if (hour >= 1 && hour <= 9) {
          hour = '0' + hour
        }
        if (minute >= 0 && minute <= 9) {
          minute = '0' + minute
        }
        if (second >= 0 && second <= 9) {
          second = '0' + second
        }
        return (
          year +
          '-' +
          month +
          '-' +
          date +
          ' ' +
          hour +
          ':' +
          minute +
          ':' +
          second
        )
      },
    },
    mounted() {
      this.initWebSocket()
    },
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
}
body{
  background-color: rgb(239, 239, 239);
}
.chatContainer {
  overflow-y: scroll;
  width: 100%;
  position: absolute;
  top: 85px;
  bottom: 50px;
  margin-bottom: 10px;
}

.title{
  text-align: center;
  width: 100%;
  height:45px;
  line-height: 45px;
  border-bottom: 1px solid #d9d9d9;
  background: rgba(255, 255, 255, 0.8);
  position: absolute;
  top: 0;
  left: 0;
}
/* 聊天块 */
.contentDiv{
  padding: 20px 10px 0px 10px;
}
.contentDiv .time{
  text-align: center;
  color: #999;
  font-size: 13px;
  margin-bottom: 10px;
}
img{
  width: 40px;
  height: 40px;
  border-radius: 6px;
  margin-top: 4px;
}
.content{
  width: 80%;
  display: flex;
  margin-bottom: 16px;
}
.shop div{
  margin: 0 5px;
}
.content div span{
  font-size: 12px;
  color: #999;
}
.content div p{
  margin-top: 4px;
  font-size: 14px;
  background-color: #fff;
  padding: 10px;
  border-radius:5px;
  line-height: 23px;
  text-align: justify;
}

/* 用户话语 */
.user{
  width: 80%;
  margin-left: 20%;
  display: flex;
  flex-direction: row-reverse;
}
.user div{
  margin-right: 8px;
}
.user div span{
  display: block;
  text-align: right;
}
.sendDiv{
  margin-top: 30px;
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 60px;
  background-color: rgb(246, 246, 246);
  border-top: 1px solid #d9d9d9;
  display: flex;
  align-items: center;
  padding: 0 10px;
  box-sizing: border-box;
}
.sendDiv input{
  width: 75%;
  border-radius: 27px;
  background-color: #fff;
  height: 35px;
  line-height: 35px;
  border: 1px solid #fff;
  padding-left: 10px;
  font-size: 13px;
  margin-right: 10px;
}
.sendDiv button{
  flex: 1;
  background-color: green;
  color: #fff;
  border-radius: 24px;
  border: none;
  height: 30px;
  line-height: 30px;
  font-size: 13px;
}

</style>
