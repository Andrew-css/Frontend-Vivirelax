<script setup>
import { ref, onMounted } from 'vue';
import { useRouter, useRoute } from 'vue-router';
import { useStoreServicio } from '../stores/servicio.js';
import { useStoreReserva } from '../stores/reserva.js';
import { useQuasar } from 'quasar';

const router = useRouter();
const route = useRoute();
const useServicio = useStoreServicio();
const useReserva = useStoreReserva();
const idServicio = ref();
const servicio = ref();
const $q = useQuasar();
const showModal = ref(false);
const currentImgIndex = ref(0);
const dialogoAbierto = ref(false);
const mensaje = ref('Hola, me gustaría solicitar más información sobre este servicio y su disponibilidad. ¿Podrían proporcionarme detalles, por favor?');
const nombre = ref();
const email = ref();
const telefono = ref();
const loading = ref(false);

// Notify the user
function notificar(tipo, msg) {
    $q.notify({
        type: tipo,
        message: msg,
        position: "top",
    });
}

// Fetch services for the selected service type
async function cargarServicio(id) {
    try {
        const response = await useServicio.getPorId(id);

        if (response) {
            servicio.value = response;
        }
    } catch (error) {
        notificar('negative', 'Error al cargar el servicio');
        console.error('Error al cargar', error);
    }
}

// Open the modal and set the index of the clicked image
function openModal(index) {
    currentImgIndex.value = index;
    showModal.value = true;
}

function formatearDescripcion(texto) {
    // Si el texto es undefined o null, devolvemos una cadena vacía.
    return texto ? texto.replace(/\n/g, '<br>') : '';
}

const cerrarFormulario = () => {
  dialogoAbierto.value = false;
}

const enviarFormulario = async () => {
    loading.value = true;
  const data = {
    mensaje_res: mensaje.value,
    nombre_res: nombre.value,
    correo_res: email.value,
    telefono_res: telefono.value,
    idServicio: idServicio.value
  };


  try {
    const response = await useReserva.registro(data);
    console.log(response);

    if (useReserva.estatus === 200) {
      notificar('positive', "Reserva enviada con éxito");
      limpiar();
    } else if (useReserva.estatus === 400) {
      notificar('negative', useReserva.validacion);
    }
  } catch (error) {
    console.log(error);
    notificar('negative', 'Error al enviar la reserva. Intenta nuevamente.');
  } finally {
    loading.value = false;
  }
};

const limpiar = () => {
  nombre.value = '';
  email.value = '';
  telefono.value = '';
  mensaje.value = 'Hola, me gustaría solicitar más información sobre este servicio y su disponibilidad. ¿Podrían proporcionarme detalles, por favor?';
  dialogoAbierto.value = false;
}

onMounted(async () => {
    const Servicio = route.query.id;
    if (Servicio) {
        idServicio.value = Servicio;
        await cargarServicio(Servicio); // Fetch services based on the selected type
    }
});
</script>

