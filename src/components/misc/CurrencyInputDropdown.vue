<template>
    <div>
        <div class="curr_in_drop">
            <div class="max_in_cont hover_border">
                <button class="max_but" @click="maxOut">MAX</button>
                <big-num-input
                    ref="bigIn"
                    @change="amount_in"
                    class="bigIn"
                    contenteditable="bigIn"
                    :max="max_amount"
                    :denomination="denomination"
                    :step="stepSize"
                    :placeholder="placeholder"
                ></big-num-input>
            </div>
            <BalanceDropdown
                :disabled_assets="disabled_assets"
                v-model="asset_now"
            ></BalanceDropdown>
        </div>
        <div class="balance">
            <p>
                Balance:
                <span>{{ maxAmountBig.toLocaleString(denomination) }}</span>
            </p>
            <div></div>
        </div>
    </div>
</template>
<script lang="ts">
import 'reflect-metadata'
import { Vue, Component, Prop, Emit, Watch } from 'vue-property-decorator'

import BN from 'bn.js'
// import Big from 'big.js';
import Dropdown from '@/components/misc/Dropdown.vue'
// import BigNumInput from "@/components/misc/BigNumInput";

// @ts-ignore
import { BigNumInput } from '@avalabs/vue_components'
import AvaAsset from '@/js/AvaAsset'
import { ICurrencyInputDropdownValue } from '@/components/wallet/transfer/types'
import { IWalletAssetsDict, IWalletBalanceDict } from '@/store/types'

import BalanceDropdown from '@/components/misc/BalancePopup/BalanceDropdown.vue'
import { avm } from '@/AVA'
import Big from 'big.js'
import { bnToBig } from '@/helpers/helper'
interface IDropdownValue {
    label: string
    key: string
    data: any
    disabled: boolean
}

@Component({
    components: {
        Dropdown,
        BigNumInput,
        BalanceDropdown,
    },
})
export default class CurrencyInputDropdown extends Vue {
    amount: BN = new BN(0)
    asset_now: AvaAsset = this.walletAssetsArray[0]

    @Prop({ default: () => [] }) disabled_assets!: AvaAsset[]
    @Prop({ default: '' }) initial!: string

    mounted() {
        if (this.isEmpty) return
        if (this.initial) {
            let initialAsset = this.walletAssetsDict[this.initial]
            this.drop_change(initialAsset)
        } else {
            this.drop_change(this.walletAssetsArray[0])
        }
    }

    @Watch('asset_now')
    drop_change(val: AvaAsset) {
        this.asset_now = val
        this.amount_in(new BN(0))
    }

    get stepSize() {
        return new BN(1)
    }
    maxOut() {
        // @ts-ignore
        this.$refs.bigIn.maxout()
    }

    amount_in(val: BN) {
        this.amount = val
        this.onchange()
    }

    // onchange event for the Component
    @Emit('change')
    onchange(): ICurrencyInputDropdownValue {
        return {
            asset: this.asset_now,
            amount: this.amount,
        }
    }

    onfocus() {
        console.log('focus')
    }

    get isEmpty(): boolean {
        if (this.walletAssetsArray.length === 0) {
            return true
        } else {
            return false
        }
    }

    get display(): string {
        return ''
    }

    get placeholder(): string {
        if (this.isEmpty || !this.asset_now) return '0.00'
        let deno = this.asset_now.denomination
        let res = '0'
        if (deno > 0) {
            res += '.'
            for (var i = 0; i < deno; i++) {
                res += '0'
            }
        }
        return res
    }

    get denomination(): number {
        if (!this.asset_now) return 0
        return this.asset_now.denomination
    }

    get walletAssetsArray(): AvaAsset[] {
        return this.$store.getters.walletAssetsArray
    }

    get walletAssetsDict(): IWalletAssetsDict {
        return this.$store.getters['walletAssetsDict']
    }

    get avaxAsset(): AvaAsset {
        return this.$store.getters['Assets/AssetAVA']
    }

    get max_amount(): null | BN {
        if (!this.asset_now) return null

        let assetId = this.asset_now.id
        let balance = this.walletAssetsDict[assetId]

        let avaxId = this.avaxAsset.id

        // Max amount is BALANCE - FEE for AVAX
        if (assetId === avaxId) {
            let fee = avm.getTxFee()
            // console.log(fee);
            if (fee.gte(balance.amount)) {
                return new BN(0)
            } else {
                return balance.amount.sub(fee)
            }
        }

        if (balance.amount.isZero()) return null
        return balance.amount
    }

    get maxAmountBig(): Big {
        if (!this.max_amount) return Big(0)
        return bnToBig(this.max_amount, this.denomination)
    }
}
</script>
<style scoped lang="scss">
@use '../../main';

.bigIn {
    width: 100%;
    border: none !important;
    font-size: 15px;
    font-family: monospace;
    /*background-color: #303030;*/
}

.max_in_cont {
    position: relative;
    display: grid;
    grid-template-columns: max-content 1fr;
    padding: 8px 14px;
}

.curr_in_drop {
    display: grid;
    grid-template-columns: 1fr 140px;
    background-color: transparent;
    //font-size: 12px;
    height: 40px;
    width: 100%;
    outline: none;
    text-align: right;
    column-gap: 10px;

    > * {
        background-color: var(--bg-light);
        border-radius: 2px;
    }
}

input {
    flex-grow: 1;
    outline: none;
    text-align: right;
    flex-basis: 0px;
    width: 0px;
    color: var(--primary-color);
}

.max_but {
    opacity: 0.4;
    font-size: 13px;
    &:hover {
        opacity: 1;
    }
}

.dropdown {
    /*flex-basis: 140px;*/
    width: 100%;
    /*border-left: 1px solid #d2d2d2;*/
}

.balance {
    display: grid;
    column-gap: 10px;
    grid-template-columns: 1fr 140px;
    font-size: 14px;
    color: var(--primary-color-light);
    padding: 2px 8px;

    p {
        text-align: right;
        padding: 2px 8px;
    }

    span {
        font-family: monospace;
        padding-left: 14px;
    }
}

@include main.mobile-device {
    .curr_in_drop {
        grid-template-columns: 1fr 70px;
    }

    .balance {
        grid-template-columns: 1fr 70px;
    }
}
</style>
