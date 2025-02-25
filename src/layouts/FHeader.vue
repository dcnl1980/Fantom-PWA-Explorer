<template>
    <header class="f-header" :class="cCssClass">
        <div class="narrow-container">
            <div class="row no-collapse align-items-center">
                <div class="col-2">
                    <router-link to="/" class="logo" :aria-label="$t('view_home.back_to_home')"><img height="42px" src="/camino-logo-new.png" alt="" class="not-fluid"></router-link>
                </div>
                <div class="col right-col">
                    <f-navigation
                        :items="cNavigation"
                    ></f-navigation>
                    <f-dark-mode-switch ref="darkModeSwitch" />
                    <f-search-box ref="searchBox" class="small" expandable v-show="!cHomeView"></f-search-box>
                    <f-hamburger-switch
                        thickness="2"
                        two-lines
                        @hamburger-switch-on="onHamburgerSwitchOn"
                        @hamburger-switch-off="onHamburgerSwitchOff"
                    ></f-hamburger-switch>
                </div>
            </div>
        </div>

        <div class="f-drawer" @click="onDrawerClick">
            <div class="body"></div>
            <div class="footer">
                <f-social-media-links></f-social-media-links>
                <div class="copyright">
                    <a href="https://chain4travel.com/" target="_blank" rel="nofollow">©2022 Chain4Travel AG</a>
                </div>
            </div>
        </div>
    </header>
</template>

<script>
    import FNavigation from "../components/FNavigation.vue";
    import FHamburgerSwitch from "../components/FHamburgerSwitch.vue";
    import FSocialMediaLinks from "../components/FSocialMediaLinks.vue";
    import { mapState } from 'vuex';
    import FSearchBox from "../components/FSearchBox.vue";
    import FDarkModeSwitch from "@/components/FDarkModeSwitch.vue";

    /**
     * Renderes header and takes care of navigation.
     */
    export default {
        components: {
            FDarkModeSwitch,
            FSearchBox,
            FNavigation,
            FHamburgerSwitch,
            FSocialMediaLinks
        },

        data() {
            return {
                /** Is drawer visible? */
                dDrawerOn: false
            }
        },

        computed: {
            /**
             * Get navigation by current language.
             *
             * @return {array}
             */
            cNavigation() {
                const messages = this.$i18n.messages[this.$i18n.locale];

                return messages.navigation || [];
            },

            /**
             * Container's css classes.
             *
             * @retun {object}
             */
            cCssClass() {
                return {
                    'drawer-on': this.dDrawerOn
                }
            },

            /**
             * Is current route home view?
             *
             * @retun {boolean}
             */
            cHomeView() {
                return (this.$route.name === 'home');
            },

            ...mapState(['breakpoints'])
        },

        watch: {
            /**
             * Watches for vue route change.
             */
            $route() {
                this.hamburgerSwitchOff();
            },

            /**
             * Watches for `breakpoints` state change.
             *
             * @param {object} _breakpoints
             */
            breakpoints(_breakpoints) {
                const menuMobileBreakpoint = _breakpoints['menu-mobile'];

                if (menuMobileBreakpoint && !menuMobileBreakpoint.matches) {
                    this.hamburgerSwitchOff();
                }
            }
        },

        methods: {
            moveNavigationToDrawer() {
                const { $refs } = this;

                if (!this.$el) {
                    return;
                }

                const eFNavigation = this.$el.querySelector('.f-navigation');
                const eDrawer = this.$el.querySelector('.f-drawer .body');

                if (eFNavigation && eDrawer) {
                    eDrawer.appendChild(eFNavigation);

                    if ($refs.darkModeSwitch) {
                        eDrawer.prepend($refs.darkModeSwitch.$el);
                    }
                }
            },

            moveNavigationFromDrawer() {
                const { $refs } = this;

                if (!this.$el) {
                    return;
                }

                const eFNavigation = this.$el.querySelector('.f-navigation');
                const eRightCol = this.$el.querySelector('.right-col');

                if (eFNavigation && eRightCol) {
                    eRightCol.insertBefore(eFNavigation, eRightCol.firstChild);
                    if ($refs.searchBox && $refs.darkModeSwitch) {
                        $refs.searchBox.$el.parentNode.insertBefore($refs.darkModeSwitch.$el, $refs.searchBox.$el);
                    }
                    // eRightCol.appendChild(eFNavigation);
                }
            },

            hamburgerSwitchOff() {
                this.$children.forEach(_child => {
                    if (_child.$options._componentTag === 'f-hamburger-switch') {
                        _child.dOn = false;
                    }
                });
            },

            onHamburgerSwitchOn() {
                this.moveNavigationToDrawer();
                this.dDrawerOn = true;
            },

            onHamburgerSwitchOff() {
                this.dDrawerOn = false;
                setTimeout(() => {
                    this.moveNavigationFromDrawer();
                }, 300);
            },

            onDrawerClick() {
                this.hamburgerSwitchOff();
            }
        }
    }
