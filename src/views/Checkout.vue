<template>
    <div class="container mt-5">
        <h1>Checkout</h1>

        <form @submit.prevent>
  <div class="form-group">
    <label for="formGroupExampleInput">Localidad</label>
    <input type="text" class="form-control" v-model="localidad" id="formGroupExampleInput" placeholder="Example input">
  </div>
  <div class="form-group">
    <label for="formGroupExampleInput2">calle</label>
    <input type="text" class="form-control" v-model="calle" id="formGroupExampleInput2" placeholder="Another input">
  </div>
    <div class="form-group">
    <label for="formGroupExampleInput2">Numero</label>
    <input type="text" class="form-control" v-model="numero" id="formGroupExampleInput2" placeholder="Another input">
  </div>
  <button v-if="loading === false"  class="btn btn-success p-3" @click="pagar">Pagar  </button>
  <button v-else disabled="true"  class="btn btn-success p-3" @click="pagar">Espere......  </button>

</form>
    </div>
</template>

<script>
import * as firebase from "firebase/app";
import moment from 'moment'
import 'firebase/firestore';
import {mapGetters} from 'vuex'

    export default {
            data(){
              return{
                localidad: '',
                calle: '',
                numero: '',
                loading: false,
                productos: []
              }
            },
            methods: {

              

             async pagar(){

                this.loading = true
                let arr = []
                for(let val of this.carrito){

                     let encontrado = this.productos.find(valor => valor.id === val.id)
                     let cantidadFinal = Number(encontrado.cantidad) - Number(val.cantidad)

                 await firebase.firestore().collection("articulos").doc(encontrado.id).update({cantidad: cantidadFinal })


                    let obj  = {
                      nombre: val.nombre,
                      cantidad: val.cantidad,
                      precio: val.precio
                    }

                    arr.push(obj)
                }
                  
                  let obj = {
                    compra: arr,
                    precioFinal: this.precioTotal,
                    usuario: this.currentUser.email,
                    fecha: moment().format('MM-DD-YYYY'),
                    direccion:{
                      localidad: this.localidad,
                      calle: this.calle,
                      numero: this.numero
                    }
                  }

              const result =    await firebase.firestore().collection('compras').add(obj)
              console.log('guardando envio',result)
              this.loading = false
                 this.$store.dispatch('setVaciar')

              this.$router.push('/cliente')
                 }
            }
            ,
            computed:{
                   ...mapGetters(['carrito','precioTotal','currentUser'])

            },
            async created(){
                  const snapshot = await firebase.firestore().collection('articulos').get()
            let docs = []
             snapshot.forEach((doc) => {
            docs.push({ ...doc.data(), id: doc.id });
          });

      
        
        this.productos = docs


        console.log('productoskssksk',this.productos)
            }
    }
    
</script>

<style lang="scss" scoped>

</style>