<template>
    <ul class="category-nest">
        <li v-for="category in categories"  class="category-item" :key="category" @click="hideCategoryPaper">
            <router-link  :to="{ name: 'frontend.product', query: {category : category.slug} }" class="category-menu">
                <span class="text-sm font-medium capitalize">{{ category.name }}</span>
                <i v-if="category.children.length > 0" class="lab-line-chevron-right"></i>
            </router-link>

            <MenuChildrenComponent  v-if="category.children.length > 0" :key="category" :categories="category.children" v-on:hideCategoryPaper="hideCategoryPaper"  />
        </li>
    </ul>


</template>
<script>
export default {
    name: "MenuChildrenComponent",
    props: {
        categories: {
            type: Array,
            required: true
        }
    },
    methods: {
        hideCategoryPaper: function(){
            this.$emit('hideCategoryPaper');
        }
    }
}
</script>