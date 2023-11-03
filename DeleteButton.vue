<template>
        <LoadingButton 
            :attributes="attributes" 
            :label="route.label" 
            :loading="loading.delete"
            :Loadinglabel="route.loadinglabel"
            @click="deleteRecord(route.id)"
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
                loading:{},
                dData:{},
             }
        },
        props: {
           route:{
                type: Object,
                required:true,
                validator: function (obj) { return 'name' in obj && 'id' in obj && 'label' in obj && 'loadinglabel' in obj}
           },
           data:{
                type: Object,
                required:false,
                // default: () => { return {data:[{id:this.route.id}], page:0}},
           },
           attributes:{
                type: Object,
           },
           success:{

           },

        },
        created(){
            this.dData = this.data
        },
        mounted(){
          if(!this.dData) this.dData = {data:[{id:this.route.id}], page:0}
        },
        methods: {
           deleteRecord(id) { 
                if(!this.dData.data.map(item => item.id).includes(id)){
                   this.$swal('', 'Record missing in passed data!', 'error');
                   return; 
                } 
                this.$swal({
                    title: 'Are you sure?',
                    text: 'You can\'t revert your action',
                    type: 'warning',
                    icon: 'warning',
                    showCancelButton: true,
                    confirmButtonText: 'Yes Delete it!',
                    cancelButtonText: 'No, Keep it!',
                    showCloseButton: true,
                    showLoaderOnConfirm: true
                }).then((result) => {
                    if(result.value) { 
                        Reflect.set(this.loading, `delete`, true);
                        let delete_route = this.$router.resolve({name: this.route.name, params: { id: id }}).href;
                        this.axios
                            .delete(delete_route)
                            .then(response => { response;
                                if(this.dData.data.map(item => item.id).includes(id)){
                                    let i = this.dData.data.map(item => item.id).indexOf(id);
                                    this.dData.data.splice(i, 1);
                                }
                                this.success(id);
                                if(this.dData.data.length == 0 && this.page > 1) this.page = this.page-1;
                                window.notify({ group: 'notices', type: 'success', title: 'Deleted', text: 'Record was deleted!' });
                                Reflect.set(this.loading, `delete`, false);
                            }).catch((error) =>{ 
                            Reflect.set(this.loading, `delete`, false);
                            console.log('erros : ',error); 
                            this.$swal('', error.message + ' <br> Please try again later', 'error');
                        });
                    }
                })
            },
            
        },
    
   }
</script>