<template>
  <b-modal
    id="buy-modal"
    :hide-header="true"
    @change="onChange($event)"
  >
    <span
      v-if="withPin"
      class="badge badge-pill badge-dialog"
      :class="{'item-selected-badge': isPinned}"
      @click.prevent.stop="togglePinned()"
    >
      <span
        class="svg-icon inline color icon-16"
        v-html="icons.pin"
      ></span>
    </span>
    <div>
      <span
        class="svg-icon icon-12 close-icon"
        aria-hidden="true"
        @click="hideDialog()"
        v-html="icons.close"
      ></span>
    </div>
    <div
      v-if="item != null"
      class="content"
    >
      <div class="inner-content">
        <slot
          name="item"
          :item="item"
        >
          <div v-if="showAvatar">
            <avatar
              :show-visual-buffs="false"
              :member="user"
              :avatar-only="true"
              :hide-class-badge="true"
              :with-background="true"
              :override-avatar-gear="getAvatarOverrides(item)"
              :sprites-margin="'0px auto 0px -24px'"
            />
          </div>
          <item
            v-else-if="item.key != 'gem'"
            class="flat bordered-item"
            :item="item"
            :item-content-class="item.class"
            :show-popover="false"
          />
        </slot>
        <h4 class="title">
          {{ itemText }}
        </h4>
        <div v-html="itemNotes"></div>
        <slot
          name="additionalInfo"
          :item="item"
        >
          <equipmentAttributesGrid
            v-if="showAttributesGrid"
            class="attributesGrid"
            :item="item"
            :user="user"
          />
        </slot>
        <div
          v-if="item.value > 0"
          class="purchase-amount"
        >
          <div
            v-if="showAmountToBuy(item)"
            class="how-many-to-buy"
          >
            <strong>{{ $t('howManyToBuy') }}</strong>
          </div>
          <div v-if="showAmountToBuy(item)">
            <div class="box">
              <input
                v-model.number="selectedAmountToBuy"
                type="number"
                min="0"
                step="1"
              >
            </div>
            <span :class="{'notEnough': notEnoughCurrency}">
              <span
                class="svg-icon inline icon-32"
                aria-hidden="true"
                v-html="icons[getPriceClass()]"
              ></span>
              <span
                class="cost"
                :class="getPriceClass()"
              >{{ item.value }}</span>
            </span>
          </div>
          <div
            v-else
            class="d-flex align-items-middle"
          >
            <span
              class="svg-icon inline icon-32 ml-auto my-auto"
              aria-hidden="true"
              v-html="icons[getPriceClass()]"
            ></span>
            <span
              class="cost mr-auto my-auto"
              :class="getPriceClass()"
            >{{ item.value }}</span>
          </div>
        </div>
        <div
          v-if="item.key === 'gem'"
          class="gems-left"
        >
          <strong v-if="gemsLeft > 0">{{ gemsLeft }} {{ $t('gemsRemaining') }}</strong>
          <strong v-if="gemsLeft === 0">{{ $t('maxBuyGems') }}</strong>
        </div>
        <div v-if="attemptingToPurchaseMoreGemsThanAreLeft">
          {{ $t('notEnoughGemsToBuy') }}
        </div>
        <div
          v-if="nonSubscriberHourglasses"
          class="hourglass-nonsub mt-3"
        >
          {{ $t('mysticHourglassNeededNoSub') }}
        </div>
        <button
          v-if="getPriceClass() === 'gems'
            && !enoughCurrency(getPriceClass(), item.value * selectedAmountToBuy)"
          class="btn btn-primary"
          @click="purchaseGems()"
        >
          {{ $t('purchaseGems') }}
        </button>
        <button
          v-else-if="nonSubscriberHourglasses"
          class="btn btn-primary"
          @click="viewSubscriptions(item)"
        >
          {{ $t('viewSubscriptions') }}
        </button>
        <button
          v-else
          class="btn btn-primary"
          :disabled="item.key === 'gem' && gemsLeft === 0 ||
            attemptingToPurchaseMoreGemsThanAreLeft || numberInvalid"
          :class="{'notEnough': !preventHealthPotion ||
            !enoughCurrency(getPriceClass(), item.value * selectedAmountToBuy)}"
          @click="buyItem()"
        >
          {{ $t('buyNow') }}
        </button>
      </div>
    </div>
    <div
      v-if="item.event && item.owned == null"
      class="limitedTime"
    >
      <span
        class="svg-icon inline icon-16 clock-icon"
        v-html="icons.clock"
      ></span>
      <span class="limitedString">{{ limitedString }}</span>
    </div>
    <div
      v-if="item.key === 'rebirth_orb' && item.value > 0 && user.stats.lvl >= 100"
      class="free-rebirth d-flex align-items-center"
    >
      <div class="m-auto">
        <span
          class="svg-icon inline icon-16 mr-2 pt-015"
          v-html="icons.whiteClock"
        ></span>
        <span v-html="$t('nextFreeRebirth', {days: nextFreeRebirth})"></span>
      </div>
    </div>
    <div
      slot="modal-footer"
      class="d-flex"
    >
      <span class="balance mr-auto">{{ $t('yourBalance') }}</span>
      <balanceInfo
        class="ml-auto"
        :currency-needed="getPriceClass()"
        :amount-needed="item.value"
      />
    </div>
  </b-modal>
