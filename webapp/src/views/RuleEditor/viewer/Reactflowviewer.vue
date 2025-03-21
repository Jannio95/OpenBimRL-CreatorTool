<template>
    <div id="viewerContainer">
        <reactFlow 
            id="reactflowviewer" 
            style="width: 100%; height: 100%;" 
            :elements="localElements" 
            :nodeTypes="nodeTypes"
            :onMove="onMove"
            :onDragOver="onDragOver"
            :onDrop="onDrop"
            :onEdgeUpdate="onEdgeUpdate"
            :onConnect="onConnect"
            :onSelectionChange="onSelectionChange"
            :onNodeDoubleClick="onNodeDoubleClick"
            :onNodeDragStop="onNodeDragStop"
            :onNodeDrag="onNodeDrag"
            :defaultZoom="$store.state.viewer.currentZoom" 
            :minZoom="$store.state.viewer.minZoom" 
            :maxZoom="$store.state.viewer.maxZoom">

            <!-- vue based self implementation of a react flow control panel -->
            <reactflowcontrols :handleRenderBGGrid="handleRenderBGGrid" :versionInformation="$store.state.version"/>

            <!-- vue based self implementation of a react flow background -->
            <reactflowbackground v-if="$store.state.settings.renderBackground" :panePosition="panePosition" />

        </reactFlow>

        <!-- The modal -->
        <b-modal id="text-input-modal" title="Text Input Modal">
            <b-input-group>
                <template #prepend>
                    <b-input-group-text>Label</b-input-group-text>
                </template>
                <b-form-input v-model="dblClickSelectedNode.data.label"></b-form-input>
            </b-input-group>
        </b-modal>
    </div>
</template>

<script>
import ReactFlow, { ReactFlowProvider, isNode, isEdge, removeElements } from 'react-flow-renderer';
import Reactflowcontrols from './Reactflowcontrols.vue';
import Reactflowbackground from './Reactflowbackground.vue';
import Parser, { createUniqueID } from '../../../core/ParserOpenBIMRL.ts';
import { 
    DEFAULT_NODE_WIDTH,
    //createInputType, 
    //createFunctionType, 
    //createRuleIdentifier, 
    creatFancyFunctionType, 
    createFancyInputType, 
    createFancyRuleIdentifier,
    updateGroup
} from "../../../core/CustomNodeSetup.ts";
import exampleData from "../../../../resources/defaultGraphExample.json";
import xmljs from 'xml-js';

