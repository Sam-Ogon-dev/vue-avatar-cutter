<template>
    <div id="app">
        <Avatar :src="image" style="width: 300px; height: 300px; border: 1px solid black" @miniAvatar="setMiniAvatar"/>
        <input type="file" @change="loadImage">
        <img :src="miniAvatar" width="100px" height="100px" style="border-radius: 50px"/>
    </div>
</template>

<script>
    import Avatar from "./components/Avatar";

    export default {
        name: 'App',
        components: {
            Avatar
        },
        data() {
            return {
                image: {},
                miniAvatar: ""
            }
        },
        methods: {
            loadImage(e) {
                this.image = URL.createObjectURL(e.target.files[0]);
            },
            setMiniAvatar(src) {
                this.miniAvatar = src;
            }
        }
    }
</script>

<style lang="scss">
    html, body {
        margin: 0;
        padding: 0;
        height: 100%;
    }

    * {
        box-sizing: border-box;
    }

    #app {
        height: 100%;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
    }
</style>
