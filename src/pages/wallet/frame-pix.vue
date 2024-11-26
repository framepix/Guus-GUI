<template>
<q-page class="frame-pix-page">
    <div class="header row items-center justify-center q-pt-md">
        <q-btn-toggle
            v-model="screen"
            toggle-color="primary"
            color="secondary"
            :options="[
                {label: $t('titles.framePix.staking'), value: 'staking'},
                {label: $t('titles.framePix.registration'), value: 'registration'}
            ]"
        />
    </div>
    <FramePixStaking v-if="screen === 'staking'"/>
    <FramePixRegistration v-if="screen === 'registration'" />
</q-page>
</template>

<script>
import FramePixStaking from "components/frame_pix_staking"
import FramePixRegistration from "components/frame_pix_registration"
export default {
    data () {
        return {
            screen: "staking",
        }
    },
    components: {
        FramePixStaking,
        FramePixRegistration
    }
}
</script>

<style lang="scss">
</style>
