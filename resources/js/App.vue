<template>
    <Header />
    <component :is="layout">
        <router-view></router-view>
    </component>
    <Footer />
</template>

<script>
import { mapActions } from 'vuex'
import Header from "@/src/layout/Header.vue";
import Footer from "@/src/layout/Footer.vue";
import Default from "@/src/layout/Default.vue";
import FullWidth from "@/src/layout/FullWidth.vue";
import emitter from '@/EventBus.js'

export default {
    name: "App",
    components: {
        Header,
        Footer,
        Default,
        FullWidth
    },
    data() {
        return {
            layout: null,
        };
    },

    created(){
        this.getRecipeItemCategories()
        this.getSpecialtiesCategories()
        this.checkAuth()
        emitter.on('set-title', (title) => document.title = title)
    },

    methods:{
        ...mapActions('global', [
            'getRecipeItemCategories',
            'getSpecialtiesCategories'
        ]),
        ...mapActions('auth', [
            'checkAuth'
        ])
    },

    watch: {
        $route(to) {
            document.title = to.meta?.title || 'Yükleniyor..'

            if (to.meta.layout !== undefined && to.meta.layout.length > 0) {
                this.layout = to.meta.layout;
            } else {
                this.layout = "Default";
            }
        },
    },
}
</script>

<style scoped>

</style>
