<template>
    <nav v-if="data.data && data.last_page > 1" aria-label="Page navigation"> 
        <ul :class="classes">
            <li v-if="data.current_page!==1" class="page-item" @click.prevent="changePage(1)">
                <a class="page-link" href="#" aria-label="Previous">
                    <span aria-hidden="true">&laquo;</span>
                </a>
            </li>

            <li v-for="page in data.last_page" :key="page" @click.prevent="page!==data.current_page?changePage(page) : null" :class="`page-item`+(page===data.current_page? ' active' : '')"><a class="page-link" href="#">{{page}}</a></li>

            <li v-if="data.last_page!==data.current_page" class="page-item"  @click.prevent="changePage(data.last_page)">
                <a class="page-link" href="#" aria-label="Next">
                    <span aria-hidden="true">&raquo;</span>
                </a>
            </li>
        </ul>
    </nav>
</template>
<script>
    export default {
        props:{
            classes:{

            },
            data:{
                type: Object,
            },
            changePage:{
                type: Function,
            }
        },
        created(){
            
        },
        methods:{

        }
    }
</script>
