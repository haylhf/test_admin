<template xmlns:v-on="http://www.w3.org/1999/xhtml" xmlns:v-bind="http://www.w3.org/1999/xhtml">
    <div>
        <!--<video :src="getBgVideo()"-->
        <!--style="right:0; bottom: 0; z-index: -100;min-width: 100%;min-height: 100%;position: fixed;height: auto;width: auto;-webkit-filter: grayscale(20%)"-->
        <!--loop="loop" autoplay="autoplay" muted ></img >-->
        <img :src="getBgVideo()"
             style="min-width: 100%;min-height: 100%;position: fixed;z-index: -100;right:0; bottom: 0;"/>
        <div class="homeDiv">
            <el-row style="margin-top: 30px">
                <el-col :offset="5" :span="3" style="text-align: center">
                    <span style="font-size: 30px;color: white;font-weight: lighter">{{currentTime}}</span>
                </el-col>
                <el-col :offset="2" :span="4" style="text-align: center;margin-top: 30px">
                    <span style="font-size: 40px;color: white;">{{title}}</span>
                </el-col>
                <el-col :offset="2" :span="4" style="text-align: center">
                    <span style="font-size: 28px;color: white;font-weight: lighter">签到：</span>
                    <span style="font-size: 30px;color: white;font-weight: lighter">{{getSignIn()}}</span>
                </el-col>
            </el-row>
            <div id="divRoot" style="margin:10%;background-color: red">

            </div>
            <div id="root" class="rootDiv">
                <div id="container" class="container">
                </div>
            </div>
        </div>
        <!--<div >-->
        <!--<el-form >-->
        <!--<el-col :span="24" >-->
        <!--<el-form-item label="内容：" label-width="100" style="width: 100%" >-->
        <!--<el-input v-model="sendText" type="textarea"-->
        <!--:rows="5" clearable style="font-size: 18px" ></el-input >-->
        <!--</el-form-item >-->
        <!--</el-col >-->
        <!--<el-button type="primary" @click="onSend" size="small" >Send</el-button >-->
        <!--</el-form >-->
        <!--</div >-->
    </div>
</template>

