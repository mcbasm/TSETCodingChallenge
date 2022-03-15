<script lang="ts">
import { Vue } from 'vue-class-component';

interface Price {
    id: number,
    name: string,
    priceValue: string,
    realPrice: number,
    removable: boolean,
    editablePrice: boolean
    editableName: boolean,
    mouseHoverName: boolean,
    mouseHoverPrice: boolean
}

export default class Challenge extends Vue {

    currentId = 2;
    totalAmount = 1;
    newName = "";
    newPrice = 0;
    toBeCreated = false;

    prices: Price[] = [{ id: 1, name: "Baseprice", priceValue: "1.0", realPrice: 1.0, removable: false, editablePrice: false, editableName: false, mouseHoverName: false, mouseHoverPrice: false }];

    addPrice() {
        const newPrice: Price = { id: 0, name: this.newName, priceValue: "", realPrice: this.newPrice, removable: true, editablePrice: false, editableName: false, mouseHoverName: false, mouseHoverPrice: false };
        if (this.validatePrice(newPrice)) {
            this.formatPrice(newPrice);
            this.prices.push({ id: this.currentId++, name: newPrice.name, priceValue: newPrice.priceValue, realPrice: newPrice.realPrice, removable: true, editablePrice: false, editableName: false, mouseHoverName: false, mouseHoverPrice: false })
            this.getCurrentTotal();
            this.clearForm();
        }
    }

    getCurrentTotal() {
        let total = 0;
        this.prices.forEach(price => {
            total += price.realPrice;
        });
        this.totalAmount = total;
    }


    removePrice(price: Price) {
        this.prices = this.prices.filter(x => x != price);
        this.getCurrentTotal();
    }

    completePriceEditions(price: Price) {
        price.mouseHoverName = false;
        price.mouseHoverPrice = false;
        price.editableName = false;
        price.editablePrice = false;
        this.formatPrice(price);
        this.getCurrentTotal();
    }

    private validatePrice(price: Price): boolean {
        if (price.name.length < 1) {
            alert("The minimun length of the name must be 1.")
            return false;
        }
        if (price.realPrice < 0) {
            alert("The minimum value of the price must be 0.");
            price.realPrice = 0;
            return false;
        }
        if (Number.isInteger(price.realPrice)) {
            alert("The value must be a decimal.")
            return false;
        }
        return true
    }

    private formatPrice(price: Price) {
        /* If the price is empty, set the values to 0 and 0.0 */
        if (price.realPrice) {
            /* Get the Real Price as a String to get the number of decimals */
            const numStr = String(price.realPrice);

            /* If it has at least two decimals, show only two, else just one decimal */
            if (numStr.includes('.')) {
                if (numStr.split('.')[1].length >= 2) {
                    price.priceValue = price.realPrice.toFixed(2);
                } else {
                    price.priceValue = price.realPrice.toFixed(1);
                }
            } else {
                price.priceValue = price.realPrice + ".0";
            }
        } else {
            price.realPrice = 0;
            price.priceValue = "0.0";
        }

    }

    private clearForm() {
        this.newName = "";
        this.newPrice = 0;
        this.toBeCreated = false;
    }
}
</script>

<template>
    <div class="container-fluid">
        <div class="row">
            <div class="col-12 align-right">Total: {{ totalAmount.toFixed(2) }} EUR/kg</div>
        </div>
        <div class="row">
            <div class="col-md-6 offset-md-3">
                <table class="table">
                    <thead>
                        <tr>
                            <th scope="col">Price Name</th>
                            <th scope="col">Value</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr
                            v-for="price in prices"
                            :key="price.id"
                            @mouseleave="completePriceEditions(price)"
                            v-on:keyup.enter="completePriceEditions(price)"
                        >
                            <td>
                                <span
                                    @mouseover="price.mouseHoverName = true"
                                    v-if="!price.editableName || price.id == 1"
                                >{{ price.name }}</span>
                                <input
                                    type="text"
                                    @mouseleave="price.mouseHoverName = false"
                                    v-on:blur="completePriceEditions(price)"
                                    v-if="price.editableName && price.id != 1"
                                    v-model="price.name"
                                    autofocus
                                />
                                <button
                                    v-if="price.mouseHoverName && price.id != 1"
                                    @click="price.editableName = true;"
                                    class="btn btn-outline-secondary"
                                >
                                    <i class="fas fa-edit"></i>
                                </button>
                            </td>
                            <td>
                                <div class="row">
                                    <div class="col-12">
                                        <input
                                            v-if="!price.editablePrice"
                                            @click="price.editablePrice = true;"
                                            @mouseover="price.mouseHoverPrice = true;"
                                            type="text"
                                            v-model="price.priceValue"
                                        />
                                        <input
                                            type="number"
                                            @mouseover="price.mouseHoverPrice = true;"
                                            v-on:blur="completePriceEditions(price)"
                                            v-else
                                            v-model="price.realPrice"
                                            autofocus
                                        />
                                        <button
                                            v-if="price.id != 1 && price.removable && price.mouseHoverPrice"
                                            @click="removePrice(price)"
                                            class="btn btn-danger"
                                        >
                                            <i class="fas fa-times"></i>
                                        </button>
                                    </div>
                                </div>
                                <div
                                    class="row"
                                    v-if="price.mouseHoverPrice && !price.editablePrice"
                                >
                                    <div class="col-12">
                                        <span>{{ price.realPrice }}</span>
                                    </div>
                                </div>
                            </td>
                        </tr>
                    </tbody>
                </table>
                <div class="row">
                    <div class="col-5">
                        <input
                            :class="{
                                disabled: !toBeCreated
                            }"
                            type="text"
                            placeholder="Insert Name"
                            @click="toBeCreated = true"
                            v-model="newName"
                        />
                    </div>
                    <div class="col-5">
                        <input
                            :class="{ disabled: !toBeCreated }"
                            type="number"
                            placeholder="0.0"
                            step="any"
                            @click="toBeCreated = true"
                            v-model="newPrice"
                        />
                    </div>
                    <div class="col-1">
                        <button v-if="toBeCreated" @click="addPrice()" class="btn btn-success">
                            <i class="fas fa-save"></i>
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<style>
.disabled {
    color: gray;
    background-color: lightgray;
}
</style>