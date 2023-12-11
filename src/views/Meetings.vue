<template>
    <div class='meetings'>
        <div class="main-wrapper">
            <v-card class="mx-auto px-6 py-8" max-width="600px">
                <div>
                    <video class="local-media" muted id='local-media' autoplay></video>
                    <br>
                    <div>
                        <div class="d-flex camera-preview-actions">
                            <li v-on:click="muteVideo()" class="icon" :class="{ 'icon-disabled': !isVideo }">
                                <mdicon v-if="isVideo" name="camera"/>
                                <mdicon v-else name="camera-off"/>
                            </li>
                            <li v-on:click="muteAudio()" class="icon" :class="{ 'icon-disabled': !isAudio }">
                                <mdicon v-if="isAudio" name="microphone"/>
                                <mdicon v-else name="microphone-off"/>
                            </li>
                        </div>
                    </div>
                </div>
                <div>
                    <div class="room-input-form">
                        <input
                            v-model='routRoomId'
                            placeholder='Insira o id de uma nova reunião'
                            required
                            type="text"
                            class="roomid-input"
                            id='roomid-input'
                        />
                        <input
                            v-model='username'
                            type="text"
                            placeholder='Insira um nome ou apelido'
                            class="username-input"
                            id='username-input'
                        />
                    </div>
                    <div class="d-flex">
                        <v-btn color="info" v-on:click='startCall()'>
                            <span v-if="routRoomId">Entrar na</span>
                            <span v-else>Criar uma</span> chamada
                        </v-btn>
                    </div>
                </div>
            </v-card>
        </div>
    </div>
</template>

<script>
import { mapState } from 'vuex'
import { v4 as uuidv4 } from 'uuid'

export default {
    name: 'Meetings',
    data: function () {
        return {
            meetingsList: [],
            localStream: null,
            isAudio: true,
            isVideo: true,
            username: '',
            peerConn: null,
            routRoomId: '',
            startCall () {
                debugger
                const randomRoomId = this.routRoomId ? this.routRoomId : uuidv4()

                this.$router.push({ name: 'room/', params: { roomId: randomRoomId } })
            },
            startMedia () {
                const params = {
                    video: {
                        frameRate: 24,
                        width: {
                            min: 480, ideal: 720, max: 1280
                        },
                        aspectRatio: 1.33333
                    },
                    audio: true
                }
                navigator.getUserMedia = (
                    navigator.getUserMedia ||
                    navigator.webkitGetUserMedia ||
                    navigator.mozGetUserMedia ||
                    navigator.msGetUserMedia
                )

                if (typeof navigator.mediaDevices.getUserMedia === 'undefined') {
                    navigator.getUserMedia(params, this.userStreamHandler, console.log)
                } else {
                    navigator.mediaDevices.getUserMedia(params).then(this.userStreamHandler).catch(console.log)
                }
            },
            muteAudio () {
                this.isAudio = !this.isAudio
                this.localStream.getAudioTracks()[0].enabled = this.isAudio
            },
            muteVideo () {
                this.isVideo = !this.isVideo
                this.localStream.getVideoTracks()[0].enabled = this.isVideo
            }
        }
    },
    methods: {
        userStreamHandler (stream) {
            this.localStream = stream
            document.getElementById('local-media').srcObject = this.localStream
        }
    },
    watch: {
        username (newUsername) {
            localStorage.username = newUsername
        }
    },
    computed: mapState(['APIData']),
    mounted: function () {
        if (localStorage.username) {
            this.username = localStorage.username
        }
    },
    created: function () {
        this.startMedia()
    }
}
</script>

<style scoped>

    input {
        width: 100%;
        padding: 12px 20px;
        margin: 8px 0;
        display: flex;
        border: 1px solid #ccc;
        border-radius: 4px;
        box-sizing: border-box;
    }

    .meetings {
        height: 100%;
        background-color: #E5ECE9;
    }

    .camera-preview-actions {
        width: 100%;
    }

    .room-input-form {
        display: flex;
        justify-content: space-between;
        margin: 18px 0;
    }

    .roomid-input {
        font-size: 14px;
        margin-right: 12px;
    }

    .username-input {
        font-size: 14px;
        margin-left: 12px;
    }

    .icon {
        position: relative;
        background: #ffffff;
        border-radius: 50%;
        padding: 25px;
        margin: 10px;
        width: 30px;
        height: 30px;
        font-size: 18px;
        display: flex;
        justify-content: center;
        align-items: center;
        flex-direction: column;
        box-shadow: 0 10px 10px rgb(0 0 0 / 10%);
        transition: all 0.2s cubic-bezier(0.68, -0.55, 0.265, 1.55);
    }

    .icon-disabled {
        background: rgb(0 0 0 / 12%);
    }

    .icon:hover {
        text-shadow: 0px -1px 0px rgba(0, 0, 0, 0.1);
        opacity: 0.5;
        color: #1877F2;
    }

    .main-wrapper {
        padding: 40px;
    }
    .local-media {
        max-width: 100%;
    }
</style>
