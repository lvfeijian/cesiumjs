<template>
	<div>
		<div id="cesiumContainer"></div>
		<button @click="destoryLayer">销毁影像图层</button>
		<button @click="addProvider">添加图层</button>
		<button @click="addTerrain">添加地形、水流</button>
		<button @click="addCZMLDataSource">添加czml数据</button>
		<button @click="addEntities">加载glTF和entities数据</button>
		<button @click="add3dTiles">添加3d Tiles</button>
		<button @click="addPrimitive">添加几何体(geometry)</button>
	</div>
</template>

<script>
Cesium.Ion.defaultAccessToken =
	"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiJjODAxZTQ1ZS0yM2RhLTRmMGQtODgyZi0zZTY2MGQ1YTY5MTkiLCJpZCI6OTE3MjMsImlhdCI6MTY1MTIwMjU1N30.WkHgoYh3hceaxm2w5dyU_uBDAQF0a7qpF4R70DzHnwI";
export default {
	data() {
		return {
			option: {
				infoBox: false, // 解决控制台报错
				animation: false,
				BaseLayerPicker: false,
				fullscreenButton: false,
				vrButton: false,
				geocoder: false,
				homeButton: false,
				sceneModePicker: false,
				selectionIndicator: false,
				timeline: false,
				navigationHelpButton: false,
				navigationInstructionsInitiallyVisible: false,
				shouldAnimate: true,
				// terrainProvider: new Cesium.GoogleEarthEnterpriseTerrainProvider(),
				// imageryProvider 加载影像
				// imageryProvider: new Cesium.ArcGisMapServerImageryProvider({
				// 	url: "https://services.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer",
				// }),
				// imageryProvider: new Cesium.IonImageryProvider({ assetId: 2 }),
			},
			viewer: null,
		};
	},
	mounted() {
		this.initCesium();

		const destination = Cesium.Cartesian3.fromDegrees(-122.4175, 37.655, 400); // 经纬度转世界坐标
		console.log("世界坐标", destination);
		const cartographic = Cesium.Cartographic.fromCartesian(destination); // 世界坐标系转弧度
		console.log("弧度", cartographic);
		let lat = Cesium.Math.toDegrees(cartographic.latitude); // 弧度转经度
		let lng = Cesium.Math.toDegrees(cartographic.longitude); // 弧度转纬度
		console.log(lng, lat, cartographic.height);

		// flyTo 将相机从当前位置飞到新位置。
		this.viewer.camera.flyTo({
			destination,
			orientation: {
				heading: Cesium.Math.toRadians(0.0), // 将度数转化为弧度
				// pitch: Cesium.Math.toRadians(-60.0), // 有pitch建议使用viewer.flyTo
			},
		});
		// entity点击事件
		this.viewer.selectedEntityChanged.addEventListener(function (entity) {
			if (Cesium.defined(entity)) {
				console.log(entity.id, entity);
			}
		});

		this.clickScreenGetCoordinate();
	},
	methods: {
		/**
		 * 初始化cesium
		 */
		initCesium() {
			this.viewer = new Cesium.Viewer("cesiumContainer", this.option);
			// 去掉logo
			this.viewer._cesiumWidget._creditContainer.style.display = "none";
		},

		/**
		 * 用户点击屏幕获取屏幕坐标，以及屏幕坐标转世界坐标
		 */
		clickScreenGetCoordinate() {
			new Cesium.ScreenSpaceEventHandler(
				this.viewer.scene.canvas
			).setInputAction((movement) => {
				console.log("屏幕坐标:", movement);
				var cartesian = this.viewer.scene.globe.pick(
					this.viewer.camera.getPickRay(movement.position),
					this.viewer.scene
				);
				console.log("世界坐标", cartesian);
			}, Cesium.ScreenSpaceEventType.LEFT_CLICK);
		},
		/**
		 * 销毁最外层影像图层
		 */
		destoryLayer() {
			// 获取图层的个数
			let length = this.viewer.imageryLayers.length;
			if (length <= 1) {
				return;
			}
			let isDestoryLayer = this.viewer.imageryLayers.remove(
				this.viewer.imageryLayers.get(length - 1) // 获取影像图层
			);
			console.log(isDestoryLayer);
		},

		/**
		 * 添加影像
		 */
		addProvider() {
			this.viewer.imageryLayers.addImageryProvider(
				new Cesium.ArcGisMapServerImageryProvider({
					url: "https://services.arcgisonline.com/ArcGIS/rest/services/NatGeo_World_Map/MapServer/",
				})
			);
			// this.viewer.imageryLayers.add(
			// 	new Cesium.ImageryLayer(
			// 		new Cesium.ArcGisMapServerImageryProvider({
			// 			url: "https://services.arcgisonline.com/ArcGIS/rest/services/NatGeo_World_Map/MapServer/",
			// 		})
			// 	)
			// );
		},

		/**
		 * 添加地形，以及水流
		 */
		addTerrain() {
			this.viewer.terrainProvider = new Cesium.CesiumTerrainProvider({
				url: Cesium.IonResource.fromAssetId(1),
				requestVertexNormals: true,
				requestWaterMask: true,
			});
		},

		/**
		 * 添加czml数据源
		 */
		addCZMLDataSource() {
			const dataSourcePromise = this.viewer.dataSources.add(
				Cesium.CzmlDataSource.load("/DataSource/Vehicle.czml")
			);
			dataSourcePromise.then((dataSource) => {
				// this.viewer.camera.flyTo;
				this.viewer.scene.camera.setView({
					destination: Cesium.Cartesian3.fromDegrees(-116.52, 35.02, 95000),
					orientation: {
						heading: 6,
					},
				});
			});
		},

		/**
		 * entities 添加实体(加载glTF数据)
		 */
		addEntities() {
			const position = Cesium.Cartesian3.fromDegrees(
				-123.0744619,
				44.0503706,
				5000
			);
			const entity = this.viewer.entities.add({
				position,
				model: {
					uri: "/DataSource/Cesium_Air.glb",
					minimumPixelSize: 128,
					maximumScale: 20000,
				},
			});
			// console.log(entity);
			this.viewer.trackedEntity = entity;
		},

		/**
		 * 添加3D Tiles
		 */
		add3dTiles() {
			const tileset = this.viewer.scene.primitives.add(
				// 添加3D Tiles
				new Cesium.Cesium3DTileset({
					url: "/DataSource/InstancedQuantizedOct32POrientation/tileset.json",
					maximumScreenSpaceError: 2,
				})
			);
			tileset.style = new Cesium.Cesium3DTileStyle({
				color: "color('red')",
			});
			tileset.readyPromise.then((tileset) => {
				// this.viewer.zoomTo(tileset);
				// 设置相机位置，调整角度
				this.viewer.camera.viewBoundingSphere(
					tileset.boundingSphere,
					new Cesium.HeadingPitchRange(0, -0.5, 0)
				);
			});
		},

		/**
		 * 图元 Primitive绘制, 添加几何实例
		 */
		addPrimitive() {
			// 创建一个几何图形圆
			const circle = new Cesium.CircleGeometry({
				center: Cesium.Cartesian3.fromDegrees(-75.59777, 40.03883),
				radius: 1000000.0,
				height: 100000,
			});
			this.viewer.scene.primitives.add(
				new Cesium.Primitive({
					// 设置几何体实例
					geometryInstances: new Cesium.GeometryInstance({
						geometry: circle,
					}),
					// 设置椭圆体表面上的外观
					appearance: new Cesium.MaterialAppearance({
						closed: true,
						material: Cesium.Material.fromType("Checkerboard"),
					}),
				})
			);
		},
	},
};
</script>

<style lang="less" scoped>
#cesiumContainer {
	width: 100%;
}
</style>