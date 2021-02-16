<template>
    <div id="box" class="box">
        
    </div>
</template>

<script>
    export default {
        name: 'Box',

        props: {
            corner0: {
                type: object,
                validator: (obj) => {
                    return (obj.x && Number.isInteger(obj.x) && obj.y && Number.isInteger(obj.y) );
                },
                default: () => {
                    return {x: 0, y: 0}
                }
            },
            corner1: {
                type: object,
                validator: (obj) => {
                    return (obj.x && Number.isInteger(obj.x) && obj.y && Number.isInteger(obj.y) );
                },
                default: () => {
                    return {x: 0, y: 0}
                }
            },
            corner2: {
                type: object,
                validator: (obj) => {
                    return (obj.x && Number.isInteger(obj.x) && obj.y && Number.isInteger(obj.y) );
                },
                default: () => {
                    return {x: 0, y: 0}
                }
            },
            corner3: {
                type: object,
                validator: (obj) => {
                    return (obj.x && Number.isInteger(obj.x) && obj.y && Number.isInteger(obj.y) );
                },
                default: () => {
                    return {x: 0, y: 0}
                }
            },
            imageSize: {
                type: object,
                validator: (obj) => {
                    return (obj.width > 0 && Number.isInteger(obj.width) && obj.height > 0 && Number.isInteger(obj.height) );
                },
                default: () => {
                    return {width: 0, height: 0}
                }
            }
        },

        computed: {

        },

        data () {
            return {
                
            };
        },

        mounted() {
        },

        methods: {
        }

}
</script>

<style>
.box {
    background-color: rgba(255, 196, 0, 0.692);
    border: 0.8px solid red;
}

</style>
