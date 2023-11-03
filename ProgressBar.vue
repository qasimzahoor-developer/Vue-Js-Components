<template>
        <div class="progress">
            <div 
                :class="`progress-bar complete `+classes" 
                role="progressbar" 
                :style="`width: `+complete+`%;`" 
                :aria-valuenow="complete" 
                aria-valuemin="0" 
                aria-valuemax="100"
            >
                {{complete}}% {{ fillLabel }}
            </div>
            <div 
                class="progress-bar progressbar-nobg" 
                role="progressbar" 
                :style="`width: `+unfilled+`%;`" 
                :aria-valuenow="unfilled" 
                aria-valuemin="0" 
                aria-valuemax="100"
            >
                {{unfilled}}% {{ unfillLabel }}
            </div>
        </div>
</template>
<script>
export default ({
    data(){
        return { 
            unfilled:0,
        }
    },
    props: {
        showLabel: {
                type: Boolean,
        },
        fillLabel: {
                type: String,
                default:'%',
        },
        unfillLabel: {
                type: String,
                default:'%',
        },
        complete: {
                type: Number,
                validator:function (value) {
                    return (value >= 0 && value <= 100);
                },
        },
        classes: {
                type: String,
        },
    },
    mounted(){
       this.unfilled =  100 - this.complete;
    },
    methods:{
        
    },
})
</script>
<style>
    .progressbar-nobg{ background: none; color:#000}
</style>
