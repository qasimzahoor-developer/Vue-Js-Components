<template>
    <div :class="containerClasses" :style="`height:${height}px; margin-top:${marginTop}px;`">
        <div class="loading-container">
            <div :class="'spinner-'+spinnerType+' '+spinnerClasses" role="status"></div>
            <p :class="loadingLabelClasses">{{loadingLabel}}</p>
        </div>
    </div>
</template>
<script>
export default {
    data(){
        return{
            height:null,
            marginTop:0,

        }
    },
    props:{
            containerClasses:{
                type:String,
                default:'text-center'
             },
             spinnerType:{
                type:String,
                default:'grow',
             },
             spinnerClasses:{
                type:String,
             },
             loadingLabel:{
                type:String,
                default:'Loading...'
             },
             loadingLabelClasses:{
                type:String,
             },
    },
    mounted(){
        this.height = this.$parent.$el.offsetHeight;
        this.marginTop = (this.$el.firstChild.offsetHeight/2)+(this.height/2)
    },
}
</script>
