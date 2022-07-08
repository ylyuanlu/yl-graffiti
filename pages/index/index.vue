<template>
	<view class="full-screen flex-column flex-center bg-light">
		<view v-if="!picture" class="flex-column flex-center square-30vw round shadow bg-blue pdt-5" @click="choseImage">
		    <yl-icon icon="custom-icon-add-image" size="size-35" color="text-white"></yl-icon>
		    <text class="text-df text-white mgt-2">选择图片</text>
		</view>
        <view v-else-if="!writing" class="fill-parent">
            <view class="w-100 h-90vh flex-column flex-center">
                <image :src="picture" @click="writing = true" :style="canvasStyle"></image>
            </view>
            <view class="pos-fixed bottom-0p bg-white w-100 flex-row align-center justify-around pdtb-5">
                <view v-for="(item, idx) in options" :key="idx" class="flex-column flex-center pd-5" @click="selectOption(idx)">
                    <yl-icon :icon="item.icon" size="size-22" color="text-black"></yl-icon>
                    <text class="text-sm text-black mgt-2">{{item.label}}</text>
                </view>
            </view>
        </view>
        <view v-else class="full-screen">
            <view class="h-75vh flex-column flex-center">
                <yl-graffiti
                    ref="graffiti"
                    canvas-id="myCanvas"
                    :width="canvasStyle.w"
                    :height="canvasStyle.h"
                    :shape="curShape"
                    :lineColor="lineColor"
                    :lineWidth="lineSize"
                    :bgImage="picture"
                    @stepChanged="stepChanged">
                </yl-graffiti>
            </view>
            <view class="w-100 h-25vh pos-fixed bottom-0p bg-white pdtb-5 flex-column justify-between">
                <view v-if="optIndex == 0" class="flex-row align-center justify-around mglr-50 pdb-5" style="height: 9vh;">
                    <view v-for="(item, idx) in shapes"
                        :key="idx" 
                        @click="updateShape(item.shape, idx)">
                        <view v-if="item.shape == 'rect' || item.shape == 'circle'" style="width: 18px;height: 18px;" :class="[item.active ? 'bg-blue': 'bg-u-main', {'round': item.shape == 'circle', 'radius-2': item.shape == 'rect'}]"></view>
                        <yl-icon v-else :icon="item.icon" :size="item.size || 'size-22'" :color="item.active ? 'text-blue': 'text-black'"></yl-icon>
                    </view>
                </view>
                <view v-if="optIndex == 1" class="flex-row align-center justify-around mglr-50 pdb-5" style="height: 9vh;">
                    <view v-for="(item, idx) in colors"
                        :key="idx" 
                        class="flex-1 h-15p" 
                        :class="{'solids': item.active}"
                        :style="'background-color:' + item.color"
                        @click="updateColor(item.color, idx)">
                    </view>
                </view>
                <view v-if="optIndex == 2" class="flex-row align-center justify-around mglr-50 pdb-5" style="height: 9vh;">
                    <view v-for="(item, idx) in sizes" 
                        :key="idx" 
                        class="square-20p flex-row flex-center"
                        @click="updateSize(item.thickness, idx)">
                        <view class="round" 
                            :class="{'bg-u-info-disabled': !item.active, 'bg-blue': item.active}"
                            :style="'width:'+item.thickness+'px;height:'+item.thickness+'px'">
                        </view>
                    </view>
                </view>
                <view v-if="optIndex == 3" class="flex-row align-center justify-around mglr-30" style="height: 9vh;"></view>
                <view class="flex-row align-center justify-around mglr-20" style="height: 8vh;">
                    <view v-for="(item, idx) in writingOptions" 
                        :key="item.label" 
                        class="flex-column flex-center pd-5"
                        @click="selectWritingOption(idx)">
                        <view class="square-25p flex-row flex-center" :class="{'solid border-blue round': optIndex == 1 && optIndex == idx}">
                            <yl-icon :icon="item.icon" :size="optIndex == 1 && optIndex == idx ? 'size-18 lh-1': 'size-22'" :color="idx == 1 ? lineColor: optIndex == idx ? 'text-blue': item.color"></yl-icon>
                        </view>
                        <text class="text-sm mgt-2" :class="[{'text-blue': optIndex == idx}, optIndex != idx ? item.color: '']">{{item.label}}</text>
                    </view>
                </view>
                <view class="flex-row align-center justify-between padding-lr pdtb-5 mgt-15 mgb-10" style="height: 8vh;">
                    <yl-icon icon="cuicon-close" size="size-28" color="text-black" @click="writing = false"></yl-icon>
                    <text class="text-xl text-black">涂鸦</text>
                    <yl-icon icon="cuicon-check" size="size-28" color="text-black" customClass="pdlr-5 pdtb-2" @click="savePicture"></yl-icon>
                </view>
            </view>
        </view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				title: '涂鸦',
                picture: "",
                writing: false,
                swiperCurrent: -1,
                canvasStyle: {},
                optIndex: 0,
                options: [{
                    label: "涂鸦",
                    icon: "cuicon-write"
                }, {
                    label: "删除",
                    icon: "cuicon-delete"
                }],
                shapes: [ //画笔形状
                    { icon: "cuicon-write", shape: "curve", active: true },
                    { icon: "cuicon-move", shape: "line", active: false },
                    { icon: "cuicon-square", shape: "rect", active: false },
                    { icon: "cuicon-square", shape: "hollow-rect", active: false },
                    { icon: "cuicon-round", shape: "circle", active: false },
                    { icon: "cuicon-round", shape: "hollow-circle", active: false }
                ],
                colors: [ //画笔颜色 
                    { color: "#ff0000", active: true },
                    { color: "#1c9d02", active: false },
                    { color: "#000000", active: false },
                    { color: "#006ce6", active: false },
                    { color: "#efaa03", active: false }
                ],
                sizes: [ //画笔粗细
                    { thickness: 2, active: true },
                    { thickness: 4, active: false },
                    { thickness: 6, active: false },
                    { thickness: 8, active: false },
                    { thickness: 10, active: false }
                ],
                stepInfo: {
                    curStep: -1,
                    len: 0
                },
                saving: false,
                systemInfo: null
			}
		},
		onLoad() {
            this.systemInfo = uni.getSystemInfoSync();
		},
        computed: {
            /**
             * 涂鸦操作项
             */
            writingOptions() {
                const {
                    curStep,
                    len
                } = this.stepInfo;
                return [{
                    label: "涂鸦",
                    icon: "cuicon-write",
                    color: 'text-black'
                }, {
                    label: "颜色",
                    icon: "cuicon-radioboxfill",
                    color: 'text-black'
                }, {
                    label: "粗细",
                    icon: "cuicon-radiobox",
                    color: 'text-black'
                }, {
                    label: "撤销",
                    icon: "cuicon-repeal",
                    color: curStep == -1 ? 'text-gray': 'text-black'
                }, {
                    label: "重做",
                    icon: "cuicon-repeal transform-180",
                    color: curStep == len-1 ? 'text-gray': 'text-black'
                }, {
                    label: "清空",
                    icon: "custom-icon-clear",
                    color: curStep == -1 ? 'text-gray': 'text-black'
                }]
            },
            
            /**
             * 当前形状
             */
            curShape() {
                return this.shapes.filter(shape => shape.active)[0].shape;
            },
            
            /**
             * 线条颜色
             */
            lineColor() {
                return this.colors.filter(color => color.active)[0].color;
            },
            
            /**
             * 线条粗细
             */
            lineSize() {
                return this.sizes.filter(size => size.active)[0].thickness;
            },
        },
		methods: {
            /**
             * 选择图片
             */
            choseImage() {
            	uni.chooseImage({
            		count: 1,
            		sizeType: ['original', 'compressed'], //可以指定是原图还是压缩图，默认二者都有
            		sourceType: ['camera', 'album'] //拍照或从相册选择
            	}).then(res => {
                    const picture = res[1].tempFiles[0].path;
            		uni.getImageInfo({
            		    src: picture,
            		    success: image => {
            		        this.canvasStyle = this.caclCanvasStyle(image.height, image.width);
                            this.picture = picture;
            		    },
            		    fail: err => {
            		        console.log("getImageInfo err: ", err);
            		    }
            		});
            	}).catch(res => {})
            },
            
            /**
             * 计算高
             * @param {Object} imgHeight
             * @param {Object} imgWidth
             */
            caclCanvasStyle(imgHeight, imgWidth) {
                let width = 0, height = 0;
                
                if (imgWidth > imgHeight) {
                    if (this.systemInfo.windowWidth <= imgWidth) {
                        width = this.systemInfo.windowWidth;
                        height = (this.systemInfo.windowWidth / imgWidth) * imgHeight;
                    } else {
                        width = imgWidth;
                        height = imgHeight;
                    }
                } else {
                    if (this.systemInfo.windowHeight * 0.7 <= imgHeight) {
                        height = this.systemInfo.windowHeight * 0.7;
                        width = (this.systemInfo.windowHeight * 0.7 / imgHeight) * imgWidth;
                        if (this.systemInfo.windowWidth <= width) {
                            width = this.systemInfo.windowWidth * 0.8;
                            height = (this.systemInfo.windowWidth * 0.8 / width) * height;
                        }
                    } else {
                        width = imgWidth;
                        height = imgHeight;
                    }
                }
                
                return {
                    width: width.toFixed(0) + "px",
                    height: height.toFixed(0) + "px",
                    w: parseInt(width.toFixed(0)),
                    h: parseInt(height.toFixed(0)),
                    style: `width:${width.toFixed(0)}px;height:${height.toFixed(0)}px;`
                }
            },
            
            /**
             * 图片操作
             * @param {Object} index
             */
            selectOption(index) {
                switch (index) {
                    case 0:
                        this.writing = true;
                        break;
                    case 1:
                        this.picture = "";
                        break;
                    default:
                        break;
                }
            },
            
            /**
             * 选择涂鸦的类型
             * @param {Object} index
             */
            selectWritingOption(index) {
                switch (index) {
                    case 0:
                    case 1:
                    case 2:
                        this.optIndex = index;
                        break;
                    case 3:
                        this.$refs.graffiti.repeal();
                        break;
                    case 4:
                        this.$refs.graffiti.redo();
                        break;
                    case 5:
                        this.$refs.graffiti.clearBoard();
                        break;
                    default:
                        break;
                }
            },
            
            /**
             * 当前位置变化
             * @param {Object} e
             */
            stepChanged(e) {
                this.stepInfo = e;
            },
            
            /**
             * 更新线条形状
             * @param {Object} shape
             * @param {Object} index
             */
            updateShape(shape, index) {
                this.shapes.forEach((item, idx) => item.active = idx == index);
            },
            
            /**
             * 更新线条颜色
             * @param {Object} color
             * @param {Object} index
             */
            updateColor(color, index) {
                this.colors.forEach((item, idx) => item.active = idx == index);
            },
            
            /**
             * 更新线条尺寸
             * @param {Object} size
             * @param {Object} index
             */
            updateSize(size, index) {
                this.sizes.forEach((item, idx) => item.active = idx == index);
            },
            
            /**
             * 保存涂鸦
             */
            savePicture() {
                uni.showLoading();
                this.$refs.graffiti.saveCanvas().then(res => {
                    this.picture = res;
                    this.writing = false;
                    uni.hideLoading();
                });
            }
		}
	}
</script>

<style>
</style>
