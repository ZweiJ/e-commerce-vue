<template>
    <div class="page-checkout">
        <div class="columns is-multiline">
            <div class="column is-12">
                <h1 class="title">Checkout</h1>
            </div>

            <div class="column is-12-box">
                <table class="table is-fullwidth">
                    <thead>
                        <tr>
                            <th>Product</th>
                            <th>Price</th>
                            <th>Quantity</th>
                            <th>Total</th>
                        </tr>
                    </thead>

                    <tbody>
                        <tr
                            v-for="item in cart.items"
                            v-bind:key="item.product.id"
                        >
                            <td>{{ item.product.name }}</td>
                            <td>Rp{{ item.product.price }}</td>
                            <td>{{ item.quantity }}</td>
                            <td>Rp{{ getItemTotal(item).toFixed(3) }}</td>
                        </tr>
                    </tbody>

                    <tfoot>
                        <tr>
                            <td colspan="2">Total</td>
                            <td>{{ cartTotalLength }}</td>
                            <td>Rp{{ cartTotalPrice.toFixed(3) }}</td>
                        </tr>
                    </tfoot>
                </table>
            </div>

            <div class="column is-12 box">
                <h2 class="subtitle">Shipping details</h2>

                <p class="has-text-grey mb-4">* All Fields are required</p>

                <div class="columns is-multiline">
                    <div class="column is-6">
                        <div class="field">
                            <label>First name*</label>
                            <div class="control">
                                <input type="text" class="input" v-model="first_name">
                            </div>
                        </div>

                        <div class="field">
                            <label>Last name*</label>
                            <div class="control">
                                <input type="text" class="input" v-model="last_name">
                            </div>
                        </div>

                        <div class="field">
                            <label>E-mail*</label>
                            <div class="control">
                                <input type="email" class="input" v-model="email">
                            </div>
                        </div>
                        
                        <div class="field">
                            <label>Phone*</label>
                            <div class="control">
                                <input type="text" class="input" v-model="phone">
                            </div>
                        </div>

                        <div class="field">
                            <label>Address*</label>
                            <div class="control">
                                <input type="text" class="input" v-model="address">
                            </div>
                        </div>

                        <div class="field">
                            <label>Zipcode*</label>
                            <div class="control">
                                <input type="text" class="input" v-model="zipcode">
                            </div>
                        </div>

                        <div class="field">
                            <label>Place*</label>
                            <div class="control">
                                <input type="text" class="input" v-model="place">
                            </div>
                        </div>
                    </div>
                </div>
                <div class="notification is-danger mt-4" v-if="errors.length">
                        <p v-for="error in errors" v-bind:key="error">{{ error }}</p>
                    </div>

                    <hr>

                    <div id="card-element" class="mb-5"></div>

                    <template v-if="cartTotalLength">
                        <hr>

                        <button class="button is-dark" @click="submitForm">Checkout</button>
                    </template>
            </div>
        </div>
    </div>
</template>


<script>
import axios from 'axios'
export default {
    name:'Checkout',
    data() {
        return {
            cart:{
                items:[]
            },
            stripe: {},
            card: {},
            first_name: '',
            last_name: '',
            email: '',
            phone:'',
            address:'',
            zipcode:'',
            place:'',
            errors: []
        }
    },
    mounted() {
        document.title = 'checkout | Lawak FnB'

        this.cart = this.$store.state.cart

        if (this.cartTotalLength > 0){
            this.stripe = Stripe('pk_test_51LqLVPLl7ZRByXe2Af2r3Rpi4EVLjkriTbzRpgzBIaNlq3ohBISZQayyT08Occn2e6Trb44yNSjS9KqEeDXCOQ0P00XdbZP8Z6')
            const elements = this.stripe.elements();
            this.card = elements.create('card',{hidePostalCode:true})

            this.card.mount('#card-element')
        }
    },
    methods: {
        getItemTotal(item){
            return item.quantity * item.product.price
        },
        submitForm(){
            this.errors=[]

            if (this.first_name ===''){
                this.errors.push('nama depan tidak boleh kosong')
            }

            if (this.last_name ===''){
                this.errors.push('nama belakang tidak boleh kosong')
            }

            if (this.email ===''){
                this.errors.push('email tidak boleh kosong')
            }

            if (this.address ===''){
                this.errors.push('alamat tidak boleh kosong')
            }

            if (this.phone ===''){
                this.errors.push('no hp tidak boleh kosong')
            }

            if (this.place ===''){
                this.errors.push('alamat tidak lengkap')
            }

            if (this.errors.length){
                this.$store.commit('setIsLoading', false)

                this.stripe.createToken(this.card).then(result =>{
                    if (result.error){
                        this.$store.commit('setIsLoading', false)

                        this.errors.push('Ada yang salah dengan stripe, Silahkan Coba lagi')

                        console.log(result.error.message)
                    } else{
                        this.stripeTokenHandler(result.token)
                    }
                })
            }
        },
        async stripeTokenHandler(token){
            const items = []

            for(let i=0; i < this.cart.items.length; i++){
                const item = this.cart.items[i]
                const obj = {
                    product: item.product.id,
                    quantity: item.quantity,
                    price: item.product.price * item.quantity
                }

                items.push(obj)
            }

            const data ={
                'first_name': this.first_name,
                'last_name': this.last_name,
                'email': this.email,
                'address': this.address,
                'zipcode': this.zipcode,
                'place': this.place,
                'phone': this.phone,
                'items': items,
                'stripe_token': token.id,
            }

            await axios
                .post('/api/v1/checkout/',data)
                .then(response =>{
                    this.$store.commit('clearCart')
                    this.$router.push('/cart/success')
                })
                .catch(error =>{
                    this.errors.push('Terjadi kesalahan , silahkan coba lagi')

                    console.error();
                })

                this.$store.commit('setIsLoading', false)
        }
    },
    computed: {
        cartTotalPrice() {
            return this.cart.items.reduce((acc, curVal) => {
                return acc += curVal.product.price * curVal.quantity
            }, 0)
        },
        cartTotalLength() {
            return this.cart.items.reduce((acc, curVal) => {
                return acc += curVal.quantity
            }, 0)
        }
    },
}
</script>