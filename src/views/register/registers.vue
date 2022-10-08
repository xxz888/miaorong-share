<!-- 订单 -->
<template>
  <div class="register_bg">
    <!-- <div class="bg_btn" @click="registerTrueFalseBy = true">立即下载</div> -->
    <div class="brandId_title" >
      <!-- <div class="tit_logo"><img src="../../assets/register/title.png"/></div> -->
      <!-- <div class="tit_zi"><h1> 让美好生活变成一种习惯</h1></div> -->
    </div>

    <div   class="register">

      <div class="register_box">
        <div class="logo">
          <div class="border">
          <img src="../../assets/register/logo.png" alt="" />
          </div>
        </div>
        <ul class="list">
          <li class="item">
            <div class="title">手机号</div>
            <div class="input">
              <div class="left">
                <img  src="../../assets/register/phone_icon.png" alt="" />
              </div>
              <div class="right">
                <van-field
                  class='ra'
                  v-model="rphone"
                  type="digit"
                  clearable
                  maxlength="11"
                  placeholder="请输入手机号"
                />
              </div>
            </div>
          </li>

          <li class="item">
            <div class="title">密码</div>
            <div class="input">
              <div class="left">
                <img  src="../../assets/register/v_pwd.png" alt="" />
              </div>
              <div class="right">
                <van-field
                  class='ra'
                  type="password"
                  v-model="password"
                  center
                  maxlength="16"
                  clearable
                  placeholder="请输入6-16位数字/字母组合密码"
                >
                </van-field>
              </div>
            </div>
          </li>

          <li class="item">
            <div class="title">验证码</div>
            <div class="input">
              <div class="left">
                <img  src="../../assets/register/v_code_icon.png" alt="" />
              </div>
              <div class="right">
                <van-field
                  class='ra'
                  v-model="sms"
                  center
                  clearable
                  placeholder="请输入验证码"
                >
                  <template #button>
                    <!-- <van-button type="primary" size="small" slot="button" :color="num==60?'#08C416':'#08C416'" loading-type="spinner" :loading="num==60?false:true" :loading-text="num==60?message:message" @click="getCode()">获取验证码</van-button> -->
                    <div class="sms" @click=" message == '获取验证码' ? getCode(): '' ">{{ message }}</div>
                  </template>
                </van-field>
              </div>
            </div>
          </li>
         
        </ul> 

         <a class="btn2" @click="_getDownload ()">我已注册，立即下载</a>
        <div class="btn" @click="_register()">立即注册</div>
      </div>
    </div>
  </div>
