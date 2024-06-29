<template>
    <div id="mapDiv" class="map"></div>
    <div id="popup" class="ol-popup" v-show="popupVisible">
        <a href="#" id="popup-closer" class="ol-popup-closer" @click="closePopup"></a>
        <div id="popup-content">

            <p><strong>城市名:</strong> {{ popupName }}</p>
        </div>
    </div>
</template>

<script setup>
    import "ol/ol.css";
    import { onMounted, ref } from 'vue';
    import { Map, View } from "ol";
    import TileLayer from "ol/layer/Tile";
    import OSM from "ol/source/OSM";
    import ImageLayer from "ol/layer/Image";
    import ImageWMS from "ol/source/ImageWMS";
    import Overlay from 'ol/Overlay';
    import { fromLonLat, toLonLat } from 'ol/proj';
    import XYZ from 'ol/source/XYZ';

    import { mapTile, popupName, popupVisible, popupElement, popup } from "./Map.js"



    const popupCoordinates = ref('');
    let isAnimating = false;

    const closePopup = (event) => {
        event.preventDefault();
        popupVisible.value = false;
    };
    const view = new View({
        center: fromLonLat([104.195397, 35.86166]),
        zoom: 4
    });



    onMounted(() => {
        mapTile.value = new Map({
            target: "mapDiv",
            layers: [
                new TileLayer({
                    source: new OSM()
                }),
                new ImageLayer({
                    source: new ImageWMS({
                        url: 'http://localhost:8088/geoserver/ChinaCoal/wms',
                        params: {
                            'LAYERS': 'ChinaCoal:37cityPro',
                            'VERSION': '1.1.0',
                            'FORMAT': 'image/png'
                        },
                        serverType: 'geoserver'
                    })
                }),
                new TileLayer({
                    source: new XYZ({
                        url: 'http://wprd0{1-4}.is.autonavi.com/appmaptile?lang=zh_cn&size=1&style=8&x={x}&y={y}&z={z}'
                    })
                }),

            ],
            view: view
        });

        popupElement.value = document.getElementById('popup');
        popup.value = new Overlay({
            element: popupElement.value,
            autoPan: true,
            autoPanAnimation: {
                duration: 250
            }
        });
        mapTile.value.addOverlay(popup.value);

        mapTile.value.on('singleclick', async function (evt) {
            const viewResolution = mapTile.value.getView().getResolution();
            const url = mapTile.value.getLayers().getArray()[1].getSource().getFeatureInfoUrl(
                evt.coordinate, viewResolution, 'EPSG:3857',
                { 'INFO_FORMAT': 'application/json' }
            );
            console.log(url)
            if (url) {
                fetch(url)
                    .then(response => response.json())
                    .then(async (data) => {
                        if (data.features.length > 0) {
                            const feature = data.features[0];
                            const coordinate = evt.coordinate;
                            const lonLat = toLonLat(coordinate);

                            popupName.value = feature.properties.NAME;
                            popupCoordinates.value = `${lonLat[0].toFixed(6)}, ${lonLat[1].toFixed(6)}`;
                            popup.value.setPosition(coordinate);
                            popupVisible.value = true;

                            isAnimating = true;
                            view.animate({
                                center: coordinate,
                                zoom: 7,
                                duration: 1000
                            });
                            isAnimating = false;
                        } else {
                            popupVisible.value = false;
                        }
                    });
            }
        });
    });
</script>

<style>
    .map {
        width: 100%;
        height: 100vh;
        /* 使地图全屏显示 */
    }

    .ol-popup {
        position: absolute;
        background-color: white;
        box-shadow: 0 1px 4px rgba(0, 0, 0, 0.2);
        padding: 15px;
        border-radius: 10px;
        border: 1px solid #cccccc;
        bottom: 12px;
        left: -50px;
        min-width: 200px;
    }

    .ol-popup:after,
    .ol-popup:before {
        top: 100%;
        border: solid transparent;
        content: " ";
        height: 0;
        width: 0;
        position: absolute;
        pointer-events: none;
    }

    .ol-popup:after {
        border-top-color: white;
        border-width: 10px;
        left: 48px;
        margin-left: -10px;
    }

    .ol-popup:before {
        border-top-color: #cccccc;
        border-width: 11px;
        left: 48px;
        margin-left: -11px;
    }

    .ol-popup-closer {
        text-decoration: none;
        position: absolute;
        top: 2px;
        right: 8px;
    }

    .ol-popup-closer:after {
        content: "✖";
    }
</style>