</template>

<style lang="scss">
  @import '~@/assets/scss/colors.scss';
  @import '~@/assets/scss/modal.scss';

  #buy-modal {
    @include centeredModal();

    .modal-body {
      padding-bottom: 0px;
    }

    .modal-dialog {
      width: 330px;
    }

    .avatar {
      cursor: default;
      margin: 0 auto;
    }

    .content {
      text-align: center;
    }

    .item-wrapper {
      margin-bottom: 0 !important;
    }

    .inner-content {
      margin: 33px auto auto;
      width: 282px;
    }

    .purchase-amount {
      margin-top: 24px;

      .how-many-to-buy {
        margin-bottom: 16px;
      }

      .box {
        display: inline-block;
        width: 74px;
        height: 40px;
        border-radius: 2px;
        background-color: $white;
        box-shadow: 0 2px 2px 0 rgba(26, 24, 29, 0.16), 0 1px 4px 0 rgba(26, 24, 29, 0.12);
        margin-right: 24px;

        input {
          width: 100%;
          border: none;
        }

        input::-webkit-contacts-auto-fill-button {
          visibility: hidden;
          display: none !important;
          pointer-events: none;
          position: absolute;
          right: 0;
        }
      }
    }

    span.svg-icon.inline.icon-32 {
      height: 32px;
      width: 32px;

      margin-right: 8px;

      vertical-align: middle;
    }

    .cost {
      width: 28px;
      height: 32px;
      font-size: 24px;
      font-weight: bold;
      line-height: 1.33;

      vertical-align: middle;

      &.gems {
        color: $gems-color;
      }

      &.gold {
        color: $gold-color;
      }

      &.hourglasses {
        color: $hourglass-color;
      }
    }

    button.btn.btn-primary {
      margin-top: 24px;
      margin-bottom: 24px;
      min-width: 6rem;
    }

    .balance {
      width: 74px;
      height: 16px;
      font-size: 12px;
      font-weight: bold;
      line-height: 1.33;
      color: $gray-200;
    }

    .modal-footer {
      height: 48px;
      background-color: $gray-700;
      border-bottom-right-radius: 8px;
      border-bottom-left-radius: 8px;
      display: block;
    }

    .badge-dialog {
      color: $gray-300;
      position: absolute;
      left: -14px;
      padding: 8px 10px;
      top: -12px;
      background: white;
      cursor: pointer;

      &.item-selected-badge {
        background: $purple-300;
        color: $white;
      }
    }

    .notEnough {
      pointer-events: none;
      opacity: 0.55;
    }

    .limitedTime {
      height: 32px;
      background-color: $purple-300;
      width: calc(100% + 30px);
      margin: 0 -15px; // the modal content has its own padding

      font-size: 12px;
      line-height: 1.33;
      text-align: center;
      color: $white;

      display: flex;
      align-items: center;
      justify-content: center;

      .limitedString {
        height: 16px;
        margin-left: 8px;
      }
    }

    .attributesGrid {
      margin-top: 8px;
      border-radius: 2px;
      background-color: $gray-500;

      margin: 10px 0 24px;
    }

    .gems-left {
      margin-top: .5em;
    }

    .free-rebirth {
      background-color: $yellow-5;
      color: $white;
      height: 2rem;
      line-height: 16px;
      margin: auto -1rem -1rem;
    }

    .pt-015 {
      padding-top: 0.15rem;
    }
  }
</style>

