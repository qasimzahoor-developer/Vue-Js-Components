<template>
    <div class="row">
        <div class="col-12">
            <div class="row">
                <div class="col-6 text-left">
                    <div class="float-start m-1 dropdown">
                        <a href="javascript:void(0)" class="nav-link dropdown-toggle" data-bs-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false">
                            <Icons
                                :name="`table`"
                                :classes="`icon-table ms-1`"
                                :color="`#009688`"
                                :width="13"
                                :height="13"
                            />
                        </a>
                        <ul class="dropdown-menu pt-1 pb-1" @click="function(e){ e.stopPropagation() }">
                            <li class="dropdown-item" v-for="column in columns" :key="column.name">
                                <CheckBox 
                                    class="pl-2 pr-2 pt-1 pb-1 d-block" 
                                    :style="`cursor: pointer;`"
                                    onselectstart="return false"
                                    :name="`showcol_`+tblID+`[]`" 
                                    :checked="colshow.includes(column.name)" 
                                    :val="column.name" 
                                    v-model:value="colshow"  
                                    :label="column.label"
                                    :checkbox_events="{change:function(){setLocalStorage('colshow'+tblID, colshow);}}"
                                />
                            </li>
                        </ul>
                    </div>
                    <div class="float-start m-1">
                        <select v-if="options.perPageList.length" @change="getPerPage($event)" class="form-control">
                            <option v-for="(list, index) in options.perPageList" :key="index" :selected="perPage == list">{{ list }}</option>
                        </select>
                        <select v-else  @change="getPerPage($event)" class="form-control">
                            <option v-for="(list, index) in [10,20,50,100,200,'all']" :key="index" :selected="perPage == list">{{ list }}</option>
                        </select>
                    </div>
                </div>
                <div class="col-6 text-right">
                    <div class="row float-end">
                        <label class="control-label col-5">Search: <br>All pages<input type="checkbox" class="m-1" v-model="isFilterPages"></label>
                        <div class="col-7 p-r-0">
                            <input class="form-control" type="text" @keyup="isFilterPages ? filterPages($event) : filterRows($event)">
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <div class="col-12">
            <div :class="attributes.responsive? 'table-responsive' : null"> 
            <table :class="attributes.class">
                <thead> 
                    <tr>
                        <th v-if="options.bulkSelect.status"><input v-model="selectAll" checked="selectAll" type="checkbox"></th>
                        <th v-for="column in cp_columns" :key="column.name" v-html="column.label"  :width="column.width? column.width+`%` : ''" :class="(column.sortable ? (column.name === orderBy? 'sortable '+order : 'sortable') : '') + (column.searchable ? ' searchable' : '')" @click="column.sortable && loading.results ? sort(column.name) : null"></th> 
                        <th v-if="routes.view || routes.edit || routes.delete">Actions</th>
                    </tr> 
                </thead> 
                <tbody>
                    <tr v-if="!loading.results">
                        <td colspan="100%">
                            <Loading 
                                :spinnerType="`grow`"
                            />
                        </td>
                    </tr>
                    <tr v-else v-for="item in data.data" :key="item[keyColumn]" :class="((typeof rowCallback) == 'function' ? ` data-row clickable ` : ` data-row `)+(selectedRow === item[keyColumn] ? ` table-warning ` : ``)" :data-id="item[keyColumn]" v-on="(typeof rowCallback) == 'function' ? {click:rowCallback} : {}"> 
                        <td v-if="options.bulkSelect.status"><input :checked="selectAll" type="checkbox" :name="options.bulkSelect.column" :id="options.bulkSelect.column+`_`+item.id" :value="item[options.bulkSelect.column]" class="select-all"></td>
                        <td v-for="column in cp_columns" :key="column.name+`_`+item.id" :width="column.width? column.width+`%` : ''" :class="column.name" v-html="vHtml(column, item)"></td>
                        <td  v-if="routes.view || routes.edit || routes.delete">
                            <!-- <img :src="'/images/logo.png'" class="img-fluid"/> -->
                            <div :class="routes.groupBtns?'btn-group' : ''" role="group">
                                <router-link v-if="routes.view" :to="{name: routes.view.name, params: { id: item.id }}" class="btn btn-sm btn-info" v-html="routes.view.label"></router-link>
                                <router-link v-if="routes.edit" :to="{name: routes.edit.name, params: { id: item.id }}" class="btn btn-sm btn-primary" v-html="routes.edit.label" ></router-link>
                                <router-link v-for="(route, index) in getButtons()" v-on:click="clickButton(route.click, item.id)" :key="`btn`+index" :to="{}" :class="route.classes" v-html="route.label" ></router-link>
                                <LoadingButton v-for="(route, index) in getButtons(`loader`)" @click="clickButton(route.click, item.id)" :key="`loader_btn`+index" :attributes="{class:route.classes}" :label="route.label" :loading="route.isLoading"/>
                                <LoadingButton v-if="routes.delete" @click="$emit('click', deletePost(item.id))" :key="item.id" :attributes="{class:'btn btn-sm btn-danger'}" :label="routes.delete.label" :loading="loading.delete"/>
                            </div>
                        </td>
                    </tr> 
                    <tr v-if="loading.results && options.paginate">
                        
                        <td colspan="100%"><Pagination :classes="`pagination`" :data="data" :changePage="getResults" /></td>
                    </tr>
                    <tr v-if="loading.results && data.data.length == 0">
                        <td colspan="100%"><div class="alert alert-dismissible alert-warning text-center">There is no data to display!</div></td>
                    </tr>
                </tbody>
            </table>
            </div>
        </div>

    </div>
