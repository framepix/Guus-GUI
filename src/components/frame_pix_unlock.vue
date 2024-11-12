<template>
<div class="frame-node-unlock" v-if="frame_pixs.length > 0">
    <div class="q-pa-md">
        <div class="q-pb-sm header">{{ $t('titles.currentlyStakedNodes') }}</div>
        <q-list class="frame-node-list" no-border>
            <q-item v-for="node in frame_pixs" :key="node.key">
                <q-item-main>
                    <q-item-tile class="ellipsis" label>{{ node.key }}</q-item-tile>
                    <q-item-tile sublabel class="non-selectable">{{ $t('strings.contribution') }}: <FormatGuus :amount="node.amount" /></q-item-tile>
                </q-item-main>
                <q-item-side>
                    <q-btn
                        color="primary"
                        size="md"
                        :label="$t('buttons.unlock')"
                        :disabled="!is_ready || unlock_status.sending"
                        @click="unlockWarning(node.key)"
                    />
                </q-item-side>
                 <q-context-menu>
                    <q-list link separator style="min-width: 150px; max-height: 300px;">
                        <q-item v-close-overlay @click.native="copyKey(node.key, $event)">
                            <q-item-main :label="$t('menuItems.copyFrameNodeKey')" />
                        </q-item>
                        <q-item v-close-overlay @click.native="openExplorer(node.key)">
                            <q-item-main :label="$t('menuItems.viewOnExplorer')" />
                        </q-item>
                    </q-list>
                </q-context-menu>
            </q-item>
        </q-list>
    </div>

    <q-inner-loading :visible="unlock_status.sending" :dark="theme=='dark'">
        <q-spinner color="primary" :size="30" />
    </q-inner-loading>
</div>
</template>

<script>
const objectAssignDeep = require("object-assign-deep");
import { mapState } from "vuex"
import { required } from "vuelidate/lib/validators"
import { frame_pix_key } from "src/validators/common"
import GuusField from "components/guus_field"
import WalletPassword from "src/mixins/wallet_password"
import FormatGuus from "components/format_guus"

export default {
    name: "FrameNodeUnlock",
    computed: mapState({
        theme: state => state.gateway.app.config.appearance.theme,
        unlock_status: state => state.gateway.frame_pix_status.unlock,
        our_address: state => {
            const primary = state.gateway.wallet.address_list.primary[0]
            return (primary && primary.address) || null
        },
        is_ready (state) {
            return this.$store.getters["gateway/isReady"]
        },
        frame_pixs (state) {
            const nodes = state.gateway.daemon.frame_pixs
            const getContribution = node => node.contributors.find(c => c.address === this.our_address)
            // Only show nodes that we contributed to
            return nodes.filter(getContribution).map(n => {
                const ourContribution = getContribution(n)
                return {
                    key: n.frame_pix_pubkey,
                    amount: ourContribution.amount
                }
            })
        }
    }),
    validations: {
        node_key: { required, frame_pix_key }
    },
    watch: {
        unlock_status: {
            handler(val, old){
                if(val.code == old.code) return
                switch(this.unlock_status.code) {
                    case 0:
                        this.key = null
                        this.password = null

                        this.$q.notify({
                            type: "positive",
                            timeout: 1000,
                            message: this.unlock_status.message
                        })
                        this.$v.$reset();
                        break;
                    case 1:
                        // Tell the user to confirm
                         this.$q.dialog({
                            title: this.$t("dialog.unlockFrameNode.confirmTitle"),
                            message: this.unlock_status.message,
                            ok: {
                                label: this.$t("dialog.unlockFrameNode.ok")
                            },
                            cancel: {
                                flat: true,
                                label: this.$t("dialog.buttons.cancel"),
                                color: this.theme=="dark"?"white":"dark"
                            }
                        }).then(() => {
                            this.gatewayUnlock(this.password, this.key, true);
                        }).catch(() => {
                        })
                        break;
                    case -1:
                        this.key = null
                        this.password = null

                        this.$q.notify({
                            type: "negative",
                            timeout: 3000,
                            message: this.unlock_status.message
                        })
                        break;
                    default: break;
                }
            },
            deep: true
        },
    },
    methods: {
        unlockWarning (key) {
            this.$q.dialog({
                title: this.$t("dialog.unlockFrameNodeWarning.title"),
                message: this.$t("dialog.unlockFrameNodeWarning.message"),
                ok: {
                    label: this.$t("dialog.unlockFrameNodeWarning.ok")
                },
                cancel: {
                    flat: true,
                    label: this.$t("dialog.buttons.cancel"),
                    color: this.theme === "dark" ? "white" : "dark"
                }
            }).then(() => {
                this.unlock(key)
            }).catch(() => {})
        },
        unlock (key) {
            // We store this as it could change between the 2 step process
            this.key = key

            this.showPasswordConfirmation({
                title: this.$t("dialog.unlockFrameNode.title"),
                noPasswordMessage: this.$t("dialog.unlockFrameNode.message"),
                ok: {
                    label: this.$t("dialog.unlockFrameNode.ok")
                },
            }).then(password => {
                this.password = password
                this.gatewayUnlock(this.password, this.key, false);
            }).catch(() => {
            })
        },
        gatewayUnlock (password, key, confirmed = false) {
            this.$store.commit("gateway/set_snode_status", {
                unlock: {
                    code: 2, // Code 1 is reserved for can_unlock
                    message: "Unlocking...",
                    sending: true
                }
            })
            this.$gateway.send("wallet", "unlock_stake", {
                password,
                frame_pix_key: key,
                confirmed
            })
        },
        copyKey (key, event) {
            event.stopPropagation()
            for(let i = 0; i < event.path.length; i++) {
                if(event.path[i].tagName == "BUTTON") {
                    event.path[i].blur()
                    break
                }
            }
            clipboard.writeText(key)
            this.$q.notify({
                type: "positive",
                timeout: 1000,
                message: this.$t("notification.positive.copied", { item: "Frame node key" })
            })
        },
        openExplorer (key) {
            this.$gateway.send("core", "open_explorer", {type: "frame_pix", id: key})
        }
    },

    mixins: [WalletPassword],
    components: {
        GuusField,
        FormatGuus
    }
}
</script>

<style lang="scss">
.frame-node-unlock {
    user-select: none;
    .header {
        font-weight: 450;
    }
    .q-item-sublabel, .q-list-header {
        font-size: 14px;
    }
}
</style>
