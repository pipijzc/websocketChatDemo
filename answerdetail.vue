<template>
  <div style="margin-top:30px">
    <el-row>
      <el-col :span="14" :offset="5">
           <div class="title">专家咨询</div>
         <!-- 主体   -->
        <div class="triangle" ref="box">
          <ul v-for="(item,index) in list" :key="index">
            <li :class="item.type==='he' ?  'textLeft': 'textRight'">
              <span :class="item.type==='he' ?  'leftspan': 'rightspan'" v-html="analyzeEmoji(item.content)">{{analyzeEmoji(item.content)}}</span>
              <img :src="item.type==='he' ? robotIcon : User.avatarUrl" alt="" class="usericon" :class="item.type==='he' ?  'left': 'right'">
            </li>
          </ul>   
        </div>
          <!-- 输入框 -->
          <div class="input-group">
            <el-input class="inputcontent" placeholder="发送消息" type="text" v-model="contentText"></el-input>
            <el-button v-on:keyup.enter="sendText" @click="sendText" size="medium" class="buttonSize">发送</el-button>
          </div>
            <!-- 表情包 -->
           <div :class="pBody?'OwO':'OwO OwO-open'">
            <div class="OwO-logo" @click="pBody=!pBody">
              <span class="emojButton">{{pBody==false?'再点一下':'表情😄'}}</span>
            </div>
            <div class="OwO-body">
              <ul class="OwO-items OwO-items-show">
                <li
                  class="OwO-item"
                  v-for="(oitem,index) in OwOlist"
                  :key="'oitem'+index"
                  @click="choseEmoji(oitem.title)"
                >
                  <img :src="'https://avatar-img.gz.bcebos.com/' +oitem.url" alt />
                </li>
              </ul>
            </div>
          </div>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import { getappoinement, getTeacherAppoinement,getTheirAvatar } from "../ajax/ajax";
