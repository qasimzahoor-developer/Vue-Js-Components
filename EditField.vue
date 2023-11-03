<template>
    <div :tabindex="editable? 0 : ''" :class="`editField d-inline `+ (editable ? `editable`: `` )" v-on="editable ? {dblclick : showEdit, keyup:onEnterShowEdit } : {}">
        <div class="editInput d-inline"> 
            <Icons
                v-if="(editable && !isEditing && !autoload && !axiosCall)"
                :name="`edit`"
                :classes="`icon-edit`"
                :width="12"
                :height="12"
                role="button"
             />
            <slot v-if="axiosCall && type!=='text-input'">
                <div class="spinner-border spinner-border-sm" role="status"></div>
            </slot>
            <slot v-if="(!axiosCall && type == 'text') || type=='text-input'">
                <label v-show="!isEditing && !autoload">{{val !='' ? val : '---'}}</label>
                <input v-if="editable || type==='text-input'" :placeholder="placeHolder" v-show="isEditing || autoload" :id="`editField`+$.uid" :class="inputClass +' '+(isEditing ? `d-block` : ``)" type="text" v-on:input="isModified = true" v-on:keyup="updateEdit" v-model="val" :tabindexe="$.uid">
            </slot>
            <slot v-if="!axiosCall && type == 'textarea'">
                <label v-show="!isEditing">{{val !='' ? val : '---'}}</label>
                <textarea  v-if="editable"  :placeholder="placeHolder" v-show="isEditing" :id="`editField`+$.uid" :class="inputClass +' '+(isEditing ? `d-block` : ``)" type="text" v-on:input="isModified = true" v-on:keyup="updateEdit" v-model="val" :tabindexe="$.uid"></textarea>
            </slot>
            <slot v-if="!axiosCall && type == 'select'">
                <label v-show="!isEditing">{{options[val] != '' && hasval() ? options[val]: '---'}}</label>
                <select  v-if="editable" v-show="isEditing" :id="`editField`+$.uid" :class="inputClass" v-on:input="isModified = true" v-on:keyup="updateEdit" v-model="val" :tabindexe="$.uid">
                    <option v-for="(label, key) in options" :key="key" :value="key" :selected="val == key">{{label}}</option>
                </select>
            </slot>
            <slot v-if="!axiosCall && type == 'radio'">
                <label v-show="!isEditing">{{options[val] != '' ? options[val] : '---'}}</label>
                <label v-show="isEditing"  v-for="(label, key, index) in options" :key="key" :class="(isEditing ? `d-block` : ``)" :for="`editField`+$.uid+index">
                    <input  v-if="editable" :placeholder="placeHolder" :id="`editField`+$.uid+index" :class="inputClass +' '+(isEditing ? `` : ``)" type="radio" v-on:input="isModified = true" v-on:keyup="updateEdit" :value="key" v-model="val" :tabindexe="$.uid+index" :checked="val == key"> 
                    {{label}}
                </label>
            </slot>
            <slot v-if="!axiosCall && type == 'checkbox'">
                <label v-show="!isEditing">
                    <span v-for="selectedVal in val" :key="selectedVal">{{options[selectedVal]}}</span>
                    <span v-if="val.length == 0">--</span>
                </label>
                <label v-show="isEditing"  v-for="(label, key, index) in options" :key="key" :class="(isEditing ? `d-block` : ``)" :for="`editField`+$.uid+index">
                    <input  v-if="editable" :placeholder="placeHolder" :id="`editField`+$.uid+index" :class="inputClass +' '+(isEditing ? `d-block` : ``)" type="checkbox" v-on:input="isModified = true" v-on:keyup="updateEdit" :value="key" v-model="val" :tabindexe="$.uid+index" :checked="val.includes(key)"> 
                    {{label}}
                </label>
            </slot>
            <slot v-if="!axiosCall && type == 'multiselect'">
                <label v-show="!isEditing">
                    <span v-for="(selectedVal, index) in val" :key="selectedVal">{{options[selectedVal]}} <i v-if="Object.keys(options).length > 0 && index < val.length-1">,</i></span>
                    <span v-if="val.length == 0 || !hasval()">---</span>
                </label>
                <select  v-if="editable" multiple v-show="isEditing" :id="`editField`+$.uid" :class="inputClass +' '+(isEditing ? `d-block` : ``)" v-on:input="isModified = true" v-on:keyup="updateEdit" v-model="val" :tabindexe="$.uid">
                    <option v-for="(label, key) in options" :key="key" :value="key" :selected="val == key">{{label}}</option>
                </select>
            </slot>
            <slot v-if="!axiosCall && type == 'editor'">
                <label v-show="!isEditing && !autoload" v-html="val !='' ? val : '---'"></label>
                <div  v-if="editable && isEditing" :class="inputClass +' editor '+(isEditing ? `d-block` : ``)">
                    <Icons
                        v-if="(editable && isEditing && !autoload && !axiosCall)"
                        :name="`save`"
                        :classes="`icon-save`"
                        :width="12"
                        :height="22"
                        role="button"
                    />
                    <ckeditor :id="`editField`+$.uid" v-on:input="isModified = true" v-on:keyup="updateEdit" v-model="val" :editor="copyrightNoteEditor" :config="editorConfig"></ckeditor>
                    <span v-if="characterCounter">
                        Character count: {{characterOnlyCount(val).length}}
                    </span>
                    <span v-if="characterMin && characterOnlyCount(val).length < characterMin" :class="`text-danger ps-1`">
                        Min character limit: {{characterMin}}
                    </span>
                    <span v-if="characterMax && characterOnlyCount(val).length > characterMax" :class="`text-danger ps-1`">
                        Max character limit: {{characterMax}}
                    </span>
                </div>
            </slot>
        </div>
        <ul v-if="autoLoadShow && autoload" class="dropdown-menu show" aria-labelledby="dropdownMenuLink">
            <li v-for="(aloitem, index) in autoloadOptions" :key="aloitem" @click="setAutoLoad(index,aloitem)"><span class="dropdown-item" v-html="aloitem.replace(new RegExp( '(' + val + ')', 'gi' ), '<strong>$1</strong>')"></span></li>
        </ul>
    </div>
