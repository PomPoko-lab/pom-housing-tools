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
        { id: 1, desc: 'Good Location', weight: 5 },
        { id: 2, desc: 'Sidings', weight: 5 },
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
        weight: 0,
        fulfilled: false,
    }
    tableRows.value.push(newRow)
}

const removeTableRow = (row) => {
    tableRows.value = tableRows.value.filter((r) => {
        return r.id !== row.id
    })
}

const totalWeights = () => {
    if (tableRows.value.length === 0) {
        return 0
    }

    return tableRows.value.reduce((acc, row) => {
        if (row.fulfilled) return acc
        return acc + Number(row.weight)
    }, 0)
}

</script>

<template>
    <div class="container py-3">
        <div class="row gap-3">
            <section class="col">
                <h4>Requirements</h4>
                <table class="table table-hover">
                    <thead>
                        <tr>
                            <th>Fulfilled</th>
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
                                <div class="d-flex gap-2">
                                    <div class="input-group">
                                        <input type="text" class="form-control" v-model="row.weight">
                                        <span class="input-group-text">%</span>
                                    </div>
                                    <button class="btn btn-outline-danger" @click="removeTableRow(row)">
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
            <main class="col">
                <h5>Calculated Offer Price</h5>
                <p class="lead">
                    {{ moneyFormatter.format(houseListingPrice - (houseListingPrice * totalWeights() / 100)) }}
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
