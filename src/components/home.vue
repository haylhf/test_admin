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
			$('#img-slider').redraw();
//                    var index = $('#img-slider').roundabout('getChildInFocus');
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

window.onclick = ()=> {
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
var emptyCount = 0;
var showIndex = 0;//最左边开始算
var photoIndex = 0;//最左边是集合最后一个元素
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
		promise.then(()=> {
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
		sleep(500)
		let removeToIndex = 0;
		for (let i = 0; i < dataList.length; i++) {
			removeToIndex = i + 1;
			let data = dataList[i];
			//如果已经满了，填补原来为空的元素，并播动画
			if (emptyCount > 0) {
				for (let j = _this.userList.length - 1; j >= 0; j--) {// 从后面更新数据，因为动画是倒着播的
					if (isUndefined(_this.userList[j].name)) {
						photoIndex = j;
						data.index = j;
						_this.userList.splice(photoIndex, 1, Object.assign(data))
						if (showIndex != 0) {
							showIndex--;
							if (showIndex < 0) {
								showIndex = TotalItems - 1;
							}
							playAnimationToNext(showIndex);
						} else { //第一次不要动画
							showIndex--;
							if (showIndex < 0) {
								showIndex = TotalItems - 1;
							}
						}
						emptyCount--;
						break;
					}
				}
			}
			else {//6个元素幅值
				photoIndex--;
				if (photoIndex < 0) {
					photoIndex = TotalItems - 1;
				}
                data.index = photoIndex;
				_this.userList.splice(photoIndex, 1, Object.assign(data))
//				for (let i = 0; i < 2; i++) {
//					let index = photoIndex - 3;
//					if (index < 0) {
//						index = TotalItems - 1;
//					}
//					_this.userList.splice(_this.userList[index], 1, {})
//				}
				playAnimationToNext(showIndex);
				showIndex--;
				if (showIndex < 0) {
					showIndex = TotalItems - 1;
				}
			}
		}
		if (removeToIndex > 0) {
			dataList.splice(0, removeToIndex);
		}
		sleep(200)
	}

}

function playAnimationTry(index, count = 0) {
	setTimeout(()=> {
		if (count > 3) {
			return;
		}
		try {
			$('#img-slider').roundabout('animateToChild', index);
		} catch (e) {
			sleep(1000)
			playAnimationTry(index, ++count)
		}
	}, 500);
}

function playAnimationToNext(index) {
	let promise = new Promise(function (resolve, reject) {
		sleep(200)
		playAnimationTry(index);
		resolve();
	});

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
			signInNum: 0
		}
	},
	methods: {
		btnTest()
		{

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
        "name": "string2",
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
        "name": "string3",
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
			emptyCount++;
		}
		currentInterval = setInterval(function doAnimation() {

			_this.currentTime = new Date().format("MM月dd日 hh:mm");
//			    $.ajax({
//				    url: HOST + "user/getStaffNum",
//				    type: 'GET',
//				    dataType: 'json',
//				    success: function (data) {
//					    if (data.code == 200) {
//						    _this.staffNum = data.data;
//					    }
//				    },
//				    error: function (data) {
//
//				    }
//			    })
//			    $.ajax({
//				    url: HOST + "user/getStaffSignInNum",
//				    type: 'GET',
//				    dataType: 'json',
//				    success: function (data) {
//					    if (data.code == 200) {
//						    _this.signInNum = data.data;
//					    }
//				    },
//				    error: function (data) {
//
//				    }
//			    })

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
