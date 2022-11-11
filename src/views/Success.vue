<template>
    <div class="page-success">
        <div class="columns is-multiline">
            <div class="column is-12">
                <h1 class="title">Terima Kasih</h1>

                <p>Pesanan Anda Sedang Di Proses, Mohon Di Tunggu</p>
            </div>
        </div>
    </div>
</template>


<script>
export default {
    name: 'Success',
    mounted() {
        document.title = 'Success | Lawak FnB'
    },
}
</script>