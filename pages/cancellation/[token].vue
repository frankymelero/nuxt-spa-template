<template>
    <NavBar></NavBar>
<section class="confirmation">
    <h1>Appointment cancellation</h1>
   <p>{{ message }} </p> </section>
</template>

<script setup>
const { token } = useRoute().params
const message = ref("")

const fetchData = async () => {
  try {
    const response = await fetch(`https://spa-template-backend-express.vercel.app/api/v1/appointment/${token}`, {
      method: 'DELETE',
    });

    if (response.status === 200) {
      const data = await response.json();

      if (data.deleted) {
        // La cita se eliminó con éxito
        message.value = 'Appointment has been successfully canceled.';
      } else {
        // La cita no se eliminó o hubo algún otro problema
        message.value = 'Appointment not found';

      }
    } else if (response.status === 404) {
      const data = await response.json();
      message.value = data.message;
    } else {
      // Otro código de estado, maneja el error aquí
      throw new Error('Error en la solicitud: ' + response.status);
    }
  } catch (error) {
    // Manejo de errores de red u otros errores
    console.error('Error en la solicitud:', error.message);
  
  }
};

onMounted(() => {
  fetchData();
});
</script>

<style scoped>
section.confirmation{
    min-height: 30vh;
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    gap: 10vh;
}
section.confirmation > p {
    font-size: 24px;
}
</style>