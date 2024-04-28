<script setup>
import { watch, ref } from 'vue'
import { RouterLink, RouterView } from 'vue-router'
import HelloWorld from './components/HelloWorld.vue'

const moneyFormatter = new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
})

let tableRows
let housingListingPrice
let houseListingPrice

// Grab data from localstorage
let storedTableRows = localStorage.getItem('tableRows')
if (storedTableRows) {
    tableRows = ref(JSON.parse(storedTableRows))
} else {
    tableRows = ref([
        { id: 1, desc: 'Good Location', weightPercent: 5, weightFlat: 0 },
        { id: 2, desc: 'Sidings', weightPercent: 5, weightFlat: 0 },
    ])
}

const storedHouseListingPrice = localStorage.getItem('houseListingPrice')
if (storedHouseListingPrice) {
    houseListingPrice = ref(Number(storedHouseListingPrice))
} else {
    houseListingPrice = ref(0)
}

// Set up watches to update localstorage
watch(tableRows, (newRows) => {
    localStorage.setItem('tableRows', JSON.stringify(newRows))
}, { deep: true })

watch(houseListingPrice, (newPrice) => {
    localStorage.setItem('houseListingPrice', newPrice)
})

const addTableRow = () => {
    const newRow = {
        id: tableRows.value + 1,
        desc: 'New Requirement',
        weightPercent: 0,
        weightFlat: 0,
    }
    tableRows.value.push(newRow)
}

const removeTableRow = (row) => {
    tableRows.value = tableRows.value.filter((r) => {
        return r.id !== row.id
    })
}

const getTotalWeights = () => {
    if (tableRows.value.length === 0) {
        return 0
    }

    return tableRows.value.reduce((acc, row) => {
        const percentAmt = Number(row.weightPercent) * houseListingPrice.value / 100
        const flatAmt = Number(row.weightFlat)
        return acc + percentAmt + flatAmt
    }, 0)
}

const calculateOfferPrice = () => {
    const totalWeights = getTotalWeights()
    return houseListingPrice.value - totalWeights
}

</script>

<template>
    <div class="container py-3">
        <div class="row gap-3">
            <section class="col-12 col-md-8">
                <h4>Requirements</h4>
                <table class="table table-hover">
                    <thead>
                        <tr>
                            <th><span class="d-none d-md-block">Fulfilled</span></th>
                            <th>Description</th>
                            <th>Weight</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="row in tableRows" :key="row.id">
                            <td>
                                <div>
                                    <input type="checkbox" class="form-check-input" :id="'requirement-' + row.id"
                                        v-model="row.fulfilled">
                                </div>
                            </td>
                            <td>
                                <input type="text" class="form-control" v-model="row.desc">
                            </td>
                            <td>
                                <div class="row gap-2 gx-0">
                                    <div class="input-group col-12 col-lg">
                                        <input type="number" class="form-control" v-model="row.weightPercent">
                                        <span class="input-group-text">%</span>
                                    </div>
                                    <div class="input-group col-12 col-lg">
                                        <span class="input-group-text">$</span>
                                        <input type="number" class="form-control" v-model="row.weightFlat">
                                    </div>
                                    <button class="btn btn-outline-danger col col-lg-2" @click="removeTableRow(row)">
                                        <i class="bi-trash"></i>
                                    </button>
                                </div>
                            </td>
                        </tr>
                    </tbody>
                </table>
                <div class="text-center">
                    <button class="btn btn-primary" @click="addTableRow">
                        <i class="bi-plus-lg"></i>
                    </button>
                </div>
            </section>
            <main class="col-12 col-md">
                <h5>Calculated Offer Price</h5>
                <p class="lead">
                    {{ moneyFormatter.format(calculateOfferPrice()) }}
                </p>
                <div class="mb-3">
                    <label for="house-listing-price" class="form-label">House Listing Price</label>
                    <div class="input-group">
                        <span class="input-group-text">$</span>
                        <input type="number" id="house-listing-price" class="form-control" v-model="houseListingPrice">
                    </div>
                </div>
            </main>

        </div>
    </div>
</template>

<style scoped></style>
