<template>
    <div class="center">
        <div v-if="(error == false)">
            <div v-if="ssid_name">
                <div class="card apcard" v-on:click="generate_ap()">
                    <div class="columns">
                        <div class="column col-5">
                            <center><img :src="ssid_qr" id="qrcode"></center>
                        </div>
                        <div class="divider-vert white-bg" data-content="OR"></div>
                        <div class="column col-5"><br />
                            <span class="light-grey">Network name: </span><br />
                            <h4>{{ ssid_name }}</h4>
                            <span class="light-grey">Network password: </span><br />
                            <h4>{{ ssid_password }}</h4>
                        </div>
                    </div>
                </div>
                <br /><br /><br /><br /> <br /><br /><br /><br /><br /><br />
                <!-- Requite a CSS MEME for that shit :) -->
                <span class="legend">Tap the white frame to generate a new network.</span>
            </div>
            <div v-else>
                <img src="@/assets/loading.svg"/>
                <p class="legend">We generate an ephemeral network for you.</p>
            </div>
        </div>
        <div v-else>
            <p>
                <strong>Unfortunately, we got some issues.</strong>
                <br /><br />
                Please verify that you've two Wifi interfaces on your device<br />
                and restart it by clicking on the button below.<br /> 
            </p>
            <button class="btn" v-on:click="reboot()">Reboot the device</button>
        </div>
    </div>
    
</template>

<script>
import axios from 'axios'
import router from '../router'

export default {
    name: 'generate-ap',
    components: {},
    data() {
        return {
            ssid_name: false,
            ssid_qr: false,
            ssid_password: false,
            capture_token: false,
            capture_start: false,
            interval: false,
            error: false
        }
    },
    methods: {
        generate_ap: function() {
            clearInterval(this.interval)
            this.ssid_name = false
            axios.get(`/api/network/ap/start`, { timeout: 70000 })
                .then(response => (this.show_ap(response.data)))
        },
        show_ap: function(data) {
            if (data.status) {
                this.ssid_name = data.ssid
                this.ssid_password = data.password
                this.ssid_qr = data.qrcode
                this.start_capture() // Start the capture before client connect.
            } else {
                this.error = true
            }
        },
        start_capture: function() {
            axios.get(`/api/capture/start`, { timeout: 30000 })
                .then(response => (this.get_capture_token(response.data)))
        },
        reboot: function() {
            axios.get(`/api/misc/reboot`, { timeout: 30000 })
                .then(response => { console.log(response)})
        },
        get_capture_token: function(data) {
            if (data.status) {
                this.capture_token = data.capture_token
                this.capture_start = Date.now()
                this.get_device()
            }
        },
        get_device: function() {
            this.interval = setInterval(() => {
                axios.get(`/api/device/get/${this.capture_token}`, { timeout: 30000 })
                    .then(response => (this.check_device(response.data)))
            }, 500);
        },
        check_device: function(data) {
            if (data.status) {
                clearInterval(this.interval);
                var capture_token = this.capture_token
                var capture_start = this.capture_start
                var device_name = data.name
                router.replace({
                    name: 'capture',
                    params: {
                        capture_token: capture_token,
                        capture_start: capture_start,
                        device_name: device_name
                    }
                });
            }
        }
    },
    created: function() {
        this.generate_ap();
    }
}
</script>

