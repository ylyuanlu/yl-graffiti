# yl-graffiti涂鸦组件使用说明
## 组件引用方式
### easycom方式引用 
将yl-graffiti组件放入根目录下的components中:
```
|--components
|    |--yl-graffiti
|        |-yl-graffiti.vue
|--pages
|--pages.json
```

然后在pages.json文件中添加easycom声明：
```
{
    "easycom": {
        "autoscan": true,
        "custom": {
            "^yl-(.*)": "@/components/yl-$1/yl-$1.vue"
        }
    },
    ...
}
```

### 代码引用 
同上，将yl-graffiti组件放入根目录下的components中或放在使用该组件页面对应的目录下，然后在页面中导入该组件：
```
<script>
    import ylGraffiti from "./components/yl-graffiti/yl-graffiti.vue";
    export default {
        components: {
            ylGraffiti
        },
        ...
    }
</script>
```

## 示例代码
组件使用相关代码，详细代码请参考示例工程，下面贴出主要代码。
```
<view>
    ...
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
    <!-- 涂鸦组件控制视图，省略 -->
    ...
</view>
<script>
    // 涂鸦组件功能由页面来控制
    import ylGraffiti from "./components/yl-graffiti/yl-graffiti.vue";
    
    export default {
        components: {
            ylGraffiti
        },
        data() {
            return {
                stepInfo: { // 用来控制撤销和重做
                    curStep: -1,
                    len: 0
                },
                saving: false
            }
        },
        ...,
        methods: {
            ...,
            
            /**
             * 当前位置变化
             * @param {Object} e
             */
            stepChanged(e) {
                this.stepInfo = e;
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
             * 保存涂鸦
             */
            savePicture() {
                this.saving = true;
                this.$refs.graffiti.saveCanvas().then(res => {
                    this.pictures[this.swiperCurrent].url = res;
                    setTimeout(_ => this.saving = this.writing = false, 100);
                });
            }
        }
    }
</script>
```

## 组件接口
### 属性

| 参数             | 说明                  | 类型              | 默认值        |
| --------------   | ------------         | ----------------  | ------------ |
| canvasId          | 画布ID              | <em>string</em>   |    `MyCanvas`           |
| width     | 画布宽度            | <em>number</em>    | `300`        |
| height     | 画布高度            | <em>number</em>    | `225`        |
| shape     | 画笔绘制图形形状        | <em>string</em>     | `curve`   |
| lineColor         | 画笔颜色              | <em>string</em>    | `#091A22`      |
| lineWidth       | 画笔宽度      | <em>number</em> | `5`  |
| bgColor       | 背景颜色           | <em>string</em>   |        |
| bgImage       | 背景图片           | <em>string</em>   |        |


## 参考说明
参考链接：https://ext.dcloud.net.cn/plugin?id=782 感谢作者的付出，给我提供了一些思路，并做了如下优化：
+ 亲测支持H5、小程序和APP-PLUS等平台
+ 添加更多涂鸦图形的选择，并解决了绘制过程中的重影问题
+ 支持撤销、重做等功能，并优化了小程序绘制闪屏问题
+ 可将图片和涂鸦合成保存

