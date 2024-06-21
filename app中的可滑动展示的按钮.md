Layout/index.vue

这个组件的作用是作为一个架子，需要展示悬浮按钮的页面，将整个页面都放在这个组件内，原理就是插槽。

```vue

<template>
  <view class="layout-view">
      
		<Floating  style="position: relative; z-index: 10"
			@click="SET_SHOW(true)" />
<!-- 这个插槽是装app中的你需要展示悬浮按钮的页面,这里很巧妙 -->
    <slot></slot>			
  </view>
</template>

<script>
import {
	mapMutations,
} from 'vuex';
export default {
    data() {
        return {}
    },
    computed: {
        problemShow() {
            return this.$store.state.problemShow
        },
        hasPermi() {
            return true
        }
    },
    methods: {
        ...mapMutations(['SET_SHOW']), 
    },
};
</script>

<style scoped lang="scss">
.layout-view {
  width: 100%;
  height: 100%;
}
</style>

```

Floating/index.vue

```vue
<template>
	<view>
		<!-- 点击屏幕任意地方隐藏 -->
		<view class="all" v-if="isC" @click="isC = false"></view>
		<!-- 单个未点击的球 -->
		<view class="qiu" :style="'transform: translate(' + x + 'px,' + y + 'px);'" :class="{ chu: isC }"
			@click="clickCros" @touchstart="touchS" @touchmove.stop.prevent="touchM" @touchend="touchE">
			<image :class="{ zhuan: isC }" src="../../static/plus1.png" mode=""></image>
		</view>



		<view v-if="checkPermissionMove('H5:problem')" @click="toProblem" class="model modelA"
			:style="three?'transform: translate(' +(x-2) + 'px,' + (y-3) + 'px);':'transform: translate(' +(x+10) + 'px,' + (y -30) + 'px);'"
			:class="{ model1: isC }">

			<view class="yuan">
				<image class="pai" mode="widthFix" src="/static/images/pai1.png">
				</image>
				<text class="textPai">安全随手拍</text>
			</view>
		</view>

		<view v-if="checkPermissionMove('convenientTake')" @click="toPai" class="model modelB"
			:style="three?'transform: translate(' +(x+24) + 'px,' + (y+44) + 'px);':'transform: translate(' + (x+10) + 'px,' + (y + 26) + 'px);'"
			:class="{ model1: isC }">


			<view class="yuan">
				<image class="pai" mode="widthFix" src="/static/images/pai2.png">
				</image>
				<text class="textPai">质量随手拍</text>
			</view>

		</view>

		<view v-if="three" @click="clickThree" :style="'transform: translate(' + (x+25) + 'px,' + (y -47) + 'px);'"
			class="model modelC" :class="{ model1: isC }">


			<view class="yuan">
				<image class="pai" mode="widthFix" src="/static/images/pai2.png">
				</image>
				<text class="textPai">问题反馈</text>
			</view>

		</view>

	</view>
</template>

<script>
	import {mapMutations} from 'vuex';
	import problemMixin from "@/mixins/problemMixin.js";
	import store from "@/store";
	import md5 from "@/common/lib/md5.min.js";
	import {
		randomNum
	} from "@/utils/index.js";
	import globalConfig from "@/config";
	export default {
		mixins: [problemMixin],
		data() {
			return {
				isC: false,
				start: {
					left: 0,
					top: 0
				},
				x: 0,
				y: 0,
				oldx: 0,
				oldy: 0,

			}
		},
		props: {
			// 是否有问题反馈按钮
			three: {
				default: false,
				type: Boolean,
			}
		},
		watch: {
			'$route.path': {
				handler(val) {
					this.isC = false
				}
			},
            //这里的作用是
			'getBtnY':{
				immediate:true,
				handler(val){
					this.y = this.getBtnY
					this.oldy=this.getBtnY
				}
			}

		},
		mounted(){
			
		},
		computed:{
            //获取到悬浮按钮在前一个页面在y轴距离原始位置移动的距离
			getBtnY(){
				return this.$store.state.btnY
			}
			
		},
		
		methods: {
...mapMutations(['setBtnPosition']),
			clickCros() {

				this.isC = !this.isC

			},
			// 点击第一个小球
			toProblem() {
				this.isC = false;
				uni.navigateTo({
					url: '/pages/Snapshot/index?type=1',
					animationType: 'pop-in',
					animationDuration: 200
				});
			},
			// 点击第二个小球
			toPai() {
				this.isC = false
				uni.navigateTo({
					url: '/pages/Snapshot/index?type=0',
					animationType: 'pop-in',
					animationDuration: 200
				});

			},
			// 点击第三个小球
			clickThree() {
				this.isC = false
			},


			touchS(val) {
				this.isMove = true
				this.start.left = val.changedTouches[0].clientX
				this.start.top = val.changedTouches[0].clientY
			},
			touchM(val) {
				if (this.isMove == false) {
				  return;
				}
			
				   // 安全区域的高度
					let	height= uni.getSystemInfoSync().windowHeight

				if (val.changedTouches[0].clientY > height - 60) {
					val.changedTouches[0].clientY = height - 60
					this.y = this.oldy + val.changedTouches[0].clientY - this.start.top
				}
				//上面
				if (val.changedTouches[0].clientY < 60) {
					val.changedTouches[0].clientY = 60
					this.y = this.oldy + val.changedTouches[0].clientY - this.start.top
				}
				this.x = this.oldx + val.changedTouches[0].clientX - this.start.left
				this.y = this.oldy + val.changedTouches[0].clientY - this.start.top
				console.log('val.changedTouches[0].clientY',val.changedTouches[0].clientY);
				console.log('this.start.top',this.start.top);
			},
			touchE(val) {
				
				this.x = 0
				this.oldx = this.x
				this.oldy = this.y
				this.$store.commit('setBtnPosition',this.y)
				this.isMove = false
			},
		}
	};
</script>

<style scoped lang="scss">
	.all {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		z-index: 100;
	}

	.qiu {
		// z-index: 99999;
		position: fixed;
		top: 45vh;
		right: 0;
		width: 38px;
		height: 38px;
		border-radius: 19px;
		background: #4285F4;
		transition-property: right, top, height, width;
		transition-duration: 0.5s;
		display: flex;
		justify-content: center;
		align-items: center;

		image {
			width: 34px;
			height: 34px;
			transition: all 0.4s;
		}
	}



	.model {
		position: fixed;
		z-index: 101;
		opacity: 0;
		transition: all 0.5s;
		border-radius: 10px;
		display: flex;
		justify-content: space-around;

		color: #fff;
		text-align: center;
		font-size: 14px;
		display: flex;
		flex-direction: column;

		.yuan {
			display: flex;
			flex-direction: column;
			justify-content: center;
			align-items: center;
			width: 44px;
			height: 44px;
			font-weight: 800;
			font-size: 16px;
			line-height: 36px;
			text-align: center;
			background: #4285F4;
			border-radius: 22px;
			margin-bottom: 4px;

			.pai {
				width: 12px;
				height: 14px;

			}

			.textPai {
				width: 27px;
				height: 20px;
				font-family: Source Han Sans CN, Source Han Sans CN;
				font-weight: 400;
				font-size: 9px;
				color: #FFFFFF;
				line-height: 10px;
				text-align: center;
				font-style: normal;
				text-transform: none;
			}
		}
	}

	.modelA {
		// calc(72% - 5px)
		// left: 73%;
		right: 47px;
		top: 45vh;
	}

	.modelB {
		// left: 73%;
		right: 47px;
		top: 45vh;
	}

	.modelC {
		// left: 73%;
		right: 47px;
		top: 45vh;
	}

	.model1 {
		opacity: 1;
	}
</style>
```