<script>
    var container = null;
    var rootContainer = null;
    var indexPiece = 0,
        htmlPic = '',
        arrayPic = null,
        rotate = 0;//40
    var transform = function (element, value, key) {
        key = key || "Transform";
        ["Moz", "O", "Ms", "Webkit", ""].forEach(function (prefix) {
            element.style[prefix + key] = value;
        });
        return element;
    };

    var hostname = MqttServer,
        port = ServerPort,
        clientId = `client-${newGuid()}`,
        timeout = 5,
        keepAlive = 100,
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

    function onConnect() {
        console.log("connect successfully");
        for (let item of ServerTOPIC)//订阅主题
        {
            console.log(`subscribed server topic: ${item}`);
            client.subscribe(item);
        }
    };

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
    window.onload = function () {
        client.connect(options);//连接服务器并注册连接成功处理事件
        client.onConnectionLost = onConnectionLost;//注册连接断开处理事件
        client.onMessageArrived = onMessageArrived;//注册消息接收处理事件


        var $ = function (selector) {
            return document.querySelector(selector);
        };
        // 获取元素
        container = $("#container");
        container.addEventListener("click", function () {
            transform(container, `rotateY(${(1 * rotate * ++indexPiece)}deg)`);
        });

        if (container != null) {
            arrayPic = [1, 2, 3, 4, 5, 6, 7, 8, 9] // load image list
            rotate = 360 / arrayPic.length;
            for (let i of arrayPic) {
                let img = require(`../assets/img/female.png`);
                if (i % 2 == 0) {
                    img = require(`../assets/img/male.png`);
                }
                htmlPic = htmlPic + `<div id="div${i}" class="col-center-block text-center" style="background: darkgrey;border:black solid 1px;">
                <img id="img${i}" src="${img}" style="width: 150px;height: 150px;margin-top: 20px;" />
                <br/>
                <br/>
                <p class="text-center">test${i}</p>
                <p class="text-center">test${i}</p>
                </div>`;
            }
            container.innerHTML = htmlPic;
            var transZ = 35 * arrayPic.length;
            arrayPic.forEach(function (i, j) {
//			      console.log(`${i} image,  rotation degree:${j * rotate}`);
                transform($(`#div${i}`), `rotateY(${j * rotate}deg) translateZ(${transZ}px)`);
            });
            //console.log(JSON.stringify(container.innerHTML))
        }
    };

    function onConnectionLost(responseObject) {
        if (responseObject.errorCode !== 0) {
            console.log("onConnectionLost:" + responseObject.errorMessage);
            console.log("连接已断开");
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
        if (data != null) {
            switch (message.destinationName) {
                case ServerTOPIC[0]: //statff
                    onStaffSign(data);
                    break;
                case ServerTOPIC[1]://vip
                    onVipSign(data);
                    break;
                default:
                    console.log("未知主题消息...")
                    break;
            }
        }
    }


    function onStaffSign(signDataList) {
        if (!rootContainer) {
            rootContainer = document.getElementById("divRoot");
        }
        if (rootContainer.childElementCount > 3) {
            rootContainer.innerHTML = "";
        }
        for (let i = 0; i < signDataList.length; i++) {
            if (rootContainer.childElementCount > 3) {
                rootContainer.innerHTML = "";
            }
            let signData = signDataList[i];
            _this.user = Object.assign(_this.user, signData.person.person_information);
            //_this.user.signTime = signData.person.upload_time.format("yyyy-MM-dd HH:mm:ss");
            _this.user.photo = `http://api.vaiwan.com:8081/image/${signData.person.face_list[0].face_image_id}`;
            _this.userList.push(_this.user);
            rootContainer.innerHTML += `<div id="divuser${i}" class="col-center-block text-center"
                     style="background: darkgrey;border:black solid 1px; width: 300px;height: 400px; float: right;margin: 20px; left: 0px;" >
                    <img src="${_this.user.photo}" style="width: 150px;height: 150px;margin-top: 20px;"/>
                    <br/>
                    <br/>
                    <p class="text-center">${_this.user.name}</p>
                    <p class="text-center">${_this.user.company}</p>
                </div>`
            let uiPhoto = document.getElementById(`divuser${i}`);
            var id = window.setTimeout(() => {
                let currentX = uiPhoto.offsetLeft
                currentX += 300 * i;
                uiPhoto.style.left = `${currentX}px`;
                clearTimeout(id);
            }, 0);
            sleep(200);
        }
    }

    function onVipSign(signData) {
        if (!rootContainer) {
            rootContainer = $("#divRoot");
        }

    }

    import Vue from 'vue'

    var _this
    var currentInterval
    export default {
        name: "home",
        components: {},
        data() {
            _this = this;
            return {
                sendText: "Hello mqtt",
                userList: [],
                user: {
                    photo: require(`../assets/img/female.png`),
                    signTime: "",
                },
                title: HOME_SCREEN_TITLE,
                currentTime: "00:00",
                staffNum: 0,
                signInNum: 0
            }
        },
        methods: {
            getBgVideo() {
                return require('../assets/img/bg.png');
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
                if (!isActived) {
                    return;
                }
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

                var id = window.setTimeout(() => {
                    if (container != null) {
                        if (isActived) {
                            transform(container, `rotateY(${(1 * rotate * ++indexPiece)}deg)`); //改正负确定旋转方向
                            //console.log(`indexPiece: ${indexPiece}`);
                        }
                    }
                    clearTimeout(id);
                }, 500);

            }, 1000);//定时器
        },
        destroyed: function () {
            clearInterval(currentInterval);
        }
    }

</script>
<style>
    span {
        text-align: center;
    }
</style>