</template>
<script>
   import LoadingButton from './LoadingButton.vue';
   import Pagination from './Pagination.vue';
   import Loading from './Loading.vue';
   import CheckBox from './CheckBox.vue';
   export default {
        components: {
            LoadingButton,   
            Pagination,
            CheckBox,
            Loading, 
        },
         data() {
            return {
                data: {},
                filter:'',
                current_page: 1,
                perPage: 20,
                order: 'asc',
                orderBy: '',
                selectAll:false,
                searchTerm: '',
                searchAbles:[],
                isFilterPages: false,
                loading:{},
                colshow:[],
                tblID:'',
            }
        },
        props: {
            routes: {
                type: Object,
                required: true,
            },
            columns: {
                type: Array,
                required: true,
            },
            options: {
                type: Object,
            },
            attributes: {
                type: Object,
            },
            ApplyFilter: {
                type: Object,
                default:function(){ return {}; },
            },
            rowCallback:{
                type: Function,
            },
            selectedRow:{},
            keyColumn:{
                type: String,
                default:'id',
            }
        },
        watch: { 
                ApplyFilter: {
                    handler: function() { this.getResults(1); },
                    deep: true,
                }
        },
        created() { 
            this.tblID = this.routes.index.name.replace(/[^a-z0-9-_]/g, "");
            this.perPage = this.options.perPage ? this.options.perPage : this.perPage;
            this.perPage = this.getLocalStorage('perPage'+this.tblID) ? this.getLocalStorage('perPage'+this.tblID) : this.perPage;
            this.order = this.getLocalStorage('order'+this.tblID);
            this.orderBy = this.getLocalStorage('orderBy'+this.tblID);
            this.colshow = this.getLocalStorage('colshow'+this.tblID);
            this.getResults(1); 
            if(this.colshow && (typeof this.colshow) !== Array) this.colshow = this.colshow.split(',');
            if(!this.colshow){
                this.colshow = [];
                this.columns.forEach((key) => { 
                    this.colshow.push(key.name); 
                });
                this.setLocalStorage('colshow'+this.tblID, this.colshow);
            }
        },
        mounted(){ 
            this.columns.forEach((key) => {
                    if(key.searchable) this.searchAbles.push(key.name);
            });
        }, 
        computed: {
            cp_columns:function (){
                return this.columns.filter(i => this.colshow.includes(i.name));
            },
        },
        methods: {
            getResults(page = 1) { 
                if(Object.keys(this.ApplyFilter).length > 0 && !this.ApplyFilter.startFilter) return;
                Reflect.set(this.loading, `results`, false);
                this.current_page = page;
                this.order = (this.order)? this.order : 'asc';
                this.orderBy = (this.orderBy) ? this.orderBy : this.columns[0].name;
                var doSearch = this.searchTerm ? '&search='+this.searchTerm : '';
                var searchColumns = (this.searchAbles.length) ? '&search_in=' + this.searchAbles.toString() :'';
                const applyFilters = this.ApplyFilter;
                // var setFilters = (Object.keys(applyFilters).length > 0) ?  ('&filters='+Object.keys(applyFilters).map(key => `${key}=${applyFilters[key] ? applyFilters[key].replace('&','amp;') : applyFilters[key]}`).join("|*|")) : '';
                delete applyFilters.startFilter;
                var setFilters = (Object.keys(applyFilters).length > 0) ?  ('&filters='+Object.keys(applyFilters).map(key => `${key}=${applyFilters[key] ? applyFilters[key] : applyFilters[key]}`).join("|*|")) : '';
                this.axios
                    .get(`${this.routes.index.name}?page=${page}&order=${this.order}&orderBy=${this.orderBy}&perPage=${this.perPage}${doSearch}${searchColumns}${setFilters}`)
                    .then(response => {
                        if(response.data.success == true){ 
                            this.data = response.data.data;
                            this.filter = this.data.data;
                            Reflect.set(this.loading, `results`, response.data.success);
                        }else{
                            this.$swal('', response.data.message + ' <br> Please try again later', 'error');
                        }
                    }).catch((error) =>{ 
                        Reflect.set(this.loading, `results`, false);
                        console.log('erros : ',error); 
                        this.$swal('', error.message + ' <br> Please try again later', 'error');
                    });
            },
            getPerPage:function(event){ event;
                this.perPage = event.target.value;
                this.setLocalStorage('perPage'+this.tblID, this.perPage);
                this.getResults();
            },
            sort:function(s) {
                if(s === this.orderBy) {
                    this.order = (this.order==='asc')?'desc':'asc';
                }
                this.orderBy = s;
                this.setLocalStorage('order'+this.tblID, this.order);
                this.setLocalStorage('orderBy'+this.tblID, this.orderBy);
                this.getResults(this.current_page);
            }, 
            filterPages(event) {  
                this.searchTerm = event.target.value;
                this.getResults();
            },
            filterRows(event) {  
                this.searchTerm = event.target.value;
                this.data.data =  this.filter.filter(row => {
                    var match = [];
                    this.searchAbles.forEach((searchColumn) => { //console.log(row[searchColumn]); return;
                    if(!row[searchColumn]) console.log(searchColumn);
                        searchColumn = row[searchColumn].toString().toLowerCase();
                        match.push(searchColumn.includes(this.searchTerm));
                    });
                return match.includes(true);
                });
            },
            vHtml:function(column, item) {
                return column.formatter ? column.formatter(item[column.name]) : item[column.name];
            },
            getButtons(loader = null){ 
                const addLoader = (loader === 'loader') ? true : false; 
                const obj = this.routes.buttons;
                const LoaderBtns = [];
                const Btns = [];
                for(const btn in obj){ 
                    if(obj[btn].isLoader){ 
                        LoaderBtns.push(obj[btn]);
                    }
                    if(!obj[btn].isLoader){
                        Btns.push(obj[btn]);
                    }
                }
                if(addLoader) return LoaderBtns;
                if(!addLoader) return Btns;
            },
            clickButton(fn, id){
                if(fn) fn(id);
            },
            deletePost(id) { 
                if(!this.routes.delete) return;
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
                        let delete_route = this.$router.resolve({name: this.routes.delete.name, params: { id: id }}).href;
                        this.axios
                            .delete(delete_route)
                            .then(response => { response;
                                let i = this.data.data.map(item => item.id).indexOf(id);
                                this.data.data.splice(i, 1);
                                if(this.data.data.length == 0 && this.current_page > 1) this.current_page = this.current_page-1;
                                console.log(this.current_page);
                                this.getResults(this.current_page);
                                window.notify({ group: 'notices', type: 'success', title: 'Record Deleted', text: `Record #${id} was deleted!`, duration:2, });
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
<style scoped>
    .sortable{
        cursor: pointer;
        position: relative;
    }
    .sortable:before{
        right: 1em;
        content: "\2193";
    }
    .sortable:after{
        right: 0.5em;
        content: "\2191";
    }
    .sortable:before, .sortable:after{
        position: absolute;
        bottom: 0.9em;
        display: block;
        opacity: 0.3;
    }
    .sortable.asc:before{
        opacity: 1;
    }
    .sortable.desc:after{
        opacity: 1;
    }
    .clickable{
        cursor: pointer;
    }
</style>