export default {
    name: "Reactflowviewer",

    props : ["currentfile"],

    data() {
        return {
            dblClickSelectedNode : { data: { label: ""}}, //Initially: empty placeholder node for modal rendering
            panePosition: { x: 0, y:0, zoom: 1}, //A placeholder for the current transformation of the grid pane
            nodeTypes: { //Sets the NodeStyles dispayed
                inputType : createFancyInputType, 
                functionType : creatFancyFunctionType, 
                ruleIdentifier : createFancyRuleIdentifier
                //inputType : createInputType,
                //functionType : createFunctionType,
                //ruleIdentifier : createRuleIdentifier
            },
            localElements: []
        }
    },
    methods: {
        onNodeDrag(event, node){
            let storedNode = this.findElement(node.id);
            storedNode.position = node.position;
            
            if(storedNode.parentNode){
                updateGroup(storedNode.parentNode);
                this.notify();
            }
        },
        onNodeDragStop(event, node){
            let storedNode = this.findElement(node.id);
            storedNode.position = node.position;

            if(storedNode.parentNode){
                updateGroup(storedNode.parentNode);
                this.notify();
            }
        },
        /* onMove: updates the position of the pane transformation to update the background rendering. */
        onMove(position){
            this.panePosition = position;
        },
        /* onDragOver: enables the draging effect and drop onto the pane. */
        onDragOver(event){
            event.preventDefault();
            event.dataTransfer.dropEffect = 'move';
        },
        /* onDrop: reads data droped on the pane and registers the node. */
        onDrop(event){
            event.preventDefault();

            let dropData = event.dataTransfer.getData('application/reactflow');

            if(!Array.isArray(dropData) && dropData !== null && dropData !== ""){
                
                //Retrieve the node template 
                const nodeTemplateInstance = JSON.parse(
                    dropData
                );

                //Retrieve pane transformation information
                let zoomObj = event.target.offsetParent.__zoom;
                
                /* Old offsets
                let offsetX = 125 + 25 * 1/zoomObj.k;
                let offsetY = 125 * 1/zoomObj.k + 25;
                */

                let offsetX = DEFAULT_NODE_WIDTH/2;
                let offsetY = 50 * 1/zoomObj.k + 25;

                //Caluclate drop position
                const position = {
                    x: (event.clientX - zoomObj.x) * 1/zoomObj.k - offsetX,
                    y: (event.clientY - zoomObj.y) * 1/zoomObj.k - offsetY
                };

                //Register postion to the node instance and create a new unique node ID
                nodeTemplateInstance.position = position;
                nodeTemplateInstance.id = createUniqueID(); 

                //add node to scene and reload elements
                this.$store.state.modelCheck.elements.set(nodeTemplateInstance.id, nodeTemplateInstance);

                this.localElements = Array.from(
                    this.$store.state.modelCheck.elements.values()
                );
            }
        },
        /* onEdgeUpdate: update and relace edge instance */
        onEdgeUpdate(oldEdge, newConnection){
            //Retrieve the source element and find the label
            let element = this.findElement(newConnection.source);
            let outLabel = element.data.outputs[newConnection.sourceHandle].name;

            oldEdge.id = createUniqueID(); 
            oldEdge.source = newConnection.source;
            oldEdge.target = newConnection.target;
            oldEdge.sourceHandle = newConnection.sourceHandle;
            oldEdge.targetHandle = newConnection.targetHandle;
            //oldEdge.label = outLabel;
        },
        /* onConnect: creates a new edge using the source and target information */
        onConnect(params){
            //Retrieve the source element and find the label
            let element = this.findElement(params.source);
            let outLabel = element.data.outputs[params.sourceHandle].name;

            //Create and register new edge
            let newEdge = { 
                "id": createUniqueID(), 
                "source": params.source, 
                "target": params.target, 
                "sourceHandle":params.sourceHandle,
                "targetHandle":params.targetHandle, 
                //"label": outLabel, 
                "style": {
                    "strokeWidth" : 4
                }
            }
            
            this.$store.state.modelCheck.elements.set(newEdge.id, newEdge);

            this.localElements = Array.from(
                this.$store.state.modelCheck.elements.values()
            );

        },
        /* onNodeDoubleClick: displays a modal input menu for the selected node */
        onNodeDoubleClick(event, node){
            event.preventDefault();
            this.dblClickSelectedNode = node;
            this.$bvModal.show("text-input-modal");
        },
        /* onSelectionChange: saves the selected nodes and sets their satus to selected */
        onSelectionChange(elements) {
            //Deselects current selection first (removes BoxShadow)
            if(this.$store.state.selectedElements){
                for(let index in this.$store.state.selectedElements){
                    if(isNode(this.$store.state.selectedElements[index])){
                        this.$store.state.selectedElements[index].data.selected = false;
                    }
                    if(isEdge(this.$store.state.selectedElements[index])){
                        //TODO - custom deselection style
                    }
                }
            }

            //Enable ongoing selection, its tied to Control keydown event
            if(this.$store.state.settings.multiSelectionMode && elements && this.$store.state.selectedElements){
                this.$store.state.selectedElements = elements.concat(this.$store.state.selectedElements)
            }else{
                this.$store.state.selectedElements = elements;
            }

            //Selects current elements (applies BoxShadow)
            for(let index in this.$store.state.selectedElements){
                if(isNode(this.$store.state.selectedElements[index])){
                    this.$store.state.selectedElements[index].data.selected = true;
                }
                if(isEdge(this.$store.state.selectedElements[index])){
                    //TODO - custom selection style
                }
            }
        },
        /* deleteSelectedElements: deletes the current selection from the scene */
        deleteSelectedElements(){
            let eleArr = Array.from(this.$store.state.modelCheck.elements.values());
            let currentKeys = Array.from(this.$store.state.modelCheck.elements.keys());
            let selection = this.$store.state.selectedElements;

            //Returns an array of elements without the ones from elementsToRemove. 
            //Also removes all incoming/outgoing edges.
            let reducedList = removeElements(
                selection, 
                eleArr
            ); 

            for(let activeElement of reducedList){
                const index = currentKeys.indexOf(activeElement.id);
                if (index > -1) { // only splice array when item is found
                    currentKeys.splice(index, 1); // 2nd parameter means remove one item only
                }
            }
            
            for(let removedElementKey of currentKeys){
                this.$store.state.modelCheck.elements.delete(removedElementKey);
            }
            
            this.localElements = reducedList;
        },
        findElement(id){
            return this.$store.state.modelCheck.elements.get(id);
        },
        handleRenderBGGrid(){
            this.$store.state.settings.renderBackground = !this.$store.state.settings.renderBackground;
        },
        notify(){
            //manuel update to handle reactivity of maps
            this.localElements = Array.from(
                this.$store.state.modelCheck.elements.values()
            );
        }
    },
    mounted: function () {
        //Load sample data when mounted
        if(this.$store.state.modelCheck.elements.size === 0){
            
            this.localElements = exampleData.elements;
            let elements = exampleData.elements;
            this.$store.state.modelCheck.elements = new Map();
            for(let ele of elements){
                this.$store.state.modelCheck.elements.set(ele.id, ele);
            }

            this.$store.state.modelCheck.subChecks = exampleData.subChecks;
            this.$store.state.modelCheck.resultSets = exampleData.resultSets;

            this.$store.state.subChecks = [{
                id: "0",
                name: "Absatz 1: Test Prüfung",
                applicability: [],
                rulesOrRuleSets: [],
                resultSets: []
            }];
        }

        //Local referenz to the current vue instance
        let _self = this;

        //Global key down events to enable node and edge removal
        window.addEventListener('keydown', function(e) {
            if(e.key === "Delete"){
                _self.deleteSelectedElements();
            }
        });
        window.addEventListener('keydown', function(e) {
            if(e.key === "Control"){
                _self.$store.state.settings.multiSelectionMode = true;
            }
        });
        window.addEventListener('keyup', function(e) {
            if(e.key === "Control"){
                _self.$store.state.settings.multiSelectionMode = false;
            }
        });

        //const rootElement = document.getElementById("reactflowviewer");
    },
    watch: {
        // You can also set up a watcher for name here if you like
        currentfile() { 
            // POST request using fetch
            fetch("http://localhost:3000/load", {
                method: "POST",
                mode: 'cors',
                headers: { "Content-Type": "application/json" },
                body: JSON.stringify({ file: this.currentfile })
            }).then(response => response.text()) //.then(response => response.json())
            .then(data => {
                let jsonStrData = xmljs.xml2json(data, {compact: true, spaces: 4});
                let jsonData = JSON.parse(jsonStrData);
                
                let opts = this.$store.state.settings;
                let parser = new Parser();
                let elements = parser.parse(jsonData, opts);

                this.$store.state.modelCheck.elements = new Map();
                for(let ele of elements){
                    this.$store.state.modelCheck.elements.set(ele.id, ele);
                }
            }); //response.json()
        }
    },
    components: {
        ReactFlow,
        Reactflowcontrols,
        Reactflowbackground,
        ReactFlowProvider
    }
}
</script>

<style scoped>
#reactflowviewer {
    
}
</style>
