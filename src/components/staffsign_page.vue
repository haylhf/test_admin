<template xmlns:v-on="http://www.w3.org/1999/xhtml" xmlns:v-bind="http://www.w3.org/1999/xhtml" >
    <div >
            <el-row style="margin-top:150px;" >
                <el-col :span="12" :offset="6" >
                    <div id="rootDiv" class="col-center-block text-center" >
                    </div >
                </el-col >

            </el-row >
            <el-row style="margin-top:150px;" >
                 <el-col :span="18" :offset="3" >
                    <div style="float: left; padding-left: 10px;" v-for="u in recentList" >
                            <div class="text-center">
                                <img v-if="u.photo!=null" :src="u.photo"
                                     style="width: 120px;height: 120px;
                                  margin-left: 5px;margin-right: 5px; border-radius: 50%;
                                  align-items: center;justify-content: center;" >
                            </div >
                    </div >
                 </el-col >
            </el-row >

    </div >

</template >

<script >
    $(document).ready(function () {
//        imgHeight = (window.innerHeight - 100) / 3;
//        imgWidth = imgHeight * 3 / 4;
	    root = document.getElementById("rootDiv");
	    console.log('onload')
    })
    var imgHeight = 400, imgWidth = 300;
    var root = null;
    const TotalItems = 3;

    function playDisappearAnimation(callback, obj) {
	    setTimeout(() => {
		    if (obj) {
			    obj.style.animation = `animtran 0.6s linear`
			    obj.style.webkitAnimation = `animtran 0.6s linear;`
			    obj.title = 'delete';
			    callback(obj);
			    console.log(`playDisappearAnimation`)
		    }
	    }, 0);
    }

    function deleteUser(callback, item) {
	    setTimeout(() => {
		    if (item != null) {
			    root.removeChild(item);
			    console.log(`delete old user`)
		    }
		    sleep(500)
		    callback();
	    }, 0);
    }

    function callAnimation(callback, data) {
	    if (root == null) {
		    root = document.getElementById("rootDiv");
	    }
	    var imgObj = document.createElement("div");
	    imgObj.className = "liDiv";

	    imgObj.style.height = imgHeight + 'px';
	    imgObj.style.width = imgWidth + 'px';

	    imgObj.innerHTML = `
                       <img src="${data.photo}"
                             style="width: 225px;height: 225px;border-radius: 50%;margin-top: 10px;" />
                       <div class="col-center-block text-center label" >
                            <div style="margin-top: 20px;font-size: 30px;" >
                                ${data.name}
                            </div >
                            <br/>
                            <span style="font-size: 30px;" >
                                ${data.signTime}
                            </span >
                       </div >`;

	    setTimeout(() => {
		    if (root != null && root.childElementCount >= TotalItems) {
			    //先动画变小，动画执行完再删除
			    var obj = root.childNodes[0];
			    playDisappearAnimation((item)=> {
				    deleteUser(()=> {
					    root.appendChild(imgObj);
				    }, item);
			    }, obj);
		    }
		    else {
			    root.appendChild(imgObj);
		    }
		    if (_this.recentList.length >= 10) {
			    _this.recentList.splice(0, 1);
		    }
		    _this.recentList.push(Object.assign(data));//下边列表
		    _this.dataList.splice(0, 1);
		    sleep(500)
		    callback();
	    }, 0);
    }

    function showUserAndPlay() {
	    while (_this.dataList.length > 0) {
		    let data = null;
		    for (let i = 0; i < 1; i++) {
			    data = _this.dataList[i];
			    break;
		    }
		    setTimeout(() => {
			    callAnimation(()=> {
				    showUserAndPlay();
			    }, data);
		    }, 300);
		    break;
	    }
    }

    import Vue from 'vue'
    var _this
    export default {
	    name: "StaffSignPage",
	    components: {},
	    data() {
		    _this = this;
		    return {
			    userList: [],
			    recentList: [],
			    dataList: [],
		    }
	    },
	    methods: {
		    updateData(list)
		    {
			    console.log(`staff updateData: ${list.length}`)

			    if (_this.dataList.length > 0) {
				    _this.dataList = _this.dataList.concat(list);
			    }
			    else {
				    _this.dataList = list;
				    setTimeout(() => {
					    showUserAndPlay();
				    }, 0);
			    }
		    },
	    },
	    computed: {},
	    filters: {},
	    created: function () {

	    },
	    mounted: function () {

	    },
	    destroyed: function () {

	    }
    }

</script >
<style >

     @-webkit-keyframes animtran {
	     from {
		     transform: scale(1.0, 1.0);
	     }
	     to {
		     transform: scale(0, 0);
	     }
     }

     span {
	     text-align: center;
     }

     .liDiv {
	     border: green solid 0.5px;

	     float: left;
	     margin-left: 10px;
     }
</style >
