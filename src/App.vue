<template>
    <div id="app">
        <b-modal id="camModal" size="xl" no-close-on-backdrop>
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
                    <b-button v-if="modalMode == 'confirm'" class="float-right" variant="primary" @click="ok(); modalMode = 'cam'">Confirm</b-button>
                </b-container>
            </template>
        </b-modal>

        <b-modal id="loadingModal" size="lg" no-close-on-backdrop>
            <template #modal-header="{  }">     <!-- Header -->
                <!-- Emulate built in modal header close button action -->
                <label class="modal-header header">Google is doing smart things so we don't have to</label>
            </template>

            <template #default="{  }">      <!-- Body -->
                <b-container class="container">
                    <img src="./assets/sending-to-google.gif" alt="fancy loading gif" class="image">
                </b-container>
            </template>
            <template #modal-footer="{  }">     <!-- Footer -->
            </template>
        </b-modal>


        <b-container>
            <b-row>
                <b-col lg="4">
                    <b-form-file
                        v-model="pdf"
                        :state="Boolean(pdf)"
                        placeholder="Choose a file or drop it here..."
                        drop-placeholder="Drop file here..."
                        @input="readFile"
                        accept="image/*"
                    ></b-form-file>
                </b-col>
                <b-col lg="2">
                    <b-button id="button" variant="success" @click="newPic()">New pic</b-button>
                </b-col>
                <b-col lg="2">
                    <b-button id="button" variant="success" @click="compare()">Compare</b-button>
                </b-col>
            </b-row>

            <b-row class="no-gutters margined">
                <b-col sm="auto">
                    <img id="pdf" :src="pdf64" class="image pdf" />
                    <box
                        v-for="(n, index) of pdfNotPic"
                        :boxId="'pdf'+index"
                        :key="'pdf'+index"
                        :corner0="n[0]"
                        :corner1="n[1]"
                        :corner2="n[2]"
                        :corner3="n[3]"
                        :imageSize="{'width':pdfWidth, 'height':pdfHeight}"
                        class="layer-2"
                    ></box>
                </b-col>
            </b-row>
            <b-row class="no-gutters margined">
                <b-col sm="auto">
                    <img id="pic" :src="img" class="image pdf" />
                    <box
                        v-for="(n, index) of picNotPdf"
                        :boxId="'pic'+index"
                        :key="'pic'+index"
                        :corner0="n[0]"
                        :corner1="n[1]"
                        :corner2="n[2]"
                        :corner3="n[3]"
                        :imageSize="{'width':picWidth, 'height':picHeight}"
                        class="layer-2"
                    ></box>
                </b-col>
            </b-row>
        </b-container>
    </div>
</template>

<script>
    import WebCam from "./components/webcam.vue";
    import box from "./components/Box.vue";
    import SECURE from "./assets/secure.json";
    import axios from 'axios';

    export default {
        name: 'App',

        components: { "vue-web-cam": WebCam, box },

        computed: {
            device: function() {
                return this.devices.find(n => n.deviceId === this.deviceId);
            },
            test: () => {
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

                modalMode: 'cam',

                gCloudVisionUrl: null,

                picData: null,
                picHeight: null,
                picWidth: null,
                pdfData: null,
                pdfWidth: null,
                pdfHeight: null,
                pdf64: null,
                pdf: null,

                picNotPdf: [],
                pdfNotPic: []
            };
        },

        async mounted() {
            // Fetch image from q or receive from input link
            this.$bvModal.show('camModal');

            this.gCloudVisionUrl = `https://vision.googleapis.com/v1/images:annotate?key=${SECURE.privateKey}`;
        },

        methods: {

            newPic() {
                this.$bvModal.show('camModal');
                setTimeout(()=>{
                    this.onStart();
                }, 0);
            },

            async readFile(file) {
                this.picNotPdf = [];
                this.pdfNotPic = [];

                var reader = new FileReader();
                reader.readAsDataURL(file);
                reader.onload = () => {
                    this.pdf64 = reader.result;
                };
                reader.onerror = function (error) {
                    console.log('Error: ', error);
                };
            },

            async readPic() {
                const imageURL = this.img.split(",")[1];
                
                let requestBody = { requests: [ { image: { content: imageURL }, features: [ { type: "TEXT_DETECTION", maxResults: 1000 } ] } ] };
                
                await axios.post( this.gCloudVisionUrl, requestBody).then(response => {
                    this.picData = response.data.responses[0]
                    this.picWidth = this.picData.fullTextAnnotation.pages[0].width;
                    this.picHeight = this.picData.fullTextAnnotation.pages[0].height;
                }).catch(error => {
                    console.log(error);
                    console.log(error.response.data.error.message);
                });

            },

            async readPdf() {
                const pdfURL = this.pdf64.split(",")[1];
                
                let requestBody = { requests: [ { image: { content: pdfURL }, features: [ { type: "TEXT_DETECTION", maxResults: 1000 } ] } ] };
                
                await axios.post( this.gCloudVisionUrl, requestBody).then(response => {
                    this.pdfData = response.data.responses[0];
                    this.pdfWidth = this.pdfData.fullTextAnnotation.pages[0].width;
                    this.pdfHeight = this.pdfData.fullTextAnnotation.pages[0].height;
                }).catch(error => {
                    console.log(error);
                    console.log(error.response.data.error.message);
                });

            },

            extractWords(set) {
                var picWords = {};
                var picWordsFull = {};
                this[set].textAnnotations.forEach(element => {
                    if (!element.locale) { 
                        picWords[element.description] = element.boundingPoly.vertices; 
                    } else { 
                        picWordsFull[element.description] = element.boundingPoly.vertices; 
                    }
                });
                return picWords
            },

            async compare() {
                this.$bvModal.show('loadingModal');

                await this.readPdf();
                await this.readPic();

                const picWords = this.extractWords('picData');
                const pdfWords = this.extractWords('pdfData');

                const picNotPdfWords = Object.keys(picWords).filter(x => !Object.keys(pdfWords).includes(x));
                const pdfNotPicWords = Object.keys(pdfWords).filter(x => !Object.keys(picWords).includes(x));

                var word;
                for (word of picNotPdfWords) {
                    this.picNotPdf.push(picWords[word])
                }
                for (word of pdfNotPicWords) {
                    this.pdfNotPic.push(pdfWords[word])
                }
                this.$bvModal.hide('loadingModal');
            },

            opacitySlide(event) {
                document.getElementById('pic').style.opacity = event.target.value;
            },

            onCapture() {
                this.picNotPdf = [];
                this.pdfNotPic = [];
                this.img = this.$refs.webcam.capture();
                this.modalMode = 'confirm';
            },

            retake() {
                this.modalMode = 'cam'
            },
            onStarted() {
            },
            onStopped() {
            },
            onStop() {
                this.$refs.webcam.stop();
            },
            onStart() {
                this.$refs.webcam.start();
            },
            // onError(error) {
            onError() {
            },
            onCameras(cameras) {
                this.devices = cameras;
            },
            onCameraChange(deviceId) {
                this.deviceId = deviceId;
                this.camera = deviceId;
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
    max-height: 100%;
    object-fit: contain;
    margin: 0px;
    display: inline-block;
    padding: 0 !important;
    margin: 0 !important;
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

.layer-2 {
  z-index: 1;
  position: absolute;
}

.pdf {
    height: 480px;
    width: auto;
}

.img-container {
    min-width: 20px;
    min-height: 20px;
    display: inline-block;
    padding: 0%;
}

.margined {
    margin: 1%;
}

</style>
