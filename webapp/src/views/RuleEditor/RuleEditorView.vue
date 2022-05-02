<template>
    <div>
        <div id="ruleEditor">
            <reactflowTopMenu class="ptree"></reactflowTopMenu>
            <reactflowviewer class="viewer" :currentfile="$store.state.currentfile"></reactflowviewer>
            <modelCheckView></modelCheckView>
        </div>
    </div>
</template>

<script>
import Reactflowviewer from './viewer/Reactflowviewer.vue';
import ReactflowTopMenu from './ReactflowTopMenu.vue';
import ModelCheckView from './RuleSetMenu/ModelCheckView.vue';

export default {
    name: "RuleEditorView",

    props : [ ],

    data() {
        return {
            //filenames: []
        }
    },

    methods: {
        /*
        addItem: function(filename){
            this.filenames.push({
                "Filename" : filename
            });
        }
        */
    },

    mounted() {
        /*
        fetch("http://localhost:3000/entries", {
            method: "GET",
            mode: 'cors',
            headers: { "Content-Type": "application/json" }
        }).then(response => response.json()) //response.json()
        .then(data => {
            data.forEach(element => {
                this.addItem(element);      
            });
        });
        */
    },
    
    components: {
       Reactflowviewer,
       ReactflowTopMenu,
       ModelCheckView
    }
}
</script>

<style scoped>
#ruleEditor {
    display: grid;
    grid-template-columns: 1fr 4fr;
    grid-template-rows: 50px auto;
    grid-template-areas: 
    "ptree ptree"
    "viewer viewer";
    overflow: hidden;

    border-radius: 10px;
    margin: 15px;
    height: calc(100vh - 30px);
    box-shadow: 0 3px 10px rgb(0 0 0 / 0.2);
    border: 1px solid lightgrey;
}

.ptree{
    grid-area: ptree;
    background-color: gainsboro;
}

.viewer{
    grid-area: viewer;
    background-color: white;
}
</style>
