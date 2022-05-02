<template>
    <div>
        
        <router-view/>

        <!-- Graph editor should be accessed via routing -->
        <!-- 
        <projectOverview />
        <ruleEditor />
        -->
    </div>
</template>

<script>
import RuleEditorView from './RuleEditor/RuleEditorView.vue';

export default {
    name: "Mainframe",

    props : [ ],

    methods: {
        pageLoad(event){
            console.log(event);
        }
    },
    
    components: {
        RuleEditorView
    }
}
</script>

<style scoped>

</style>
