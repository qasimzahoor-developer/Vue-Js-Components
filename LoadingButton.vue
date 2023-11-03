<template>
    <button 
        :data-label="`loadingbutton`"
        v-on="$attrs"
        @click="loadingStatus"
        v-bind="attributes"
        :disabled="loading && loadingMe" v-html="loading && loadingMe? `<span class='spinner-border spinner-border-sm'></span>`+Loadinglabel : (icon ? `<i class='fa ${icon}'></i>` : '') +label"
    >
    </button>
</template>
<script>
   export default {
        data(){
            return { 
                loadingMe: false,
             }
        },
        props: {
            label: {
                type: String,
                required: true,
            },
            Loadinglabel: {
                type: String,
                default: 'Loading...',
            },
            icon: {
                type: String,
            },
            loading: {
                type: Boolean,
                default: false,
                required: true,
            },
            attributes: {
                type: Object,
            },

        },
        methods: {
            loadingStatus() {
                const groupEls = document.querySelectorAll(`[data-label="loadingbutton"]`); 
                groupEls.forEach((el)=>{                                                                                                          
                    if(!el.__vueParentComponent.props.loading) el.__vueParentComponent.data.loadingMe = false;
                });
                this.loadingMe = true;
            }
        }
   }
</script>