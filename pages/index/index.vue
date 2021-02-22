<template>
	<view class="content">
		<canvas style="width: 750rpx; height: 500rpx;" canvas-id="firstCanvas" id="firstCanvas"
		@touchmove="move" @touchstart="start" @touchend="end"></canvas>
		<view class="tools">
			<view class="item">
				<view class="title">操作</view>
				<view class="btn">
					<button type="warn" size="mini" @click="clear">清除</button>
					<button type="default" size="mini" @click="eraser">橡皮擦</button>
					<button type="default" size="mini" @click="saveImage()">保存</button>
					<button type="default" size="mini" @click="backOperate()">撤销</button>
					<button type="default" size="mini" @click="recoverBackOperate()">恢复</button>
				</view>
			</view>
			
			<view class="item">
				<view class="title">色彩</view>
				<view class="btn">
					<button type="default" style="background-color: red;" size="mini" @click="changeStrokeStyle('red')">红色</button>
					<button type="default" style="background-color: green;" size="mini" @click="changeStrokeStyle('green')">绿色</button>
					<button type="default" style="background-color: yellow;" size="mini" @click="changeStrokeStyle('yellow')">黄色</button>
					<input type="text" :value="strokeStyle" @input="setColor" class="color-input"/>
				</view>
			</view>
			
			<view class="item">
				<view class="title">粗细</view>
				<view class="btn">
					<button type="default" size="mini" @click="incLineWidth()">增大</button>
					<button type="default" size="mini" @click="decLineWidth()">减小</button>
					<input type="number" :value="lineWidth" @input="setLineWidth"/>
				</view>
			</view>
			
			<view class="item">
				<view class="title">形状</view>
				<view class="btn">
					<button type="default" size="mini" @click="changeDrawType('rect')">矩形</button>
					<button type="default" size="mini" @click="changeDrawType('arc')">圆形</button>
					<button type="default" size="mini" @click="changeDrawType('point')">无规律</button>
				</view>
			</view>
			
			<view class="item">
				<view class="title">
					图片展示
				</view>
				<view class="">
					<image :src="imgUrl" mode=""></image>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				title: 'Hello',
				canvasH:0,
				canvasW:0,
				lineWidth: 1,
				strokeStyle: "#000000",
				context: '',
				drawType: 'point',
				points:[],
				rectX:0,
				rectY:0,
				rectW:0,
				rectH:0,
				arcX:0,
				arcY:0,
				arcR:0,
				arcF:'',
				operateArr:[],
				backOperateArr:[],
				backOperateStep:0,
				imgUrl:'',
				tmpCanvas:[],
			}
		},
		onLoad() {
			uni.stopPullDownRefresh();
		},
		onReady() {
			var context = uni.createCanvasContext('firstCanvas');
			context.setStrokeStyle(this.strokeStyle)
			context.setLineWidth(this.lineWidth);
			context.setLineCap("round");
			context.setLineJoin("round");
			
			this.context = context;
			var that = this;
			uni.createSelectorQuery().select('#firstCanvas').boundingClientRect((res)=>{
				that.canvasH = res.height;
				that.canvasW = res.width;
			}).exec();
		},
		methods: {
			//开始
			start(e){
				this.saveOperate();
				this.points.push({X:e.changedTouches[0].x,Y:e.changedTouches[0].y});
				if(this.drawType === 'rect') {
					this.rectX = e.changedTouches[0].x;
					this.rectY = e.changedTouches[0].y;
				}else if(this.drawType === 'arc') {
					this.arcX = e.changedTouches[0].x;
					this.arcY = e.changedTouches[0].y;
				}
				this.context.beginPath();
			},
			//移动
			move(e){
				// console.log(e.changedTouches[0].x);
				// console.log(e.changedTouches[0].y);
				this.points.push({X:e.changedTouches[0].x,Y:e.changedTouches[0].y});
				let len = this.points.length;
				if(len >= 2){
					if(this.drawType === 'point'){
						this.draw();
					}else if(this.drawType === 'rect') {
						this.rect();
					}else if(this.drawType === 'arc') {
						this.arc();
					}
				}
			},
			//结束
			end(e){
				this.points = [];
				if(this.drawType === 'rect') {
					this.context.strokeRect(this.rectX,this.rectY,this.rectW,this.rectH);
					this.context.draw(true);
					this.rectX = 0;
					this.rectY = 0;
					this.rectW = 0;
					this.rectH = 0;
				}else if(this.drawType === 'arc') {
					console.log('info:',this.arcX+'|'+this.arcY+'|'+this.arcR)
					let x = this.arcX;
					// if (this.arcF === 'l') {
					// 	x = this.arcX - this.arcR;
					// }else if(this.arcF === 'r') {
					// 	x = this.arcX + this.arcR;
					// }
					this.context.arc(x,this.arcY,this.arcR,0,2*Math.PI);
					this.context.stroke();
					this.context.draw(true);
					this.arcX,this.arcY,this.arcR  = 0;
				}
			},
			//画画方法
			draw(){
				let point1 = this.points[0];
				let point2 = this.points[1];
				this.points.shift();
				this.context.moveTo(point1.X,point1.Y);
				this.context.lineTo(point2.X,point2.Y);
				this.context.stroke()
				this.context.draw(true);
			},
			// 绘制矩形
			rect(){
				// let clearPoint = this.points[this.points.length-2];
				
				let point = this.points[this.points.length-1];

				this.rectW = point.X-this.rectX;
				this.rectH = point.Y-this.rectY;
				// this.context.strokeRect(this.rectX,this.rectY,this.rectW,this.rectH);
				// this.context.draw();

				// console.log('p2',point2.X+'-'+point2.Y);
				// this.context.strokeRect(this.rectX,this.rectY,this.rectW,this.rectH);
				// this.context.fill();
				// console.log('text',this.rectX+'|'+this.rectY+'|'+this.rectW+'|'+this.rectH)
				// this.context.draw(true);
			},
			// 绘制圆
			arc(){
				let point = this.points[this.points.length-1];
				if(point.X > this.arcX) {
					this.arcF = 'r';
				}else{
					this.arcF = 'l';
				}
				this.arcR = Math.sqrt(Math.pow(point.X - this.arcX,2)+Math.pow(point.Y-this.arcY,2));
				
			},
			//清空画布
			clear:function(){
				let that = this;
				uni.getSystemInfo({
					success: function(res) {
						let canvasw = res.windowWidth;
						let canvash = res.windowHeight;
						that.context.clearRect(0, 0, canvasw, canvash);
						that.context.draw(true);
					},
				})
			},
			//橡皮擦
			eraser:function(){
				this.changeStrokeStyle("#ffffff");
			},
			//改变颜色
			changeStrokeStyle(color){
				this.strokeStyle = color;
				this.context.setStrokeStyle(this.strokeStyle);
			},
			setColor:function(event){
				this.changeStrokeStyle(event.detail.value);
			},
			incLineWidth(){
				this.lineWidth +=1;
				this.changeLineWidth(this.lineWidth);
			},
			decLineWidth(){
				this.lineWidth -=1;
				this.changeLineWidth(this.lineWidth);
			},
			changeLineWidth(lineWidth){
				this.context.setLineWidth(lineWidth);
			},
			setLineWidth:function(event){
				this.lineWidth = event.detail.value;
				this.changeLineWidth(this.lineWidth);
			},
			//改变绘画形状
			changeDrawType:function(type){
				this.drawType = type;
			},
			//保存当前的操作
			saveOperate:function(){
				var that = this;
				uni.canvasGetImageData({
					canvasId:'firstCanvas',
					x: 0,
					y: 0,
					width: that.canvasW,
					height: that.canvasH,
					success(res) {
						that.operateArr.push(res.data);
					}
				})
			},
			//保存撤销操作
			saveBackOperate: function(){
				var that = this;
				
				uni.canvasGetImageData({
					canvasId: 'firstCanvas',
					x:0,
					y:0,
					width: that.canvasW,
					height: that.canvasH,
					success(res) {
						that.backOperateArr.push(res.data);
					}
				})
			},
			//返回上一步操作
			backOperate:function(){
				var that = this;
				if(that.operateArr.length < 1){
					uni.showToast({
						title:"无操作可撤销",
						icon:"none"
					})
					return;
				}
				this.saveBackOperate();
				var data = that.operateArr.pop();
				this.clear();
				uni.canvasPutImageData({
					canvasId: 'firstCanvas',
					x: 0,
					y: 0,
					width: that.canvasW,
					height: that.canvasH,
					data: data,
					success(res) {
						console.log(res);
					},
					fail(res){
						console.log(res);
					}
				})
			},
			//恢复撤销
			recoverBackOperate:function(){
				var that = this;
				
				if(that.backOperateArr.length < 1){
					uni.showToast({
						title:"无撤销可恢复",
						icon:"none"
					})
					return;
				}
				var data = that.backOperateArr.pop();
				this.saveOperate()
				this.clear()
				uni.canvasPutImageData({
					canvasId: 'firstCanvas',
					x: 0,
					y: 0,
					width: that.canvasW,
					height: that.canvasH,
					data: data,
					success(res) {
						console.log(res);
					},
					fail(res){
						console.log(res);
					}
				})
			},
			//保存为图片
			saveImage:function(){
				var that = this;
				uni.canvasToTempFilePath({
				  canvasId: 'firstCanvas',
				  success: function(res) {
				    // 在H5平台下，tempFilePath 为 base64
					that.imgUrl = res.tempFilePath;
				  } 
				})
			}
		}
	}
</script>

<style lang="scss">
	page{
		width: 100%;
		height: 100%;
	}
	.tools{
		// margin: 20rpx 0 0 20rpx;
		// border: 1px solid red;
		// display:flex;
		// flex-flow: row inwrap;
		// justify-content: flex-start;
		
		.item{
			// position: relative;
			// display: flex;
			// justify-content: flex-start;
			// border: 1px solid gray;
			.btn{
				// border: 1px solid green;
				// position: relative;
				display: flex;
				justify-content: flex-start;
		
			}
			button{
				margin: 5rpx 5rpx;
			}
			input{
				margin-top: 5rpx;
				padding: 5rpx 0rpx;
				border: 1px solid #dadada;
				width: 100rpx;
			}
			.color-input{
				width: 150rpx;
			}
		}
	}
	#firstCanvas{
		border-bottom: 1px solid #333333;
	}
</style>