<template>
    <q-page class="q-pa-md">
        <div v-if="servicio">
            <!-- Collage Image Section -->
            <div class="image-collage q-mb-lg">
                <div v-for="(img, index) in servicio.galeria.slice(0, 4)" :key="img._id" class="collage-image"
                    @click="openModal(index)">

                    <!-- Si no es la última imagen, muestra la imagen normal -->
                    <q-img v-if="index < 3" :src="img.url" alt="gallery image" />

                    <!-- Si es la última imagen, muestra el overlay con "Ver más" -->
                    <div v-else class="last-image">
                        <q-img :src="img.url" alt="gallery image" />
                        <div class="overlay-text">Ver más...</div>
                    </div>
                </div>
            </div>

            <!-- Modal for viewing all images -->
            <q-dialog v-model="showModal" full-width full-height>
                <q-carousel v-model="currentImgIndex" :value="currentImgIndex" arrows navigation infinite
                    control-color="black">
                    <q-carousel-slide v-for="(img, index) in servicio.galeria" :key="img._id" :name="index">
                        <q-img :src="img.url" />
                    </q-carousel-slide>
                </q-carousel>
            </q-dialog>

            <!-- Service Details -->
            <q-card class=" service-details-card" style="max-width: 900px; margin: 0 auto;">
                <q-card-section>
                    <div class="text-h2 text-bold text-center">{{ servicio.nombre_serv }}</div>
                    <p class="q-mt-lg" v-html="formatearDescripcion(servicio.descripcion)"></p>

                    <!-- Beneficios -->
                    <div>
                        <p class="text-bold text-h5 ">Beneficios:</p class="text-bold text-h5 ">
                        <p v-for="beneficio in servicio.beneficios" :key="beneficio._id">
                            <q-icon name="check_circle" color="green" />
                            {{ beneficio.descrip }}
                        </p>
                    </div>

                    <!-- Precio y Duración -->
                    <div class="q-mt-md">
                        <p><strong>Precio: </strong> {{ servicio.precio }}</p>
                        <p><strong>Duración: </strong> {{ servicio.duracion }}</p>
                    </div>

                    <!-- Botón para Reservar -->
                    <div class="text-center">
                        <q-btn label="Reservar" color="dark" class="q-mt-lg" size="18px" @click="dialogoAbierto = true" />
                    </div>

                </q-card-section>
            </q-card>
        </div>

        <q-dialog v-model="dialogoAbierto" persistent>
            <q-card style="min-width: 400px">
                <q-card-section class="text-h6 text-bold text-uppercase">
                    {{ servicio.nombre_serv }}
                </q-card-section>

                <q-card-section>
                    <div class="text-subtitle1 text-bold">Pide más información</div>
                    <p class="text-body2">
                        Rellena este formulario y el SPA VIVIRELAX se pondrá en contacto contigo en breve.
                        Todos los datos que envíes serán tratados de forma confidencial.
                    </p>

                    <!-- Formulario -->
                    <q-form @submit="enviarFormulario" class="q-gutter-md">
                        <q-input filled v-model="mensaje" color="black" label="Mensaje" type="textarea" :rows="4" />
                        <q-input filled v-model="nombre" color="black" label="Digite su nombre y apellidos"
                            :rules="[val => !!val || 'El nombre es obligatorio']" />
                        <q-input filled v-model="email" color="black" label="Digite su correo electrónico" type="email"
                            :rules="[val => !!val || 'El correo es obligatorio', val => /.+@.+\..+/.test(val) || 'Correo no válido']" />
                        <q-input filled v-model="telefono" color="black" label="Digite su número telefónico" type="number"
                            :rules="[val => !!val || 'El teléfono es obligatorio']" />
                    </q-form>

                </q-card-section>

                <!-- Botones del diálogo -->
                <q-card-actions align="right">
                    <q-btn flat label="Cancelar" color="black" @click="cerrarFormulario" />
                    <q-btn label="Enviar" color="black" @click="enviarFormulario" :loading="loading" :disabled="loading"/>
                </q-card-actions>
            </q-card>
        </q-dialog>
    </q-page>
</template>

<style scoped>
.image-collage {
    display: flex;
    justify-content: center;
    gap: 10px;
    width: 100%;
}

.collage-image {
    width: 250px;
    /* Fijamos el ancho del contenedor */
    height: 170px;
    /* Fijamos la altura */
    position: relative;
    overflow: hidden;
    border-radius: 8px;
}

.collage-image q-img {
    object-fit: cover;
    /* Asegura que la imagen llene el contenedor sin deformarse */
    width: 100%;
    height: 100%;
}

.last-image {
    position: relative;
    width: 250px;
    /* Aseguramos que también tenga el mismo ancho que las demás imágenes */
    height: 150px;
    border-radius: 8px;
}

.overlay-text {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    color: white;
    background-color: rgba(0, 0, 0, 0.5);
    /* Fondo negro semitransparente */
    padding: 10px;
    font-weight: bold;
    border-radius: 5px;
    text-align: center;
    pointer-events: none;
}

.service-details-card {
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
    border-radius: 8px;
    overflow: hidden;
}

.q-mt-md {
    margin-top: 16px;
}

.q-mt-lg {
    margin-top: 24px;
}
</style>
