<template>
    <div class="funiswap-swap funiswap">
        <f-card>
            <div class="funiswap__box">
                <div class="funiswap__token__balance">
                    <span>
                        From <template v-if="showFromEstimated">(estimated)</template>
                        <!--<pulse-loader color="#1969ff" size="10px" :loading="fromValueLoading"></pulse-loader>-->
                    </span>
                    <span class="balance">
                        Balance:
                        <f-token-value
                            :token="fromToken"
                            :value="fromTokenBalance"
                            :use-placeholder="false"
                            :add-decimals="addDeciamals"
                            no-currency
                        />
                    </span>
                </div>
                <div class="funiswap__token__body">
                    <div class="funiswap__token__sign">-</div>
                    <span>
                        <input
                            :id="`text-input-${id}`"
                            ref="fromInput"
                            type="text"
                            inputmode="decimal"
                            autocomplete="off"
                            placeholder="0"
                            class="text-input no-style"
                            @input="onFromInput"
                            @keydown="onInputKeydown"
                        />
                    </span>
                    <button class="btn small secondary max-amount" @click="onMaxAmountClick">Max</button>
                    <f-select-button
                        collapsed
                        aria-label="pick a token"
                        class="bigger-arrow"
                        @click.native="onFromTokenSelectorClick"
                    >
                        <f-crypto-symbol :token="fromToken" img-width="24px" img-height="auto" />
                    </f-select-button>
                </div>
            </div>

            <div class="funiswap__swap-cont">
                <button class="btn round same-size light" title="Swap Tokens" @click="swapTokens">
                    <icon data="@/assets/svg/arrow-left.svg" width="12" height="12" dir="left" aria-hidden="true" />
                </button>
            </div>

            <div class="funiswap__box">
                <div class="funiswap__token__balance">
                    <span>
                        To <template v-if="showToEstimated">(estimated)</template>
                        <!--<pulse-loader color="#1969ff" size="10px" :loading="toValueLoading"></pulse-loader>-->
                    </span>
                    <span class="balance">
                        Balance:
                        <f-token-value
                            :token="toToken"
                            :value="toTokenBalance"
                            :use-placeholder="false"
                            :add-decimals="addDeciamals"
                            no-currency
                        />
                    </span>
                </div>
                <div class="funiswap__token__body">
                    <div class="funiswap__token__sign">+</div>
                    <span>
                        <input
                            :id="`text-input-${id}-2`"
                            ref="toInput"
                            type="text"
                            inputmode="decimal"
                            autocomplete="off"
                            placeholder="0"
                            class="text-input no-style"
                            @input="onToInput"
                            @keydown="onInputKeydown"
                        />
                    </span>
                    <f-select-button
                        v-if="toToken.address"
                        collapsed
                        aria-label="pick a token"
                        class="bigger-arrow"
                        @click.native="onToTokenSelectorClick"
                    >
                        <f-crypto-symbol :token="toToken" img-width="24px" img-height="auto" />
                    </f-select-button>
                    <button
                        v-else
                        class="btn small secondary funiswap__select-token-btn"
                        type="button"
                        @click="onToTokenSelectorClick"
                    >
                        Select a token <icon data="@/assets/svg/chevron-down.svg" width="12" height="12" />
                    </button>
                </div>
            </div>

            <template v-if="showPriceInfo">
                <div class="funiswap-swap__exchange-price">
                    <div class="funiswap-swap__exchange-price__row">
                        <div class="defi-label">Price</div>
                        <div class="value">
                            <f-placeholder :content-loaded="!!perPrice" replacement-text="000.0000 fUSD per fETH">
                                {{ perPrice }}
                            </f-placeholder>
                        </div>
                        <div class="swap-price">
                            <button class="btn light same-size round" @click="swapPerPrice">
                                <icon data="@/assets/svg/exchange-alt.svg" />
                            </button>
                        </div>
                    </div>
                    <div class="funiswap-swap__exchange-price__row">
                        <div class="defi-label">Slippage Tolerance</div>
                        <div class="value">{{ slippageTolerance * 100 }}%</div>
                    </div>
                </div>
            </template>

            <div class="funiswap__submit-cont">
                <button ref="submitBut" class="btn large" :disabled="submitBtnDisabled" @click="onSubmit">
                    {{ submitLabel }}
                </button>
            </div>
        </f-card>

        <transition name="scale-center">
            <div v-if="showPriceInfo" class="funiswap__bottom-box">
                <div class="row no-vert-col-padding no-collapse">
                    <div class="col defi-label">
                        <template v-if="minimumReceived > 0">Minimum Received</template>
                        <template v-else>Maximum Sold</template>
                        <f-info window-closeable window-class="light" icon-size="16" class="uniswap-f-info">
                            Your transaction will revert if there is a large, unfavorable price movement before it is
                            confirmed.
                        </f-info>
                    </div>
                    <div class="col align-right">
                        <!--<f-token-value ref="minimumReceived" :value="0" :token="toToken" />-->
                        <f-token-value v-if="minimumReceived > 0" :value="minimumReceived" :token="toToken" />
                        <f-token-value v-else :value="maximumSold" :token="fromToken" />
                    </div>
                </div>
                <div class="row no-vert-col-padding no-collapse">
                    <div class="col defi-label">
                        Price Impact
                        <f-info window-closeable window-class="light" icon-size="16" class="uniswap-f-info">
                            The difference between the market price and estimated price due to trade size.
                        </f-info>
                    </div>
                    <div class="col align-right">{{ priceImpact }}</div>
                </div>
                <div class="row no-vert-col-padding no-collapse">
                    <div class="col defi-label">
                        Liquidity Provider Fee
                        <f-info window-closeable window-class="light" icon-size="16" class="uniswap-f-info">
                            A portion of each trade ({{ liquidityProviderFee * 100 }}%) goes to liquidity providers as a
                            protocol incentive.
                        </f-info>
                    </div>
                    <div class="col align-right">
                        <f-token-value
                            :value="fromValue_ * liquidityProviderFee"
                            :token="fromToken"
                            :add-decimals="addDeciamals"
                        />
                    </div>
                </div>
            </div>
        </transition>

        <defi-token-picker-window
            ref="pickFromTokenWindow"
            :tokens="fromTokens"
            @defi-token-picked="onFromTokenPicked"
        />
        <defi-token-picker-window ref="pickToTokenWindow" :tokens="toTokens" @defi-token-picked="onToTokenPicked" />
    </div>
