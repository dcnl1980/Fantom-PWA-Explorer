<template>
    <div class="address-asset-list-dt">
        <f-data-table
            :columns="columns"
            :items="items"
            first-m-v-column-width="6"
            class="f-data-table-body-bg-color"
        >
            <template v-slot:column-asset="{ value, item, column }">
                <div v-if="column" class="row no-collapse no-vert-col-padding">
                    <div class="col-6 f-row-label">{{ column.label }}</div>
                    <div class="col break-word">
                        <f-crypto-symbol :token="item" />
                    </div>
                </div>
                <template v-else>
                    <f-crypto-symbol :token="item" />
                </template>
            </template>

            <template v-slot:column-available="{ value, item, column }">
                <div v-if="column" class="row no-collapse no-vert-col-padding">
                    <div class="col-6 f-row-label">{{ column.label }}</div>
                    <div class="col break-word">
                        <f-token-value
                            :value="value"
                            :token="item"
                            :use-placeholder="false"
                            :max-decimals="MAX_TOKEN_DECIMALS_IN_TABLES"
                            no-currency
                        />
                    </div>
                </div>
                <template v-else>
                    <f-token-value
                        :value="value"
                        :token="item"
                        :use-placeholder="false"
                        :max-decimals="MAX_TOKEN_DECIMALS_IN_TABLES"
                        no-currency
                    />
                </template>
            </template>


            <template v-slot:column-deposited="{ value, item, column }">
                <div v-if="column" class="row no-collapse no-vert-col-padding">
                    <div class="col-6 f-row-label">{{ column.label }}</div>
                    <div class="col break-word">
                        <f-token-value
                            :value="value"
                            :token="item"
                            :use-placeholder="false"
                            :max-decimals="MAX_TOKEN_DECIMALS_IN_TABLES"
                            no-currency
                        />
                        <!-- <span class="currency-light">{{ $defi.getTokenSymbol(item) }}</span>-->
                    </div>
                </div>
                <template v-else>
                    <f-token-value
                        :value="value"
                        :token="item"
                        :use-placeholder="false"
                        :max-decimals="MAX_TOKEN_DECIMALS_IN_TABLES"
                        no-currency
                    />
                    <!-- <span class="currency-light">{{ $defi.getTokenSymbol(item) }}</span>-->
                </template>
            </template>

            <template v-slot:column-borrowed="{ value, item, column }">
                <div v-if="column" class="row no-collapse no-vert-col-padding">
                    <div class="col-6 f-row-label">{{ column.label }}</div>
                    <div class="col break-word">
                        <f-token-value
                            :value="value"
                            :token="item"
                            :use-placeholder="false"
                            :max-decimals="MAX_TOKEN_DECIMALS_IN_TABLES"
                            no-currency
                        />
                        <!-- <span class="currency-light">{{ $defi.getTokenSymbol(item) }}</span>-->
                    </div>
                </div>
                <template v-else>
                    <f-token-value
                        :value="value"
                        :token="item"
                        :use-placeholder="false"
                        :max-decimals="MAX_TOKEN_DECIMALS_IN_TABLES"
                        no-currency
                    />
                    <!-- <span class="currency-light">{{ $defi.getTokenSymbol(item) }}</span>-->
                </template>
            </template>
        </f-data-table>
    </div>
</template>

<script>
import FDataTable from '../components/core/FDataTable/FDataTable.vue';
import FCryptoSymbol from '../components/core/FCryptoSymbol/FCryptoSymbol.vue';
import { stringSort } from '../utils/array-sorting.js';
import {MAX_TOKEN_DECIMALS_IN_TABLES} from "../plugins/defi/defi.js";
import FTokenValue from "@/components/core/FTokenValue/FTokenValue.vue";
import { USD_TOKEN, WRAPPED_NATIVE_TOKEN } from '@/utils/transactions.js';