</template>
<script>
import {
  NavBar,
  DropdownMenu,
  DropdownItem,
  PullRefresh,
  List,
  Empty,
  Popup,
  Field,
} from "vant";
import { authPhone, smsSend, getDownload, register } from "@/api/register";
export default {
  data() {
    return {
      scrollTop: 0, //滚动位置
      phone: "",
      token: "",
      brandId:100,
      userId: "",
      ip: "",
      active: {
        activeCountRecord: 0,
        inviteCount: 0,
        realNameCountRecord: 0,
      },
      direct: {
        directPushAcitveCount: 0,
        directPushCount: 0,
        stagePushAcitveCount: 0,
        stagePushCount: 0,
        twoStagePushAcitveCount: 0,
        twoStagePushCount: 0,
      },
      yearList: [],
      yaer: "",
      month: "",
      monthList: [
        { text: "01月", value: "01" },
        { text: "02月", value: "02" },
        { text: "03月", value: "03" },
        { text: "04月", value: "04" },
        { text: "05月", value: "05" },
        { text: "06月", value: "06" },
        { text: "07月", value: "07" },
        { text: "08月", value: "08" },
        { text: "09月", value: "09" },
        { text: "10月", value: "10" },
        { text: "11月", value: "11" },
        { text: "12月", value: "12" },
      ],
      type: 2,
      payList: [],
      dateTime: "", //当前时间
      bankIconList: [],
      quickList: [],
      isLoading: false,
      isupLoading: false,
      finished: false,
      totalElements: 0,
      page: 1,
      size: 20,
      registerTrueFalseBy: true,
      sms: "",
      rphone: "",
      num: 60,
      message: "获取验证码",
      prePhone: "",
      password:''
    };
  },
  created() {
    localStorage.clear();
    let urlArr = location.href.split("?")[1].split("&");
    this.prePhone = urlArr[0].split("=")[1];
    this.brandId = urlArr[1].split("=")[1];
  },
  mounted() {
    window.addEventListener("scroll", this.handleScroll);
  },
  components: {
    [NavBar.name]: NavBar,
    [DropdownMenu.name]: DropdownMenu,
    [DropdownItem.name]: DropdownItem,
    [PullRefresh.name]: PullRefresh,
    [List.name]: List,
    [Empty.name]: Empty,
    [Popup.name]: Popup,
    [Field.name]: Field,
  },
  methods: {
    // 获取验证码
    getCode() {
      if (this.rphone == "") {
        this.$toast({ message: "请输入手机号", position: "bottom" });
      } else if (this.rphone.length != 11) {
        this.$toast({
          message: "请确认手机号的格式是否正确",
          position: "bottom",
        });
      } else {
         authPhone(this.brandId, this.rphone).then((res) => {
          if (res.resp_code == "000000") {
            //用户已注册，注册了，直接跳下载页面
            this._getDownload()
          } else {
            //用户未注册，发送验证码
              smsSend(this.brandId, this.rphone).then((res) => {
                if (res.resp_code == "000000") {
                  let timer = setInterval(() => {
                    this.message = "(" + this.num + ")秒";
                    this.num--
                    if (this.num == 0) {
                      clearInterval(timer);
                      this.num = 60;
                      this.message = "获取验证码";
                    }
                  }, 1000);
                }
                this.$toast({ message: res.resp_message, position: "bottom" });
              });
          }
        })
      }
    },
    directDown(){
      if(!this.rphone){
         this._getDownload()
      }else{
         this._authPhone()
      }
    },
    
     _register() {
          if (this.sms == "") {
            this.$toast({ message: "请先获取验证码", position: "bottom" });
            return;
          }
          if (/^\d+$/.test(this.password)) {
            this.$toast({ message: "密码不能为纯数字", position: "bottom" });
            return;
          }
          if (!/^\w{6,16}$/.test(this.password)){
            this.$toast({ message: "密码只能由6-16位数字/字母组成", position: "bottom" });
            return;
          }
          register(this.brandId, this.rphone, this.sms, this.prePhone,this.password).then(
            (res) => {
              if (res.resp_code == "000000") {
                this.$toast({ message: "注册成功", position: "bottom" });
                this._getDownload()
              }else{
                  this.$toast({ message: res.resp_message, position: "bottom" });
              }
            }
          )
    },
    // _authPhone() {
    //     authPhone(this.brandId, this.rphone).then((res) => {
    //       if (res.resp_code == "000000") {
    //         this._getDownload()
    //       } else {
    //         if (this.sms == "") {
    //           this.$toast({ message: "请先获取验证码", position: "bottom" });
    //           return;
    //         }
    //         if (/^\d+$/.test(this.password)) {
    //           this.$toast({ message: "密码不能为纯数字", position: "bottom" });
    //           return;
    //         }
    //         if (!/^\w{6,16}$/.test(this.password)){
    //           this.$toast({ message: "密码只能由6-16位数字/字母组成", position: "bottom" });
    //           return;
    //         }
    //         register(this.brandId, this.rphone, this.sms, this.prePhone,this.password).then(
    //           (res) => {
    //             if (res.resp_code == "000000") {
    //               this.$toast({ message: "注册成功", position: "bottom" });
    //               // this.$router.push("/down");
    //               this._getDownload()
    //             }else{
    //                 this.$toast({ message: res.resp_message, position: "bottom" });
    //             }
    //           }
    //         )
    //       }
    //     })
    // },
    _getDownload() {
      getDownload(this.brandId).then((res) => {
        if (res.resp_code == "000000") {                        
          var u = navigator.userAgent,
            app = navigator.appVersion;
          var isAndroid = u.indexOf("Android") > -1 || u.indexOf("Linux") > -1; //g
          var isIOS = !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/); //ios终端
          if (isAndroid) {
            if (
              res.result.downloadAndroidUrl != null &&
              res.result.downloadAndroidUrl != ""
            ) {
                this.$toast({ message: "注册成功", position: "bottom" });
              // window.location.href = res.result.downloadAndroidUrl;
            } else {
              this.$toast({ message: "当前无下载链接", position: "bottom" });
            }
          }
          if (isIOS) {
            if (res.result.iOS.url != null && res.result.iOS.url != "") {
                              this.$toast({ message: "注册成功", position: "bottom" });

              // window.location.href = res.result.iOS.url;
            } else {
              this.$toast({ message: "当前无下载链接", position: "bottom" });
            }
          }
        }
      });
    },
  },
};
/*  */
</script>
<style scoped>
.register_bg {
  height: 100vh;
  /* height: 825px; */
  width: 100%;
  background: url("../../assets/register/bg.png") center no-repeat;
  background-size: 100% 100%;
  position: relative;
}
.bg_btn {
  width: 296px;
  height: 44px;
  background: #ffffff;
  box-shadow: 0px 3px 8px 0px rgba(93, 32, 0, 0.2);
  border-radius: 22px;
  position: absolute;
  bottom: 20px;
  left: 0;
  right: 0;
  margin: 0 auto;
  color: #ff4f2d;
  font-size: 20px;
  line-height: 44px;
  text-align: center;
}
.register {
  overflow-y: unset !important;
   background-color:#fff;
   width: 268px;
   position:absolute;
   top: 25%;
   left: 15%;
   border-radius: 15px;
}
.register_box {
  width: 275px;
  position: relative;
  z-index: 1000;
}

