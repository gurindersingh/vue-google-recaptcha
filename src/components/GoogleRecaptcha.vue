<template>
    <div id="google-recaptcha-container" v-if="show">
        <h3 class="label">Verify you are human, Check the box below</h3>
        <div ref="captchaZone" style="min-height: 78px;"></div>
        <input type="hidden" :name="input" v-model="response" v-if="input">
    </div>
</template>

<script>
    export default {
        name: "recaptcha",

        props: {
            captchaScript: {
                type: String,
                default: 'https://www.google.com/recaptcha/api.js?render=explicit'
            },
            dataShow: {
                type: Boolean,
                default: false
            },
            dataReset: {
                type: Boolean,
                default: false
            },
            input: {
                type: String
            },
            siteKey: {
                type: String
            },
            theme: {
                type: String,
                default: 'light'
            }
        },

        data() {
            return {
                show: false,
                captchaId: null,
                response: null,
                captcahLoaded: false,
                options: {
                    callback: this.verifyCallback,
                    sitekey: this.siteKey ? this.siteKey : window.App.recaptcha.sitekey,
                    theme: this.theme
                }
            }
        },

        mounted() {
            this.loadScript();
        },

        methods: {

            loadScript() {
                let head = document.getElementsByTagName("head")[0];

                let script = document.createElement("script");
                script.charset = "utf-8";
                script.type = "text/javascript";
                script.async = true;
                script.id = this.captchaScript;
                script.src = this.captchaScript;
                script.loadJS = "watermark";
                head.appendChild(script);

                this.checkCaptchaLoaded();
            },

            checkCaptchaLoaded(timeId = null) {
                if (window.hasOwnProperty('grecaptcha') && window.grecaptcha.hasOwnProperty('render')) {

                    this.captcahLoaded = true;

                    clearTimeout(timeId);

                    if(this.dataShow === true) {
                        this.show = true;
                    } else this.show = this.dataShow !== false;


                } else {
                    let timeId = setTimeout(() => this.checkCaptchaLoaded(timeId), 500);
                }
            },

            render() {
                this.$nextTick(() => {
                    this.captchaId = window.grecaptcha.render(this.$refs.captchaZone, this.options);
                })
            },

            verifyCallback(response) {
                this.response = response;

                this.$emit('verified', {
                    response: response,
                    captchaId: this.captchaId
                })
            }
        },

        watch: {

            show: function (newVal, oldVal) {
                if (newVal && window.hasOwnProperty('grecaptcha')) {
                    this.render()
                }
            },

            dataShow: function (val) {
                this.show = !!val;
            },

            dataReset: function (newVal, oldVal) {
                if (newVal) {
                    window.grecaptcha.reset(this.captchaId)
                }
            }

        }
    }
</script>