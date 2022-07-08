<template>
    <view style="position: relative;" :style="canvasStyle">
        <canvas
            v-if="bgImage"
            id="defCanvas"
            canvas-id="defCanvas"
            :style="defCanvasStyle"
            :disable-scroll="true">
        </canvas>
        <canvas
            style="position: absolute;top: 0px;left: 0px;"
            :id="canvasId"
            :canvas-id="canvasId"
            :style="canvasStyle"
            :disable-scroll="true"
            @touchstart.stop="penStart"
            @touchmove.stop="penMove"
            @touchend.stop="penEnd">
        </canvas>
    </view>
</template>

<script>
    // 绘制点集合
    let drawFigures = []; 
    // 保存操作的像素数据
    let imageData = [];
    
    export default {
        name: 'yl-graffiti',
    	props: {
            // 画布ID
            canvasId: {
            	type: String,
            	default: 'myCanvas'
            },
            // 画布宽度
            width: {
            	type: Number,
            	default: 300
            },
            // 画布高度
            height: {
            	type: Number,
            	default: 225
            },
            // 图形 默认曲线 curve/line/rect/hollow-rect/circle/hollow-circle
            shape: {
            	type: String,
            	default: "curve"
            },
            // 画笔颜色
            lineColor: {
            	type: String,
            	default: '#091A22'
            },
            // 画笔宽度
            lineWidth: {
            	type: Number,
            	default: 5
            },
            // 画布背景颜色
            bgColor: {
            	type: String,
            	default: ""
            },
            // 画布背景图片
            bgImage: {
            	type: String,
            	default: ""
            }
    	},
        
        data() {
            return {
                myCanvasContext: null,
                canvasContext: null, // 画布上下文
                curStep: -1, // 当前步数
                // #ifdef MP-WEIXIN
                topLeftPoint: { // 画布左上角点
                    left: 0,
                    top: 0
                }, 
                // #endif
                startX: 0, // 前一个绘制节点X轴
                startY: 0, // 前一个绘制节点Y轴
                curBgImage: null,
                canDraw: false
            }
        },
        
        /**
         * 初始化画板
         */
        mounted() {
            // 初始化上下文
            this.canvasContext = uni.createCanvasContext(this.canvasId, this);
            
            // 让线条圆润
            this.canvasContext.setLineCap('round');
            this.canvasContext.strokeStyle = this.lineColor;
            this.canvasContext.setFillStyle(this.lineColor);
            this.canvasContext.setLineWidth(this.lineWidth);
            
            // 有背景图片初始化
            if (this.bgImage) {
                this.myCanvasContext = uni.createCanvasContext("defCanvas", this);
                
                // 绘制图片
                this.myCanvasContext.drawImage(this.bgImage, 0, 0, this.width, this.height);
                this.myCanvasContext.draw();
                
                // 让线条圆润
                this.myCanvasContext.setLineCap('round');
                this.myCanvasContext.strokeStyle = this.lineColor;
                this.myCanvasContext.setFillStyle(this.lineColor);
                this.myCanvasContext.setLineWidth(this.lineWidth);
            }
            
            // 设置背景色
            if (this.bgColor) {
                this.setBackgroundColor();
            }
            
            // #ifdef MP-WEIXIN
            this.getBoundingClientRect(`#${this.canvasId}`, this).then(res => {
                this.topLeftPoint = res;
            })
            // #endif
            
            drawFigures = [];
            imageData = [];
        },
        
        computed: {
            /**
             * 画布样式
             */
            canvasStyle() {
                if (this.bgImage) {
                    const bgImage = this.curBgImage || this.bgImage;
                    return `width:${this.width}px;height:${this.height}px;background-image:url(${bgImage});background-size:${this.width}px ${this.height}px;`;
                }
                return `width:${this.width}px;height:${this.height}px;`;
            },
            
            /**
             * 画布样式
             */
            defCanvasStyle() {
                return `width:${this.width}px;height:${this.height}px;`;
            }
        },
        
        watch: {
            /**
             * 设置线条颜色
             */
            lineColor() {
                this.canvasContext.strokeStyle = this.lineColor;
                this.canvasContext.setFillStyle(this.lineColor);
                if (this.bgImage) {
                    this.myCanvasContext.strokeStyle = this.lineColor;
                    this.myCanvasContext.setFillStyle(this.lineColor);
                }
            },
            
            /**
             * 设置线条颜色
             */
            lineWidth() {
                this.canvasContext.setLineWidth(this.lineWidth);
                if (this.bgImage) {
                    this.myCanvasContext.setLineWidth(this.lineWidth);
                }
            },
            
            /**
             * 当前位置
             */
            curStep() {
                this.$emit("stepChanged", {
                    curStep: this.curStep,
                    len: imageData.length
                })
            }
        },
        
        methods: {
            /**
             * 清空画板
             */
            clearBoard(clearData = true) {
                // 清空
                this.canvasContext.clearRect(0, 0, this.width, this.height);
                this.canvasContext.draw();
                
                // 初始化颜色和线宽
                this.canvasContext.strokeStyle = this.lineColor;
                this.canvasContext.setFillStyle(this.lineColor);
                this.canvasContext.setLineWidth(this.lineWidth);
                
                if (this.bgImage) {
                    // 清空
                    this.myCanvasContext.clearRect(0, 0, this.width, this.height);
                    this.myCanvasContext.draw();
                    
                    if (clearData) {
                        // 绘制图片
                        this.myCanvasContext.drawImage(this.bgImage, 0, 0, this.width, this.height);
                        this.myCanvasContext.draw();
                    }
                    
                    // 初始化颜色和线宽
                    this.myCanvasContext.strokeStyle = this.lineColor;
                    this.canvasContext.setFillStyle(this.lineColor);
                    this.myCanvasContext.setLineWidth(this.lineWidth);
                }
                
                if (clearData) {
                    // 重置数据
                    this.curStep = -1;
                    imageData = [];
                    if (this.bgImage) {
                        this.curBgImage = null;
                    }
                }
            },
            
            /**
             * 撤销操作
             */
            repeal() {
                if (this.curStep != -1) {
                    this.curStep = this.curStep - 1;
                    this.clearBoard(false);
                    if (this.curStep != -1) {
                        this.restoreImageData();
                    } else if (this.bgImage) {
                        this.curBgImage = null;
                    }
                }
            },
            
            /**
             * 重做操作
             */
            redo() {
                if (this.curStep < imageData.length - 1) {
                    this.curStep = this.curStep + 1;
                    this.clearBoard(false);
                    if (this.curStep < imageData.length) {
                        this.restoreImageData();
                    }
                }
            },
            
            /**
             * 设置画板背景颜色
             */
            setBackgroundColor() {
            	this.canvasContext.setFillStyle(this.bgColor);
            	this.canvasContext.fillRect(0, 0, this.width, this.height);
            	this.canvasContext.fill();
            	this.canvasContext.draw();
                
                if (this.bgImage) {
                    this.myCanvasContext.setFillStyle(this.bgColor);
                    this.myCanvasContext.fillRect(0, 0, this.width, this.height);
                    this.myCanvasContext.fill();
                    this.myCanvasContext.draw();
                }
            },
            
            /**
             * 保存画布内容
             */
            saveCanvas() {
                return new Promise((resolve, reject) => {
                    // 定时器线程中执行绘制，否则因绘制占用主线程导致加载动画不显示
                    setTimeout(_ => {
                        uni.canvasToTempFilePath({
                            canvasId: this.bgImage ? "defCanvas": this.canvasId,
                            quality: 1,
                            success: res => {
                                console.log("save path: ", res.tempFilePath)
                                resolve(res.tempFilePath);
                            },
                            fail: err => {
                                console.log('failed: ', err);
                                reject(err);
                            }
                        }, this);
                    }, 50);
                })
            },
            
            /**
             * 保存画布内容
             */
            saveImageData() {
                uni.canvasToTempFilePath({
                    canvasId: this.bgImage ? "defCanvas": this.canvasId,
                    quality: 1,
                    success: res => {
                        console.log("saveImageData path: ", res.tempFilePath)
                        if (this.curStep < imageData.length - 1) {
                            imageData = imageData.filter((item, index) => index <= this.curStep);
                        }
                        imageData.push(res.tempFilePath);
                        this.curStep += 1;
                    },
                    fail: err => {
                        console.log('saveImageData failed: ', err);
                    }
                }, this);
            },
            
            /**
             * 将保存的画布内容重新写入画布
             */
            restoreImageData(index = this.curStep) {
                if (this.bgImage) {
                    this.curBgImage = imageData[index];
                    this.myCanvasContext.drawImage(imageData[index], 0, 0, this.width, this.height);
                    this.myCanvasContext.draw();
                } else {
                    this.canvasContext.drawImage(imageData[index], 0, 0, this.width, this.height);
                    this.canvasContext.draw();
                }
            },
            
            /**
             * 绘制所有图形
             */
            draw() {
                drawFigures.forEach(figure => {
                    const {
                        shape,
                        lineColor,
                        lineWidth,
                        points
                    } = figure;
                    // 初始化颜色和线宽
                    this.canvasContext.strokeStyle = lineColor;
                    this.canvasContext.setFillStyle(lineColor);
                    this.canvasContext.setLineWidth(lineWidth);
                    
                    const startX = points[0].x;
                    const startY = points[0].y;
                    const endX = points[1] ? points[1].x: 0;
                    const endY = points[1] ? points[1].y: 0;
                    
                    switch (shape) {
                        case "curve":
                            this.drawCurveOnce(points, this.canvasContext);
                            break;
                        case "line":
                            this.drawLine(startX, startY, endX, endY, this.canvasContext);
                            break;
                        case "rect":
                            this.drawRect(startX, startY, endX, endY, this.canvasContext);
                            break;
                        case "hollow-rect":
                            this.drawHollowRect(startX, startY, endX, endY, this.canvasContext);
                            break;
                        case "circle":
                            this.drawCircle(startX, startY, endX, endY, this.canvasContext);
                            break;
                        case "hollow-circle":
                            this.drawHollowCircle(startX, startY, endX, endY, this.canvasContext);
                            break;
                        default:
                            break;
                    }
                })
                this.canvasContext.draw(false);
            },
            
            /**
             * 开始接触事件
             * @param {Object} e
             */
            penStart(e) {
                console.log("penStart: ", e)
                // #ifdef MP-WEIXIN
                const x = e.touches[0].pageX - this.topLeftPoint.left;
                const y = e.touches[0].pageY - this.topLeftPoint.top;
                // #endif
                // #ifndef MP-WEIXIN
                const x = e.touches[0].x;
                const y = e.touches[0].y;
                // #endif
                
                this.canvasContext.beginPath();
                if (this.bgImage) {
                    this.myCanvasContext.beginPath();
                }
                this.startX = x;
                this.startY = y;
                
                switch (this.shape) {
                    case "curve":
                        this.drawCurve(x, y);
                        break;
                    default:
                        break;
                }
                drawFigures = drawFigures.filter((figure, index) => index <= this.curStep);
            	drawFigures.push({
                    shape: this.shape,
                    lineColor: this.lineColor,
                    lineWidth: this.lineWidth,
            		points: [{
                        x,
                        y
                    }]
            	});
            },
            
            /**
             * 滑动事件
             * @param {Object} e
             */
            penMove(e) {
                if (this.intervalId) {
                    clearInterval(this.intervalId);
                    this.intervalId = null;
                }
                this.intervalId = setInterval(_ => this.canDraw = true, 100);
                
                // #ifdef MP-WEIXIN
                const x = e.touches[0].pageX - this.topLeftPoint.left;
                const y = e.touches[0].pageY - this.topLeftPoint.top;
                // #endif
                // #ifndef MP-WEIXIN
                const x = e.touches[0].x;
                const y = e.touches[0].y;
                // #endif
            	switch (this.shape) {
            	    case "curve":
                        this.addToPoints(x, y);
            	        this.drawCurve(x, y);
            	        this.canvasContext.draw(true);
            	        if (this.bgImage) {
            	            this.myCanvasContext.draw(true);
            	        }
            	        break;
            	    case "line":
            	        this.addToPoints(x, y);
            	        this.draw();
            	        break;
            	    case "rect":
            	        this.addToPoints(x, y);
            	        this.draw();
            	        break;
            	    case "hollow-rect":
            	        this.addToPoints(x, y);
            	        this.draw();
            	        break;
            	    case "circle":
            	        this.addToPoints(x, y);
            	        this.draw();
            	        break;
            	    case "hollow-circle":
            	        this.addToPoints(x, y);
            	        this.draw();
            	        break;
            	    default:
            	        break;
            	}
            },
            
            /**
             * 离开屏幕事件
             * @param {Object} e
             */
            penEnd(e) {
                console.log("penEnd: ", e);
                
                // #ifdef MP-WEIXIN
                const x = e.changedTouches[0].pageX - this.topLeftPoint.left;
                const y = e.changedTouches[0].pageY - this.topLeftPoint.top;
                // #endif
                // #ifndef MP-WEIXIN
                const x = e.changedTouches[0].x;
                const y = e.changedTouches[0].y;
                // #endif
                
            	switch (this.shape) {
                    case "curve":
                        this.addToPoints(x, y);
                        this.drawCurve(x, y);
                        break;
                    case "line":
                        if (this.bgImage) {
                            this.drawLine(this.startX, this.startY, x, y, this.myCanvasContext);
                            this.myCanvasContext.draw(true);
                        }
                        this.draw();
                        break;
                    case "rect":
                        if (this.bgImage) {
                            this.drawRect(this.startX, this.startY, x, y, this.myCanvasContext);
                            this.myCanvasContext.draw(true);
                        }
                        this.addToPoints(x, y);
                        this.draw();
                        break;
                    case "hollow-rect":
                        if (this.bgImage) {
                            this.drawHollowRect(this.startX, this.startY, x, y, this.myCanvasContext);
                            this.myCanvasContext.draw(true);
                        }
                        this.addToPoints(x, y);
                        this.draw();
                        break;
                    case "circle":
                        if (this.bgImage) {
                            this.drawCircle(this.startX, this.startY, x, y, this.myCanvasContext);
                            this.myCanvasContext.draw(true);
                        }
                        this.addToPoints(x, y);
                        this.draw();
                        break;
                    case "hollow-circle":
                        if (this.bgImage) {
                            this.drawHollowCircle(this.startX, this.startY, x, y, this.myCanvasContext);
                            this.myCanvasContext.draw(true);
                        }
                        this.addToPoints(x, y);
                        this.draw();
                        break;
                    default:
                        break;
                }
                this.saveImageData();
            },
            
            /**
             * 添加到点数组
             * @param {Object} x
             * @param {Object} y
             */
            addToPoints(x, y) {
                const drawFigure = drawFigures[this.curStep+1];
                if (this.shape == "curve") {
                    drawFigure.points.push({
                    	x,
                    	y
                    });
                } else if (drawFigure.points.length == 1) {
                    drawFigure.points.push({
                    	x,
                    	y
                    });
                } else {
                    drawFigure.points[1] = {
                    	x,
                    	y
                    };
                }
            },
            
            /**
             * 一次绘制完整曲线
             */
            drawCurveOnce(points, ctx = this.canvasContext) {
                points.forEach((point, index) => {
                    if (index != 0) {
                        ctx.moveTo(points[index-1].x, points[index-1].y);
                        ctx.lineTo(point.x, point.y);
                        ctx.stroke();
                    }
                })
            },
            
            /**
             * 绘制曲线中间添加点
             * @param {Object} x
             * @param {Object} y
             */
            drawCurve(x, y) {
                this.canvasContext.beginPath();
            	this.canvasContext.moveTo(this.startX, this.startY)
                this.canvasContext.strokeStyle = this.lineColor;
                this.canvasContext.setLineWidth(this.lineWidth);
            	this.canvasContext.lineTo(x, y);
            	this.canvasContext.stroke();
                
                if (this.bgImage) {
                    this.myCanvasContext.beginPath();
                    this.myCanvasContext.moveTo(this.startX, this.startY)
                    this.myCanvasContext.lineTo(x, y);
                    this.myCanvasContext.stroke();
                }
                
            	this.startX = x;
            	this.startY = y;
            },
            
            /**
             * 绘制直线添加点
             * @param {Object} startX
             * @param {Object} startY
             * @param {Object} x
             * @param {Object} y
             * @param {Object} ctx
             */
            drawLine(startX, startY, x, y, ctx = this.canvasContext) {
                ctx.beginPath();
                ctx.moveTo(startX, startY)
            	ctx.lineTo(x, y);
            	ctx.stroke();
            },
            
            /**
             * 这里是画实体矩形
             * @param {Object} startX
             * @param {Object} startY
             * @param {Object} x
             * @param {Object} y
             * @param {Object} ctx
             */
            drawRect(startX, startY, x, y, ctx = this.canvasContext) {
            	let width = Math.abs(x - startX);
            	let height = Math.abs(y - startY);
                
                ctx.beginPath();
            	ctx.rect(startX, startY, width, height);
            	ctx.fill();
            },
            
            /**
             * 画空心矩形
             * @param {Object} startX
             * @param {Object} startY
             * @param {Object} x
             * @param {Object} y
             * @param {Object} ctx
             */
            drawHollowRect(startX, startY, x, y, ctx = this.canvasContext) {
            	let pointLT = {};
            	let pointRB = {};
            	let pointRT = {};
            	let pointLB = {};
            	let center = {};
                
            	pointLT.X = Math.min(startX, x);
            	pointLT.Y = Math.min(startY, y);
            
            	pointRB.X = Math.max(startX, x);
            	pointRB.Y = Math.max(startY, y);
            
            	pointRT.X = pointRB.X;
            	pointRT.Y = pointLT.Y;
            
            	pointLB.X = pointLT.X;
            	pointLB.Y = pointRB.Y;
            
                let width = Math.abs(x - startX);
                let height = Math.abs(y - startY);
                
                ctx.beginPath();
            	ctx.rect(startX, startY, width, height);
            	ctx.stroke();
            },
            
            /**
             * 画实心圆
             * @param {Object} startX
             * @param {Object} startY
             * @param {Object} x
             * @param {Object} y
             * @param {Object} ctx
             */
            drawCircle(startX, startY, x, y, ctx = this.canvasContext) {
            	let pointLT = {};
            	let pointRB = {};
            	let pointRT = {};
            	let pointLB = {};
            	let center = {};
            
                pointLT.X = Math.min(startX, x);
                pointLT.Y = Math.min(startY, y);
                            
                pointRB.X = Math.max(startX, x);
                pointRB.Y = Math.max(startY, y);
            
            	pointRT.X = pointRB.X;
            	pointRT.Y = pointLT.Y;
            
            	pointLB.X = pointLT.X;
            	pointLB.Y = pointRB.Y;
            
            	center.X = (pointRB.X + pointLT.X) / 2;
            	center.Y = (pointRB.Y + pointLT.Y) / 2;
            
            	let dx = pointRB.X - pointLT.X;
            	let dy = pointRB.Y - pointLT.Y;
            	let r = Math.sqrt(dx * dx + dy * dy) / 2;
                
                ctx.beginPath();
            	ctx.arc(center.X, center.Y, r, 0, 2 * Math.PI);
            	ctx.fill();
            },
            
            /**
             * 这里是画空心圆
             * @param {Object} startX
             * @param {Object} startY
             * @param {Object} x
             * @param {Object} y
             * @param {Object} ctx
             */
            drawHollowCircle(startX, startY, x, y, ctx = this.canvasContext) {
            	let pointLT = {};
            	let pointRB = {};
            	let center = {};
            
                pointLT.X = Math.min(startX, x);
                pointLT.Y = Math.min(startY, y);
                            
                pointRB.X = Math.max(startX, x);
                pointRB.Y = Math.max(startY, y);
            
            	center.X = (pointRB.X + pointLT.X) / 2;
            	center.Y = (pointRB.Y + pointLT.Y) / 2;
            
            	let dx = pointRB.X - pointLT.X;
            	let dy = pointRB.Y - pointLT.Y;
            	let r = Math.sqrt(dx * dx + dy * dy) / 2;
                
                ctx.beginPath();
                ctx.arc(center.X, center.Y, r, 0, 2 * Math.PI);
            	ctx.stroke();
            },
            
            /**
             * 获取组件信息
             * @param {Object} selector
             * @param {Object} queryIn
             */
            getBoundingClientRect(selector, queryIn) {
                const selectors = selector.split(",");
                if (selectors.length == 1) {
                    return new Promise((resolve) => {
                        const query = uni.createSelectorQuery();
                        if (queryIn) {
                            query.in(queryIn).select(selector).boundingClientRect(res => resolve(res)).exec();
                        } else {
                            query.select(selector).boundingClientRect(res => resolve(res)).exec();
                        }
                    })
                } else {
                    return new Promise((resolve) => {
                        const query = uni.createSelectorQuery();
                        if (queryIn) {
                            query.in(queryIn).selectAll(selector).boundingClientRect(res => resolve(res)).exec();
                        } else {
                            query.selectAll(selector).boundingClientRect(res => resolve(res)).exec();
                        }
                    })
                }
            }
        }
    }
</script>

<style>
</style>