.logo {
  height: 60px;
  width: 100%;
  text-align: center;
  padding: 10px;
}
.logo .border{
  width: 115px;
  height: 115px;
  position: relative;
  margin: 0px auto;
  bottom: 60px;
  overflow: hidden;
}
.logo  img {
  position:absolute;
  height:80px;
  width: 80px;
  text-align: center;
  right: 12px;
  margin-top: 10px;
  background: #fff;

}
.list {
  font-size: 14px;
  padding: 0 35px;
}
.item .title {
  color: #3AC461;
  text-align: left;
  padding-top: 10px;
}
.input {
  border-bottom: 2px solid #3AC461; 
  display: flex;
  
}
.input .left {
  width: 30px;
}
.input .left img {
  height: 24px;
  width: 18px;
  margin-top: 8px;
}
.btn {
  width: 180px;
  height: 35px;
  /* background-image:linear-gradient(rgb(236,138,101), rgb(225, 84, 49));; */
  background-color:  #3AC461;
  /* border-radius: 29px; */
  border-radius: 12px;
  font-size: 14px;
  text-align: center;
  line-height: 35px;
  color: #fff;
  font-weight: 500;
  margin: 20px auto 30px auto;
}
.btn2 {
  display: inline-block;
  font-size: 14px;
  width: 100%;
    text-align: center;
  color:  #3AC461;
  margin-top: 10px ;
}
.sms {
  background-color:  #3AC461;
  color: #fff;
  font-size: 13px;
  padding: 0px 2px;
}
/* .ra{
    border-radius: 30px;
} */
.brandId_title{
  padding-top:50px; 
  display:flex;
  height: 30px;
}
.brandId_title .tit_logo{
  flex: 1;
  text-align: left;
  margin-left: 20px;
}
.brandId_title .tit_zi{
  flex: 3;
}

.brandId_title img{
  width: 250px;
}
.brandId_title h1{
  color: rgb(239,130,0);
  font-size: 20px;
  line-height: 50px;
  text-align: left;
  padding-left:10px ;
} 

.input >>> .van-field{
  background: none;
}

</style>