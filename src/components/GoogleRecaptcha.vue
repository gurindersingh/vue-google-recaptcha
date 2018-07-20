<template>
    <div id="google-recaptcha-container" v-if="show">
        <h3 class="label">Verify you are human, Check the box below</h3>
        <div ref="captchaZone" style="min-height: 78px;"></div>
        <input type="hidden" :name="name" v-model="response" v-if="input">
    </div>
</template>

<script>
    export default {
        name: "recaptcha",

        props: {
            dataShow: {
                type: Boolean,
                default: false
            },
            dataReset: {
                type: Boolean,
                default: false
            },
            input: {
                type: Boolean,
                default: false
            },
            name: {
                type: String,
                default: 'gRecaptchaResponse'
            },
            siteKey: {
                type: String,
                required: true
            },
            theme: {
                type: String,
                default: 'light'
            }
        },

        data() {
            return {
                show: true,
                captchaId: null,
                response: null,
                captcahLoaded: false,
                options: {
                    'callback': this.verifyCallback,
                    'sitekey': this.siteKey,
                    'theme': this.theme
                }
            }
        },

        mounted() {
            if (!this.captcahLoaded) {
                this.checkCaptchaLoaded();
            }
        },

        methods: {

            checkCaptchaLoaded(timeId = null) {
                if (window.hasOwnProperty('grecaptcha') && window.grecaptcha.hasOwnProperty('render')) {
                    this.captcahLoaded = true;
                    clearTimeout(timeId);
                    this.captchaId = window.grecaptcha.render(this.$refs.captchaZone, this.options)
                } else {
                    let timeId = setTimeout(() => this.checkCaptchaLoaded(timeId), 500);
                }
            },

            verifyCallback(response) {
                this.response = response;

                this.$emit('verified', {
                    response: response,
                    captchaId: this.captchaId
                })
            },
        },

        watch: {

            show: function (newVal, oldVal) {
                if (newVal && window.hasOwnProperty('grecaptcha')) {
                    this.render()
                }
            },

            dataShow: function (val) {
                if (!val) {
                    this.show = false
                }
            },

            dataReset: function (newVal, oldVal) {
                if (newVal) {
                    window.grecaptcha.reset(this.captchaId)
                }
            }

        }
    }
</script>