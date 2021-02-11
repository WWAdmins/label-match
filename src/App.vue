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
                <h2>Captured Image</h2>
                <figure class="figure">
                    <img :src="img" class="image" />
                </figure>
            </b-row>
        </b-container>

        <b-modal id="camModal" size="xl">
            <template #modal-header="{  }">     <!-- Header -->
                <!-- Emulate built in modal header close button action -->
                <label class="modal-header" v-if="modalMode == 'cam'">Take image</label>
                <label class="modal-header" v-if="modalMode == 'confirm'">Confirm image</label>
            </template>

            <template #default="{  }">      <!-- Body -->
                <b-container v-show="modalMode == 'cam'" class="container">
                    <vue-web-cam
                        ref="webcam"
                        :device-id="deviceId"
                        @started="onStarted"
                        @stopped="onStopped"
                        @error="onError"
                        @cameras="onCameras"
                        @camera-change="onCameraChange"
                    />
                    <select v-model="camera">
                        <option>-- Select Device --</option>
                        <option
                            v-for="device in devices"
                            :key="device.deviceId"
                            :value="device.deviceId"
                        >{{ device.label }}</option>
                    </select>
                </b-container>
                <b-container v-show="modalMode == 'confirm'" class="container">
                    <b-row>
                        <b-col cols="10"  offset="1">
                            <img class="image" :src="img" alt="taken image">
                        </b-col>
                    </b-row>
                    
                </b-container>
            </template>

            <template #modal-footer="{ cancel, ok }">     <!-- Footer -->
                <b-container>
                    <!-- Emulate built in modal footer ok and cancel button actions -->
                    <b-button v-if="modalMode == 'cam'" class="float-left" variant="danger" @click="cancel()">Cancel</b-button>
                    <b-button v-if="modalMode == 'confirm'" class="float-left" variant="danger" @click="retake()">Cancel</b-button>
                    <b-button v-if="modalMode == 'cam'" class="float-right" variant="primary" @click="onCapture()">Capture Photo</b-button>
                    <b-button v-if="modalMode == 'confirm'" class="float-right" variant="primary" @click="ok()">Confirm</b-button>
                </b-container>
            </template>
        </b-modal>
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
                devices: [],

                modalMode: 'cam'
            };
        },

        mounted() {
            // Fetch image from q or receive from input link
            this.$bvModal.show('camModal')
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
                this.modalMode = 'confirm';
            },

            retake() {
                this.modalMode = 'cam'
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
    background-color: rgba(254, 254, 254, 1);
    margin-top: 60px;
}

.dotted {
    border-style: dotted;
}

.image { 
    max-width: 100%; 
    object-fit: cover;
}

</style>
