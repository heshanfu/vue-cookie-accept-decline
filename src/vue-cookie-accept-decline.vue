<template>
    <transition appear :name="transitionName">
        <div class="cookie" :class="containerPosition" v-if="isOpen">
            <div class="cookie__content">
                <slot class="cookie__content" name="message">{{ message }}</slot>
            </div>
            <div class="cookie__buttons">
                <button @click="decline" class="cookie__buttons__button cookie__buttons__button--decline">Opt Out</button>
                <button @click="accept" class="cookie__buttons__button cookie__buttons__button--accept">Got It!</button>
            </div>
        </div>
    </transition>
</template>

<script>
import * as tinyCookie from 'tiny-cookie'

export default {
    name: 'vue-cookie-accept-decline',
    props: {
        debug: {
            type: Boolean,
            default: false
        },

        message: {
            type: String,
            default: 'We use cookies to ensure you get the best experience on our website.'
        },

        // bottom, top
        position: {
            type: String,
            default: 'bottom'
        },

        // slideFromBottom, slideFromTop, fade
        transitionName: {
            type: String,
            default: 'slideFromBottom'
        }
    },
    data () {
        return {
            status: null,
            supportsLocalStorage: true,
            isOpen: false
        }
    },
    computed: {
        containerPosition () {
            return `cookie--${this.position}`
        }
    },
    created () {
        this.checkLocalStorageFunctionality()
        this.init()
    },
    methods: {
        init () {
            let visitedType = this.getCookieStatus()
            if (visitedType && (visitedType === 'accept' || visitedType === 'decline')) {
                this.isOpen = false
            }

            if (!visitedType) {
                this.isOpen = true
            }

            this.status = visitedType
            this.$emit('status', visitedType)
        },
        checkLocalStorageFunctionality () {
            // Check for availability of localStorage
            try {
                const test = '__vue-cookie-accept-decline-check-localStorage'
                window.localStorage.setItem(test, test)
                window.localStorage.removeItem(test)
            } catch (e) {
                console.error('Local storage is not supported, falling back to cookie use')
                this.supportsLocalStorage = false
            }
        },
        setCookieStatus (type) {
            if (this.supportsLocalStorage) {
                if (type === 'accept') {
                    localStorage.setItem('vue-cookie-accept-decline', 'accept')
                }
                if (type === 'decline') {
                    localStorage.setItem('vue-cookie-accept-decline', 'decline')
                }
            } else {
                if (type === 'accept') {
                    tinyCookie.set('vue-cookie-accept-decline', 'accept')
                }
                if (type === 'decline') {
                    tinyCookie.set('vue-cookie-accept-decline', 'decline')
                }
            }
        },
        getCookieStatus () {
            if (this.supportsLocalStorage) {
                return localStorage.getItem('vue-cookie-accept-decline')
            } else {
                return tinyCookie.get('vue-cookie-accept-decline')
            }
        },
        accept () {
            if (!this.debug) {
                this.setCookieStatus('accept')
            }

            this.status = 'accept'
            this.isOpen = false
            this.$emit('clickedAccept')
        },
        decline () {
            if (!this.debug) {
                this.setCookieStatus('decline')
            }

            this.status = 'decline'
            this.isOpen = false
            this.$emit('clickedDecline')
        }
    },
    mounted () {
    }
}
</script>

<style lang="scss" scoped>
    $light-grey: #EEEEEE;
    $green: #4caf50;
    $dark-green: darken($green, 10%);
    $red: #f44336;
    $dark-red: darken($red, 10%);
    $white: #fff;
    $black: #333;
    $transition: all 0.2s ease;

    .cookie {
        position: fixed;
        overflow: hidden;
        box-sizing: border-box;
        z-index: 9999;
        width: 100%;
        background: $light-grey;
        padding: 20px 20px;
        display: flex;
        justify-content: space-between;
        align-items: center;
        flex-direction: column;
        box-shadow: 0 -2px 2px 0 rgba($black, 0.02);
        font-size: 1rem;
        font-family: -apple-system, BlinkMacSystemFont, Roboto, Oxygen, Ubuntu, Cantarell, “Fira Sans”, “Droid Sans”, “Helvetica Neue”, Arial, sans-serif;
        line-height: 1.5;

        @media (min-width: 768px) {
            flex-direction: row;
        }

        &--bottom {
            bottom: 0;
            left: 0;
            right: 0;
        }

        &--top {
            top: 0;
            left: 0;
            right: 0;
        }

        &__content {
            margin-right: 0;
            margin-bottom: 20px;
            font-size: 0.9rem;

            @media (min-width: 768px) {
                margin-right: 10px;
                margin-bottom: 0;
            }
        }

        &__buttons {
            transition: $transition;
            display: flex;
            flex-direction: column;
            width: 100%;

            @media (min-width: 768px) {
                flex-direction: row;
                width: auto;
            }

            &__button {
                display: inline-block;
                font-weight: 400;
                text-align: center;
                white-space: nowrap;
                vertical-align: middle;
                user-select: none;
                border: 1px solid transparent;
                padding: 0.375rem 0.75rem;
                line-height: 1.5;
                border-radius: 4px;
                font-size: 0.9rem;

                &:hover {
                    cursor: pointer;
                    text-decoration: none;
                }

                &--accept {
                    background: $green;
                    color: $white;

                    &:hover {
                        background: $dark-green;
                    }
                }

                &--decline {
                    background: $red;
                    color: $white;
                    margin-bottom: 10px;

                    &:hover {
                        background: $dark-red;
                    }

                    @media (min-width: 768px) {
                        margin-bottom: 0;
                        margin-right: 10px;
                    }
                }
            }
        }
    }

    //Animations
    .slideFromTop-enter, .slideFromTop-leave-to {
        transform: translate(0px, -12.500em);
    }

    .slideFromTop-enter-to, .slideFromTop-leave {
        transform: translate(0px, 0px);
    }

    .slideFromBottom-enter, .slideFromBottom-leave-to {
        transform: translate(0px, 12.500em);
    }

    .slideFromBottom-enter-to, .slideFromBottom-leave {
        transform: translate(0px, 0px);
    }

    .slideFromBottom-enter-active,
    .slideFromBottom-leave-active,
    .slideFromTop-enter-active,
    .slideFromTop-leave-active, {
        transition: transform .4s ease-in;
    }

    .fade-enter-active, .fade-leave-active {
        transition: opacity .5s
    }
    .fade-enter, .fade-leave-to {
        opacity: 0
    }
</style>