<template>
    <div>
        <b-row>
            <b-col cols="10" offset="1" class="box">
                <video
                    id="video"
                    ref="video"
                    :width="width"
                    :height="height"
                    :src="source"
                    :autoplay="autoplay"
                    :playsinline="playsinline"
                />
            </b-col>
        </b-row>
        <b-row>
            <b-col cols="1"><label class="zoomIcon">-</label></b-col>
            <b-col cols="10">
                <input 
                    type="range" 
                    :min="minZoom" 
                    :max="maxZoom"
                    :step="stepZoom" 
                    value=0
                    class="slider"
                    id="zoomSlide"
                >
            </b-col>
            <b-col cols="1"><label class="zoomIcon">+</label></b-col>
        </b-row>
    </div>
</template>

<script>
export default {
    name: "VueWebCam",

    props: {
        width: {
            type: [Number, String],
            default: "100%"
        },
        height: {
            type: [Number, String],
            default: 500
        },
        autoplay: {
            type: Boolean,
            default: true
        },
        screenshotFormat: {
            type: String,
            default: "image/jpeg"
        },
        selectFirstDevice: {
            type: Boolean,
            default: false
        },
        deviceId: {
            type: String,
            default: null
        },
        playsinline: {
            type: Boolean,
            default: true
        },
        resolution: {
            type: Object,
            default: () =>  {
              return {
                height: { ideal: 2160 },
                width: { ideal: 4096 }
                };
            },
            validator: value => {
                return value.height && value.width;
            }
        }
    },

    data() {
        return {
            source: null,
            canvas: null,
            camerasListEmitted: false,
            cameras: [],

            minZoom: null,
            maxZoom: null,
            zoomable: false,
            stepZoom: null,
            zoom: null,

            delayTimer: null
        };
    },

    watch: {
        deviceId: function(id) {
            this.changeCamera(id);
        }
    },

    mounted() {
        this.setupMedia();
    },

    beforeDestroy() {
        this.stop();
    },

    methods: {
        /**
         * get user media
         */
        legacyGetUserMediaSupport() {
            return constraints => {
                // First get ahold of the legacy getUserMedia, if present
                let getUserMedia =
                    navigator.getUserMedia ||
                    navigator.webkitGetUserMedia ||
                    navigator.mozGetUserMedia ||
                    navigator.msGetUserMedia ||
                    navigator.oGetUserMedia;

                // Some browsers just don't implement it - return a rejected promise with an error
                // to keep a consistent interface
                if (!getUserMedia) {
                    return Promise.reject(
                        new Error("getUserMedia is not implemented in this browser")
                    );
                }

                // Otherwise, wrap the call to the old navigator.getUserMedia with a Promise
                return new Promise(function(resolve, reject) {
                    getUserMedia.call(navigator, constraints, resolve, reject);
                });
            };
        },

        /**
         * setup media
         */
        setupMedia() {
            if (navigator.mediaDevices === undefined) {
                navigator.mediaDevices = {};
            }

            if (navigator.mediaDevices.getUserMedia === undefined) {
                navigator.mediaDevices.getUserMedia = this.legacyGetUserMediaSupport();
            }

            this.testMediaAccess();
        },

        /**
         * load available cameras
         */
        loadCameras() {
            navigator.mediaDevices
                .enumerateDevices()
                .then(deviceInfos => {
                    for (let i = 0; i !== deviceInfos.length; ++i) {
                        let deviceInfo = deviceInfos[i];
                        if (deviceInfo.kind === "videoinput") {
                            this.cameras.push(deviceInfo);
                        }
                    }
                })
                .then(() => {
                    if (!this.camerasListEmitted) {
                        if (this.selectFirstDevice && this.cameras.length > 0) {
                            this.deviceId = this.cameras[0].deviceId;
                        }

                        this.$emit("cameras", this.cameras);
                        this.camerasListEmitted = true;
                    }
                })
                .catch(error => this.$emit("notsupported", error));
        },

        /**
         * change to a different camera stream, like front and back camera on phones
         */
        changeCamera(deviceId) {
            this.stop();
            this.$emit("camera-change", deviceId);
            this.loadCamera(deviceId);
        },

        /**
         * load the stream to the
         */
        loadSrcStream(stream) {
            if ("srcObject" in this.$refs.video) {
                // new browsers api
                this.$refs.video.srcObject = stream;
            } else {
                // old broswers
                this.source = window.HTMLMediaElement.srcObject(stream);
            }

            const zoomSlider = document.getElementById("zoomSlide");
            if (this.zoomable) {
                const [track] = stream.getVideoTracks();
                zoomSlider.oninput = function(event) {
                    this.zoom = event.target.value;
                    track.applyConstraints({advanced: [ {zoom: event.target.value} ]});
                }
            } else {
                zoomSlider.oninput = function(event) {
                    this.zoom = event.target.value;
                    document.getElementById("video").style.transform = `scale(${event.target.value}, ${event.target.value})`;
                }
            }

            // Emit video start/live event
            this.$refs.video.onloadedmetadata = () => {
                this.$emit("video-live", stream);
            };

            this.$emit("started", stream);
        },

        /**
         * stop the selected streamed video to change camera
         */
        stopStreamedVideo(videoElem) {
            let stream = videoElem.srcObject;
            let tracks = stream.getTracks();

            tracks.forEach(track => {
                // stops the video track
                track.stop();
                this.$emit("stopped", stream);

                this.$refs.video.srcObject = null;
                this.source = null;
            });
        },

        // stop the video
        stop() {
            if (this.$refs.video !== null && this.$refs.video.srcObject) {
                this.stopStreamedVideo(this.$refs.video);
            }
        },

        // start the video
        start() {
            if (this.deviceId) {
                this.loadCamera(this.deviceId);
            }
        },

        // pause the video
        pause() {
            if (this.$refs.video !== null && this.$refs.video.srcObject) {
                this.$refs.video.pause();
            }
        },

        // resume the video
        resume() {
            if (this.$refs.video !== null && this.$refs.video.srcObject) {
                this.$refs.video.play();
            }
        },

        /**
         * test access
         */
        testMediaAccess() {
            let constraints = { video: true };

            if (this.resolution) {
                constraints.video = {};
                constraints.video.height = this.resolution.height;
                constraints.video.width = this.resolution.width;
            }

            navigator.mediaDevices
                .getUserMedia(constraints)
                .then(stream => {
                    //Make sure to stop this MediaStream
                    let tracks = stream.getTracks();
                    this.initZoom(tracks[0]);
                    tracks.forEach(track => {
                        track.stop();
                    });
                    this.loadCameras();
                })
                .catch(error => this.$emit("error", error));
        },

        /**
         * load the camera passed as index!
         */
        loadCamera(device) {
            let constraints = { video: { deviceId: { exact: device } } };

            if (this.resolution) {
                constraints.video.height = this.resolution.height;
                constraints.video.width = this.resolution.width;
            }

            navigator.mediaDevices
                .getUserMedia(constraints)
                .then(stream => this.loadSrcStream(stream))
                .catch(error => this.$emit("error", error));
        },

        /**
         * capture screenshot
         */
        capture() {
            return this.getCanvas().toDataURL(this.screenshotFormat);
        },

        /**
         * get canvas
         */
        getCanvas() {
            let video = this.$refs.video;
            if (!this.ctx) {
                let canvas = document.createElement("canvas");
                canvas.height = video.videoHeight;
                canvas.width = video.videoWidth;
                this.canvas = canvas;

                this.ctx = canvas.getContext("2d");
            }

            const { ctx, canvas } = this;
            ctx.drawImage(video, 0, 0, canvas.width, canvas.height);

            return canvas;
        },

        initZoom(track) {
            const capabilities = track.getCapabilities();
            const settings = track.getSettings();
            if (!('zoom' in settings)) {
                this.zoomable = false;
                this.minZoom = 1;
                this.maxZoom = 4;
                this.stepZoom = 0.01;
                this.zoom = 1;
            } else {
                this.zoomable = true;
                this.minZoom = capabilities.zoom.min;
                this.maxZoom = capabilities.zoom.max;
                this.stepZoom = capabilities.zoom.step;
                this.zoom = settings.zoom;
            }
        }
    }
};
</script>

