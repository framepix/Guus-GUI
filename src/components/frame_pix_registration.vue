<template>
<div class="frame-pix-registration">
    <div class="q-pa-md">
        <i18n path="strings.framePixRegistrationDescription" tag="div" class="description q-mb-lg">
            <b place="registerCommand">register_frame_pix</b>
            <b place="prepareCommand">prepare_registration</b>
        </i18n>
        <GuusField :label="$t('fieldLabels.framePixCommand')" :error="$v.registration_string.$error" :disabled="registration_status.sending">
            <q-input
                v-model="registration_string"
                type="textarea"
                :dark="theme=='dark'"
                @blur="$v.registration_string.$touch"
                placeholder="register_frame_pix ..."
                :disabled="registration_status.sending"
                hide-underline
            />
        </GuusField>
        <q-field class="q-pt-sm">
            <q-btn color="primary" @click="register()" :label="$t('buttons.registerFramePix')" :disabled="registration_status.sending"/>
        </q-field>
    </div>

    <q-inner-loading :visible="registration_status.sending" :dark="theme=='dark'">
        <q-spinner color="primary" :size="30" />
    </q-inner-loading>
</div>
</template>

<script>
const objectAssignDeep = require("object-assign-deep");
import { mapState } from "vuex"
import { required } from "vuelidate/lib/validators"
import GuusField from "components/guus_field"
import WalletPassword from "src/mixins/wallet_password"

export default {
    name: "FramePixRegistration",
    computed: mapState({
        theme: state => state.gateway.app.config.appearance.theme,
        registration_status: state => state.gateway.frame_pix_status.registration,
    }),
    data () {
        return {
            registration_string: "",
        }
    },
    validations: {
        registration_string: { required }
    },
    watch: {
        registration_status: {
            handler(val, old){
                if(val.code == old.code) return
                switch(this.registration_status.code) {
                    case 0:
                        this.$q.notify({
                            type: "positive",
                            timeout: 1000,
                            message: this.registration_status.message
                        })
                        this.$v.$reset();
                        this.registration_string = ""
                        break;
                    case -1:
                        this.$q.notify({
                            type: "negative",
                            timeout: 3000,
                            message: this.registration_status.message
                        })
                        break;
                }
            },
            deep: true
        },
    },
    methods: {
        register: function() {
            this.$v.registration_string.$touch()

            if (this.$v.registration_string.$error) {
                this.$q.notify({
                    type: "negative",
                    timeout: 1000,
                    message: this.$t("notification.errors.invalidFramePixCommand")
                })
                return
            }

            this.showPasswordConfirmation({
                title: this.$t("dialog.registerFramePix.title"),
                noPasswordMessage: this.$t("dialog.registerFramePix.message"),
                ok: {
                    label: this.$t("dialog.registerFramePix.ok")
                },
            }).then(password => {
                this.$store.commit("gateway/set_snode_status", {
                    registration: {
                        code: 1,
                        message: "Registering...",
                        sending: true
                    }
                })
                this.$gateway.send("wallet", "register_frame_pix", {
                    password,
                    string: this.registration_string
                })
            }).catch(() => {
            })

        }
    },
    mixins: [WalletPassword],
    components: {
        GuusField
    }
}
</script>

<style lang="scss">
</style>