</template>

<script>
import { mapGetters } from 'vuex';
import FCryptoSymbol from '../../components/core/FCryptoSymbol/FCryptoSymbol.vue';
import FSelectButton from '../../components/core/FSelectButton/FSelectButton.vue';
import DefiTokenPickerWindow from '../../components/windows/DefiTokenPickerWindow/DefiTokenPickerWindow.vue';
import FTokenValue from '@/components/core/FTokenValue/FTokenValue.vue';
import FCard from '@/components/core/FCard/FCard.vue';
import FInfo from '@/components/core/FInfo/FInfo.vue';
// import PulseLoader from 'vue-spinner/src/PulseLoader.vue';
import Web3 from 'web3';
import { debounce, defer, getUniqueId } from '../../utils';
import { pollingMixin } from '@/mixins/polling.js';
import FPlaceholder from '@/components/core/FPlaceholder/FPlaceholder.vue';

export default {
    name: 'FUniswapSwap',

    components: {
        FPlaceholder,
        FInfo,
        FCard,
        FTokenValue,
        DefiTokenPickerWindow,
        FSelectButton,
        FCryptoSymbol,
    },

    mixins: [pollingMixin],

    props: {
        slippageTolerance: {
            type: Number,
            default: 0.005,
        },
    },

    data() {
        return {
            fromValue: '',
            toValue: '',
            fromValue_: 0,
            toValue_: 0,
            toTokenPrice: 0,
            fromTokenPrice: 0,
            minimumReceived: 0,
            maximumSold: 0,
            // perPrice: 0,
            /** Per price direction. true - from -> to, false - to -> from */
            perPriceDirF2T: true,
            submitBtnDisabled: true,
            fromValueLoading: false,
            toValueLoading: false,
            priceImpact: '0%',
            /** @type {DefiToken} */
            fromToken: {},
            /** @type {DefiToken} */
            toToken: {},
            /** @type {DefiToken[]} */
            tokens: [],
            sliderLabels: ['0%', '25%', '50%', '75%', '100%'],
            id: getUniqueId(),
            liquidityProviderFee: 0.003,
            submitLabel: 'Enter an amount',
            dPair: {},
            addDeciamals: 2,
        };
    },

    computed: {
        ...mapGetters(['currentAccount']),

        /**
         * @return {{fromToken: DefiToken, toToken: DefiToken}}
         */
        params() {
            const { $route } = this;

            return $route && $route.params ? $route.params : {};
        },

        fromInputValue() {
            return this.formatFromInputValue(this.fromValue);
        },

        fromTokens() {
            return this.getPickerTokens('from');
        },

        toTokens() {
            return this.getPickerTokens('to');
        },

        fromTokenBalance() {
            const { fromToken } = this;
            let balance =
                this.$defi.fromTokenValue(fromToken.availableBalance, fromToken) - (fromToken.symbol === 'FTM' ? 2 : 0);

            if (balance < 0) {
                balance = 0;
            }

            return balance;
        },

        toTokenBalance() {
            return this.$defi.fromTokenValue(this.toToken.availableBalance, this.toToken);
        },

        showFromEstimated() {
            return this.toToken.address && this.toValue_ > 0 && this.maximumSold > 0;
        },

        showToEstimated() {
            return this.toToken.address && this.toValue_ > 0 && this.minimumReceived > 0;
        },

        maxFromInputValue() {
            /*
            if (this.fromToken.symbol === 'FUSD') {
                // subtract 0.5% fee
                max = this.fromTokenBalance - this.fromTokenBalance * 0.005;
            } else {
            */
            // }

            return this.fromTokenBalance;
        },

        maxToInputValue() {
            return Number.MAX_SAFE_INTEGER;
        },

        submitDisabled() {
            return !this.currentAccount || this.correctFromInputValue(this.fromValue_) === 0;
        },

        showPriceInfo() {
            return this.toToken.address && this.toValue_ > 0;
        },

        perPrice() {
            const fromToken = this.perPriceDirF2T ? this.fromToken : this.toToken;
            const toToken = this.perPriceDirF2T ? this.toToken : this.fromToken;

            if (!fromToken.address || !toToken.address) {
                return '';
            }

            const perPrice = this.perPriceDirF2T ? this.fromTokenPrice : this.toTokenPrice;
            const { $defi } = this;

            return `${perPrice.toFixed(4)} ${$defi.getTokenSymbol(fromToken)} per ${$defi.getTokenSymbol(toToken)}`;
        },
    },

    watch: {
        fromValue(_value, _oldValue) {
            if (_value !== _oldValue) {
                this.fromValue_ = !_value ? 0 : parseFloat(_value);

                this._fromValueChanged();
            }
        },

        toValue(_value, _oldValue) {
            if (_value !== _oldValue) {
                this.toValue_ = !_value ? 0 : parseFloat(_value);

                this._toValueChanged();
            }
        },

        async fromToken(_value, _oldValue) {
            if (_value !== _oldValue) {
                if (_value.address && this.toToken.address) {
                    const dPair = await this.getUniswapPair();

                    if (dPair.pairAddress !== this.dPair.pairAddress) {
                        this.dPair = dPair;
                    }

                    defer(() => {
                        this.updateSubmitLabel();
                    });
                }
            }
        },

        async toToken(_value, _oldValue) {
            if (_value !== _oldValue) {
                if (_value.address && this.fromToken.address) {
                    const dPair = await this.getUniswapPair();

                    if (dPair.pairAddress !== this.dPair.pairAddress) {
                        this.dPair = dPair;
                    }
                }
            }
        },

        currentAccount(_value, _oldValue) {
            if (_value !== _oldValue) {
                this.onAccountPicked();
            }
        },
    },

    created() {
        this.init();

        if (!this.currentAccount) {
            this.submitLabel = 'Connect Wallet';
        }

        this._fromValueChangedDebounced = debounce(() => {
            return this.fromValueChanged();
        }, 250);

        this._toValueChangedDebounced = debounce(() => {
            return this.toValueChanged();
        }, 250);
    },

    mounted() {
        this.$refs.submitBut.disabled = true;

        this._polling.start(
            'update-funiswap-swap-prices',
            () => {
                if (this.showFromEstimated) {
                    this.toValueChanged();
                } else if (this.showToEstimated) {
                    this.fromValueChanged();
                }
            },
            4000
        );
    },

    methods: {
        _fromValueChanged(_value) {
            return this._fromValueChangedDebounced(_value);
        },

        _toValueChanged() {
            return this._toValueChangedDebounced();
        },

        async fromValueChanged() {
            this.toValueLoading = true;

            this.toValue_ = await this.convertFrom2To(this.fromValue_);

            this.setTPrices();
            this.setMinMaxReceived(true);

            this.updateInputColor(this.fromValue_);
            this.updateInputColor(this.toValue_, true);
            this.updateSubmitLabel();

            this.setToInputValue(this.toValue_);
            // this.setToInputValue(this.correctToInputValue(this.toValue_));
            this.toValueLoading = false;

            this.setPriceImpact();
        },

        async toValueChanged() {
            this.fromValueLoading = true;

            this.fromValue_ = await this.convertTo2From(this.toValue_);

            this.setTPrices();
            this.setMinMaxReceived();

            this.updateInputColor(this.toValue_, true);
            this.updateInputColor(this.fromValue_);
            this.updateSubmitLabel();

            this.setFromInputValue(this.fromValue_);

            this.fromValueLoading = false;
            // this.setFromInputValue(this.correctFromInputValue(this.fromValue_));

            this.setPriceImpact();
        },

        async init() {
            const { $defi } = this;
            const { params } = this;
            const result = await Promise.all([
                $defi.fetchTokens(this.currentAccount ? this.currentAccount.address : ''),
                $defi.init(),
            ]);

            this.tokens = result[0];

            // if (params.fromToken && params.toToken) {
            if (params.fromToken) {
                this.fromToken = this.tokens.find((_item) => _item.symbol === params.fromToken.symbol);
                // this.toToken = this.tokens.find((_item) => _item.symbol === params.toToken.symbol);
            } else if (this.tokens.length >= 2) {
                this.fromToken = this.tokens[0];
                // this.toToken = this.tokens[1];
            }
        },

        async getUniswapPair() {
            const addressA = this.fromToken.address;
            const addressB = this.toToken.address;

            if (addressA && addressB) {
                return await this.$defi.fetchUniswapPairs(this.currentAccount ? this.currentAccount.address : '', '', [
                    addressA,
                    addressB,
                ]);
            }

            return {};
        },

        swapTokens() {
            const hToken = this.fromToken;
            // const hValue = this.fromValue_;

            this.fromToken = this.toToken;
            this.toToken = hToken;

            this.fromValue = 0;
            this.toValue = 0;

            // this.setFromInputValue('');
            // this.setToInputValue('');
        },

        /**
         * @param {number} _value
         */
        formatToInputValue(_value) {
            return _value !== 0 ? _value.toFixed(this.$defi.getTokenDecimals(this.toToken) + this.addDeciamals) : '';
        },

        /**
         * @param {number} _value
         */
        formatFromInputValue(_value) {
            return _value !== 0 ? _value.toFixed(this.$defi.getTokenDecimals(this.fromToken) + this.addDeciamals) : '';
        },

        /**
         * @param {number} _value
         */
        correctFromInputValue(_value) {
            return Math.min(Math.max(_value, 0), this.maxFromInputValue);
        },

        /**
         * @param {number} _value
         */
        correctToInputValue(_value) {
            return Math.min(Math.max(_value, 0), this.maxToInputValue);
        },

        /**
         * @param {number} _value
         */
        async convertFrom2To(_value) {
            const { fromToken } = this;
            const { toToken } = this;
            const value = parseFloat(_value);

            if (toToken.address && value > 0) {
                let amounts = await this.$defi.fetchUniswapAmountsOut(
                    Web3.utils.toHex(this.$defi.shiftDecPointRight(value.toString(), fromToken.decimals)),
                    [fromToken.address, toToken.address]
                );

                return this.$defi.fromTokenValue(amounts[1], toToken);
            }

            return 0;
        },

        /**
         * @param {number} _value
         */
        async convertTo2From(_value) {
            const { fromToken } = this;
            const { toToken } = this;
            const value = parseFloat(_value);

            if (toToken.address && value > 0) {
                const amounts = await this.$defi.fetchUniswapAmountsIn(
                    Web3.utils.toHex(this.$defi.shiftDecPointRight(value.toString(), toToken.decimals)),
                    [fromToken.address, toToken.address]
                );

                return this.$defi.fromTokenValue(amounts[0], fromToken);
            }

            return 0;
        },

        /**
         * Get token list for `defi-token-picker-window`.
         *
         * @type {('from'|'to')} [_type]
         * @return {DefiToken[]}
         */
        getPickerTokens(_type = 'from') {
            // const fromTokenAddress = this.fromToken.address;
            let token = _type === 'from' ? this.fromToken : this.toToken;
            let fromTokenAddress = token.address;

            // if no 'to' token is selected
            if (_type === 'to' && !this.toToken.address) {
                fromTokenAddress = this.fromToken.address;
            }

            return this.tokens.map((_item) => {
                return { ..._item, _disabled: _item.address === fromTokenAddress };
            });
        },

        resetInputValues() {
            this.fromValue = '';
            this.toValue = '';
        },

        setFromInputValue(_value) {
            defer(() => {
                this.$refs.fromInput.value = this.formatFromInputValue(_value || 0);
            });
        },

        setToInputValue(_value) {
            defer(() => {
                this.$refs.toInput.value = this.formatToInputValue(_value || 0);
            });
        },

        setMinMaxReceived(_fromValueChanged) {
            if (_fromValueChanged) {
                this.minimumReceived = this.toValue_ * (1 - this.slippageTolerance);
                this.maximumSold = 0;
            } else {
                this.maximumSold = this.fromValue_ * (1 + this.slippageTolerance);
                this.minimumReceived = 0;
            }
        },

        setTPrices() {
            this.toTokenPrice = this.toValue_ / this.fromValue_;
            this.fromTokenPrice = this.fromValue_ / this.toValue_;
        },

        async setPriceImpact() {
            // const tokenPrices = await this.$defi.fetchTokenPrices([this.fromToken.symbol, this.toToken.symbol]);
            const { dPair } = this;
            const address = this.currentAccount ? this.currentAccount.address : '';

            if (!dPair.pairAddress) {
                return;
            }

            const pair = await this.$defi.fetchUniswapPairs(address, dPair.pairAddress, [
                this.fromToken.address,
                this.toToken.address,
            ]);

            const fromTokenTotal = this.$defi.totalTokenLiquidity(this.fromToken, pair);
            const toTokenTotal = this.$defi.totalTokenLiquidity(this.toToken, pair);
            const tokenPrices = [fromTokenTotal / toTokenTotal, toTokenTotal / fromTokenTotal];
            let p1 = 0;
            let p2 = 0;
            let priceImpact = 0;

            if (this.showFromEstimated) {
                p1 = tokenPrices[0] * this.toValue_;
                p2 = this.fromTokenPrice * this.toValue_;
                priceImpact = ((p2 - p1) / p2) * 100;
            } else if (this.showToEstimated) {
                p1 = tokenPrices[1] * this.fromValue_;
                p2 = this.toTokenPrice * this.fromValue_;
                priceImpact = ((p1 - p2) / p1) * 100;
            }

            this.priceImpact = `${priceImpact.toFixed(2)}%`;
        },

        updateInputColor() {},

        /*
        updateInputColor(_value, _toInput = false) {
            const cValue = _toInput ? this.correctToInputValue(_value) : this.correctFromInputValue(_value);
            const eInput = _toInput ? this.$refs.toInput : this.$refs.fromInput;

            if (_value > cValue) {
                eInput.classList.add('invalid');
            } else {
                eInput.classList.remove('invalid');
            }
        },
        */

        updateSubmitLabel() {
            const fromValue = this.fromValue_;
            const toValue = this.toValue_;

            this.submitBtnDisabled = true;

            if (!this.currentAccount) {
                this.submitLabel = 'Connect Wallet';
            } else if (fromValue && fromValue !== '0' && toValue && toValue !== '0') {
                if (parseFloat(fromValue) > this.maxFromInputValue || parseFloat(toValue) > this.maxToInputValue) {
                    this.submitLabel = `Insufficient ${this.$defi.getTokenSymbol(this.fromToken)} balance`;
                } else {
                    this.submitLabel = 'Swap';
                    this.submitBtnDisabled = false;
                }
            } else if (fromValue && fromValue !== '0') {
                this.submitLabel = 'Select a token';
            } else {
                this.submitLabel = 'Enter an amount';
            }

            // this.$refs.submitBut.innerText = submitLabel;
            // this.$refs.submitBut.disabled = submitBtnDisabled;
        },

        swapPerPrice() {
            this.perPriceDirF2T = !this.perPriceDirF2T;
        },

        onMaxAmountClick() {
            if (this.fromValue_ !== this.maxFromInputValue) {
                this.fromValue = 0;
                defer(() => {
                    this.fromValue = this.maxFromInputValue;
                    this.setFromInputValue(this.fromValue);
                });
            }
        },

        onFromTokenSelectorClick() {
            this.$refs.pickFromTokenWindow.show();
        },

        onToTokenSelectorClick() {
            this.$refs.pickToTokenWindow.show();
        },

        /**
         * @param {DefiToken} _token Picked token.
         */
        onFromTokenPicked(_token) {
            if (_token.address === this.toToken.address) {
                this.swapTokens();
            } else {
                this.fromToken = _token;
                this.resetInputValues();
            }
        },

        /**
         * @param {DefiToken} _token Picked token.
         */
        onToTokenPicked(_token) {
            if (_token.address === this.fromToken.address) {
                this.swapTokens();
            } else {
                this.toToken = _token;
                this.fromValueChanged();
            }
        },

        /**
         * @param {InputEvent} _event
         */
        onFromInput(_event) {
            this.fromValue = _event.target.value;
        },

        /**
         * @param {InputEvent} _event
         */
        onToInput(_event) {
            this.toValue = _event.target.value;
        },

        /**
         * Prevent typing '+' or '-'.
         * @param {KeyboardEvent} _event
         */
        onInputKeydown(_event) {
            if (_event.key === '+' || _event.key === '-') {
                _event.preventDefault();
            }
        },

        onSubmit() {
            const { fromToken } = this;
            const { toToken } = this;
            // const ftmTokens = ['FTM', 'WFTM'];
            const params = {
                fromValue: this.fromValue_,
                toValue: this.toValue_,
                fromToken: { ...fromToken },
                toToken: { ...toToken },
                slippageTolerance: this.slippageTolerance,
                steps: 2,
                step: 1,
                minimumReceived: this.minimumReceived,
                maximumSold: this.maximumSold,
                max: this.maxFromInputValue === this.fromValue,
            };

            if (!this.submitDisabled) {
                this.$router.push({
                    name: 'funiswap-swap-confirmation',
                    params,
                });
            }
        },

        onAccountPicked() {
            this.init();
            this.resetInputValues();
        },
    },
};
</script>

<style lang="scss">
@import 'style';
</style>
