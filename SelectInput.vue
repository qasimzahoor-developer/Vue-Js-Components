<template>
    <div class="selectInput">
        <label> {{label}} </label>
        <select v-model="selection" :required="required" :multiple="Array.isArray(getSelected)">
            <option v-if="placeHolder" :selected="getSelected.length" v-text="placeHolder"></option>
            <option 
                v-for="(option, index) in options" :key="index" 
                :selected="getSelected.includes(option.value)"
                :value="option.value"
                >
                    {{option.label}}
                </option>
        </select>
        <div class="sb">
            <div 
                :class="displayOptions? 'siOpen form-control' :'form-control'"
                @click="togleDisplay"
                v-html="selectedLabel"
            >
            </div>
            <ul v-if="displayOptions" class="list-group">
                <li class="list-group-item list-group-item-action" v-if="placeHolder" :selected="selection.length == 0" @click="selectOption" v-text="placeHolder"></li>
                <li
                    class="list-group-item list-group-item-action"
                    v-for="(option, index) in options" :key="index" 
                    :value="option.value" 
                    :selected="selection.includes(option.value)"
                    @click="selectOption"
                >
                    {{option.label}}
                </li>
            </ul>
        </div>
    </div>
</template>
<script>
   export default {
        data() {
            return { 
              displayOptions: false,  
              selection: [],
              getSelected: '',
              selectedLabel:'',
             }
        },
        props: {
            label: {
                type: String,
                default: 'Select',
                required: true,
            },
            placeHolder: {
                type: String,
                default: ' -- Select --',
            },
            options: {
                type: Array,
                default: function(){ return []; },
                required: true,
            },
            selected: {
                type: [Array,String],
                required: true,
            },
            required: {
                type: Boolean,
            },

        },
        created() {
            this.getSelected = this.selected;
        },
        mounted() {
            this.selection = this.getSelected;
            this.getSelection(this.selection);
            document.addEventListener('click', this.close)
        },
        methods: {
            togleDisplay() {
                this.displayOptions = this.displayOptions? false : true;
            },
            selectOption(option) {
                const value = option.target.getAttribute('value');
                if(Array.isArray(this.getSelected)){
                    if(!this.selection.includes(value)){
                        this.selection.push(value);
                    }else{
                        this.selection = this.selection.filter(function(val){ if(val !== value) return val; });
                    }
                }else{
                    this.selection = value;
                }
                if(null === value){
                    this.selection = [];
                }
                this.displayOptions = this.displayOptions? false : true;
                this.getSelection(this.selection);  
            },
            getSelection(sOpts) {
                const sOpt = this.options.find(o => sOpts.includes(o.value));
                if(typeof sOpt !== 'undefined'){ 
                    this.selectedLabel = '';
                    if(Array.isArray(sOpts)){
                        this.options.forEach((option) => {
                            if(sOpts.includes(option.value)){
                                this.selectedLabel = this.selectedLabel + ' <div value="'+option.value+'" class="badge badge-primary">' +option.label+ ' <i class="unselect">×</i></div>';
                            }
                        });
                    }else{
                        this.selectedLabel =  '<span>' + sOpt.label+ '</span>';                   
                    }
                }
                else{
                    this.selectedLabel =  '<span>' + this.placeHolder+ '</span>';
                }
                // let spans = document.querySelectorAll('i.unselect');
                // spans.forEach(el => {el.addEventListener('click', this.selectOption);})
                // console.log(spans);
            },
            close(e) {
                if (!this.$el.contains(e.target)) {
                    this.displayOptions = false
                }
            }
    }
}
</script> 
<style>
    .selectInput{ display: inline-block; position: relative;  }
    .selectInput > select{ display: none; }
    .selectInput .siOpen{ border-color: #00635a; }
    .selectInput .badge{ background-color: #e8e8e8; border: 1px solid #ddd; color: #222; font-size: 14px; font-weight: normal; padding: 4px 5px; }
    .selectInput .badge .unselect{ cursor: pointer; display: none; }
    .selectInput .list-group > li[selected="selected"]{ background: #009688; color:#fff; position: relative; }
    .selectInput .list-group > li:not(:first-child)[selected="selected"]:after{ content: "\f00c";color: #fff;font-family: FontAwesome;font-size: 14px;position: absolute;right: 5px; }
</style> 