<template>
    <view class="conbox">
        <u-modal :show="show" :content='content' @confirm="show=false"></u-modal>
        <u-popup :show="show2" @close="show2=false" mode="bottom" :round="10" :overlay="false">
            <div class="box">
				<div class="tip">作者:吕小布</div>
                <div class="bli" v-for='(e,i) in list' :key='i'>
                    <div class="label">
                        <u--input v-model="e.name" @change="save()"></u--input>
                    </div>
                    <div class="value">
                        <u-number-box @change="save()" color="#ffffff" bgColor="#2979ff" iconStyle="color: #fff" integer :min="1" v-model="e.value"></u-number-box>
                    </div>
                    <div class="del">
                        <u-button icon="trash" type="error" size="mini" shape="circle" text="" @click="list.splice(i,1)"></u-button>
                    </div>
                </div>
                <div class="bli">
                    <u-button icon="plus" size="small" color="#0ab99c" type="primary" shape="circle" text="" @click="list.push({name: '', value: 1})"></u-button>
                </div>
            </div>
        </u-popup>
        <view class="container" @touchmove="handletouchmove" @touchstart="handletouchstart" @touchend="handletouchend">
            <!-- 背景 -->
            <image src="../../static/images/bg.gif" class="cont" mode=""></image>
            <view class="main" style="padding-top: 30vh">
                <view class="canvas-container">
                    <view :animation="animationData" class="canvas-content" id="zhuanpano" style="">
                        <view class="canvas-line">
                            <view class="canvas-litem" v-for="(item,index) in list" :key="index" :style="{transform:'rotate('+(index * width + width / 2)+'deg)'}"></view>
                        </view>
                        <view class="canvas-list">
                            <view class="canvas-item" :style="{transform: 'rotate('+(index * width)+'deg)', zIndex:index, }" v-for="(iteml,index) in list" :key="index">
                                <view class="canvas-item-text" :style="'transform:rotate('+(index )+')'">
                                    <text class="b" style="font-size: 32upx;">{{iteml.name}}</text>
                                </view>
                            </view>
                        </view>
                    </view>
                    <view @tap="playReward" class="canvas-btn" v-bind:class="btnDisabled">开始 </view>
                </view>
            </view>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            list: [
                { name: '上拉试试', value: 1 },
            ],
            width: 0,
            flag: 0,//1向左滑动,2向右滑动,3向上滑动 4向下滑动
            lastX: 0,
            lastY: 0,
            animationData: {},
            btnDisabled: '',
            show: false,
            show2: false,
            content: '',
        }
    },
    watch: {
        list(newVal, oldVal) {
            console.log(newVal, '监听')
            this.width = 360 / this.list.length
            this.save()

        },
    },
    onLoad: function () {
        // 获取奖品列表
        this.width = 360 / this.list.length
        try {
            if (uni.getStorageSync('list').length) this.list = uni.getStorageSync('list')

        } catch (error) {

        }
    },
    methods: {
        save() {
            try {
                uni.setStorageSync('list', this.list)
                console.log('已保存');

            } catch (error) {
            }
        },
        animation(index, duration) {
            //中奖index
            var list = this.list;
            var runNum = 4; //旋转8周

            // 旋转角度
            this.runDeg = this.runDeg || 0;
            this.runDeg = this.runDeg + (360 - this.runDeg % 360) + (360 * runNum - index * (360 / list.length)) + 1
            //创建动画
            var animationRun = uni.createAnimation({
                duration: duration,
                timingFunction: 'ease'
            })
            animationRun.rotate(this.runDeg).step();
            this.animationData = animationRun.export();
            this.btnDisabled = 'disabled';

        },
        //发起抽奖
        playReward() {
            Array.prototype.sample = function () {
                if (!this.length) return;
                return this[~~(Math.random() * this.length)];
            }
            function random(list) {
                let arr = []
                list.forEach((e, i) => {
                    for (let index = 0; index < e.value; index++) {
                        arr.push(e.name)
                    }
                })
                let name = arr.sample()
                return list.findIndex(item => {
                    return item.name == name
                })
            };
            let index = random(this.list), duration = 4000
            if (index == -1) return uni.$u.toast('出错了')
            this.animation(index, duration)
            setTimeout(() => {
                this.content = this.list[index].name
                this.show = true
                this.btnDisabled = '';
            }, duration + 500)

        },
        handletouchmove(event) {
            if (this.flag !== 0) {
                return;
            }
            let currentX = event.changedTouches[0].pageX;
            let currentY = event.changedTouches[0].pageY;
            let tx = currentX - this.lastX;
            let ty = currentY - this.lastY;

            //左右方向滑动
            if (Math.abs(tx) > Math.abs(ty)) {
                if (tx < 0) {
                    // 向左滑动
                    this.flag = 1;

                } else if (tx > 0) {
                    //向右滑动 
                    this.flag = 2;
                }
            }
            //上下方向滑动
            else {
                if (ty < 0) {
                    //向上滑动
                    this.show2 = true
                    this.flag = 3;
                } else if (ty > 0) {
                    //向下滑动
                    this.flag = 4;
                    this.show2 = false

                }
            }
            //将当前坐标进行保存以进行下一次计算
            this.lastX = currentX;
            this.lastY = currentY;
        },
        handletouchstart(event) {
            this.lastX = event.changedTouches[0].pageX;
            this.lastY = event.changedTouches[0].pageY;
        },
        handletouchend(event) {
            //停止滑动
            this.flag = 0;
        }
    }
}
</script>
<style lang="scss" scoped>
.icon-awrad {
    font-size: 50upx !important;
}