</template>
<script> 
    import CKEditor from '@ckeditor/ckeditor5-vue';
    import ClassicEditor from '@ckeditor/ckeditor5-build-classic';

export default ({
    components: {
            ckeditor: CKEditor.component,
        },
    data(){
        return { 
            isEditing: false,
            isModified: false,
            val:'',
            oval:'',
            autoloadOptions: {},
            autoLoadShow:false,
            readyToLoad: false,
            axiosCall:false,
            copyrightNoteEditor: ClassicEditor,
            editorConfig: this.$ckConfig
        }
    },
    props: {
        type: {
            type: String,
            default: 'text',
            required: true,
            validator: (value) => ['multiselect',  'checkbox',  'radio',  'select', 'textarea', 'text', 'text-input'].includes(value)
        },
        value: {
            default: '',
        },
        placeHolder:{
            type: String,
        },
        record: {
            default: false,
        },
        options: {
                type: Object,
                default: function(){ return {}; }, 
        },
        editable:{
            type: Boolean,
            default: true,
        },
        autoload: { 
            type: Boolean,
            default: false,
        },
        characterCounter: { 
            type: Boolean,
            default: false,
        },
        characterMin: { 
            type: Number,
        },
        characterMax: { 
            type: Number,
        },
        autoLoadUrl: {
            type: String,
            default: 'custom/autosearch',
        },
        autoSearchLimit:{
            type:Number,
            default: 1,
        },
        inputClass:{
            type: String,
        },
        axiosStatus:{
            type: Boolean,
            default: false,
        }
    },
    created(){ 
        if(!this.value){
            this.val = ['checkbox', 'multiselect'].includes(this.type) ? [] : '';
        }else
        if(['checkbox', 'multiselect'].includes(this.type) && !Array.isArray(this.value)){
          throw new TypeError('Value must be given array for multiselect type!');   
        }else
        if(this.value){
            this.val = this.value;
        }
        if(this.autoload && this.autoLoadUrl === ''){
          throw new TypeError('autoLoadUrl prop is missing!');
        }   
        this.axiosCall = this.axiosStatus;     
    },
    mounted(){
       document.addEventListener('click',this.updateEdit);
       document.addEventListener("keydown", this.updateEdit);
    },
    watch: { 
        value: function() { this.val = this.oval = this.value; },
        axiosStatus: function() { this.axiosCall = this.axiosStatus; },
        val: {  
            handler: function() { console.log(this.val); this.autoLoadCall(); },
            deep: true,
        }
    },
    methods:{
       hasval(){
           let r = false;
           if(Array.isArray(this.val)){ 
                const options = this.options;
                this.val.forEach(function(sval){
                    if(Object.keys(options).includes(sval)) r = true;
                });
                return r;
           }else{ 
                if(Object.keys(this.options).includes(this.val.toString())) return true;
           }
           return r;
       },
       onEnterShowEdit(e){
            if(!["Enter"].includes(e.key)) return;
            this.showEdit(e);
       },
       showEdit(e){ 
           const et = e.currentTarget.localName === 'a' ? e.currentTarget.parentElement : e.currentTarget;
           this.isEditing = true;  
        //    document.querySelector('.ck.ck-content').addEventListener("keydown", this.updateEdit);
           window.setTimeout(function (et, type){ 
               if(['checkbox', 'radio', 'text'].includes(type)) type = 'input[type="'+type+'"]';
               if(['multiselect'].includes(type)) type = 'select';
               if(et.querySelector(type)) et.querySelector(type).focus();
           }, 0 , et , this.type);
       },
       autoLoadCall(){ 
           this.$emit('update:value', this.val);
           if(!this.readyToLoad || this.val.length === 0){
               this.readyToLoad = true;
               return;
           }
           if(!this.autoload || this.type !== 'text-input') return;
           if(this.autoSearchLimit > (this.val.length)){
               this.hideAutoLoad();
               return;
           }
           this.axios
                    .get(this.autoLoadUrl+'?search='+this.val+'&record='+this.record)
                    .then(response => {
                        if(response.data.success == true){ 
                            this.autoloadOptions = (Array.isArray(response.data.data) && response.data.data.length > 0) ? response.data.data : {nodata:'No data found!'};
                            this.autoLoadShow = true;
                        }else{
                            this.$swal('', response.data.message + ' <br> Please try again later', 'error');
                        }
                    }).catch((error) =>{ 
                        Reflect.set(this.loading, `results`, false);
                        console.log('erros : ',error); 
                        this.$swal('', error.message + ' <br> Please try again later', 'error');
                    });
       },
       setAutoLoad(index,item){
           this.autoLoadShow = false;
           if(index !== 'nodata'){
               this.val = item;
               this.readyToLoad = false;
               this.autoloadOptions = {};
            }
       },
       hideAutoLoad(){
         this.autoLoadShow = false;
       },
       characterOnlyCount(v){
          return this.decodeEntities(v).replace(/(<([^>]+)>)/gi, "").replace(/\s/g, "").replace(/ +?/g, "");
       },
       decodeEntities(encodedString) {
            var translate_re = /&(nbsp|amp|quot|lt|gt);/g;
            var translate = {
                "nbsp":" ",
                "amp" : "&",
                "quot": "\"",
                "lt"  : "<",
                "gt"  : ">"
            };
            return encodedString.replace(translate_re, function(match, entity) {
                return translate[entity];
            }).replace(/&#(\d+);/gi, function(match, numStr) {
                var num = parseInt(numStr, 10);
                return String.fromCharCode(num);
            });
        },
       updateEdit(e){
           if ((window.navigator.platform.match("Mac") ? e.metaKey : e.ctrlKey)  && e.keyCode == 83) {
              e.preventDefault();
              if(e.target.__vnode && e.target.__vnode.scopeId != this.$.type.__scopeId) return;  
            }
          else{ 
              if (['keyup', 'keydown'].includes(e.type) && ["Escape"].includes(e.key)){ this.val = this.oval; this.isModified = false; this.isEditing = false; return; }
              if (['keyup', 'keydown'].includes(e.type) && !["Enter", "Escape"].includes(e.key)) return;
              if(e.target.__vnode && e.target.__vnode.scopeId === this.$.type.__scopeId && !["Enter", "Escape"].includes(e.key)) return;
              if(e.target.__vnode && e.target.__vnode.scopeId === this.$.type.__scopeId && !["Enter", "Escape"].includes(e.key)) return;
              if(e.currentTarget.activeElement.classList[0] === 'ck' && !["Escape"].includes(e.key) || e.target.classList[0]){ return; }
          }
          this.hideAutoLoad();
          if(!this.editable) return;
          this.isEditing = false;
          if(!this.isModified){ this.isModified = false; return; }
          this.isModified = false;
          this.axiosCall = true;
          this.axios
            .post(`custom/postfield`, {record:this.record, value:this.val })
            .then((response) => {
                if(response.data.success == true){ 
                    window.notify({ group: 'notices', type: 'success', title: 'Success', text: 'Record added/updated successfully!' });
                }else{
 
                    this.$swal('', response.data.message + ' <br> Please try again later', 'error');
                }
                this.axiosCall = false;
            })
            .catch((error) =>{ 
                if(error.response && error.response.data.errors) this.errors = error.response.data.errors;
                this.$swal('', error.message + ' <br> Please try again later', 'error');
                this.axiosCall = false;
            });
       },
    },
})
</script>
<style scoped>
    .editField{
        position: relative;
    }
    .editInput{
        position: relative;
        z-index: 1;
    }
    .dropdown-menu.show{
        top:inherit;
        margin-top: 0px;
        border-radius: 0;
        z-index: 2;
    }
    .dropdown-item{
        cursor: pointer;
    }
    .editor{
        position: relative;
        clear: both;
    }
    .icon-save{
        cursor: pointer;
        position: absolute;
        right: 14px;
        z-index: 100;
    }
</style>