</script>

<style lang="scss">
    .f-header {
        --f-header-background-color: #{$theme-color};
        --f-header-link-color: #{$secondary-color-lighter};

        /*position: -webkit-sticky;*/
        /*position: sticky;*/
        position: fixed;
        top: 0;
        z-index: 10;
        width: 100%;
        height: $f-header-height;
        color: #fff;
        background-color: var(--f-header-background-color);
        transition: height $transition-length ease;
        border-bottom: 1px solid transparent;

        .narrow-container {
            padding-top: 0;
            padding-bottom: 0;
            height: 100%;

            > .row {
                height: 100%;
            }
        }

        .logo {
            > img {
                max-height: 42px;
                display: block;
                margin-bottom: 0 !important;
                transition: opacity $transition-length ease;
            }
        }

        .f-hamburger-switch {
            position: fixed;
            /*top: 12px;*/
            top: 16px;
            right: 16px;
            z-index: 11;
            transform: scale(0.9);
            display: none;
        }

        .f-drawer {
            position: fixed;
            z-index: 10;
            width: 100%;
            height: 100%;
            top: 0;
            padding: 64px 8px 8px 8px;
            background-color: var(--f-header-background-color);
            transform: translateX(100%);
            transition: all $transition-length ease;
            display: none;
            flex-direction: column;

            .body {
                flex: 1;
            }

            .footer {
                .f-social-media-links {
                    text-align: center;
                }
            }

            @include links() {
                color: var(--f-header-link-color);
                transition: color $transition-length ease;
            }

            a:not(.btn):hover {
                color: #fff;
                text-decoration: none;
            }

            .copyright {
                padding-top: 8px;
                text-align: center;
            }
        }

        .f-search-box {
            color: $body-color;
        }

        .right-col {
            //text-align: end;
            display: flex;
            align-items: center;
            justify-content: flex-end;

            .f-navigation {
                margin-inline-end: 16px;
            }
        }

        .f-dark-mode-switch {
            margin-inline-end: 16px;
        }

        &.drawer-on {
            .f-drawer {
                transform: translateX(0);
            }
        }
    }

    .search-box-on {
        .f-header {
            .right-col {
                padding-inline-end: 52px;
            }
        }
    }

    @include media-max($bp-menu) {
        .f-header {
            .logo {
                position: fixed;
                /*top: 10px;*/
                top: 14px;
                left: 16px;
                z-index: 11;

                > img {
                    max-height: 30px;
                }
            }

            &.drawer-on {
                .logo {
                    > img {
                        opacity: 0.5;
                    }
                }

                .f-search-box:not(.expanded) {
                    opacity: 0.5;
                }
            }

            .f-hamburger-switch {
                display: inline-block;

                &.on {
                }
            }

            .f-drawer {
                display: flex;
            }

            .right-col {
                .f-navigation {
                    display: none;
                    margin-inline-end: 0;
                }
            }

            .f-search-box {
                position: fixed;
                z-index: 12;
                top: 8px;
                left: 50%;
                margin-inline-end: 0;
                margin-inline-start: 0;
                /*transform: scale(0.5);*/

                &.small.expandable {
                    &:not(.expanded) {
                        input:not(.def):not([type=submit]).large {
                            background-color: transparent;
                            box-shadow: none;
                        }

                        button[type="submit"] {
                            color: #fff;

                            &:hover {
                                background-color: transparent !important;
                            }
                        }
                    }
                }
            }

            .f-dark-mode-switch {
                display: none;
            }

            &.drawer-on {
                .f-navigation {
                    display: block;
                }

                .f-dark-mode-switch {
                    display: block;
                    text-align: end;
                    margin-inline-end: 8px;
                }
            }
        }
    }
</style>
