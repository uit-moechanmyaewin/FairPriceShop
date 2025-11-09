<template>
    <LoadingComponent :props="loading" />
    <div class="row">
        <div class="col-12 lg:col-8">
            <div class="flex items-center rounded-2xl w-fit mb-6 text-focus bg-[#EAF6FF]">
                <div class="relative cursor-pointer">
                    <input @change="changeOrderType(orderTypeEnum.DELIVERY)" id="checkout-delivery"
                        :checked="orderType === orderTypeEnum.DELIVERY" :value="orderTypeEnum.DELIVERY"
                        class="cart-switch w-full h-full absolute top-0 left-0 opacity-0 cursor-pointer" type="radio">
                    <label class="py-1.5 px-3.5 rounded-2xl text-sm font-semibold capitalize transition cursor-pointer"
                        for="checkout-delivery">{{ $t('label.delivery') }}</label>
                </div>
                <div v-if="setting.site_pick_up == enums.activityEnum.ENABLE" class="relative cursor-pointer">
                    <input @change="changeOrderType(orderTypeEnum.PICK_UP)" id="checkout-delivery"
                        :checked="orderType === orderTypeEnum.PICK_UP" :value="orderTypeEnum.PICK_UP"
                        class="cart-switch w-full h-full absolute top-0 left-0 opacity-0 cursor-pointer" type="radio">
                    <label class="py-1.5 px-3.5 rounded-2xl text-sm font-semibold capitalize transition cursor-pointer"
                        for="checkout-delivery">{{ $t('label.pick_up') }}</label>
                </div>
            </div>

            <div v-if="orderType === orderTypeEnum.PICK_UP" class="mb-6 rounded-2xl shadow-card">
                <h4 class="font-bold capitalize p-4 border-b border-gray-100">{{ $t('label.store_location') }}</h4>

                <div v-if="outlets.length > 0" v-for="outlet in outlets" class="px-4 pt-4">
                    <div class="flex p-2 border transition-all rounded-lg"
                        :class="outlet.id === modelOutlet.id ? 'border-primary/50 bg-[#E7FFF3]' : 'border-[#F7F7F7] bg-[#F7F7F7]'">
                        <input type="radio" @change="outletAddress($event)" :id="outlet.name" :name="outlet.name"
                            :value="outlet" :key="outlet" v-model="modelOutlet">
                        <label :for="outlet.name" class="px-2 text-sm capitalize cursor-pointer ">
                            <span class="font-semibold">{{ outlet.name }}</span> - {{ outlet.address }}, {{ outlet.city
                            }}, {{ outlet.state }}, {{ outlet.zip_code }}
                        </label>
                    </div>
                </div>
            </div>

            <AddressComponent v-if="orderType === orderTypeEnum.DELIVERY" :show="true"
                :selectedAddress="getDeliveryAddress" :method="deliveryAddress" />

            <div class="max-lg:hidden flex items-center justify-between gap-5 mt-10">
                <router-link :to="{ name: 'frontend.checkout.cartList' }"
                    class="field-button w-fit font-semibold tracking-wide normal-case text-secondary bg-[#F7F7FC]">
                    {{ $t('button.back_to_cart') }}
                </router-link>

                <button @click.prevent="selectAddress"
                    class="field-button w-fit font-semibold tracking-wide normal-case"
                    :class="parseFloat(getDeliveryZone.minimum_order_amount) > 0 && parseFloat(getDeliveryZone.minimum_order_amount) > subtotal && orderType === orderTypeEnum.DELIVERY ? 'bg-primary/50 pointer-events-disable text-sm' : ''">
                    {{ $t('button.save_and_pay') }}
                    <strong
                        v-if="parseFloat(getDeliveryZone.minimum_order_amount) > 0 && parseFloat(getDeliveryZone.minimum_order_amount) > subtotal && orderType === orderTypeEnum.DELIVERY"
                        class="text-red-400">
                        ({{ $t('label.min_order') + ': ' + getDeliveryZone.currency_minimum_order_amount }})
                    </strong>
                </button>
            </div>
        </div>

        <div class="col-12 lg:col-4">
            <CouponComponent />
            <SummeryComponent />

            <div class="max-lg:flex hidden flex-col-reverse sm:flex-row items-center justify-between gap-5 mt-10">
                <router-link :to="{ name: 'frontend.checkout.cartList' }"
                    class="field-button font-semibold tracking-wide normal-case text-secondary bg-[#F7F7FC]">
                    {{ $t('button.back_to_cart') }}
                </router-link>

                <button @click.prevent="selectAddress" class="field-button font-semibold tracking-wide normal-case"
                    :class="parseFloat(getDeliveryZone.minimum_order_amount) > 0 && parseFloat(getDeliveryZone.minimum_order_amount) > subtotal && orderType === orderTypeEnum.DELIVERY ? 'bg-primary/50 pointer-events-disable text-sm' : ''">
                    {{ $t('button.save_and_pay') }}
                    <strong
                        v-if="parseFloat(getDeliveryZone.minimum_order_amount) > 0 && parseFloat(getDeliveryZone.minimum_order_amount) > subtotal && orderType === orderTypeEnum.DELIVERY"
                        class="text-red-400">
                        ({{ $t('label.min_order') + ': ' + getDeliveryZone.currency_minimum_order_amount }})
                    </strong>
                </button>

            </div>
        </div>
    </div>
