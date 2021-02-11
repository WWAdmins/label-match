<template>
    <div id="app">
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


        <b-container>
            <b-row>
                <b-col cols="5">
                    <label for="stockInput" class="main-title">Stock code:</label>
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
                <b-col cols="10">
                    <label for="pic" class="header">Captured Image</label>
                </b-col>
            </b-row>

            <b-row>
                <b-col cols="10 display">
                    <img id="pic" :src="img" class="image layer-2" />
                    <img id="pdf" src="./assets/522572.jpg" class="image pdf" />
                </b-col>
            </b-row>
            
            <b-row>
                <input 
                    type="range" 
                    min=0
                    max=1
                    step=0.01
                    value=0.5
                    class="slider"
                    id="opacitySlider"
                    @input="opacitySlide"
                >
            </b-row>
        </b-container>
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
            document.getElementById('pic').style.opacity = 0.5
        },

        methods: {

            opacitySlide(event) {
                document.getElementById('pic').style.opacity = event.target.value;
            },

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
.dotted {
    border-style: dotted;
}

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

.standard-input {
    float: left;
    width: 100%;
    padding: 4%;
    border: 1px solid lightgrey;
    border-radius: 8px;
    font-size: 120%;
    outline-width: 3px;
    outline-color: darkgrey;
}

.main-title {
  font-size: 200%;
  text-align: left;
  white-space: pre-wrap;
  float: left;
  font-weight: 500;
  color: rgba(65,65,65,1);
}

.header {
  font-weight: 500;
  padding: 15px 10px 5px 10px;
  text-align: left;
  font-size: 150%;
  white-space: pre-wrap;
}

.form-header {
  padding: 15px 10px 5px 10px;
  text-align: left;
  font-size: 25px;
  white-space: pre-wrap;
}

.display {
    object-fit: cover;
    padding: 2%;
    border: 4px groove lightgrey;
    border-radius: 9px;
}

.layer-2 {
  z-index: 1;
  position: absolute;
  left: 2%;
  width: 80%;
}

.pdf {
    height: 480px;
    width: auto;
}

</style>
