<template xmlns:v-on="http://www.w3.org/1999/xhtml" xmlns:v-bind="http://www.w3.org/1999/xhtml" >
    <div >

        <!--<video :src="getBgVideo()"-->
        <!--style="right:0; bottom: 0; z-index: -100;min-width: 100%;min-height: 100%;position: fixed;height: auto;width: auto;-webkit-filter: grayscale(20%)"-->
        <!--loop="loop" autoplay="autoplay" muted ></img >-->
        <img :src="getBgVideo()"
             style="min-width: 100%;min-height: 100%;position: fixed;z-index: -100;right:0; bottom: 0;" />
        <div class="homeDiv" >
            <el-row style="margin-top: 30px" >
                <el-col :offset="5" :span="3" style="text-align: center" >
                    <span style="font-size: 30px;color: white;font-weight: lighter" >{{currentTime}}</span >
                </el-col >
                <el-col :offset="2" :span="4" style="text-align: center;margin-top: 30px" >
                    <span style="font-size: 40px;color: white;" >{{title}}</span >
                </el-col >
                <el-col :offset="2" :span="4" style="text-align: center" >
                    <span style="font-size: 28px;color: white;font-weight: lighter" @click="btnTest" >签到：</span >
                    <span style="font-size: 30px;color: white;font-weight: lighter" >{{getSignIn()}}</span >
                </el-col >
            </el-row >
        </div >

        <div id="tour" class="zebra" >
            <div class="wrap" >
                <div class="switcher-wrap slider" >
                    <a class="prev jQ_sliderPrev" href="" ></a >
                    <a class="next jQ_sliderNext" href="" ></a >
                    <ul id="img-slider" style="height: 800px" >
                        <li class="li_img text-center" v-for="u in userList" style="float: left" >
                            <img :src="u.photo"
                                 style="width: 225px;height: 225px;border-radius: 50%;align-items: center;justify-content: center;overflow: hidden;margin-top: 60px" />
                            <div class="col-center-block text-center label" >
                                <div style="min-height: 80px;margin-top: 50px" >
                                    {{u.name}}
                                </div >
                                <span style="font-size: 24px;" >
                                        {{u.signTime}}-{{u.index}}
                                    </span >
                            </div >
                        </li >
                    </ul >
                </div >
            </div >
        </div >
        <table style="width: 100%;height: 200px; position: fixed;bottom: 0;" >
            <tr >
                <td style="width: 5%;" ></td >
                <td style="text-align: center; vertical-align: middle" >
                    <div class="center-block text-center" v-for="u in recentList"
                         style="float: right; border: solid lightgoldenrodyellow 0.5px;margin-left: 5px;margin-right: 5px;" >
                        <img :src="u.photo"
                             style="width: 150px;height: 150px;
                             margin-left: 5px;margin-right: 5px; border-radius: 50%;
                             align-items: center;justify-content: center;" >
                    </div >
                </td >
                <td style="width: 5%;" ></td >
            </tr >
        </table >
    </div >

</template >

