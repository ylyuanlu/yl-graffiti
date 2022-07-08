<script>
	export default {
		onLaunch: function() {
			console.log('App Launch')
		},
		onShow: function() {
			console.log('App Show')
		},
		onHide: function() {
			console.log('App Hide')
		}
	}
</script>

<style lang="scss">
    /*
     * 这是一款适应于H5、微信小程序、安卓、ios、支付宝的高颜值，高度自定义的Css组件库.
     * 但colorui使用了复合样式，因而不支持nvue页面开发，index.scss里面已经对使用平台
     * 做了编译控制
     */
    @import "libs/colorui/index.scss";
    
    /**
     * 工程自定义样式
     * 不支持的复合样式，条件编译在内部解决
     */
    @import "common/css/index.scss";
</style>
