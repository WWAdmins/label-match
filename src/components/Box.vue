<template>
    <div :id="'box'+boxId" class="holo-box">
        
    </div>
</template>

<script>
    export default {
        name: 'Box',

        props: {
            boxId: {
                type: String
            },           
            corner0: {
                type: Object,
                validator: (obj) => {
                    return (obj.x && Number.isInteger(obj.x) && obj.y && Number.isInteger(obj.y) );
                },
                default: () => {
                    return {x: 0, y: 0}
                }
            },
            corner1: {
                type: Object,
                validator: (obj) => {
                    return (obj.x && Number.isInteger(obj.x) && obj.y && Number.isInteger(obj.y) );
                },
                default: () => {
                    return {x: 0, y: 0}
                }
            },
            corner2: {
                type: Object,
                validator: (obj) => {
                    return (obj.x && Number.isInteger(obj.x) && obj.y && Number.isInteger(obj.y) );
                },
                default: () => {
                    return {x: 0, y: 0}
                }
            },
            corner3: {
                type: Object,
                validator: (obj) => {
                    return (obj.x && Number.isInteger(obj.x) && obj.y && Number.isInteger(obj.y) );
                },
                default: () => {
                    return {x: 0, y: 0}
                }
            },
            imageSize: {
                type: Object,
                validator: (obj) => {
                    return (obj.width > 0 && Number.isInteger(obj.width) && obj.height > 0 && Number.isInteger(obj.height) );
                },
                default: () => {
                    return {width: 0, height: 0}
                }
            }
        },

        data () {
            return {
                width: null,
                height: null,
                angel: null,
                elementId: null
            }
        },

        mounted() {
            const dxW = Math.abs(this.corner1.x - this.corner0.x);
            const dyW = Math.abs(this.corner0.y - this.corner1.y);

            const dxH = Math.abs(this.corner3.x - this.corner0.x);
            const dyH = Math.abs(this.corner0.y - this.corner3.y);

            this.width = Math.sqrt(dxW**2 + dyW**2)*2;
            
            this.height = Math.sqrt(dxH**2 + dyH**2);
            
            this.elementId = `box${this.boxId}`;
            
            const dxA = (this.corner1.x - this.corner0.x);
            const dyA = this.corner0.y - this.corner1.y;
            this.angel = Math.atan(dyA/dxA);

            document.getElementById(this.elementId).style.height = `${(this.height/this.imageSize.height)*100}%`;
            document.getElementById(this.elementId).style.width = `${(this.width/this.imageSize.width)*100*0.47}%`;
            document.getElementById(this.elementId).style.top = `${(this.corner0.y/this.imageSize.height)*100}%`;
            document.getElementById(this.elementId).style.left = `${(this.corner0.x/this.imageSize.width)*100}%`;
            
            document.getElementById(this.elementId).style.transform = `rotate(${this.angel * -(180/Math.PI)}deg)`;
        },

        methods: {
        }

}
</script>

<style>
.holo-box {
    background-color: rgba(255, 136, 0, 0.2);
    border: 1.5px dashed rgb(255, 0, 242);
    transform-origin: top left;
}

</style>
