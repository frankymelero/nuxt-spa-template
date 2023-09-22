<template>
    <NavBar></NavBar>
<section class="confirmation">
    <h1>Appointment status</h1>
   <p>{{ message }} </p> </section>
</template>

<script setup>
const { token } = useRoute().params
const message = ref("")

const fetchData = async () => {
    fetch(`https://spa-template-backend-express.vercel.app/api/v1/appointment/validate/${token}`, {
  method: 'GET',
})
  .then((response) => {
    if (response.status === 200) {
      // La cita se validó con éxito
      return response.json();
    } else if (response.status === 404) {
       
      return response.json();
    } else {
      // Otro código de estado, maneja el error aquí
      throw new Error('Error en la solicitud: ' + response.status);
    }
  })
  .then((data) => {
    if (data.success) {
      // La cita se validó con éxito
      message.value='Thanks for trusting in us, your appointment has been confirmed! ';
    } else {
      // La cita no se validó o hubo algún otro problema
      message.value="Appointment not found or already validated.";
      // Realiza acciones de manejo de errores aquí si es necesario
    }
  })
  .catch((error) => {
    // Manejo de errores de red u otros errores
    console.error('Error en la solicitud:', error.message);
    // Realiza acciones de manejo de errores aquí si es necesario
  });
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