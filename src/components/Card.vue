<template>
    <div class="card">
        <div>{{card.name}}</div>
        <img v-bind:src="card.img" /> 
    </div>
</template>

<script>
export default {
    name: 'Card',
    props: [
        'card'
    ]
}
</script>

<style scoped="scoped">
.card {
    padding: 10px;
    border: 1px solid #000;
    border-radius: 2px;
}
</style>