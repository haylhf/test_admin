<template xmlns:v-on="http://www.w3.org/1999/xhtml" xmlns:v-bind="http://www.w3.org/1999/xhtml" >
    <div >
        <video :src="getBgVideo()"
               style="right:0; bottom: 0; z-index: -100;min-width: 100%;min-height: 100%;position: fixed;height: auto;width: auto;-webkit-filter: grayscale(20%)"
               loop="loop" autoplay="autoplay" muted ></video >
        <div class="homeDiv" >
            <div id="root" class="rootDiv" >
                <div id="container" class="container" >

                </div >
            </div >
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

	    _this = this;
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
		    console.log(JSON.stringify(container.innerHTML))
	    }
    };
    //      function doAnimationTimer () {
    //          var id = window.setTimeout(()=> {
    //              if (container != null) {
    //                  transform(container, `rotateY(${(1 * rotate * ++indexPiece)}deg)`); //改正负确定旋转方向
    //              }
    //              clearTimeout(id);
    //              doAnimationTimer();
    //          }, 1500);
    //      }


    import Vue from 'vue'

    var _this
    var currentInterval
    var bg_video = require('../assets/img/bg.mp4');
    export default {
	    name: "home",
	    components: {},
	    data() {
		    _this = this;
		    return {}
	    },
	    methods: {
		    getBgVideo() {
			    return bg_video;
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
			    var id = window.setTimeout(() => {
				    if (container != null) {
					    if (isActived) {
						    transform(container, `rotateY(${(1 * rotate * ++indexPiece)}deg)`); //改正负确定旋转方向
						    console.log(`indexPiece: ${indexPiece}`);
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