<style lang="scss" scoped>
  @import '~@/assets/scss/colors.scss';

  .close-icon {
    position: absolute;
    top: 1rem;
    right: 1rem;
  }

  .hourglass-nonsub {
    color: $yellow-5;
    font-size: 12px;
  }
</style>

<script>
import keys from 'lodash/keys';
import reduce from 'lodash/reduce';
import moment from 'moment';
import * as Analytics from '@/libs/analytics';
import spellsMixin from '@/mixins/spells';
import planGemLimits from '@/../../common/script/libs/planGemLimits';
import numberInvalid from '@/mixins/numberInvalid';

import svgClose from '@/assets/svg/close.svg';
import svgGold from '@/assets/svg/gold.svg';
import svgGem from '@/assets/svg/gem.svg';
import svgHourglasses from '@/assets/svg/hourglass.svg';
import svgPin from '@/assets/svg/pin.svg';
import svgClock from '@/assets/svg/clock.svg';
import svgWhiteClock from '@/assets/svg/clock-white.svg';

import BalanceInfo from './balanceInfo.vue';
import currencyMixin from './_currencyMixin';
import notifications from '@/mixins/notifications';
import buyMixin from '@/mixins/buy';

import { mapState } from '@/libs/store';

import EquipmentAttributesGrid from '../inventory/equipment/attributesGrid.vue';

import Item from '@/components/inventory/item';
import Avatar from '@/components/avatar';

import seasonalShopConfig from '@/../../common/script/libs/shops-seasonal.config';
import { drops as dropEggs } from '@/../../common/script/content/eggs';


const dropEggKeys = keys(dropEggs);

const hideAmountSelectionForPurchaseTypes = [
  'gear', 'backgrounds', 'mystery_set', 'card',
  'rebirth_orb', 'fortify', 'armoire', 'keys',
  'debuffPotion', 'pets', 'mounts',
];

