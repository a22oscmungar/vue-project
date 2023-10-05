<script>
import { v4 as uuidv4 } from 'uuid';

export default {
	data() {
		return {

			verAñadirPregunta: false,
			novaPregunta: {
				"id": undefined,
				"pregunta": undefined,
				"respostes": [
					{
						"id": 1,
						"etiqueta": ""
					},
					{
						"id": 2,
						"etiqueta": ""
					},
					{
						"id": 3,
						"etiqueta": ""
					},
					{
						"id": 4,
						"etiqueta": ""
					}
				],
				"resposta_correcta": undefined,
				"imatge": undefined
			},
			mostrarPreguntaActual: false,
			'preguntaActual': null,
			preguntes: [],
			datosTransformados: null,
			mostrarDatos: false,
			mostrarStats: false,
			estadisticas: ""
		}
	}, methods: {
		toggleRespuestas(idPregunta) {

			this.preguntes.forEach((pregunta) => {
				if (pregunta.id == idPregunta) {
					pregunta.mostrarRespuestas = !pregunta.mostrarRespuestas;
				} else {
					pregunta.mostrarRespuestas = false;
				}
			});
		},
		editarPregunta(pregunta) {
			if (pregunta.editando == false || pregunta.editando == undefined) {
				pregunta.editando = true;
			} else {
				pregunta.editando = false;
			}
		}, async enviarEditada(preguntaEditada) {
			try {
				preguntaEditada.editando = false;
				const response = await fetch(`http://localhost:3005/editar`, {
					method: 'PUT',
					headers: {
						'Content-Type': 'application/json',
					},
					body: JSON.stringify(preguntaEditada),
				});
				if (!response.ok) {
					throw new Error("Error al editar pregunta");
				}
				this.cargarPreguntas();

			} catch (error) {
				console.log("error: ", error);
			}

		}
		,
		guardarEdicion(pregunta) {
			pregunta.editando = false; // Sale del modo de edición

		},
		mostrarFormulario() {
			if (this.verAñadirPregunta == false) {
				this.verAñadirPregunta = true;
			} else {
				this.verAñadirPregunta = false;
			}
		}, async borrarPregunta(idPregunta) {
			try {
				const idBorrar = parseInt(idPregunta);
				const response = await fetch(`http://localhost:3005/borrar/${idBorrar}`, {
					method: 'DELETE',

				});
				if (response.ok) {
					console.log(`Pregunta con id: ${idPregunta} borrada correctamente`);
					this.cargarPreguntas();
				} else {
					console.error(`Error al eliminar la pregunta con id: ${idPregunta}`)
				}
			} catch (error) {
				console.error(`Error de red:`, error);
			}


		}, cargarPreguntas() {
			const myHeaders = new Headers();
			fetch("http://localhost:3005/preguntas",
				{
					method: 'GET',
					headers: myHeaders,
					mode: 'cors',
					//cache: default
				}
			).then(
				(response) => {
					return response.text();
				}
			).then((data) => {
				console.log("Data => ");
				this.preguntes = JSON.parse(data);


			}).catch((error) => {
				console.error("Error: ", error)
			})
		}, async enviarPregunta() {
			const idUnica = uuidv4();
			this.novaPregunta.id = idUnica;
			console.log(JSON.stringify(this.novaPregunta));

			try {
				const response = await fetch('http://localhost:3005/enviarPregunta', {
					method: 'POST',
					headers: {
						'Content-Type': 'application/json',
					},
					body: JSON.stringify(this.novaPregunta),
				});
				if (response.ok) {
					console.log("Pregunta añadida con exito");
					window.location.reload();
				} else {
					console.error("Error al añadir la pregunta");
				}
			} catch (error) {
				console.error('Error de red: ', error);


			}
		}, async transformar() {
			try {
				const response = await fetch('http://localhost:3005/transform', {
					method: 'GET',
				});
				if (response.ok) {
					const data = await response.text();
					this.datosTransformados = data;

					// Invierte el estado de mostrarDatos
					this.mostrarDatos = !this.mostrarDatos;
				} else {
					console.error('Error al obtener la respuesta:', response.status, response.statusText);
				}
			} catch (error) {
				console.error('Error en la solicitud:', error);
			}
		}, async stats() {
			try {
				if (this.mostrarStats == false) {
					this.mostrarStats = true;
					const response = await fetch('http://localhost:3005/stats', {
						method: 'GET',
					});
					if (response.ok) {
						const data = await response.text();
						this.estadisticas = data;

					}

				} else {
					this.mostrarStats = false;
				}


			} catch (error) {
				console.error(error);
			}
		}
	},
	created() {
		this.cargarPreguntas();
	}
};
</script>