export default {
    name: 'AddressAssetList',

    components: {FTokenValue, FCryptoSymbol, FDataTable },

    props: {
        /** @type {DefiToken[]} */
        tokens: {
            type: Array,
            required: true,
            default() {
                return [];
            },
        },
        /** @type {FMintAccount} */
        fMintAccount: {
            type: Object,
            required: true,
            default() {
                return {
                    collateral: [],
                    debt: [],
                };
            },
        },
    },

    data() {
        return {
            items: [],
            /** @type {DeFi} */
            defi: this.$defi,
            columns: [
                {
                    name: 'asset',
                    label: 'Asset',
                    sortFunc: (_itemProp, _direction = 'asc') => {
                        return (_a, _b) => {
                            const a = _a.symbol;
                            const b = _b.symbol;

                            return (_direction === 'desc' ? -1 : 1) * stringSort(a, b);
                        };
                    },
                    sortDir: 'desc',
                    width: '140px',
                },
                {
                    name: 'available',
                    label: 'Available',
                    itemProp: 'availableBalance',
                    formatter: (_value, _item) => {
                        const balance = _item._availableBalance;

                        return balance > 0 ? balance : 0;
                    },
                    cssClass: 'align-end',
                    // width: '100px',
                },
                {
                    name: 'deposited',
                    label: 'Deposited',
                    itemProp: 'availableBalance',
                    formatter: (_value, _item) => {
                        const collateral = _item._deposited;

                        return collateral > 0 ? collateral : 0;
                    },
                    cssClass: 'align-end',
                    // width: '100px',
                },
                {
                    name: 'borrowed',
                    label: 'Borrowed',
                    // hidden: true,
                    formatter: (_value, _item) => {
                        const debt = _item._debt;

                        return debt > 0 ? debt : 0;
                    },
                    cssClass: 'align-end',
                },
            ],
            MAX_TOKEN_DECIMALS_IN_TABLES,
        };
    },

    watch: {
        /**
         * @param {DefiToken[]} _value
         */
        tokens(_value) {
            // this.items = _value.filter((_item) => _item.isActive && _item.canDeposit && _item.symbol !== NATIVE_TOKEN);
            this.prepareTokens(_value);

            this.items = _value.filter(
                (_token) => _token._availableBalance > 0 || _token._deposited > 0 || _token._debt > 0
            );

            this.$emit('records-count', this.items.length);
        },
    },

    methods: {
        /**
         * @param {DefiToken[]} _tokens
         */
        prepareTokens(_tokens) {
            const { $defi } = this;

            if (_tokens) {
                _tokens.forEach((_token) => {
                    const availableBalance = _token.balanceOf || _token.availableBalance;

                    if (availableBalance) {
                        _token._availableBalance = $defi.fromTokenValue(availableBalance, _token);
                    } else {
                        _token._availableBalance = 0;
                    }

                    _token._deposited = this.getCollateral(_token);
                    _token._debt = this.getDebt(_token);
                });
            }
        },

        /**
         * @param {DefiToken} _token
         * @return {*|number}
         */
        getCollateral(_token) {
            /** @type {FMintTokenBalance} */
            const tokenBalance = this.$defi.getFMintAccountCollateral(this.fMintAccount, _token);

            return this.$defi.fromTokenValue(tokenBalance.balance, _token) || 0;
        },

        /**
         * @param {DefiToken} _token
         * @return {*|number}
         */
        getDebt(_token) {
            /** @type {FMintTokenBalance} */
            const tokenBalance = this.$defi.getFMintAccountDebt(this.fMintAccount, _token);

            return this.$defi.fromTokenValue(tokenBalance.balance, _token) || 0;
        },

        /**
         * @param {DefiToken} _token
         * @return {boolean}
         */
        usedInFMint(_token) {
            return _token.symbol === WRAPPED_NATIVE_TOKEN || _token.symbol === USD_TOKEN;
        },
    },
};
</script>