export default {
  components: {
    BalanceInfo,
    EquipmentAttributesGrid,
    Item,
    Avatar,
  },
  mixins: [buyMixin, currencyMixin, notifications, numberInvalid, spellsMixin],
  props: {
    item: {
      type: Object,
    },
    priceType: {
      type: String,
    },
    withPin: {
      type: Boolean,
    },
    genericPurchase: {
      type: Boolean,
      default: true,
    },
  },
  data () {
    return {
      icons: Object.freeze({
        close: svgClose,
        gold: svgGold,
        gems: svgGem,
        hourglasses: svgHourglasses,
        pin: svgPin,
        clock: svgClock,
        whiteClock: svgWhiteClock,
      }),

      selectedAmountToBuy: 1,
      isPinned: false,
    };
  },
  computed: {
    ...mapState({ user: 'user.data' }),
    showAvatar () {
      return ['backgrounds', 'gear', 'mystery_set'].includes(this.item.purchaseType);
    },

    preventHealthPotion () {
      if (this.item.key === 'potion' && this.user.stats.hp >= 50) {
        return false;
      }

      return true;
    },

    showAttributesGrid () {
      return this.item.purchaseType === 'gear';
    },

    itemText () {
      if (this.item.text instanceof Function) {
        return this.item.text();
      }
      return this.item.text;
    },
    itemNotes () {
      if (this.item.notes instanceof Function) {
        return this.item.notes();
      }
      return this.item.notes;
    },
    limitedString () {
      return this.$t('limitedOffer', { date: moment(seasonalShopConfig.dateRange.end).format('LL') });
    },
    gemsLeft () {
      if (!this.user.purchased.plan) return 0;
      return planGemLimits.convCap
        + this.user.purchased.plan.consecutive.gemCapExtra - this.user.purchased.plan.gemsBought;
    },
    attemptingToPurchaseMoreGemsThanAreLeft () {
      if (this.item && this.item.key && this.item.key === 'gem' && this.selectedAmountToBuy > this.gemsLeft) return true;
      return false;
    },
    notEnoughCurrency () {
      return !this.enoughCurrency(this.getPriceClass(), this.item.value * this.selectedAmountToBuy);
    },
    nextFreeRebirth () {
      return 45 - moment().diff(moment(this.user.flags.lastFreeRebirth), 'days');
    },
    nonSubscriberHourglasses () {
      return (!this.user.purchased.plan.customerId && !this.user.purchased.plan.consecutive.trinkets && this.getPriceClass() === 'hourglasses');
    },
  },
  watch: {
    item: function itemChanged () {
      this.isPinned = this.item && this.item.pinned;
      this.selectedAmountToBuy = 1;
    },
  },
  methods: {
    onChange ($event) {
      this.$emit('change', $event);
    },
    buyItem () {
      // @TODO: I  think we should buying to the items.
      // Turn the items into classes, and use polymorphism
      if (this.item.buy) {
        this.item.buy();
        this.$emit('buyPressed', this.item);
        this.hideDialog();
        return;
      }

      if (
        this.item.pinType === 'premiumHatchingPotion'
        || (this.item.pinType === 'eggs' && dropEggKeys.indexOf(this.item.key) === -1)
      ) {
        let petsRemaining = 20 - this.selectedAmountToBuy;
        petsRemaining -= reduce(this.user.items.pets, (sum, petValue, petKey) => {
          if (petKey.indexOf(this.item.key) !== -1 && petValue > 0) return sum + 1;
          return sum;
        }, 0);
        petsRemaining -= reduce(this.user.items.mounts, (sum, mountValue, mountKey) => {
          if (mountKey.indexOf(this.item.key) !== -1 && mountValue === true) return sum + 1;
          return sum;
        }, 0);
        if (this.item.pinType === 'premiumHatchingPotion') {
          petsRemaining -= this.user.items.hatchingPotions[this.item.key] + 2 || 2;
        } else {
          petsRemaining -= this.user.items.eggs[this.item.key] || 0;
        }

        if (
          petsRemaining < 0
          && !window.confirm(this.$t('purchasePetItemConfirm', { itemText: this.item.text }))
        ) return;
      }

      const shouldConfirmPurchase = this.item.currency === 'gems' || this.item.currency === 'hourglasses';
      if (
        shouldConfirmPurchase
        && !this.confirmPurchase(this.item.currency, this.item.value * this.selectedAmountToBuy)
      ) {
        return;
      }

      if (this.genericPurchase) {
        this.makeGenericPurchase(this.item, 'buyModal', this.selectedAmountToBuy);
        this.purchased(this.item.text);
      }

      this.$emit('buyPressed', this.item);
      this.hideDialog();

      if (this.item.key === 'rebirth_orb') {
        window.location.reload(true);
      }
    },
    purchaseGems () {
      if (this.item.key === 'rebirth_orb') {
        Analytics.track({
          hitType: 'event',
          eventCategory: 'button',
          eventAction: 'click',
          eventLabel: 'Gems > Rebirth',
        });
      }
      this.$root.$emit('bv::show::modal', 'buy-gems');
    },
    togglePinned () {
      this.isPinned = this.$store.dispatch('user:togglePinnedItem', { type: this.item.pinType, path: this.item.path });

      if (!this.isPinned) {
        this.text(this.$t('unpinnedItem', { item: this.item.text }));
      }
    },
    hideDialog () {
      this.$root.$emit('bv::hide::modal', 'buy-modal');
    },
    getPriceClass () {
      if (this.priceType && this.icons[this.priceType]) {
        return this.priceType;
      } if (this.item.currency && this.icons[this.item.currency]) {
        return this.item.currency;
      }
      return 'gold';
    },
    showAmountToBuy (item) {
      if (hideAmountSelectionForPurchaseTypes.includes(item.purchaseType)) {
        return false;
      }
      return true;
    },
    getAvatarOverrides (item) {
      switch (item.purchaseType) { // eslint-disable-line default-case
        case 'gear':
          return {
            [item.type]: item.key,
          };
        case 'backgrounds':
          return {
            background: item.key,
          };
        case 'mystery_set': {
          const gear = {};

          item.items.forEach(setItem => {
            gear[setItem.type] = setItem.key;
          });

          return gear;
        }
      }

      return {};
    },
    viewSubscriptions (item) {
      if (item.purchaseType === 'backgrounds') {
        this.$root.$emit('bv::hide::modal', 'avatar-modal');
        let removeIndex = this.$store.state.modalStack
          .map(modal => modal.modalId)
          .indexOf('avatar-modal');
        if (removeIndex >= 0) {
          this.$store.state.modalStack.splice(removeIndex, 1);
        }
        removeIndex = this.$store.state.modalStack
          .map(modal => modal.prev)
          .indexOf('avatar-modal');
        if (removeIndex >= 0) {
          delete this.$store.state.modalStack[removeIndex].prev;
        }
      }
      this.$router.push('/user/settings/subscription');
      this.hideDialog();
    },
  },
};
</script>