<script >
    $(document).ready(function () {
	    $('#img-slider li').bind({
		    reposition: function () {
			    // var degrees = $(this).data('roundabout').degrees,
			    //     roundaboutBearing = $(this).parent().data('roundabout').bearing,
			    //     rotateY = Math.sin((roundaboutBearing - degrees) * (Math.PI/180)) * 9;
			    //
			    // $(this).css({
			    //     "-webkit-transform": 'rotate(' + rotateY + 'deg)',
			    //     "-moz-transform": 'rotate(' + rotateY + 'deg)',
			    //     "-ms-transform": 'rotate(' + rotateY + 'deg)',
			    //     "-o-transform": 'rotate(' + rotateY + 'deg)',
			    //     "transform": 'rotate(' + rotateY + 'deg)'
			    // });
		    }
	    });

	    $('.jQ_sliderPrev').on('click', function () {
		    $('#img-slider').roundabout('animateToNextChild');

		    return false;
	    });

	    $('.jQ_sliderNext').on('click', function () {
		    $('#img-slider').roundabout('animateToPreviousChild');

		    return false;
	    });

	    $('.jQ_sliderSwitch li').on('click', function () {
		    var $elem = $(this);
		    var index = $elem.index();

		    $('#img-slider').roundabout('animateToChild', index);

		    return false;
	    });

	    $('#img-slider').roundabout({
		    minScale: 1.0,
		    maxScale: 1.0,
		    duration: 750
	    }).bind({
		    animationEnd: function (e) {
//			    updateIndex(); //动画执行完后，更新当前index等数据
//                var index = $('#img-slider').roundabout('getChildInFocus');
//                $('.jQ_sliderSwitch li').removeClass('active');
//                $('.jQ_sliderSwitch li').eq(index).addClass('active');
		    }
	    });

    });
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
	    requestFullScreen();
    };
    window.onload = function () {

	    client.connect(options);//连接服务器并注册连接成功处理事件
	    client.onConnectionLost = onConnectionLost;//注册连接断开处理事件
	    client.onMessageArrived = onMessageArrived;//注册消息接收处理事件
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
	    while (isLoading) {
		    sleep(1000); //wait;
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

    const TotalItems = 6;
    var isLoading = false;
    var photoIndex = 0 // 当前正中间的元素，执行动画后变到右边
    var rightIndex = 1; //当前最右边的index
    var emptyList = [0, 1, 2, 3, 4, 5];
    var currentShowList = [5, 0, 1];
    var isneedPlay = false;
    function onStaffSign(signDataList) {
	    isLoading = true;
	    var dataList = [];
	    try {
		    console.log(signDataList.length)
		    for (let i = 0; i < signDataList.length; i++) {

			    let signData = signDataList[i];
			    let data = Object.assign(signData.person.person_information);
			    try {
				    data.signTime = new Date(signData.timestamp * 1000).format("hh:mm:ss");
			    } catch (e) {
			    }
			    data.device_id = signData.device_id;
			    data.photo = require('../assets/img/male.png'); //`http://api.vaiwan.com:8081/image/${signData.person.face_list[0].face_image_id}`;
			    //data.photo = "http://192.168.0.119" + ":9812/image/" + signData.person.face_list[0].face_image_id; //`http://api.vaiwan.com:8081/image/${signData.person.face_list[0].face_image_id}`;
			    dataList.push(data);

		    }
		    let promise = new Promise(function (resolve, reject) {
			    showUserAndPlay(dataList);
			    resolve();
		    });
		    promise.then(() => {
			    isLoading = false;
		    });
	    } catch (e) {
		    console.log(e)
	    } finally {
		    isLoading = false;
	    }

    }

    function onVipSign(signDataList) {

    }


    function showUserAndPlay(dataList) {
	    while (dataList.length > 0) {
		    let removeToIndex = 0;
		    for (let i = 0; i < 1; i++) {
			    removeToIndex = i + 1;
			    let data = dataList[i];

			    let index = 0;
			    let isEmpty = false;
			    let nextIndex = 0;
			    for (let j = 0; j < currentShowList.length; j++) {
				    index = rightIndex - j;
				    if (index < 0) {
					    index += TotalItems;
				    }
				    for (let eItem of emptyList) {
					    if (eItem == index) {
						    isEmpty = true;
						    break
					    }
				    }
				    if (isEmpty) {
					    if (rightIndex == index) {
						    nextIndex = index - 1;
						    if (nextIndex < 0) {
							    nextIndex += TotalItems;
						    }
						    isneedPlay = true;
					    } else {
						    nextIndex = index;
						    isneedPlay = false;
					    }
					    break;
				    }
			    }
			    if (!isEmpty) {
				    nextIndex = photoIndex - 2;
				    if (nextIndex < 0) {
					    nextIndex += TotalItems;
				    }
				    isneedPlay = true;
			    }
			    if (_this.recentList.length >= 10) {
				    _this.recentList.splice(_this.recentList.length - 1, 1);
			    }
			    _this.recentList.push(Object.assign(data));

			    _this.userList.splice(nextIndex, 1, Object.assign(data))
			    emptyList.splice(emptyList.indexOf(nextIndex), 1)
			    if (isneedPlay) {
				    playAnimationToNext();
			    }

		    }
		    if (removeToIndex > 0) {
			    dataList.splice(0, removeToIndex);
		    }
		    sleep(200)
	    }
    }

    function playAnimationTry(count = 0) {
	    setTimeout(() => {
		    if (count > 3) {
			    return;
		    }
		    try {
			    $('#img-slider').roundabout('animateToPreviousChild');

		    } catch (e) {
			    sleep(1000)
			    playAnimationTry(++count)
		    }
	    }, 100);
    }

    function playAnimationToNext() {
	    let promise = new Promise(function (resolve, reject) {
		    playAnimationTry();
		    updateIndex(); //动画执行完后，更新当前index等数据
		    resolve();
	    });
    }

    function updateIndex() {
	    photoIndex--;
	    if (photoIndex < 0) {
		    photoIndex += TotalItems;
	    }
	    rightIndex = photoIndex + 1;
	    if (rightIndex > TotalItems - 1) {
		    rightIndex -= TotalItems;
	    }
	    let leftIndex = photoIndex - 1;
	    if (leftIndex < 0) {
		    leftIndex += TotalItems;
	    }
	    currentShowList = [leftIndex, photoIndex, rightIndex];

	    let index = rightIndex + 2;
	    if (index > TotalItems - 1) {
		    index -= TotalItems;
	    }
	    _this.userList.splice(index, 1, {})//把上次显示过的设置为空对象
	    let isHasExist = false;
	    for (let eitem of emptyList) {
		    if (eitem == index) {
			    isHasExist = true;
			    break;
		    }
	    }
	    if (!isHasExist) {
		    emptyList.push(index);
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
			    title: HOME_SCREEN_TITLE,
			    currentTime: "",
			    staffNum: 0,
			    signInNum: 0,
			    recentList: [],
		    }
	    },
	    methods: {
		    btnTest() {

			    onStaffSign(JSON.parse(`[
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
        "name": "string1",
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
]`))
		    },
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
		    //初始化加入默认元素
		    for (let i = 0; i < TotalItems; i++) {
			    _this.userList.unshift({});
		    }
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
	    }
    }

</script >
<style >
    span {
	    text-align: center;
    }
</style >
