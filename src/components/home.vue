<template xmlns:v-on="http://www.w3.org/1999/xhtml" xmlns:v-bind="http://www.w3.org/1999/xhtml" >
    <div >
        <video :src="getBgVideo()"
               style="right:0; bottom: 0; z-index: -100;min-width: 100%;min-height: 100%;position: fixed;height: auto;width: auto;-webkit-filter: grayscale(20%)"
               loop="loop" autoplay="autoplay" muted ></video >
        <div class="homeDiv" >
            <div id="root" class="rootDiv" >
                <div id="container" class="container" >
                </div ></div >
        </div >
        <div >
            <el-form >
                <el-col :span="24" >
                    <el-form-item label="内容：" label-width="100" style="width: 100%" >
                        <el-input v-model="sendText" type="textarea"
                                  :rows="5" clearable style="font-size: 18px" ></el-input >
                    </el-form-item >
                </el-col >
                <el-button type="primary" @click="onSend" size="small" >Send</el-button >
            </el-form >
        </div >
    </div >
</template >

<script >
    var container = null;
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
    window.ondeactivate = ()=> {
	    isActived = false;
    };
    window.onactivate = ()=> {
	    isActived = true;
    };
    window.onblur = ()=> {
	    isActived = false;
    };
    window.onfocus = ()=> {
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
	    var data = "";
	    try {
		    data = jQuery.parseJSON(message.payloadString);
            console.log("解析出来的：data：" + JSON.stringify(data));
        } catch (e) {
		    console.log(e);
	    }
	    switch (message.destinationName)
	    {
		    case ServerPort[0]: //statff
			    break;
		    case ServerPort[1]://vip
			    break;
            default:
                console.log("未知主题消息...")
                break;
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
		    }
	    },
	    methods: {
		    getBgVideo() {
			    return require('../assets/img/bg.mp4');
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
			    var id = window.setTimeout(() => {
				    if (container != null) {
					    if (isActived) {
						    transform(container, `rotateY(${(1 * rotate * ++indexPiece)}deg)`); //改正负确定旋转方向
						    //console.log(`indexPiece: ${indexPiece}`);
					    }
				    }
				    clearTimeout(id);
			    }, 500);

		    }, 1500);//定时器
	    },
	    destroyed: function () {
		    clearInterval(currentInterval);
	    }
    }

</script >
<style >
    span {
	    text-align: center;
    }
</style >
