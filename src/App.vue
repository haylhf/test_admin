<template>
    <div id="app">
        <router-view class="router-view"></router-view>
    </div>
</template>

<script>
    import API from './api/API'
    const api = new API();
    export default {
        debug: true,
        name: 'app',
        data () {
            return {}
        },

        methods: {}
    }
</script>
<style>


</style>