<template>
	<div class="contenedor">

		<ul class="preguntas-lista">
			<li v-for="(pregunta, index) in preguntes" :key="pregunta.id">
				<div class="pregunta">
					<h3>{{ pregunta.pregunta }}</h3>
					<button class="botonOpciones" @click="toggleRespuestas(pregunta.id)">Opciones</button>
					<button class="botonEditar" @click="editarPregunta(pregunta)">Editar</button>
					<button class="botonEliminar" @click="borrarPregunta(pregunta.id)">Eliminar</button>
				</div>
				<div v-if="pregunta.mostrarRespuestas" class="opciones">
					<img :src="pregunta.imatge" alt="imagen de la pregunta" class="imagen-pregunta">
					<ol class="respuestas">
						<li v-for="(respuesta, index) in pregunta.respostes" :key="index">
							{{ respuesta.etiqueta }}
						</li>
						<li class="respuestaCorrecta">Respuesta Correcta: {{ pregunta.resposta_correcta }}</li>
					</ol>
				</div>
				<!-- Mostrar el formulario de edición -->
				<div v-if="pregunta.editando" class="formulario-edicion">
					<form @submit.prevent="guardarEdicion(pregunta)">
						<textarea v-model="pregunta.pregunta"></textarea> <br>

						<label for="editImagen">Editar URL de Imagen:</label>
						<input type="text" v-model="pregunta.imatge" id="editImagen"><br>

						<!-- Agrega campos de edición para las opciones aquí -->
						<div v-for="(respuesta, index) in pregunta.respostes" :key="index">
							<label for="editRespuesta{{ index }}">Editar Respuesta {{ index + 1 }}:</label>
							<input type="text" v-model="respuesta.etiqueta" :id="'editRespuesta' + index">
						</div>



						<label for="editRespuestaCorrecta">Editar Respuesta Correcta:</label>
						<select v-model="pregunta.resposta_correcta" id="editRespuestaCorrecta">
							<option value="1">{{ pregunta.respostes[0].etiqueta }}</option>
							<option value="2">{{ pregunta.respostes[1].etiqueta }}</option>
							<option value="3">{{ pregunta.respostes[2].etiqueta }}</option>
							<option value="4">{{ pregunta.respostes[3].etiqueta }}</option>
						</select>

						<button type="submit" class="btnGuardar" @click="enviarEditada(pregunta)">Guardar</button>
					</form>
				</div>
			</li>
		</ul>
		<button class="botonAñadir" @click="mostrarFormulario">AÑADIR PREGUNTA</button>
		<div v-if="verAñadirPregunta" class="formularioAñadir">

			<label for="enunciado">Enunciado de la pregunta:</label>
			<input type="text" v-model="novaPregunta.pregunta">

			<label for="imatge">URL de la imagen:</label>
			<input type="text" v-model="novaPregunta.imatge">
			<img :src="novaPregunta.imatge">

			<label for="respuesta1">Respuesta 1:</label>
			<input type="text" v-model="novaPregunta.respostes[0].etiqueta">

			<label for="respuesta2">Respuesta 2:</label>
			<input type="text" v-model="novaPregunta.respostes[1].etiqueta">

			<label for="respuesta3">Respuesta 3:</label>
			<input type="text" v-model="novaPregunta.respostes[2].etiqueta">

			<label for="respuesta4">Respuesta 4:</label>
			<input type="text" v-model="novaPregunta.respostes[3].etiqueta">

			<label for="correcta">Respuesta correcta:</label>
			<select v-model="novaPregunta.resposta_correcta" id="respuestaCorrecta">
				<option value="1">{{ novaPregunta.respostes[0].etiqueta }}</option>
				<option value="2">{{ novaPregunta.respostes[1].etiqueta }}</option>
				<option value="3">{{ novaPregunta.respostes[2].etiqueta }}</option>
				<option value="4">{{ novaPregunta.respostes[3].etiqueta }}</option>
			</select>

			<button @click="enviarPregunta" class="btnAñadir2">Añadir Pregunta</button>


		</div>

	</div>
	<button @click="transformar" class="btnDatos">Transforma</button>
	<div v-if="mostrarDatos" class="datos">
		{{ datosTransformados }}
	</div>
	<button @click="stats" class="btnDatos">Estadisticas</button>
	<div v-if="mostrarStats">
		<div v-html="estadisticas" class="estadisticas"></div>
	</div>
</template>
  
  
  



<style scoped>
#app {
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
	height: 100vh;
}
</style>