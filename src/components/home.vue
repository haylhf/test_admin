<template xmlns:v-on="http://www.w3.org/1999/xhtml" xmlns:v-bind="http://www.w3.org/1999/xhtml">
    <div>
        <img :src="getBgImg()"
             style="min-width: 100%;min-height: 100%;position: fixed;z-index: -100;right:0; bottom: 0;"
             v-show="!showAD"/>
        <div v-show="!showAD">
            <div class="homeDiv">
                <el-row style="margin-top: 30px">
                    <el-col :offset="5" :span="3" style="text-align: center">
                        <span style="font-size: 30px;color: white;font-weight: lighter">{{currentTime}}</span>
                    </el-col>
                    <el-col :offset="2" :span="4" style="text-align: center;margin-top: 30px" v-if="isShowVIP">
                        <img src="../assets/img/welcome.png"/>
                    </el-col>
                    <el-col :offset="2" :span="4" style="text-align: center;margin-top: 30px" v-else>
                        <span style="font-size: 40px;color: white;">{{title}}</span>
                    </el-col>
                    <el-col :offset="2" :span="4" style="text-align: center">
                        <span style="font-size: 28px;color: white;font-weight: lighter">签到：</span>
                        <span style="font-size: 30px;color: white;font-weight: lighter">{{getSignIn()}}</span>
                    </el-col>
                </el-row>
            </div>
            <VipPage ref="vipPage" v-show="isShowVIP"></VipPage>
            <StaffSignPage ref="staffPage" v-show="!isShowVIP"></StaffSignPage>
        </div>
        <video :src="getVideoBg()" autoplay muted loop
               style="min-width: 100%;min-height: 100%;position: fixed;z-index: -100;right:0; bottom: 0;"
               v-show="showAD">
        </video>
        <!--<StaffPage ref="staffPage" v-else ></StaffPage >-->
    </div>

</template>

