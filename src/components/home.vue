<template xmlns:v-on="http://www.w3.org/1999/xhtml" xmlns:v-bind="http://www.w3.org/1999/xhtml" >
    <div class="homeDiv" >
         <div id="root" class="rootDiv" >
             <div id="container" class="container" >

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
		      //console.log(`arrayPic:${arrayPic.length}`);
		      for (let i of arrayPic) {
			      let img = require(`../assets/img/${i}.jpg`);
			      htmlPic = htmlPic + `<img id="img${i}" src="${img}" />`;
		      }
		      container.innerHTML = htmlPic;
		      var transZ = 30 * arrayPic.length;
		      arrayPic.forEach(function (i, j) {
//			      console.log(`${i} image,  rotation degree:${j * rotate}`);
			      transform($(`#img${i}`), `rotateY(${j * rotate}deg) translateZ(${transZ}px)`);
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
      export default {
	      name: "home",
	      components: {},
	      data () {
		      _this = this;
		      return {}
	      },
	      methods: {},
	      computed: {},
	      filters: {},
	      created: function () {
		      _this = this;

	      },
	      mounted: function () {
		      currentInterval = setInterval(function doAnimation() {
			      var id = window.setTimeout(()=> {
				      if (container != null) {
					      transform(container, `rotateY(${(1 * rotate * ++indexPiece)}deg)`); //改正负确定旋转方向
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

</style >
