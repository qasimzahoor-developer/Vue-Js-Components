<template>
    <div :class="(design===`switch`? `form-check form-switch `+uuid : `switch-flip`) + ` ` + size">
            <label :for="uuid">
            <input
            :type="type"
            :ref="name"
            :name="name"
            v-bind:value="val"
            v-on:input="updateValue($event)"
            :checked="checked"
            :disabled="disabled"
            v-on="checkbox_events"
            class="form-check-input"
            :id="uuid"
            />
            <span v-if="design===`switch`" class="button-indecator">{{ label }}</span>
            <span v-if="design===`switch-flip`" class="flip-indecator" data-toggle-on="ON" data-toggle-off="OFF"></span>
            <span v-if="design===`switch-flip`" class="flip-label">{{ label }}</span>
        </label>
    </div>
</template> 
<script>
   export default {
        data() {
            return { 
                output:[],
                uuid:this._.uid+this.randomString(4),
             }
        },
        props: {
            label: {
                type: String,
            },
            name: {
                type: String,
                required: true,
            },
            val: {
                type: [String,Number,Boolean],
                default: 'on',
            },
            type:{
              type: String,
              default: 'checkbox',  
            },
            checked:{
                type: Boolean,
            },
            disabled:{
               type: Boolean,
            },
            design:{
              type: String,
              default: 'switch',  
            },
            size:{
               type: String,
               default: '', 
            },
            checkbox_events:{
                type:Object,
            },

        },
        mounted(){
            this.updateValue(this);
        },
        methods: {
            updateValue: function ($event) {  
                if(!$event.target){
                    $event.target = this;
                }
                var group = $event.target.name;
                var isSingle = group.match(/\[[^\]]*]/g) !== null ? false : true;
                this.output = isSingle ? '' : [];
                if(!isSingle){
                    const groupEls = document.getElementsByName(group);
                    groupEls.forEach((el)=>{
                        if(el.checked){ 
                            this.output.push(el.value); 
                        }else{ 
                            this.output.filter(item => item !== el.value);
                        }
                    }); 
                }else{
                    this.output = $event.target.checked ? $event.target.value : '';
                }
                this.$emit('update:value', this.output);
                 
            }
        },
    
   }
</script>