<template lang="pug">
#juego
	.aside
		Tetronomo(:tetro="activo" v-if="activo.tetro")
		.telemetria
			span posX: {{activo.posX}} | 
			span posY: {{activo.posY}}
	.tablero-contenedor
		.tablero
			.filas(
				v-for="fila in tablero"
			)
				.columna(
					v-for="col in fila"
					:class="{fill : col.act}"
				)
					span {{col.fila}}-{{col.col}}
</template>

<script>
import Tetronomo from "./Tetronomo"
import Tetronomos from "../tetronomos"
export default {
	name: "Juego",
	components:{
		Tetronomo
	},
	data(){
		return{
			tabla:{
				columnas: 10,
				filas: 20
			},
			activo:{
				posX: 0,
				posY: 0,
				rot: 0,
				tetro: null
			},
			estaticos:[],
			tetronomos: Tetronomos
		}
	},
	methods:{
		keyEvents(event){
			if(event.type === "keydown"){
				if(event.code === "ArrowDown"){
					this.activo.posY++
				}else if(event.code === "ArrowUp"){
					this.rotar()
				}else if(event.code === "ArrowLeft"){
					if(this.verMuroIzq()) this.activo.posX--
				}else if(event.code === "ArrowRight"){
					if(this.verMuroDer()) this.activo.posX++
				}
			}
		},
		verMuroDer(){
			let tetro = this.activo.tetro.forma[this.activo.rot]
			let ultEnTetro = 1
			// la ultima columna del tetronomo tiene cuadro?
			for(let row of tetro){
				if(row[row.length -1]){
					ultEnTetro = 0
					break
				}
			}
			let sigColumna = this.activo.posX + tetro[0].length - ultEnTetro
			//si este cuadro no existe entonces no permite el movimiento
			return this.tablero[0][sigColumna]
		},
		verMuroIzq(){
			let tetro = this.activo.tetro.forma[this.activo.rot]
			let primeroEnTetro = 1
			// la primeroima columna del tetronomo tiene cuadro?
			for(let row of tetro){
				if(row[0]){
					primeroEnTetro = 0
					break
				}
			}
			return !(this.activo.posX - 1 < 0 - primeroEnTetro)
		},
		rotar(){
			let nextIndex = this.activo.rot
			if(nextIndex === this.activo.tetro.forma.length -1){
				nextIndex = 0
			}else{
				nextIndex++
			}

			let nextTetro = this.activo.tetro.forma[nextIndex]

			// si se encuentra con un muro no rote inicio
			for(let posY = 0; posY < nextTetro.length; posY++){
				let posTabY = posY + this.activo.posY
				for(let posX = 0; posX < nextTetro[posY].length; posX++){
					let posTabX = posX + this.activo.posX
					// si no existe la fila en el tablero no hace la rotacion
					if(!this.tablero[posTabY]) return
					// si no existe la columna en el tablero no hace la rotacion
					if(!this.tablero[posTabY][posTabX]) return
				}
			}
			this.activo.rot = nextIndex
		}
	},
	computed:{
		tablero(){
			let tablero = []
			//creacion de tablero vacio
			for(let fila = 0; fila < this.tabla.filas; fila++){
				let filaP = []
				for(let col = 0; col < this.tabla.columnas; col++){
					filaP.push({
						fila: fila,
						col: col,
						act: false,
						est: false
					})
				}
				tablero.push(filaP)
			}
			// si hay un tetromeno activo se agrega al tablero
			if(this.activo.tetro){
				let tetro = this.activo.tetro.forma[this.activo.rot]
				for(let fila = 0; fila < tetro.length; fila++){
					for(let col = 0; col < tetro[fila].length; col++){
						if(tetro[fila][col] === 1){
							tablero[this.activo.posY + fila][this.activo.posX + col].act = true
						}
					}
				}
			}
			return tablero
		}
	},
	mounted(){
		this.activo.tetro = this.tetronomos.t
		window.addEventListener("keydown", this.keyEvents)
	},
	beforeDestroy(){
		window.removeEventListener("keydown", this.keyEvents)
	},
}
</script>

<style lang="sass">
body
	background-color: #101E2C

.tablero-contenedor
	padding: 25px
	&:after
		content: ""
		display: table
		clear: both

.tablero
	float: left
	border: 1px solid gray

.filas
	display: flex
	border-bottom: 1px solid gray
	&:last-child
		border-bottom: none

.columna
	width: 30px
	height: 30px
	border-right: 1px solid gray
	padding: 2px
	&:last-child
		border-right: none
	span
		display: block
		font-size: 0.6rem
		line-height: 1em

.fill
	background-color: red
	span
		color: white

</style>
