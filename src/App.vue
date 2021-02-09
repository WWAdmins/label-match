<template>
    <div id="app">
        <b-container>
            <b-row>
                <b-col cols="5">
                    <input
                        class="standard-input"
                        id="stockInput"
                        v-model.number="stockCode"
                        required
                        @keydown="keyDown"
                    >
                </b-col>
            </b-row>
            <b-row>
                <b-col>
                    <b-button id="picButton" class="picButton" v-on:click="takePhoto()">Take photo</b-button>
                </b-col>
            </b-row>
        </b-container>
    </div>
</template>

<script>

    export default {
        name: 'App',

        data () {
            return {
                stockCode: ''
            }
        },

        mounted() {
            // Fetch image from q or receive from input link
        },

        methods: {
            // On key press in input fields
            // Filters any key press that is not 0-9 or 'ArrowRight','ArrowLeft','Backspace', 'Tab'
            // (any filtered key presses are discarded)
            // if key down is 'Tab', force update to fields without timeout (prevents updates being skipped by quick change of field resetting timeout)
            keyDown(event) {
                const validKeys = ['ArrowRight','ArrowLeft','Backspace', 'Tab', 'Enter'];
                const keyRegex = /[0-9]/;
                if (!keyRegex.test(event.key) && validKeys.indexOf(event.key) < 0) {
                    event.preventDefault();
                }
            },
        }
    
}
</script>

<style>
#app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
}
</style>
