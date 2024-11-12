<template>
<q-page class="frame-node-page">
    <div class="header row items-center justify-center q-pt-md">
        <q-btn-toggle
            v-model="screen"
            toggle-color="primary"
            color="secondary"
            :options="[
                {label: $t('titles.frameNode.staking'), value: 'staking'},
                {label: $t('titles.frameNode.registration'), value: 'registration'}
            ]"
        />
    </div>
    <FrameNodeStaking v-if="screen === 'staking'"/>
    <FrameNodeRegistration v-if="screen === 'registration'" />
</q-page>
</template>

<script>
import FrameNodeStaking from "components/frame_pix_staking"
import FrameNodeRegistration from "components/frame_pix_registration"
export default {
    data () {
        return {
            screen: "staking",
        }
    },
    components: {
        FrameNodeStaking,
        FrameNodeRegistration
    }
}
</script>

<style lang="scss">
</style>
