<template>
    <!-- <div id="app">
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
            <b-row>
                <vue-web-cam
                    ref="webcam"
                    :device-id="deviceId"
                    width="100%"
                    @started="onStarted"
                    @stopped="onStopped"
                    @error="onError"
                    @cameras="onCameras"
                    @camera-change="onCameraChange"
                />
            </b-row>
            <b-row>
                <select v-model="camera">
                    <option>-- Select Device --</option>
                    <option
                        v-for="device in devices"
                        :key="device.deviceId"
                        :value="device.deviceId"
                    >{{ device.label }}</option>
                </select>
            </b-row>
            <b-row>
                <button type="button" class="btn btn-primary" @click="onCapture">Capture Photo</button>
                <button type="button" class="btn btn-danger" @click="onStop">Stop Camera</button>
                <button type="button" class="btn btn-success" @click="onStart">Start Camera</button>
            </b-row>
            <b-row>
                <h2>Captured Image</h2>
                <figure class="figure">
                    <img :src="img" class="img-responsive" />
                </figure>
            </b-row>
        </b-container>
    </div> -->
    <div class="container">
        <div class="row">
            <div class="col-md-6">
                <h2>Current Camera</h2>
                <code v-if="device">{{ device.label }}</code>
                <div class="border">
                    <vue-web-cam
                        ref="webcam"
                        :device-id="deviceId"
                        width="100%"
                        height="80%"
                        @started="onStarted"
                        @stopped="onStopped"
                        @error="onError"
                        @cameras="onCameras"
                        @camera-change="onCameraChange"
                    />
                </div>

                <div class="row">
                    <div class="col-md-12">
                        <select v-model="camera">
                            <option>-- Select Device --</option>
                            <option
                                v-for="device in devices"
                                :key="device.deviceId"
                                :value="device.deviceId"
                            >{{ device.label }}</option>
                        </select>
                    </div>
                    <div class="col-md-12">
                        <button type="button" class="btn btn-primary" @click="onCapture">Capture Photo</button>
                        <button type="button" class="btn btn-danger" @click="onStop">Stop Camera</button>
                        <button type="button" class="btn btn-success" @click="onStart">Start Camera</button>
                    </div>
                </div>
            </div>
            <div class="col-md-6">
                <h2>Captured Image</h2>
                <figure class="figure">
                    <img :src="img" class="img-responsive" />
                </figure>
            </div>
            <label>https://googlechrome.github.io/samples/image-capture/update-camera-zoom.html</label>
        </div>
    </div>
</template>

<script>
    import WebCam from "./components/webcam.vue";

    export default {
        name: 'App',

        components: { "vue-web-cam": WebCam },

        computed: {
            device: function() {
                return this.devices.find(n => n.deviceId === this.deviceId);
            }
        },
        watch: {
            camera: function(id) {
                this.deviceId = id;
            },
            devices: function() {
                // Once we have a list select the first one
                var preference = this.devices[0];
                this.devices.forEach(element => {
                    if (element.label.toLowerCase().includes("back")) {
                        preference = element;
                    }
                });
                if (preference) {
                    this.camera = preference.deviceId;
                    this.deviceId = preference.deviceId;
                }
            }
        },

        data () {
            return {
                stockCode: '',
                img: null,
                camera: null,
                deviceId: null,
                devices: []
            };
        },

        mounted() {
            // Fetch image from q or receive from input link
            console.log(WebCam)
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

            onCapture() {
                this.img = this.$refs.webcam.capture();
            },
            // onStarted(stream) {
            onStarted() {
                // console.log("On Started Event", stream);
            },
            // onStopped(stream) {
            onStopped() {
                // console.log("On Stopped Event", stream);
            },
            onStop() {
                this.$refs.webcam.stop();
            },
            onStart() {
                this.$refs.webcam.start();
            },
            // onError(error) {
            onError() {
                // console.log("On Error Event", error);
            },
            onCameras(cameras) {
                this.devices = cameras;
                // console.log("On Cameras Event", cameras);
            },
            onCameraChange(deviceId) {
                this.deviceId = deviceId;
                this.camera = deviceId;
                // console.log("On Camera Change Event", deviceId);
            }
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