export default {
  data() {
    return {
      // 输入的内容
      contentText: "",
      // 聊天列表
      list: "",
      // 用于连接的工单id
      workId: "",
      // 对方头像
      robotIcon:'',
      // websocket对象
      websock: null,
      // 自己的用户信息
      User:'',
      // 表情列表
      OwOlist: [
        //表情包和表情路径
        { title: "微笑", url: "weixiao.gif" },
        { title: "嘻嘻", url: "xixi.gif" },
        { title: "哈哈", url: "haha.gif" },
        { title: "可爱", url: "keai.gif" },
        { title: "可怜", url: "kelian.gif" },
        { title: "挖鼻", url: "wabi.gif" },
        { title: "吃惊", url: "chijing.gif" },
        { title: "害羞", url: "haixiu.gif" },
        { title: "挤眼", url: "jiyan.gif" },
        { title: "闭嘴", url: "bizui.gif" },
        { title: "鄙视", url: "bishi.gif" },
        { title: "爱你", url: "aini.gif" },
        { title: "泪", url: "lei.gif" },
        { title: "偷笑", url: "touxiao.gif" },
        { title: "亲亲", url: "qinqin.gif" },
        { title: "生病", url: "shengbing.gif" },
        { title: "太开心", url: "taikaixin.gif" },
        { title: "白眼", url: "baiyan.gif" },
        { title: "右哼哼", url: "youhengheng.gif" },
        { title: "左哼哼", url: "zuohengheng.gif" },
        { title: "嘘", url: "xu.gif" },
        { title: "衰", url: "shuai.gif" },
        { title: "吐", url: "tu.gif" },
        { title: "哈欠", url: "haqian.gif" },
        { title: "抱抱", url: "baobao.gif" },
        { title: "怒", url: "nu.gif" },
        { title: "疑问", url: "yiwen.gif" },
        { title: "馋嘴", url: "chanzui.gif" },
        { title: "拜拜", url: "baibai.gif" },
        { title: "思考", url: "sikao.gif" },
        { title: "汗", url: "han.gif" },
        { title: "困", url: "kun.gif" },
        { title: "睡", url: "shui.gif" },
        { title: "钱", url: "qian.gif" },
        { title: "失望", url: "shiwang.gif" },
        { title: "酷", url: "ku.gif" },
        { title: "色", url: "se.gif" },
        { title: "哼", url: "heng.gif" },
        { title: "鼓掌", url: "guzhang.gif" },
        { title: "晕", url: "yun.gif" },
        { title: "悲伤", url: "beishang.gif" },
        { title: "抓狂", url: "zhuakuang.gif" },
        { title: "黑线", url: "heixian.gif" },
        { title: "阴险", url: "yinxian.gif" },
        { title: "怒骂", url: "numa.gif" },
        { title: "互粉", url: "hufen.gif" },
        { title: "书呆子", url: "shudaizi.gif" },
        { title: "愤怒", url: "fennu.gif" },
        { title: "感冒", url: "ganmao.gif" },
        { title: "心", url: "xin.gif" },
        { title: "伤心", url: "shangxin.gif" },
        { title: "猪", url: "zhu.gif" },
        { title: "熊猫", url: "xiongmao.gif" },
        { title: "兔子", url: "tuzi.gif" },
        { title: "喔克", url: "ok.gif" },
        { title: "耶", url: "ye.gif" },
        { title: "棒棒", url: "good.gif" },
        { title: "不", url: "no.gif" },
        { title: "赞", url: "zan.gif" },
        { title: "来", url: "lai.gif" },
        { title: "弱", url: "ruo.gif" },
        { title: "草泥马", url: "caonima.gif" },
        { title: "神马", url: "shenma.gif" },
        { title: "给力", url: "geili.gif" },
        { title: "蛋糕", url: "dangao.gif" },
        { title: "发红包", url: "fahongbao.gif" },
        { title: "笑哭跳", url: "xiaokutiao.gif" },
        { title: "开心跳", url: "kaixintiao.gif" },
        { title: "咬舌跳", url: "yaoshetiao.gif" },
        { title: "鬼脸跳", url: "guiliantiao.gif" },
        { title: "无脸跳", url: "wuliantiao.gif" },
        { title: "惊讶跳", url: "jingyatiao.gif" },
        { title: "墨镜跳", url: "mojingtiao.gif" },
        { title: "口罩跳", url: "kouzhaotiao.gif" },
        { title: "龇牙跳", url: "ziyatiao.gif" }
      ],
      pBody: true //表情打开控制
    };
  },
  methods: {
    initWebSocket() {
      //初始化weosocket
      let wsuri = "ws://106.13.98.231/chat/" + this.workId;
      this.websock = new WebSocket(wsuri);
      this.websock.onmessage = this.websocketonmessage;
      this.websock.onopen = this.websocketonopen;
      this.websock.onerror = this.websocketonerror;
      this.websock.onclose = this.websocketclose;
    },
    // 开启时
    websocketonopen() {
      //连接建立之后执行send方法发送数据
      this.websocketsend(JSON.stringify(this.contentText));
    },
    // 连接错误时
    websocketonerror() {    
      //连接建立失败重连
      this.initWebSocket();
    },
    // 接受信息时
    websocketonmessage(e) {
      //数据接收
      if(JSON.parse(e.data)!==''){
         this.list = [
        ...this.list,
        { type: "he", content: JSON.parse(e.data) }
      ];
      }
    },
    // 发送信息时
    websocketsend(Data) {
      //数据发送
      this.websock.send(Data);
      // 清空输入框
      this.contentText = "";
    },
    // 连接关闭时
    websocketclose() {
      this.$message({
        message: '连接已断开',
        type: 'warning'
      })
    },

    //发送聊天信息
    sendText() {
      let that = this;
      if (this.contentText.trim() !== "") {
        //通过type字段进行区分是自己（mine）发的还是对方（he)
        this.list = [...this.list, { type: "mine", content: this.contentText }]; 
        this.websocketonopen();
      } else {
        this.$message({
          message: "请输入内容",
          type: "warning"
        });
      }
    },
    // 输入表情
     choseEmoji(inner) { 
      this.contentText += "[" + inner + "]";
    },
    analyzeEmoji(cont) {
      //编译表情替换成图片展示出来
      var pattern1 = /\[[\u4e00-\u9fa5]+\]/g;
      var pattern2 = /\[[\u4e00-\u9fa5]+\]/;
      var content = cont.match(pattern1);
      var str = cont;
      if (content) {
        for (let i = content.length - 1; i >= 0; i--) {
          for (let j = this.OwOlist.length - 1; j >= 0; j--) {
            if ("[" + this.OwOlist[j].title + "]" == content[i]) {
              var src = this.OwOlist[j].url;
              break;
            }
          }
          str = str.replace(
            pattern2,
            '<img style="vertical-align: middle" src="https://avatar-img.gz.bcebos.com/' +
              src +
              '" />'
          );
        }
      }
      return str;
    }
  },
   watch: {
      //监听list,当有修改的时候进行div的屏幕滚动，确保能看到最新的聊天
      list: function () {
        let that = this;
        setTimeout(() => {
          that.$refs.box.scrollTop = that.$refs.box.scrollHeight;
        }, 0);
        //加setTimeout的原因：由于vue采用虚拟dom，我每次生成新的消息时获取到的div的scrollHeight的值是生成新消息之前的值，
        // 所以造成每次都是最新的那条消息被隐藏掉了
      }
    },
  async mounted() {
    // 获取咨询用户的id和心理师的id
    let { id,icon } = this.$route.query;
    this.workId = id;
    // 用于获取本方头像
    this.User = JSON.parse(localStorage.getItem('psyUserInfo'))
    // 初始化websocket
    this.initWebSocket();
    // 获取对方头像
    const result = await getTheirAvatar(icon)
    this.robotIcon = result.data
  }
};
</script>

