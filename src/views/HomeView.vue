<template>
	<div>
		<div id="cesiumContainer"></div>
	</div>
</template>

<script>
export default {
	mounted() {
		new Cesium.Viewer("cesiumContainer", { infoBox: false });
	},
};
</script>

<style lang="less" scoped>
#cesiumContainer {
	width: 100vw;
	height: 100vh;
}
</style>