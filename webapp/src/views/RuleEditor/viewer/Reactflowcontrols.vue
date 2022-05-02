<template>
    <div id="controlContainer" class="react-flow__controls">
        <div class="react-flow__controls-button">
            +
        </div>
        <div class="react-flow__controls-button">
            -
        </div>
        <div class="react-flow__controls-button">
            <b-icon icon="arrows-fullscreen"></b-icon>
        </div>
        <div class="react-flow__controls-button">
            <b-icon icon="camera"></b-icon>
        </div>
        <div class="react-flow__controls-button">
            <b-icon icon="info-circle"></b-icon>
        </div>
    </div>
</template>

<script>
export default {
    name: "Reactflowcontrols",

    data() {
        return {

        }
    },

    methods: {
        /*onZoomIn(value){
            this.$emit('onZoomIn', value);
        }*/
        
    },

    mounted: function () {
        //const { zoomIn, zoomOut, setCenter } = useZoomPanHelper();
        //console.log(useZoomPanHelper());
    },
    
    components: {

    }
}
</script>

<style scoped>
#controlContainer {
    
}
</style>
