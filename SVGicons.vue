<template>
    <div :class="`svg-icon float-end ${classes}`" :style="{ width: width + 'px', height: height + 'px' }">
        <svg viewBox="0 0 16 16" :style="cssVars">
            <use :href="getSVGIconUrl()+'#icon-'+name"></use>
        </svg>
    </div>
</template>
<script>
export default {
    data(){
        return{
            colorOnHover:'',
        }
    },
    props: {
        name: {
            type: String,
            required: true,
        },
        width:{
            type: Number,
            default:16,
        },
        height:{
            type: Number,
            default:16,
        },
        color:{
            type: String,
            default:'#000000', 
        },
        hoverColor:{
            type: String,
        },
        classes: {
            type: String,
        },
    },
    methods:{
        getSVGIconUrl() {
            return require('./../../assets/icons.svg')
        }
    },
    created(){ 
        this.colorOnHover = this.hoverColor ?  this.hoverColor : this.color;
    },
    computed:{
        cssVars() {
            return {
                '--color': this.color,
                '--hover-color': this.colorOnHover
            }
        },
    }
};
</script>
<style scoped>
    svg{
        fill: var(--color);
    }
    svg:hover{
        fill: var(--hover-color);
    }
</style>
