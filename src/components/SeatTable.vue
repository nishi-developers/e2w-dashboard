<template>
    <table>
        <tr v-for="(row, i) in formation">
            <td v-for="(cell, j) in row" :style="{ backgroundColor: backgroundColor(cell, i * row.length + j) }">
                {{ text(i * row.length + j ) }}
            </td>
        </tr>
    </table>
</template>

<script>
const fillstyles = [
    '#b2bec3',
    '#e84393',
    '#00cec9'
]
export default {
    props: {
        formation: Array,
        seats: Object
    },
    methods: {
        text(index) {
            const userId = this.seats[index]
            if(userId) {
                return userId
            }
            return ''
        },
        backgroundColor(cell, index){
            const colorIndex = this.seats[index] ? 0 : cell
            return fillstyles[colorIndex]
        }
    },
    watch: {

    }
}
</script>

<style scoped>
td {
    width: 20px;
    height: 20px;
}
</style>