</template>

<script>
import orderTypeEnum from "../../../../enums/modules/orderTypeEnum";
import AddressComponent from "./AddressComponent.vue";
import SummeryComponent from "../SummeryComponent.vue";
import CouponComponent from "../CouponComponent.vue";
import router from "../../../../router";
import alertService from "../../../../services/alertService";
import LoadingComponent from "../../components/LoadingComponent.vue";
import statusEnum from "../../../../enums/modules/statusEnum";
import activityEnum from "../../../../enums/modules/activityEnum"


export default {
    name: "CheckoutComponent",
    components: { CouponComponent, SummeryComponent, AddressComponent, LoadingComponent },
    data() {
        return {
            loading: {
                isActive: false
            },
            enums: {
                statusEnum: statusEnum,
                activityEnum: activityEnum,
            },
            orderTypeEnum: orderTypeEnum,
            modelOutlet: 0
        }
    },
    computed: {
        setting: function () {
            return this.$store.getters['frontendSetting/lists'];
        },
        orderType: function () {
            return this.$store.getters['frontendCart/orderType'];
        },
        getDeliveryAddress: function () {
            return this.$store.getters['frontendCart/deliveryAddress'];
        },
        getDeliveryZone: function () {
            return this.$store.getters['frontendCart/deliveryZone'];
        },
        outlets: function () {
            return this.$store.getters['frontendOutlet/lists'];
        },
        subtotal: function () {
            return this.$store.getters['frontendCart/subtotal'];
        },
    },
    mounted() {
        this.loading.isActive = true;
        this.$store.dispatch('frontendOrderArea/lists').then(res => {
            this.loading.isActive = false;
        }).catch((err) => {
            this.loading.isActive = false;
        });

        this.loading.isActive = true;
        this.$store.dispatch('frontendOutlet/lists', {
            status: this.enums.statusEnum.ACTIVE
        }).then(res => {
            this.loading.isActive = false;
        }).catch((err) => {
            this.loading.isActive = false;
        });
    },
    methods: {
        changeOrderType: function (e) {
            this.$store.dispatch('frontendCart/updateOrderType', e)
        },
        deliveryAddress: function (e) {
            this.$store.dispatch('frontendDeliveryZone/selectDeliveryZone', e).then(res => {
                this.$store.dispatch('frontendCart/deliveryZone', res.data.data).then().catch();
                this.$store.dispatch('frontendCart/deliveryAddress', e).then().catch();
            }).catch((err) => {
                this.$store.dispatch('frontendCart/deliveryZone', {}).then().catch();
                this.$store.dispatch('frontendCart/deliveryAddress', {}).then().catch();
                alertService.error(err.response.data.message);
            });
        },
        outletAddress: function (e) {
            setTimeout(() => {
                this.$store.dispatch('frontendCart/outletAddress', this.modelOutlet).then().catch();
            }, 100);
        },
        selectAddress: function () {
            if (this.orderType === orderTypeEnum.DELIVERY) {
                if (Object.keys(this.getDeliveryAddress).length === 0) {
                    alertService.error(this.$t("message.required_delivery_address"));
                } else {
                    router.push({ name: "frontend.checkout.payment" });
                }
            } else {
                router.push({ name: "frontend.checkout.payment" });
            }
        }
    }
}
</script>