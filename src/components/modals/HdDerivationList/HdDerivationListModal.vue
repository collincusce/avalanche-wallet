<template>
    <modal ref="modal" title="HD Addresses" class="modal_main">
        <div class="hd_deriv_modal_body">
            <HDDerivationList :wallet="wallet"></HDDerivationList>
            <!--            <mnemonic-display :phrase="phrase" :row-size="3"></mnemonic-display>-->
            <!--            <p class="phrase_raw">{{phrase}}</p>-->
            <!--            <copy-text :value="phrase" class="copyBut" tooltip="Copy"></copy-text>-->
        </div>
    </modal>
</template>
<script lang="ts">
import 'reflect-metadata'
import { Vue, Component, Prop } from 'vue-property-decorator'

import Modal from '@/components/modals/Modal.vue'
// import MnemonicDisplay from "@/components/misc/MnemonicDisplay.vue";
// import CopyText from "@/components/misc/CopyText.vue";
import AvaHdWallet from '@/js/wallets/AvaHdWallet'
import HDDerivationList from '@/components/modals/HdDerivationList/HDDerivationList.vue'

@Component({
    components: {
        Modal,
        HDDerivationList,
        // MnemonicDisplay,
        // CopyText
    },
})
export default class HdDerivationList extends Vue {
    @Prop() wallet!: AvaHdWallet

    open(): void {
        let modal = this.$refs.modal as Modal
        // @ts-ignore
        modal.open()
    }
}
</script>
<style scoped lang="scss">
.hd_deriv_modal_body {
    width: 600px;
    max-width: 100%;
    padding: 0px;
}
</style>
