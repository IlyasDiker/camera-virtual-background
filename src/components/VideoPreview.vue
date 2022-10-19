<template>
    <main class="container">
        <article ref="VideoContainer" class="camera-preview" data-theme="dark">
            <template v-if="!started"> Click the &nbsp;<kbd @click="showLocalVideo">Start</kbd>&nbsp; button </template>
            <progress v-if="loading"></progress>
        </article>
        <div class="grid backgrounds-gallery">
            <template v-for="(item, index) in backgrounds" :key="index">
                <button class="" 
                    @click="applyEffect(item)"
                    :class="item.label == selectedProccesor?.label ? 'primary' : 'contrast outline'">
                    <span>{{item.label}}</span>
                </button>
            </template>
        </div>
        <div class="grid">
            <button @click="showLocalVideo" :disabled="started">Start</button>
            <button @click="stop" class="secondary" :disabled="!started">Stop</button>
            <button @click="removeFilter" class="secondary" :disabled="!started || !selectedProccesor">Remove
                Filter</button>
        </div>
    </main>
</template>

<script>
import { io } from '@tensorflow/tfjs-core';
import * as VideoProcessors from '@twilio/video-processors';
import Twilio from 'twilio-video';
export default {
    data() {
        return {
            loading: false,
            started: false,
            selectedProccesor: null,
            videoTrackInternface: null,
            backgrounds: [
                {
                    label: 'Light Blur',
                    processor: {
                        type: 'blur',
                        maskBlurRadius: 5,
                        blurFilterRadius: 5,
                    }
                },
                {
                    label: 'Heavy Blur',
                    processor: {
                        type: 'blur',
                        maskBlurRadius: 5,
                        blurFilterRadius: 10,
                    }
                },
                {
                    label: 'Classroom',
                    processor: {
                        type: 'image',
                        imageUrl: 'classroom.jpeg',
                        maskBlurRadius: 10,
                    }
                },
                {
                    label: 'Corporate Office',
                    processor: {
                        type: 'image',
                        imageUrl: 'corporate.jpg',
                        maskBlurRadius: 10,
                    }
                },
                {
                    label: 'Lounge',
                    processor: {
                        type: 'image',
                        imageUrl: 'lounge.webp',
                        maskBlurRadius: 10,
                    }
                },
                {
                    label: 'Ops room',
                    processor: {
                        type: 'image',
                        imageUrl: 'opsroom.webp',
                        maskBlurRadius: 10,
                    }
                },
                {
                    label: 'House',
                    processor: {
                        type: 'image',
                        imageUrl: 'house.png',
                        maskBlurRadius: 10,
                    }
                }
            ]
        }
    },

    methods: {
        async showLocalVideo() {
            if(this.started) return;
            let videoTrack = await Twilio.createLocalVideoTrack({
                width: 640,
                height: 480,
                frameRate: 24
            });
            this.$refs.VideoContainer.appendChild(videoTrack.attach());
            this.started = true;

            let vtInt = {
                stopTrack() {
                    videoTrack.stop();
                    return videoTrack.detach();
                },
                async applyfilter(filter) {
                    return videoTrack.addProcessor(filter);
                },
                async removeFilter(filter) {
                    return videoTrack.removeProcessor(filter)
                },
                getVideoTrack(){
                    return videoTrack
                },
                getCurrentProcessor(){
                    return videoTrack.processor ?? null
                }
            }
            this.videoTrackInternface = vtInt
            
        },
        applyEffect(item){
            if(!this.started) return;
            if(this.selectedProccesor){
                let cPssr = this.videoTrackInternface?.getCurrentProcessor()
                this.videoTrackInternface?.removeFilter(cPssr)
            }
            this.loadEffect(item.processor).then((filter)=>{
                this.videoTrackInternface?.applyfilter(filter).then((LocalMediaTrack2)=>{
                    this.selectedProccesor = item
                })
            }, ()=>{
                console.error('cannot load Filter')
            })
        },
        loadEffect(processor) {
            return new Promise(async (resolve, reject) => {
                var filter;
                if (processor?.type == 'blur') {
                    filter = new VideoProcessors.GaussianBlurBackgroundProcessor({
                        assetsPath: '/',
                        ...processor
                    });
                    await filter.loadModel();
                    resolve(filter);
                } else if (processor?.type == 'image') {
                    let img = new Image()
                    img.src = processor.imageUrl
                    img.onload = async () => {
                        const filter = new VideoProcessors.VirtualBackgroundProcessor({
                            assetsPath: '/',
                            backgroundImage: img,
                            maskBlurRadius: 5,
                            fitType: VideoProcessors.ImageFit.Cover,

                        });
                        await filter.loadModel();
                        resolve(filter);
                    }
                } else {
                    reject()
                }
            });
        },
        stop() {
            if(!this.started) return;
            let tracks = this.videoTrackInternface?.stopTrack();
            tracks.forEach(tr => tr.remove())
            this.started = false;
            this.selectedProccesor = null;
        },
        removeFilter() {
            if(!this.started) return;
            let cPssr = this.videoTrackInternface?.getCurrentProcessor()
            this.videoTrackInternface?.removeFilter(cPssr).then((LocalMediaTrack2)=>{
                this.selectedProccesor = null
            })
        }
    },
    created() {
        console.log(Twilio);
    },
    beforeDestroy() {
        if (this.started) this.stop();
    },
}
</script>

<style lang="scss">
.camera-preview {
    height: 450px;
    padding: 0;
    position: relative;
    display: flex;
    justify-content: center;
    align-items: center;

    video {
        height: 100%;
        width: 100%;
        object-fit: contain;
    }

    progress {
        position: absolute;
        top: unset;
        bottom: 0;
        left: 0;
        right: 0;
        width: 100%;
        margin-bottom: 0;
    }
}

.backgrounds-gallery {
    margin-bottom: 20px;

    article {
        margin-block: 10px;
    }
}
</style>