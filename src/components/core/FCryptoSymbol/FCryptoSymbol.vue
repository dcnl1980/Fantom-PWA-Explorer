<template>
    <div class="f-crypto-symbol">
        <span class="img" :style="{ width: imgWidth, height: imgHeight }">
            <icon v-if="svgIcon" :data="svgIcon" original />
            <img v-else-if="token.logoUrl || token.logoURL" :src="token.logoUrl || token.logoURL" class="not-fluid" :alt="$defi.getTokenSymbol(token)" />
        </span>
        <span v-if="!noSymbol">{{ $defi.getTokenSymbol(token) }}</span>
    </div>
</template>

<script>
import {NATIVE_TOKEN, WRAPPED_NATIVE_TOKEN} from '../../../utils/transactions';
import nativeIcon from '../../../assets/svg/tokens/NATIVE.svg';
import wrappedNativeIcon from '../../../assets/svg/tokens/WNATIVE.svg';

/**
 * Render crypto logo and name by given token.
 * Requires vue defi plugin ($defi).
 */
export default {
    name: 'FCryptoSymbol',

    props: {
        /** @type {DefiToken} */
        token: {
            type: Object,
            default() {
                return null;
            },
            required: true,
        },
        imgWidth: {
            type: String,
            default: '40px',
        },
        imgHeight: {
            type: String,
            default: '40px',
        },
        noSymbol: {
            type: Boolean,
            default: false,
        }
    },

    data() {
        return {
            logoUrl: this.token ? this.token.logoUrl || this.token.logoURL : '',
        };
    },

    computed: {
        svgIcon() {
            const { token } = this;

            if (token) {
                switch (token.symbol) {
                    case NATIVE_TOKEN:
                        return nativeIcon;
                    case WRAPPED_NATIVE_TOKEN:
                        return wrappedNativeIcon;
                }
            }

            return null;
        },
    },
};
</script>

<style lang="scss">
@import 'style';
</style>
