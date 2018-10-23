<template xmlns:v-on="http://www.w3.org/1999/xhtml" xmlns:v-bind="http://www.w3.org/1999/xhtml" >
    <div>
        <el-row>
            <el-col :span="6" style="margin-top:250px">
                <div style=" color: white;font-size: x-large;margin-left: 86px">
                    贵宾到访记录
                </div>
                <img src="../assets/img/line_start.png" style="margin-top: -10px"/>
                <div style="margin-top: 20px;margin-bottom: 20px;margin-left: 18px">
                    <div v-for=" item in recentVipList">
                        <el-row>
                            <el-col :span="3">
                                <div :style="getSmallPhoneBg()">
                                    <img  v-if="item.photo!=null" :src="item.photo"
                                          style="width: 80px;height: 80px;border-radius: 50%;align-items: center;justify-content: center;margin-top: 5px;margin-left: 5px" >
                                </div>
                            </el-col>
                            <el-col :span="21" style="margin-top: 20px">
                                <div>
                                    <img src="../assets/img/name_bar.png" style="margin-left: -2px">
                                    <div style="color: white;font-size: 24px; margin-left: 50px;margin-top: -40px">{{item.name}}</div>
                                </div>
                            </el-col>
                        </el-row>
                    </div>
                </div>
                <img src="../assets/img/line_end.png"/>
            </el-col>
            <el-col :span="12">
                <el-row style="text-align: center">
                    <el-col :span="8" style="text-align: center;margin-top: 260px" >
                        <div :style="getShowPhoneBg()">
                            <img  v-if="showVipList.length >1 && showVipList[1].photo!=null" :src="showVipList[1].photo"
                                  style="width: 255px;height: 255px;border-radius: 50%;align-items: center;justify-content: center;margin-top: 72px;" >
                        </div>
                        <div v-if="showVipList.length >1 && showVipList[1].name!=null"  style="color: white;font-size: 64px;margin-top: -10px">{{showVipList[1].name}}</div>
                        <img :src="vipPic" style="margin-top: 20px; width: 100%"/>
                        <img :src="vipPicShadow" style="margin-top: 10px; width: 90%"/>
                    </el-col>
                    <el-col :span="8" style="text-align: center;margin-top: 150px" >
                        <div :style="getShowPhoneBg()">
                            <img  v-if="showVipList.length >0 && showVipList[0].photo!=null" :src="showVipList[0].photo"
                                  style="width:255px;height: 255px;border-radius: 50%;align-items: center;justify-content: center;margin-top: 72px" >
                        </div>
                        <div v-if="showVipList.length >0 && showVipList[0].name!=null" style="color: white;font-size: 64px;margin-top: -10px">{{showVipList[0].name}}</div>
                        <img :src="vipPic" style="margin-top: 20px; width: 100%"/>
                        <img :src="vipPicShadow" style="margin-top: 30px; width: 90%"/>
                    </el-col>
                    <el-col :span="8" style="text-align: center;margin-top: 260px">
                        <div :style="getShowPhoneBg()">
                            <img  v-if="showVipList.length >2 && showVipList[2].photo!=null" :src="showVipList[2].photo"
                                  style="width: 255px;height: 255px;border-radius: 50%;align-items: center;justify-content: center;margin-top: 72px;" >
                        </div>
                        <div v-if="showVipList.length >2 && showVipList[2].name!=null" style="color: white;font-size: 64px;margin-top: -10px">{{showVipList[2].name}}</div>
                        <img :src="vipPic" style="margin-top: 20px; width: 100%"/>
                        <img :src="vipPicShadow" style="margin-top: 10px; width: 90%"/>
                    </el-col>
                </el-row>
            </el-col>
            <el-col :span="5" style="margin-top:250px">
                <div style=" color: white;font-size: x-large;margin-left: 86px">

                </div>
            </el-col>
        </el-row>

    </div>

</template >

<script >
    var _this
    export default {
        name: "VipPage",
        components: {},
        data() {
            _this = this;
            return {
                showVipList: [],
                recentVipList: [],
                vipPic:require('../assets/img/text_VIP.png'),
                vipPicShadow:require('../assets/img/VIP_shadow.png'),
            }
        },
        methods: {
            updateData(dataList) {
                for(let i=0; i< dataList.length; i++) {
                    if (_this.recentVipList.length >= 5) {
                        _this.recentVipList.splice(0, 1);
                    }
                    _this.recentVipList.push(Object.assign(dataList[i]));
                }
                _this.showVipList.splice(0,_this.showVipList.length);
                for(let j= _this.recentVipList.length-1; j>=0 && _this.showVipList.length < 3; j--) {
                    _this.showVipList.push(_this.recentVipList[j]);
                }
            },
            getSmallPhoneBg() {
                return "background-image: " + "url(" + require('../assets/img/small_portrait_back.png') + "); background-repeat:no-repeat; background-size:100% 100%;-moz-background-size:100% 100%;height:90px;width:90px";
            },
            getShowPhoneBg() {
                return "background-image: " + "url(" + require('../assets/img/portrait_ring.png') + "); background-repeat:no-repeat; background-size:100% 100%;-moz-background-size:100% 100%;width:400px;height:400px";
            }
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
    span {
        text-align: center;
    }
</style >
