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
	/* 注意要写在第一行，同时给style标签加入lang="scss"属性 */
	@import "uview-ui/index.scss";
  .app-bg {
    position: absolute;
    min-width: 100%;
    min-height: 100%;
    background: url('http://cdn.xkeyi.top/xiaoyequ/bg1.jpg') no-repeat;
    background-size: 100% 100%;
  }
  .content {
    position: absolute;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: flex-end;
    min-width: 100%;
    min-height: 100%;
    background: rgba(0,0,0,0.5);
    // background-color: #000;
    // opacity: 0.6;
  }
</style>
