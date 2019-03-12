<template>
	<div id="graphicContainer">
		<div class="container title">
			<div class="row justify-content-center">
				<img class="media-object" src="./assets/bitcoin.png" alt="bitcoin" width="64px" height="64px">
				<div class="col-auto">
					<h1><span class="badge badge-secondary">Précio del Bitcoin (btc_mxn)</span></h1>
				</div>
			</div>
		</div>
		<div class="container-fluid h-50 d-inline-block">
			<div class="row ">
				<div class="col-12 col-sm-12 col-md-12 col-lg-9 col-xl-9">
					<canvas id="canvas" ref="canvas"></canvas>
				</div>
				<div class="col-12 col-sm-12 col-md-12 col-lg-3 col-xl-3 align-middle">
					<div class="centerDivV">
						<button v-for="item in timesBarData" type="button" class="btn btn-secondary btn-block" v-bind:style="[sel ==item.name?{'background': 'green'}:'']" :ref="item.name"  @click="changeInterval(item.name,item.interval)"><h2>{{item.text}}</h2>
						</button>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>
	
<script>
	import axios from 'axios';
	export default{
		name: 'GraphicContainer',
		data(){
			return{
				bitcoinData: [],
				sel: 'inter1',
				inicioIntervalo: false,
				timesBarData : [
					{name: 'inter1',interval: 3600,text: '1 HR'},
					{name: 'inter2',interval: 21600,text: '6 HRS'},
					{name: 'inter3',interval: 43200,text: '12 HRS'},
					{name: 'inter4',interval: 86400,text: '24 HRS'}
				],
				provider: {
					context: null
				}
			}
		},
		methods:{
			drawGraphic(){
				const c = this.$refs['canvas'];
			    const ctx = c.getContext('2d');

			    ctx.canvas.width  = window.innerWidth;

			    if(window.innerWidth >  window.innerHeight)
		  			ctx.canvas.height = window.innerHeight;
		  		else
		  			ctx.canvas.height = window.innerHeight/2;

		  		ctx.canvas.style.width = window.innerWidth;
		  		ctx.canvas.style.height = window.innerHeight;
			    
			    var datos = this.bitcoinData;


				ctx.clearRect(0, 0,  ctx.canvas.width, ctx.canvas.height);

				var posx = 0;
				var posy = 0;
				var posxBefore = 0;
				var posyBefore = 0;
				var cWidth = ctx.canvas.width;
				var cHeight = ctx.canvas.height;

				ctx.beginPath();
				ctx.lineWidth = "2";
				ctx.fillStyle = "#DAE1E9";
				ctx.rect(0, ctx.canvas.height-40, ctx.canvas.width, 40); 
				ctx.fill();

				var intervalDate = parseInt(datos.length / 12);

				var maxValue = Math.max(...datos.map(function(item){return parseFloat(item.last);}));
				var minValue = Math.min(...datos.map(function(item){return parseFloat(item.last);}));


				ctx.fillStyle = '#6C757D';
				if(window.innerWidth >  window.innerHeight){
					ctx.font = '72px Arial';
					ctx.fillText('$'+maxValue+' MXN', 80, 100);
				}
				else{
					ctx.font = '32px Arial';
					ctx.fillText('$'+maxValue+' MXN', 30, 50);
				}

				var diferencia = maxValue - minValue;
				var segmento = ctx.canvas.height/4;
				var minY = segmento;
				var maxY = ctx.canvas.height - segmento;
				var promedio = segmento/(maxY - minY); 
				var separacion = diferencia / 10.0;

				var contIndicadoresX = 0;
				var indSupY = false;
				var indInfY = false;
				for(var i = 0; i < datos.length; i++){

				    posx = parseInt(i * cWidth/datos.length);
				    if(datos.length == 1)
				      posy = cHeight/2;
				    else{
				      posy = ((maxValue-parseFloat(datos[i].last))*cHeight)/diferencia;
				    }

				    posy = posy * promedio + segmento;

				    ctx.strokeStyle = '#DAE1E9';
					ctx.lineWidth = 2;

					if(contIndicadoresX == intervalDate || (contIndicadoresX == 10 && i == 10)){
						ctx.beginPath();
				        ctx.moveTo(posx, ctx.canvas.width);
				        ctx.lineTo(posx, segmento);
				        ctx.stroke();
				        ctx.fillStyle = '#6C757D';
					    ctx.font = '15px Arial';
					    let d = new Date(parseInt(datos[i].created_at));
					    let f = (d.getHours()<10?'0':'')+d.getHours() +':'+ (d.getMinutes()<10?'0':'') + d.getMinutes();
						ctx.fillText(f, posx, ctx.canvas.height-10);
				        contIndicadoresX = 0;
					}

					ctx.strokeStyle = '#0667D0';
					ctx.lineWidth = 3;

				    ctx.beginPath();
				    ctx.moveTo(posx, posy);
				    if(datos.length == 1)
				      ctx.lineTo(posx+3, posy);
				    else if(i > 0)
				      ctx.lineTo(posxBefore, posyBefore);

				    ctx.stroke();

				    ctx.strokeStyle = 'green';
					ctx.lineWidth = 3;

					ctx.fillStyle = '#6C757D';
					ctx.font = '15px Arial';
				    if(!indSupY && datos[i].last == maxValue){
						ctx.fillText('$'+datos[i].last, 0, posy-10);
				    	ctx.beginPath();
				        ctx.moveTo(0, posy-3);
				        ctx.lineTo(25, posy-3);
				        ctx.stroke();
				        indSupY = true;
				    }

				    ctx.strokeStyle = 'red';
					ctx.lineWidth = 3;

				    if(!indInfY && datos[i].last == minValue){
				    	ctx.fillText('$'+datos[i].last, 0, posy-10);
				    	ctx.beginPath();
				        ctx.moveTo(0, posy);
				        ctx.lineTo(25, posy);
				        ctx.stroke();
				        indInfY = true;
				    }


				    posxBefore = posx;
				    posyBefore = posy;
				    contIndicadoresX++;
				}
			},
			changeInterval(name,interval){
				this.sel = name;
				//axios.get('http://localhost:3000/api/getIntervalData/'+interval)
				axios.get('http://ec2-3-17-207-194.us-east-2.compute.amazonaws.com:8080/api/getIntervalData/'+interval)
				.then(response =>{
					this.bitcoinData = response.data;
					this.drawGraphic();
				});
			}
		},
		provide(){
			return {
				provider: this.provider
			}
		},
		mounted(){
			window.addEventListener('resize', () => {
				var userAgent = navigator.userAgent || navigator.vendor || window.opera;
				var isMobile = navigator.userAgent.match(/(iPad)|(iPhone)|(iPod)|(android)|(webOS)/i);
				if(!isMobile)
		  			this.drawGraphic();
			});
		},
		created(){
			//axios.get('http://localhost:3000/api/getIntervalData/3600')
			axios.get('http://ec2-3-17-207-194.us-east-2.compute.amazonaws.com:8080/api/getIntervalData/3600')
			.then(response =>{
				this.bitcoinData = response.data;
				this.drawGraphic();
			});

			if(!this.inicioIntervalo){
				setInterval(() => {
					if(this.bitcoinData.length > 0){
						//axios.get('http://localhost:3000/api/getLastData/'+this.bitcoinData[this.bitcoinData.length-1].created_at)
						axios.get('http://ec2-3-17-207-194.us-east-2.compute.amazonaws.com:8080/api/getLastData/'+this.bitcoinData[this.bitcoinData.length-1].created_at)
						.then(response =>{
							var newElem = response.data.length;
							/*console.log(response.data);
							console.log('Elementos: '+newElem);
							console.log('En el registro de datos: '+this.bitcoinData.length);*/
							for(let i=0; i<newElem; i++) this.bitcoinData.shift();
							this.bitcoinData = this.bitcoinData.concat(response.data);
							//console.log('En el registro de datos despues: '+this.bitcoinData.length);
							this.drawGraphic();
						});
					}
				},10000);
				this.inicioIntervalo = true;
			}
		}
	}
</script>
	
<style>
	#canvas{
		width: 100%;
		border: 2px  solid #DAE1E9;
		border-radius: 10px;
		background-color: white;
		padding-bottom: 20px;
		margin-bottom: 20px;		
	}

	.title{
		padding-top: 20px;
		padding-bottom: 20px;
	}

	@media only screen and (min-width: 992px) {
		.centerDivV{
			position: relative;
			top: 50%;
			transform: translateY(-50%);
		}

		.centerDivH{
			position: relative;
			left: 50%;
			transform: translateX(-50%);
		}
	}
</style>