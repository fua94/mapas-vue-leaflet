<template>
    <div class="container"
        v-if="ruta">
        <div class="row mt-3">
            <div class="col-12">
                <h1 class="text-center">Operadora</h1>
            </div>
        </div>
        <div class="row mt-3">
            <div class="col-4">
                <div class="card" style="width: 18rem;">
                    <div class="card-header">
                       Viajes
                    </div>
                    <template
                        v-if="rutas.length">
                        <ul class="list-group list-group-flush"
                            v-for="ruta in rutas">
                            <a href="#" class="list-group-item list-group-item-action"
                                @click="asignarViaje(ruta)"><strong>{{ruta.ID}}.-</strong> Usuario: {{ruta.USUARIO.USUARIO}}</a>
                        </ul>
                    </template>
                    <template
                        v-else>
                        <ul class="list-group list-group-flush"
                            <li class="list-group-item">No hay viajes en cola...</li>
                        </ul>
                    </template>

                </div>
                <template
                    v-if="ruta">
                    <div class="card mt-5" style="width: 18rem;">
                        <div class="card-body">
                            <h5 class="card-title text-center font-weight-bold">Detalles</h5>
                            <p class="card-text">
                                <ul class="list-group list-group-flush">
                                    <li class="list-group-item"><strong>ID:</strong> {{ruta.ID}}</li>
                                    <li class="list-group-item"><strong>Usuario:</strong> {{ruta.USUARIO.USUARIO}}</li>
                                    <li class="list-group-item"><strong>Distancia:</strong>  {{ruta.DISTANCIA}}Km.</li>
                                    <li class="list-group-item"><strong>Tiempo:</strong>  {{ruta.TIEMPO}} min. aprox.</li>
                                    <li class="list-group-item"><strong>Precio:</strong>  ${{ruta.PRECIO}} </li>
                                    <li class="list-group-item"><strong>Estado:</strong>  {{getEstado}} </li>
                                </ul>
                            </p>
                            <div class="row">
                                <div class="col-4">
                                    <a href="#" class="btn btn-primary"
                                    @click="despacharViaje"><i class="fas fa-cart-arrow-down"></i></a>
                                </div>
                                <div class="col-4">
                                    <a href="#" class="btn btn-success"
                                    @click="terminarViaje"><i class="fas fa-user-check"></i></a>
                                </div>
                                <div class="col-4">
                                    <a href="#" class="btn btn-danger"
                                    @click="terminarViaje"><i class="fas fa-trash-alt"></i></a>
                                </div>
                            </div>
                        </div>
                    </div>
                </template>

            </div>
            <div class="col-8">
                <div id="map-wrap" style="height: 80vh">
                    <no-ssr>
                        <l-map :zoom=13 :center="centro" ref="mapa"
                        v-if="ruta">
                            <l-tile-layer url="https://tile.thunderforest.com/neighbourhood/{z}/{x}/{y}.png?apikey=a345f0be1266434abcc04b97427608c7"></l-tile-layer>
                            <template
                                v-if="ruta.ORIGEN_LAT">
                                <l-marker :lat-lng="[ruta.ORIGEN_LAT, ruta.ORIGEN_LONG]">
                                    <l-icon
                                        :icon-size="[30, 30]"
                                        icon-url="https://cdn4.iconfinder.com/data/icons/dot/256/man_person_mens_room.png">
                                    </l-icon>
                                </l-marker>
                                <l-marker :lat-lng="[ruta.DESTINO_LAT, ruta.DESTINO_LONG]">
                                    <l-icon
                                        :icon-size="[30, 30]"
                                        icon-url="https://cdn.iconscout.com/icon/premium/png-256-thumb/destination-flag-3-739390.png">
                                    </l-icon>
                                </l-marker>
                            </template>

                        </l-map>
                    </no-ssr>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios'

const URI_RUTAS = 'http://puceing.edu.ec:15005/FranciscoUlloa/mytaxifinder/api/rutas'

var CONTROL = null

const DELAY = 7500

class Usuario{
    constructor(){
        this.USUARIO = ''
    }
}

class Ruta{
    constructor(){
        this.ID = -1,
        this.USUARIO_ID = -1,
        this.ORIGEN_LAT = ''
        this.ORIGEN_LONG = ''
        this.DESTINO_LAT = ''
        this.DESTINO_LONG = ''
        this.PRECIO = ''
        this.DISTANCIA = ''
        this.TIEMPO = ''
        this.ESTADO = 'P'
        this.USUARIO = new Usuario()
    }
}

export default {
    data(){
        return {
            rutas: [],
            ruta: new Ruta(),
            centro: [-0.182903, -78.467282],
            interval: null,
            isStarted: true
        }
    },
    created(){
        this.initData()
        this.interval = setInterval(function () {
            this.refreshData()
        }.bind(this), DELAY);
    },
    computed: {
        getEstado: function() {
            switch (this.ruta.ESTADO) {
                case "D":
                    return 'Despachado'
                default:
                    return 'Pendiente'
            }
        }
    },
    methods: {
        async initData(){
            try{
                let resp = await axios.get(URI_RUTAS)

                if(resp.data.length){
                    this.rutas = resp.data

                    this.$nextTick(() => {
                        CONTROL = L.Routing.control({
                            showAlternatives: false,
                            show: false,
                            draggableWaypoints: false,
                            routeWhileDragging: false,
                            autoRoute: true,
                            createMarker: function() { return null; }
                        }).addTo(this.$refs.mapa.mapObject)
                    })
                }
            }catch(error){
                console.log(error)
            }
        },
        async refreshData(){
            try{
                const resp = await axios.get(URI_RUTAS)
                const length = resp.data.length

                if(length){
                    this.rutas = resp.data
                }else{
                    this.rutas = []
                }
            }catch(error){
                console.log(error)
            }
        },
        asignarViaje(ruta) {
            this.ruta = ruta
            this.$nextTick(() => {
                this.$refs.mapa.mapObject.removeControl
                CONTROL.setWaypoints([
                    L.latLng(ruta.ORIGEN_LAT, ruta.ORIGEN_LONG),
                    L.latLng(ruta.DESTINO_LAT, ruta.DESTINO_LONG)
                ])
            })
        },
        despacharViaje(){
            this.ruta.ESTADO = "D"
            axios({
                method: 'put',
                url: URI_RUTAS,
                data: this.ruta,
                validateStatus: (status) => {
                    return true
                },
            }).catch(error => {
                console.log(error.message);
            })
        },
        terminarViaje(){
            if(confirm('¿Está seguro de terminar el viaje?')){
                axios({
                    method: 'delete',
                    url: URI_RUTAS,
                    data: this.ruta,
                    validateStatus: (status) => {
                        return true
                    },
                }).catch(error => {
                    console.log(error.message);
                }).then(response => {
                    this.$nextTick(() => {
                        this.$refs.mapa.mapObject.removeControl
                        CONTROL.setWaypoints([])
                    })
                    this.ruta = new Ruta()

                })
            }
        }
    }
}
</script>