<style>

.slidecontainer {
        width: 100%; /* Width of the outside container */
}
/* The slider itself */
.slider {
        -webkit-appearance: none;    /* Override default CSS styles */
        appearance: none;
        width: 100%; /* Full-width */
        height: 15px; /* Specified height */
        border-radius: 7px;
        background: #d3d3d3; /* Grey background */
        outline: none; /* Remove outline */
        opacity: 0.7; /* Set transparency (for mouse-over effects on hover) */
        -webkit-transition: .2s; /* 0.2 seconds transition on hover */
        transition: opacity .2s;
}
/* Mouse-over effects */
.slider:hover {
        opacity: 1; /* Fully shown on mouse-over */
}
/* The slider handle (use -webkit- (Chrome, Opera, Safari, Edge) and -moz- (Firefox) to override default look) */
.slider::-webkit-slider-thumb {
        -webkit-appearance: none; /* Override default look */
        appearance: none;
        width: 25px; /* Set a specific slider handle width */
        height: 25px; /* Slider handle height */
        border-radius: 50%;
        background: #4CAF50; /* Green background */
        cursor: pointer; /* Cursor on hover */
}
.slider::-moz-range-thumb {
        width: 25px; /* Set a specific slider handle width */
        height: 25px; /* Slider handle height */
        background: #4CAF50; /* Green background */
        cursor: pointer; /* Cursor on hover */
}

.box {
    overflow: hidden;
}

</style>
