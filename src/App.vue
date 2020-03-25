<template>
  <div id="app">
    <div class="leftBox">
      <div class="logs">
        <span v-bind:key="index" v-for="(log,index) in logs">{{log}}</span>
      </div>
    </div>
    <div class="rightBox">
      <!-- 用户登录 -->
      <div>
        <input v-model="username" type="text" placeholder="用户名" />
        <br />
        <input v-model="password" type="text" placeholder="密码" />
        <br />
        <br />
        <button v-on:click="onLogin">用户登录</button>
      </div>
      <br />
      <br />
      <!--  连接IM -->
      <div>
        <input v-model="uid" type="text" placeholder="uid" />
        <br />
        <input v-model="token" type="text" placeholder="token" />
        <br />
        <br />
        <button v-on:click="onConnectLIM">连接狸猫IM</button>
      </div>
      <br />
      <br />
      <!-- 发送消息 -->
      <div>
        <input v-model="to" type="text" placeholder="接受用户uid"/>
         <br />
          <br />
        <textarea v-model="msg" placeholder="请输入消息内容" />
        <br />
        <br />
        <button v-on:click="onSend">发送</button>
      </div>
    </div>
  </div>
</template>

<script>
/* eslint-disable */
import lim from "limao";
import axios from "axios";
export default {
  name: "App",
  components: {},
  data: () => ({
    username: "",
    password: "",
    uid: "",
    token: "",
    to: "",
    msg: '',
    logs: []
  }),
  mounted() {
    axios.defaults.baseURL = "http://49.235.59.182:8080/v1/";
    lim.options.connectURL = "ws://49.235.59.182:8030/"; // IM服务器连接地址
    // 连接状态监听
    lim.addConnectStatusListener(status => {
      if (status == 1) {
        this.logs.unshift("连接成功！");
      } else {
        this.logs.unshift("断开连接！");
      }
    });
    // 监听消息
    lim.addMessageListener((recvPacket)=>{
        this.logs.unshift("收到消息->",recvPacket);
    });
    // 发送消息
    lim.addSendMessageListener((sendPacket)=>{
        this.logs.unshift("发送消息->",sendPacket);
    });
    // 发送消息回执
    lim.addMessageStatusListener((sendackPacket)=>{
        this.logs.unshift("发送消息回执->",sendackPacket);
    })
  },
  methods: {
    async onLogin() {
      let resp = await axios.post("user/login", {
        username: this.username,
        password: this.password,
        flag: 1
      }).catch((err)=>{
          this.logs.unshift("登录失败！->",err);
      });
      if (resp && resp.data) {
        this.uid = resp.data.uid;
        this.token = resp.data.token;
      }
    },
    onConnectLIM() {
      // 连接到狸猫服务器
      lim.connect(this.uid,this.token);
    },
    onSend() {
      if(this.to=='') {
        alert('请输入接受者uid！');
          return
      }
      if(this.msg == '') {
        alert('请输入消息内容！');
        return;
      }
      // 发送消息
      lim.sendMessage({type:1,content:this.msg},{channelID:this.to,channelType:1})
    }
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  display: flex;
}
.leftBox {
  flex: 1;
}
.rightBox {
  width: 50%;
}

.logs span {
  display: block;
}
</style>
