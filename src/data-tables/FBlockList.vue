<template>
    <div class="block-list-dt">
        <template v-if="!dBlockListError">
            <f-data-table
                :columns="dColumns"
                :items="dItems"
                :disable-infinite-scroll="!dHasNext"
                :loading="cLoading"
                infinite-scroll
                fixed-header
                v-bind="{...$attrs, ...$props}"
                class="f-data-table-body-bg-color"
                @fetch-more="fetchMore"
            >
                <template v-slot:column-block="{ value, column }">
                    <div v-if="column" class="row no-collapse no-vert-col-padding">
                        <div class="col-4 f-row-label">{{ column.label }}</div>
                        <div class="col">
                            <router-link :to="{name: 'block-detail', params: {id: value}}" :title="value">{{value}}</router-link>
                        </div>
                    </div>
                    <template v-else>
                        <router-link :to="{name: 'block-detail', params: {id: value}}" :title="value">{{value}}</router-link>
                    </template>
                </template>

                <template v-slot:column-age="{ value, column }">
                    <div v-if="column" class="row no-collapse no-vert-col-padding">
                        <div class="col-4 f-row-label">{{ column.label }}</div>
                        <div class="col">
                            <timeago :datetime="timestampToDate(value)" :auto-update="1" :converter-options="{includeSeconds: true}"></timeago>
                        </div>
                    </div>
                    <template v-else>
                        <timeago :datetime="timestampToDate(value)" :auto-update="5" :converter-options="{includeSeconds: true}"></timeago>
                    </template>
                </template>
            </f-data-table>
        </template>

        <template v-else>
            <div class="query-error">{{ dBlockListError }}</div>
        </template>
    </div>
</template>

<script>
    // import FDataTable from "../components/FDataTable.vue";
    import FDataTable from "../components/core/FDataTable/FDataTable.vue";
    import gql from 'graphql-tag';
    import { NATIVE_TOKEN, WEIToNative } from "../utils/transactions.js";
    import {timestampToDate, formatDate, formatHexToInt} from "../filters.js";
    import {cloneObject} from "@/utils";

    export default {
        components: {
            FDataTable
        },

        props: {
            hiddenColumns: {
                ...FDataTable.props.hiddenColumns,
            },
            /** Number of items per page. */
            itemsPerPage: {
                type: Number,
                default: 40
            }
        },

        apollo: {
            blocks: {
                query: gql`
                    query BlockList($cursor: Cursor, $count: Int!) {
                        blocks (cursor: $cursor, count: $count) {
                            totalCount
                            pageInfo {
                                first
                                last
                                hasNext
                                hasPrevious
                            }
                            edges {
                                block {
                                    hash
                                    number
                                    timestamp
                                    transactionCount
                                    gasUsed
                                }
                                cursor
                            }
                        }
                    }
                `,
                variables() {
                    return {
                        cursor: null,
                        count: this.itemsPerPage
                    }
                },
                result(_data, _key) {
                    let data;


                    if (_key === 'blocks') {
                        // console.log('???');
                        data = _data.data.blocks;

                        const edges = cloneObject(data.edges);

                        this.dHasNext = data.pageInfo.hasNext;

                        if (this.dItems.length === 0) {
                            this.dItems = edges;
                        } else {
                            for (let i = 0, len1 = edges.length; i < len1; i++) {
                                this.dItems.push(edges[i]);
                            }
                        }

                        this.$emit('records-count', formatHexToInt(data.totalCount));
                    }
                },
                error(_error) {
                    this.dBlockListError = _error.message;
                }
            }
        },

        data() {
            return {
                dItems: [],
                dHasNext: false,
                dBlockListError: '',
                gasPrice: this.$store.state.gasPrice,
                dColumns: [
                    {
                        name: 'block',
                        label: this.$t('view_block_list.block'),
                        itemProp: 'block.number',
                        formatter: formatHexToInt
                    },
                    {
                        name: 'time',
                        label: this.$t('view_block_list.time'),
                        itemProp: 'block.timestamp',
                        formatter: (_value) => formatDate(timestampToDate(_value)),
                        width: '340px'
                    },
                    {
                        name: 'age',
                        label: this.$t('view_block_list.age'),
                        itemProp: 'block.timestamp'
                    },
                    {
                        name: 'fee',
                        label: `${this.$t('view_block_list.fee')} (${NATIVE_TOKEN})`,
                        itemProp: 'block.gasUsed',
                        formatter: (_value) => WEIToNative(_value * (this.gasPrice || 1500000000)),
                        // width: '80px'
                    },
                    {
                        name: 'transaction_count',
                        label: this.$t('view_block_list.transaction_count'),
                        itemProp: 'block.transactionCount',
                        width: '80px'
                    }
                ]
            }
        },

        computed: {
            cLoading() {
                return this.$apollo.queries.blocks.loading;
            }
        },

        methods: {
            fetchMore() {
                const {blocks} = this;

                if (blocks && blocks.pageInfo && blocks.pageInfo.hasNext) {
                    const cursor = blocks.pageInfo.last;

                    this.$apollo.queries.blocks.fetchMore({
                        variables: {
                            cursor,
                            count: this.itemsPerPage
                        },
                        updateQuery: (previousResult, { fetchMoreResult }) => {
                            // this.dHasNext = fetchMoreResult.blocks.pageInfo.hasNext;

                            return fetchMoreResult;
/*
                            return {
                                blocks: {
                                    ...fetchMoreResult.blocks,
                                    edges: [...previousResult.blocks.edges, ...fetchMoreResult.blocks.edges]
                                }
                            }
*/
                        }
                    });
                }
            },

            WEIToNative,
            timestampToDate
        }
    }
</script>