.conbox {
    width: 750upx;
    height: 100vh;
    overflow-x: hidden;
    overflow-y: scroll;
}

.container,
image.cont {
    width: 750upx;
    min-height: 100vh;
    height: auto;
    position: relative;
}

image.cont {
    height: 100%;
    position: absolute;
    z-index: 0;
}

image.caidai {
    position: absolute;
    top: 0;
    left: 0;
    width: 750upx;
    height: 1024upx;
}

.header-title > view {
    padding: 8upx 16upx;
    border: 1px solid #d89720;
    color: #d89720;
    font-size: 28upx;
    border-radius: 26upx;
}

/* 转盘 */
.canvas-container {
    margin: 0 auto;
    position: relative;
    width: 600upx;
    height: 600upx;
    background: url(./img/circle.png) no-repeat;
    background-size: cover;
    border-radius: 50%;
}

.canvas-content {
    position: absolute;
    left: 0;
    top: 0;
    z-index: 1;
    display: block;
    width: 600upx;
    height: 600upx;
    border-radius: inherit;
    /* background-clip: padding-box; */
    /* background-color: #ffcb3f; */
}

.canvas-list {
    position: absolute;
    left: 0;
    top: 0;
    width: inherit;
    height: inherit;
    z-index: 9999;
}

.canvas-item {
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    color: #e4370e;
    /* text-shadow: 0 1upx 1upx rgba(255, 255, 255, 0.6); */
}

.canvas-item-text {
    position: relative;
    display: block;
    padding-top: 46upx;
    margin: 0 auto;
    text-align: center;
    -webkit-transform-origin: 50% 300upx;
    transform-origin: 50% 300upx;
    display: flex;
    flex-direction: column;
    align-items: center;
    color: #fb778b;
}

.canvas-item-text text {
    font-size: 30upx;
}

/* 分隔线 */
.canvas-line {
    position: absolute;
    left: 0;
    top: 0;
    width: inherit;
    height: inherit;
    z-index: 99;
}

.canvas-litem {
    position: absolute;
    left: 300upx;
    top: 0;
    width: 3upx;
    height: 300upx;
    background-color: rgba(228, 55, 14, 0.4);
    overflow: hidden;
    -webkit-transform-origin: 50% 300upx;
    transform-origin: 50% 300upx;
}

/**
* 抽奖按钮
*/
.canvas-btn {
    position: absolute;
    left: 260upx;
    top: 260upx;
    z-index: 400;
    width: 80upx;
    height: 80upx;
    border-radius: 50%;
    color: #f4e9cc;
    background-color: #e44025;
    line-height: 80upx;
    text-align: center;
    font-size: 26upx;
    text-shadow: 0 -1px 1px rgba(0, 0, 0, 0.6);
    box-shadow: 0 3px 5px rgba(0, 0, 0, 0.6);
    text-decoration: none;
}

.canvas-btn::after {
    position: absolute;
    display: block;
    content: ' ';
    left: 12upx;
    top: -44upx;
    width: 0;
    height: 0;
    overflow: hidden;
    border-width: 30upx;
    border-style: solid;
    border-color: transparent;
    border-bottom-color: #e44025;
}
.canvas-btn.disabled {
    pointer-events: none;
    background: #b07a7b;
    color: #ccc;
}

.canvas-btn.disabled::after {
    border-bottom-color: #b07a7b;
}

.typecheckbox view {
    border: 1px solid #ff3637;
    background: transparent;
    color: #ff3637;
    display: flex;
    height: 60upx;
    width: 140upx;
    border-radius: 50upx;
    align-items: center;
    justify-content: center;
    display: flex;
    margin-left: 20upx;
}

.guize {
    width: 502upx;
    min-height: 300upx;
    display: flex;
    flex-direction: column;
    position: relative;
    z-index: 3;
    background-image: linear-gradient(-180deg, #f48549 0%, #f2642e 100%);
    border: 18upx solid #e4431a;
    border-radius: 16px;
    margin: 0 auto;
    margin-top: -104upx;
    padding: 48upx;
    /* box-sizing: border-box; */
    color: #fff;
}

.guize .title {
    text-align: center;
    margin-bottom: 28upx;
}

.guize .g_item {
    font-family: PingFang-SC-Medium;
    font-size: 24upx;
    color: #ffffff;
    letter-spacing: 0.5px;
    text-align: justify;
    line-height: 20px;
}

.myrewards .title {
    font-family: PingFang-SC-Bold;
    font-size: 16px;
    color: #e4431a;
    letter-spacing: 0.57px;
    display: flex;
    padding-top: 36upx;
    justify-content: center;
}
.box {
    padding: 30rpx;
    .bli {
        margin-bottom: 10rpx;
        display: flex;
        align-items: center;
        .value {
            margin: 0 5rpx;
        }
    }
}
.tip{
	font-size: 12px;
	margin-bottom: 5px;
}
</style>