<style scoped>
.triangle {
  position: relative;
  width: 100%;
  border-radius: 5px;
  height: 550px;
  overflow: scroll;
  overflow-x: hidden;
  background-color: #ebebe9;
}
.triangle ul {
  padding: 10px;
}
.triangle li {
  padding: 5px;
  margin-bottom: 10px;
}
.triangle li span {
  position: relative;
  border-radius: 7px;
  background-color: #a6e860;
  padding: 6px 10px 8px 10px;
  z-index: 1;
}
.triangle .textLeft span {
  background-color: white;
}
.leftspan{
right: -13px;
}
.rightspan{
  right: 13px;

}
.triangle li.textLeft:before {
  content: "";
  box-sizing: border-box;
  position: relative;
  left: -10px;
  top: 9px;
}
.triangle li.textLeft span:before {
  content: "";
  display: block;
  width: 0;
  height: 0;
  border: 8px solid transparent;
  border-right: 8px solid white;
  position: absolute;
  top: 8px;
  left: -16px;
}
.triangle li.textRight:after {
  /* content: url('../image/night.jpg'); */
  content:'';
  /* display: block; */
  /* box-sizing: border-box; */
  position: relative;
  width: 50px!important;
  height: 50px;
  right: -10px;
  top: 9px;
}
.triangle li.textRight span:after {
  content: "";
  display: block;
  width: 0;
  height: 0;
  border: 8px solid transparent;
  border-left: 8px solid #a6e860;
  position: absolute;
  top: 8px;
  right: -16px;
}
li {
  list-style: none;
}
.textRight {
  text-align: right;
}
.input-group {
  /* position: absolute; */
  /* bottom: 0; */
  width: 100%;
  height: 40px;
  overflow: hidden;
}
.inputcontent {
  float: left;
  width: 90%!important;
  height: 24px;
}
.buttonSize {
  float: right;
  height: 40px;
  width: 7% !important;
}
.title{
  height: 40px;
  line-height: 40px;
  border-radius: 10px 10px 0 0;
  background-color: #97dffd;
  text-align: center;
  font-weight: 700;
  font-size: 18px;
}
.usericon{

  /* position: absolute; */
  width: 40px;
  height: 40px;
  border-radius: 50%;
}
.left{
  float: left;
}
.right{
  float: right;
}

.OwO {
  position: relative;
  z-index: 1;
}
.OwO .OwO-logo {
  position: relative;
  border-radius: 4px;
  color: #fff;
  display: inline-block;
  background: #a9e4ae;
  border: 1px solid #ddd;
  font-size: 14px;
  padding: 0px 12px;
  cursor: pointer;
  height: 30px;
  box-sizing: border-box;
  z-index: 2;
  line-height: 30px;
  margin-top: 10px;
  transition: all ease-in 0.2s;
}
.OwO .OwO-logo:hover {
  background-color: #525e81;
}
.OwO .OwO-body {
  position: absolute;
  background: #fff;
  border: 1px solid #ddd;
  z-index: 1;
  top: -210px;
  border-radius: 0 4px 4px 4px;
  display: none;
}
.OwO-open .OwO-body {
  display: block;
}
.OwO-open .OwO-logo {
  border-radius: 4px 4px 0 0;
  border-bottom: none;
}
.OwO-open .OwO-logo:hover {
  animation: none;
  -webkit-animation: none;
}
.OwO .OwO-items {
  max-height: 197px;
  overflow: scroll;
  font-size: 0;
  padding: 10px;
  z-index: 1;
  /* margin-top: 10px; */
}
.OwO .OwO-items .OwO-item {
  background: #f7f7f7;
  padding: 5px 10px;
  border-radius: 5px;
  display: inline-block;
  margin: 0 10px 12px 0;
  transition: 0.3s;
  line-height: 19px;
  font-size: 20px;
  cursor: pointer;
}
.OwO .OwO-items .OwO-item:hover {
  background: #eee;
  box-shadow: 0 2px 2px 0 rgba(0, 0, 0, 0.14), 0 3px 1px -2px rgba(0, 0, 0, 0.2),
    0 1px 5px 0 rgba(0, 0, 0, 0.12);
  animation: a 5s infinite ease-in-out;
  -webkit-animation: a 5s infinite ease-in-out;
}
.OwO .OwO-body .OwO-bar {
  width: 100%;
  height: 30px;
  border-top: 1px solid #ddd;
  background: #fff;
  border-radius: 0 0 4px 4px;
  color: #444;
}
.OwO .OwO-body .OwO-bar .OwO-packages li {
  display: inline-block;
  line-height: 30px;
  font-size: 14px;
  padding: 0 10px;
  cursor: pointer;
  margin-right: 3px;
  text-align: center;
}
.OwO .OwO-body .OwO-bar .OwO-packages li:first-of-type {
  border-radius: 0 0 0 3px;
}
img {
  text-align: center;
}
</style>