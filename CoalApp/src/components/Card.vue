<script setup>
    import { ref } from 'vue'
    import { toLonLat } from 'ol/proj';
    import { mapTile } from '../views/Map.js'
    import Overlay from 'ol/Overlay';
    import TileLayer from "ol/layer/Tile";
    import TileWMS from 'ol/source/TileWMS';

    import XYZ from 'ol/source/XYZ';


    const input1 = ref('');
    let highlightLayer = ref(null);
    let baiduAnnoLayer = ref(null);

    const searchCity = async () => {
        const cityName = input1.value.trim();
        if (!cityName) return;

        if (highlightLayer.value) {
            mapTile.value.removeLayer(highlightLayer.value);
        }

        highlightLayer.value = new TileLayer({
            source: new TileWMS({
                url: 'http://localhost:8088/geoserver/ChinaCoal/wms',
                params: {
                    'LAYERS': 'ChinaCoal:37cityPro',
                    'TILED': true,
                    'INFO_FORMAT': 'application/json',
                    'CQL_FILTER': `NAME LIKE '%${cityName}%'`,
                    'STYLES': 'restricted'
                },
                serverType: 'geoserver',
                crossOrigin: 'anonymous'
            }),
            opacity: 0.7
        });

        mapTile.value.addLayer(highlightLayer.value);


    };
    const removeLayer = () => {
        console.log(highlightLayer.value)
        if (highlightLayer.value) {
            mapTile.value.removeLayer(highlightLayer.value);
        }
    };


    const BaiduVec = () => {
        if (!mapTile.value) return;
        const layers = mapTile.value.getLayers().getArray();
        console.log(layers);
        if (layers.length > 0) {
            mapTile.value.removeLayer(layers[0]);
        }
        const baiduVecLayer = new TileLayer({
            source: new XYZ({
                url: 'http://wprd0{1-4}.is.autonavi.com/appmaptile?lang=zh_cn&size=1&style=7&x={x}&y={y}&z={z}'
            })
        });
        mapTile.value.getLayers().insertAt(0, baiduVecLayer);
        console.log(mapTile.value.getLayers().getArray());
    };


    const OSMLayer = () => {
        if (!mapTile.value) return;
        const layers = mapTile.value.getLayers().getArray();
        console.log(layers);
        if (layers.length > 0) {
            mapTile.value.removeLayer(layers[0]);
        }
        const osmLayer = new TileLayer({
            source: new XYZ({
                url: 'http://tile.openstreetmap.org/{z}/{x}/{y}.png'
            })
        });
        mapTile.value.getLayers().insertAt(0, osmLayer);
        console.log(mapTile.value.getLayers().getArray());
    };

    const BaiduImg = () => {
        if (!mapTile.value) return;
        if (baiduAnnoLayer.value) {
            mapTile.value.removeLayer(baiduAnnoLayer.value);
        }

        const layers = mapTile.value.getLayers().getArray();

        // 移除第一个图层
        if (layers.length > 0) {
            mapTile.value.removeLayer(layers[0]);
        }

        // 创建百度影像地图图层
        const baiduImgLayer = new TileLayer({
            source: new XYZ({
                url: 'http://wprd0{1-4}.is.autonavi.com/appmaptile?lang=zh_cn&size=1&style=6&x={x}&y={y}&z={z}'
            })
        });

        // 创建百度注记图层
        baiduAnnoLayer.value = new TileLayer({
            source: new XYZ({
                url: 'http://wprd0{1-4}.is.autonavi.com/appmaptile?lang=zh_cn&size=1&style=8&x={x}&y={y}&z={z}'
            })
        });
        mapTile.value.getLayers().insertAt(0, baiduImgLayer);
        mapTile.value.addLayer(baiduAnnoLayer.value);

        console.log(mapTile.value.getLayers().getArray());
    };


</script>

<template>
    <div class="Card-column">
        <el-card class="Card-wrapper">
            <div class="Card-wrapper-header">
                <el-button type="primary" @click="OSMLayer">OSM图层</el-button>
                <el-button type="primary " @click="BaiduVec">百度矢量地图</el-button>
                <el-button type="primary" @click="BaiduImg">百度影像地图</el-button>
            </div>
            <div class="Card-wrapper-body">

                <div class="Card-search">
                    <el-input v-model="input1" style="width: 300px" size="large" placeholder="查询资源枯竭城市" />
                    <el-icon>
                        <Search color="#5281aa" @click="searchCity" class="Card-search-icon" />
                    </el-icon>
                </div>
            </div>
            <div class="Card-search-delete">
                <el-button type="primary" plain @click="removeLayer">清除高亮图层</el-button>
            </div>

        </el-card>

    </div>
</template>


<style>
    .Card-column {
        display: flex;
        flex-direction: column;
        justify-content: center;
        width: 400px;
        height: calc(100% - 60px);

        position: absolute;
        z-index: 2;
    }

    .Card-wrapper {
        width: 100%;
        position: relative;
        left: 100px;
        color: #5281aa;
    }

    .Card-wrapper-header {
        display: flex;
        justify-content: space-between;
    }

    .Card-search {
        font-size: 30px;
    }

    .Card-search-icon {
        margin-left: 5px;
        position: relative;
        top: 5px;
    }

    .Card-wrapper-body {
        padding-top: 20px;
        display: flex;
        flex-direction: column;
    }

    .Card-search-delete {
        padding-top: 20px;
        display: flex;
        justify-content: flex-start;
    }
</style>