<script>
    var currentInterval;
    var showADTimerId;
    var mqttReconnectInterval = null;
    var isLoading = false;
    var hostname = MqttServer,
        port = ServerPort,
        clientId = `client-${newGuid()}`,
        timeout = 30,
        keepAlive = 100,
        qos = 1,
        cleanSession = false,
        ssl = false,
        userName = 'admin',
        password = 'password',
        sendTopic = "sign_feedback";
    var client = new Paho.MQTT.Client(hostname, port, clientId);
    //建立客户端实例
    var options = {
        invocationContext: {
            host: hostname,
            port: port,
            path: client.path,
            clientId: clientId
        },
        timeout: timeout,
        keepAliveInterval: keepAlive,
        cleanSession: cleanSession,
        useSSL: ssl,
        userName: userName,
        password: password,
        onSuccess: onConnect,
        onFailure: function (e) {
            console.log(`connect failure: ${e}`);
        },
    };
    $(document).ready(function () {
        client.connect(options);//连接服务器并注册连接成功处理事件
        client.onConnectionLost = onConnectionLost;//注册连接断开处理事件
        client.onMessageArrived = onMessageArrived;//注册消息接收处理事件
    })

    function onConnect() {
        console.log("connect successfully");
        if (mqttReconnectInterval != null) {
            clearInterval(mqttReconnectInterval);
            mqttReconnectInterval = null;
        }
        for (let item of ServerTOPIC)//订阅主题
        {
            console.log(`subscribed server topic: ${item}`);
            client.subscribe(item);
        }
    }

    var isActived = true;
    window.ondeactivate = () => {
        isActived = false;
    };
    window.onactivate = () => {
        isActived = true;
    };
    window.onblur = () => {
        isActived = false;
    };
    window.onfocus = () => {
        isActived = true;
    };

    window.onclick = () => {
        //requestFullScreen();
    };

    function onConnectionLost(responseObject) {
        if (responseObject.errorCode !== 0) {
            console.log("连接已断开");
            console.log("onConnectionLost:" + responseObject.errorMessage);
            mqttReconnectInterval = setInterval(() => {
                client.connect(options);
                client.onConnectionLost = onConnectionLost;//注册连接断开处理事件
                client.onMessageArrived = onMessageArrived;//注册消息接收处理事件
            }, 2000);
        }
    }

    function onMessageArrived(message) {
        console.log("收到消息:" + message.payloadString);
        console.log("主题：" + message.destinationName);
        var data = null;
        try {
            data = jQuery.parseJSON(message.payloadString);
            console.log("解析出来的：data：" + JSON.stringify(data));
        } catch (e) {
            console.log(e);
        }
        // while (isLoading) {
        //    sleep(1000); //wait;
        // }
        if (data != null) {
            switch (message.destinationName) {
                case ServerTOPIC[0]: //statff
                    //_this.isShowVIP = false;
                    onVisitorSign(data);
                    break;
                case ServerTOPIC[1]://vip
                    _this.isShowVIP = true;
                    setTimeout(() => {
                        _this.isShowVIP = false;
                    }, 1000 * 15);
                    onVisitorSign(data);
                    break;
                case ServerTOPIC[2]:
                    if (_this.$refs.vipPage) {
                        _this.$refs.vipPage.reset();
                    }
                    if (_this.$refs.staffPage) {
                        _this.$refs.staffPage.reset();
                    }
                    break;
                default:
                    console.log("未知主题消息...")
                    break;
            }
            console.log(`show UI VIP:${_this.isShowVIP}`);
        }
    }

    function onVisitorSign(signDataList) {
        // isLoading = true;
        var dataList = [];
        try {
            console.log(signDataList.length)
            _this.showAD = false;
            //清除之前的timer，一分钟如果没有考勤信息过来，开始播放宣传片
            if (showADTimerId != null) {
                clearTimeout(showADTimerId);
            }
            showADTimerId = setTimeout(() => {
                _this.showAD = true;
            }, 1000 * 60 * 2);

            for (let i = 0; i < signDataList.length; i++) {
                let signData = signDataList[i];
                let data = Object.assign(signData.person.person_information);
                try {
                    data.signTime = new Date(signData.timestamp * 1000).format("hh:mm:ss");
                } catch (e) {
                }
                data.device_id = signData.device_id;
                //data.photo = require('../assets/img/male.png'); //`http://api.vaiwan.com:8081/image/${signData.person.face_list[0].face_image_id}`;
                data.photo = `http://api.vaiwan.com:8081/image/${signData.person.face_list[0].face_image_id}`;
                dataList.push(data);

            }
            if (_this.$refs.vipPage) {
                _this.$refs.vipPage.updateData(dataList);
            }
            if (_this.$refs.staffPage) {
                _this.$refs.staffPage.updateData(dataList);
            }
        } catch (e) {
            console.log(e)
        } finally {
            isLoading = false;
        }
    }

    var _this;
    import StaffPage from '../components/staff_page.vue';
    import VipPage from '../components/vip_page.vue';
    import StaffSignPage from '../components/staffsign_page.vue';
    var signInBg = require('../assets/img/bg.png');
    var vipBg = require('../assets/img/bg_gold.png');

    export default {
        name: "home",
        components: {
            StaffPage,
            VipPage,
            StaffSignPage,
        },
        data() {
            _this = this;
            return {
                sendText: "Hello mqtt",
                title: HOME_SCREEN_TITLE,
                currentTime: "",
                staffNum: 0,
                signInNum: 0,
                isShowVIP: false,
                showAD: true
            }
        },
        methods: {
            btnTest() {

                var id = Math.round(Math.random() * 1000000);
                onVisitorSign(JSON.parse(`
[
  {
    "device_id": "string",
    "face_id": "string",
    "face_image_id": "string",
    "identity": "STRANGER",
    "person": {
      "face_list": [
        {
          "face_id": "string",
          "face_image_id": "string",
          "scene_image_id": "string"
        }
      ],
      "identity": "STAFF",
      "meta": {},
      "person_id": "string",
      "person_information": {
        "birthday": "string",
        "company": "string",
        "employed_date": "string",
        "id": "string",
        "identity_number": "string",
        "name": "string${id}",
        "phone": "string",
        "remark": "string",
        "visit_end_timestamp": 0,
        "visit_purpose": "0",
        "visit_start_timestamp": 0,
        "visit_time_type": "0",
        "visitee_name": "string"
      },
      "tag_id_list": [
        "string"
      ],
      "upload_time": 0
    },
    "scene_image_id": "string",
    "score": 0,
    "timestamp": 0,
    "track_id": "string"
  },
  {
    "device_id": "string",
    "face_id": "string",
    "face_image_id": "string",
    "identity": "STRANGER",
    "person": {
      "face_list": [
        {
          "face_id": "string",
          "face_image_id": "string",
          "scene_image_id": "string"
        }
      ],
      "identity": "STAFF",
      "meta": {},
      "person_id": "string",
      "person_information": {
        "birthday": "string",
        "company": "string",
        "employed_date": "string",
        "id": "string",
        "identity_number": "string",
        "name": "string${id + 1}",
        "phone": "string",
        "remark": "string",
        "visit_end_timestamp": 0,
        "visit_purpose": "0",
        "visit_start_timestamp": 0,
        "visit_time_type": "0",
        "visitee_name": "string"
      },
      "tag_id_list": [
        "string"
      ],
      "upload_time": 0
    },
    "scene_image_id": "string",
    "score": 0,
    "timestamp": 0,
    "track_id": "string"
  },
  {
    "device_id": "string",
    "face_id": "string",
    "face_image_id": "string",
    "identity": "STRANGER",
    "person": {
      "face_list": [
        {
          "face_id": "string",
          "face_image_id": "string",
          "scene_image_id": "string"
        }
      ],
      "identity": "STAFF",
      "meta": {},
      "person_id": "string",
      "person_information": {
        "birthday": "string",
        "company": "string",
        "employed_date": "string",
        "id": "string",
        "identity_number": "string",
        "name": "string${id + 2}",
        "phone": "string",
        "remark": "string",
        "visit_end_timestamp": 0,
        "visit_purpose": "0",
        "visit_start_timestamp": 0,
        "visit_time_type": "0",
        "visitee_name": "string"
      },
      "tag_id_list": [
        "string"
      ],
      "upload_time": 0
    },
    "scene_image_id": "string",
    "score": 0,
    "timestamp": 0,
    "track_id": "string"
  }
]`
                ));//方法1
            },

            getBgImg() {
                var bg;
                if (_this.isShowVIP) {
                    bg = vipBg;
                } else {
                    bg = signInBg;
                }
                return bg;
            },
            getVideoBg() {
                var bg = require('../assets/img/bg.mp4');
                return bg;
            },
            onSend() {
                let strMsg = _this.sendText;// document.getElementById("msg").value;
                if (strMsg) {
                    let message = new Paho.MQTT.Message(strMsg);
                    message.destinationName = sendTopic; //发送主题
                    client.send(message);
                    console.log(`send data: ${strMsg}`)
                    _this.sendText = "";
                }
            },
            getSignIn() {
                return _this.signInNum + " / " + _this.staffNum;
            }
        },
        computed: {},
        filters: {},
        created: function () {
            _this = this;

        },
        mounted: function () {
            currentInterval = setInterval(function doAnimation() {

                _this.currentTime = new Date().format("MM月dd日 hh:mm");
                $.ajax({
                    url: HOST + "user/getStaffNum",
                    type: 'GET',
                    dataType: 'json',
                    success: function (data) {
                        if (data.code == 200) {
                            _this.staffNum = data.data;
                        }
                    },
                    error: function (data) {

                    }
                })
                $.ajax({
                    url: HOST + "user/getStaffSignInNum",
                    type: 'GET',
                    dataType: 'json',
                    success: function (data) {
                        if (data.code == 200) {
                            _this.signInNum = data.data;
                        }
                    },
                    error: function (data) {

                    }
                })

            }, 1000);//定时器

        },
        destroyed: function () {
            clearInterval(currentInterval);
            clearTimeout(showADTimerId);
        }
    }

</script>
<style>
    span {
        text-align: center;
    }
</style>
