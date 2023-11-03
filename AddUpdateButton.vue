 <template>
        <LoadingButton 
            :attributes="attributes" 
            :label="route.label" 
            :loading="loading"
            :Loadinglabel="Loadinglabel"
            :icon="icon"
            @click="postData()"
        />
</template>
<script>
   import LoadingButton from './LoadingButton.vue';
   export default {
        components: {
            LoadingButton,
        },
        data() {
            return { 
                msgLabel:'',
                loading:false,
                data_data:'',
             }
        },
        props: {
           route:{
                type: Object,
                required:true,
                validator: function (obj) { return 'name' in obj && 'id' in obj && 'label' in obj && 'update' in obj }
           },
           data:{
                type: [Object,FormData],
                required:true,
           },
           attributes:{
                type: Object,
           },
           Loadinglabel: {
                type: String,
                default: 'Wait...',
            },
            icon: {
                
            },
           headers:{
               type: Object,
           }, 
           formId: {
               type: String,
            },
           beforePost:{},
           success:{},
           afterPost:{}

        },
        created(){
            this.msgLabel = this.route.update ? 'Updated' : 'Saved';
        },
        methods: {
           postData() { 
               if(this.formId && document.getElementById(this.formId)){
                   var form = document.getElementById(this.formId);
                   this.data_data = new FormData(form);
               }else{
                   this.data_data = this.data;
               }
                this.loading = true;
                if(this.beforePost) this.beforePost(this.route.id);
                const url = this.route.update ? this.$router.resolve({name:this.route.name, params: {id:this.route.id}}) : this.$router.resolve({name:this.route.name});
                this.axios
                    .post(url.href, this.data_data, { header: this.headers }) 
                    .then((response) => {
                        if(response.data.success == true){
                            if(this.success) this.success(response.data);
                            window.notify({ group: 'notices', type: 'success', title: 'Success', text: `Record ${this.msgLabel} successfully!`, duration:5 });
                            this.loading=false;
                        }else{
                            if(this.afterPost) this.afterPost(this.route.id);
                            this.loading=false;
                            this.$swal('', response.data.message + ' <br> Please try again later', 'error');
                        }
                    }).catch((error) =>{ 
                        if(this.afterPost) this.afterPost(this.route.id);
                        this.loading=false;
                        if(error.response.data.errors && this.$parent.errors) this.$parent.errors = error.response.data.errors;
                        this.$swal('', error.message + ' <br> Please try again later', 'error');
                    });
            },

        },
    
   }
</script>