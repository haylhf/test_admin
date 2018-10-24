<template xmlns:v-on="http://www.w3.org/1999/xhtml" xmlns:v-bind="http://www.w3.org/1999/xhtml" >
    <div >
        <div id="tour" class="zebra" >
            <div class="wrap" >
                <div class="switcher-wrap slider" >
                    <ul id="img-slider" style="height: 800px" >
                        <li class="li_img text-center" v-for="u in userList" style="float: left" >
                            <img :src="u.photo"
                                 style="width: 225px;height: 225px;border-radius: 50%;align-items: center;justify-content: center;overflow: hidden;margin-top: 60px" />
                            <div class="col-center-block text-center label" >
                                <div style="min-height: 80px;margin-top: 50px" >
                                    {{u.name}}
                                </div >
                                <span style="font-size: 24px;" >
                                    {{u.signTime}}
                                    </span >
                            </div >
                        </li >
                    </ul >
                </div >
            </div >
        </div >
        <table style="width: 100%;height: 180px; position: fixed;bottom: 0;" >
            <tr >
                <td style="width: 5%;" ></td >
                <td style="text-align: center; vertical-align: middle;" v-for="u in recentList" >
                    <div class="text-center" style="width: 120px;">
                        <img  v-if="u.photo!=null" :src="u.photo"
                              style="width: 120px;height: 120px;
                          margin-left: 5px;margin-right: 5px; border-radius: 50%;
                          align-items: center;justify-content: center;" >
                    </div>
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
    const TotalItems = 6;
    var photoIndex = 0 // 当前正中间的元素，执行动画后变到右边
    var rightIndex = 1; //当前最右边的index
    var emptyList = [0, 1, 2, 3, 4, 5];
    var currentShowList = [5, 0, 1];
    var isneedPlay = false;
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
				    _this.recentList.splice(0, 1);
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
		    sleep(800)
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
		        console.log("playAnimationTry: "+e);
			    sleep(1000)
			    playAnimationTry(++count)
		    }
	    }, 500);
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
    export default {
	    name: "StaffPage",
	    components: {},
	    data() {
		    _this = this;
		    return {
			    userList: [],
			    recentList: [],
		    }
	    },
	    methods: {
		    updateData(dataList)
		    {
			    console.log(`staff updateData: ${dataList.length}`)
			    showUserAndPlay(dataList);
		    },
	    },
	    computed: {},
	    filters: {},
	    created: function () {

	    },
	    mounted: function () {
		    //初始化加入默认元素
		    for (let i = 0; i < TotalItems; i++) {
			    _this.userList.unshift({});
			    _this.recentList.push({});
		    }
	    },
	    destroyed: function () {

	    }
    }

</script >
<style >
    span {
	    text-align: center;
    }
</style >
