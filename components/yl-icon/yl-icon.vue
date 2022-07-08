<template>
    <view class="flex-row flex-center" :class="customClass" :style="[customStyle]">
        <view :class="[familyIcon, size, color]"
            :style="{
                color: color.startsWith('#') ? color: ''
            }"
             @click="$emit('click')">
        </view>
    </view>
</template>
<script>
    export default {
        name: 'yl-icon',
        props: {
            /**
             * 图标类名
             * 已支持cuicon-*（来自colorui）, uicon-*（来自uview-ui）, icon-*（svg使用）。
             * 可自定义其他字体库，需要将font-family命名其他名称防止命名冲突，如：custicon
             */
            icon: {
                type: String,
                default: ''
            },
            // 图标颜色
            color: {
                type: String,
                default: ''
            },
            // 字体大小
            size: {
                type: String,
                default: 'size-16'
            },
            // 自定义样式类
            customClass: {
                type: [String, Array],
                default: ''
            },
            // 自定义样式
            customStyle: {
                type: Object,
                default: function() {
                    return {}
                }
            }
        },

        computed: {
            /**
             * 图标类库
             */
            customPrefix() {
                return this.toFamily(this.icon);
            },

            /**
             * 图标类库+样式类
             */
            familyIcon() {
                return this.toFamilyIcon(this.icon)
            },
            
            /**
             * 图标名称
             */
            iconName() {
                return this.toIconName(this.icon, true);
            }
        },
        methods: {
            /**
             * 转换成字体库类名+图标名称
             * @param {Object} icon
             */
            toFamilyIcon(icon) {
                if (!icon) {
                    return '';
                }
            
                // uview
                if (icon.startsWith("uicon-")) {
                    return 'u-iconfont ' + icon;
                }
            
                // 自定义，为了兼容uView的图标扩展
                if (icon.startsWith("custom-icon-")) {
                    return 'custom-icon ' + icon;
                }
            
                // colorui和自定义
                return icon.split('-')[0] + ' ' + icon;
            },
            
            /**
             * 转换成字体库类名
             * @param {Object} icon
             */
            toFamily(icon) {
                if (!icon) {
                    return ''
                }
            
                // uview
                if (icon.startsWith("uicon-")) {
                    return 'u-iconfont';
                }
            
                // 自定义，为了兼容uView的图标扩展
                if (icon.startsWith("custom-icon-")) {
                    return 'custom-icon';
                }
            
                // colorui和自定义
                return icon.split('-')[0]
            },
            
            /**
             * 转换成图标名称
             */
            toIconName(icon, fromStart = false) {
                if (!icon) {
                    return ''
                }
            
                // 返回图标名称
                const index = fromStart ? icon.indexOf('-'): icon.lastIndexOf('-')
                if (index == -1) {
                    return icon
                }
                return icon.substring(index + 1)
            }
        }
    }
</script>

<style>
</style>
