<template>
    
    <!-- Service Start -->
    <div class="container-fluid px-0 py-5 my-5">
    <div class="row mx-0 justify-content-center text-center">
      <div class="col-lg-6">
        <h6 class="d-inline-block bg-light text-primary text-uppercase py-1 px-2">Our Service</h6>
        <h1>Spa & Beauty Services</h1>
      </div>
    </div>
    <div class="owl-carousel service-carousel">
      <div v-for="(service, index) in services" :key="index" class="service-item position-relative">
        <img class="img-fluid" :src="service.image" alt="" style="background: rgba(33, 30, 28, 0.7);">
        <div class="service-text text-center">
          <h4 class="text-white font-weight-medium px-3">{{ service.title }}</h4>
          <p class="text-white px-3 mb-3">{{ service.description }}</p>
          <div class="w-100 bg-white text-center p-4 description-box">
            <a class="btn btn-primary" href="">Make Order</a>
          </div>
        </div>
      </div>
    </div>
        <div class="row justify-content-center bg-appointment mx-0">
            <div class="col-lg-6 py-5">
                <div class="p-5 my-5" style="background: rgba(33, 30, 28, 0.7);">
                    <h1 class="text-white text-center mb-4">Make Appointment</h1>
                    <form @submit.prevent="handleSubmit">
                        <div class="form-row">
                            <div class="col-sm-6">
                                <div class="form-group">
                                    <input type="text" v-model="formData.name" class="form-control bg-transparent p-4" placeholder="Your Name" required />
                                </div>
                            </div>
                            <div class="col-sm-6">
                                <div class="form-group">
                                    <input type="email" v-model="formData.email" class="form-control bg-transparent p-4" placeholder="Your Email" required />
                                </div>
                            </div>
                        </div>
                        <div class="form-row">
                            <div class="col-sm-6">
                                <div class="form-group">
                                    <div class="date" id="date" data-target-input="nearest">
                                        <input type="date" v-model="formData.date" :min="minDate" :max="maxDateValue" class="form-control bg-transparent p-4 datetimepicker-input"  />
                                    </div>
                                </div>
                            </div>
                            <div class="col-sm-6">
                                <div class="form-group">
                                  
                                  <select v-model="selectedTime" @change=selectHour class="custom-select bg-transparent px-4" id="appointment-time" style="height: 47px;">
                                    
                                  <option value="">Choose an hour</option>
                                   <option v-for="time in availableTimes" :key="time">{{ time }}</option>
                                  </select>
                                </div>
                            </div>
                        </div>
                        <div class="form-row">
                            <div class="col-sm-6">
                                <div class="form-group">
                                    <select v-model="formData.selectedservice" @change="calculateServiceTime" class="custom-select bg-transparent px-4" id="services-select" style="height: 47px;">
                                        <option value="-1">Select A Service</option>
                                        <option v-for="service in availableServices" :key="service.id" :value="service.id">{{ service.title + " (" + service.duration }} minutes)</option>
                                    </select>
                                </div>
                            </div>
                            <div class="col-sm-6">
                                <button class="btn btn-primary btn-block" type="submit" style="height: 47px;">Make Appointment</button>
                            </div>
                        </div>
                    </form>
                    <p style="text-align: center;">{{ errorlogger }}</p>
                </div>
            </div>
        </div>
    </div>
    <!-- Service End -->
</template>

<script setup>
import { faBatteryEmpty } from '@fortawesome/free-solid-svg-icons';
import emailjs from 'emailjs-com';

const services = [
  {
    id: 0,
    title: 'Body Massage',
    description: 'Elitr labore sit dolor erat est lorem diam sea ipsum diam dolor duo sit ipsum',
    image: '/assets/img/service-1.jpg',
    duration: 30,
  },
  {
    id: 1,
    title: 'Stone Therapy',
    description: 'Elitr labore sit dolor erat est lorem diam sea ipsum diam dolor duo sit ipsum',
    image: '/assets/img/service-2.jpg',
    duration: 45,
  },
  {
    id: 2,
    title: 'Facial Therapy',
    description: 'Elitr labore sit dolor erat est lorem diam sea ipsum diam dolor duo sit ipsum',
    image: '/assets/img/service-3.jpg',
    duration: 15,
  },
  {
    id: 3,
    title: 'Skin Care',
    description: 'Elitr labore sit dolor erat est lorem diam sea ipsum diam dolor duo sit ipsum',
    image: '/assets/img/service-4.jpg',
    duration: 30,
  },
  {
    id: 4,
    title: 'Stream Bath',
    description: 'Elitr labore sit dolor erat est lorem diam sea ipsum diam dolor duo sit ipsum',
    image: '/assets/img/service-5.jpg',
    duration: 60,
  },
  {
    id: 5,
    title: 'Face Masking',
    description: 'Elitr labore sit dolor erat est lorem diam sea ipsum diam dolor duo sit ipsum',
    image: '/assets/img/service-6.jpg',
    duration: 45,
  },
];

const errorlogger = ref("");
let apidatamock = [];

const openedHours = ref({
  // UTC time. Adjust according your needs. Note that this is made to consider Winter/summer hours, therefore use gmt hours and you won't need to change this.
  starting: '08:00',
  finishing: '19:00',
  minuterange: 15,
  maxservicetime: 60,
})

const formData = ref({
  name: '',
  email: '',
  date: '',
  time: '',
  selectedservice: '-1',
  selectedservicetime: 0,
});

const emailsent = ref(false);

const selectedTime = ref('');

const calculatedValue = ref(0);

const availableTimes = ref([]);

const availableServices = ref([]);

// Esta función calcula los tiempos disponibles
const calculateAvailableTimes = () => {


  const times = [];
  const today = new Date();
  const todaydate = today.getDate();
  const currentMonth = (today.getMonth() + 1).toString().padStart(2, '0');
  const todayYear = today.getFullYear();
  const todayMins = today.getMinutes();
  const todayhours = today.getHours();

  // Calcula el número de minutos que faltan para el próximo cuarto de hora
  const minutesToNextQuarter = 15 - (todayMins % 15);

  // Ajusta los minutos al próximo cuarto de hora
  const roundedMinutes = (todayMins + minutesToNextQuarter) % 60;

  const formattedTime = `${todayhours.toString().padStart(2, '0')}:${roundedMinutes.toString().padStart(2, '0')}`;
  const formattedday = `${todayYear}-${currentMonth}-${todaydate}`;
  let currentMinutes;

  if (formData.value.date === formattedday) {
    if(timeToMinutes(formattedTime) >= timeToMinutes(openedHours.value.starting)){
      currentMinutes = timeToMinutes(formattedTime) + 120; 
    }else{
      currentMinutes =  timeToMinutes(openedHours.value.starting);
    }

   
  } else {
    currentMinutes = timeToMinutes(openedHours.value.starting);
  }


  const reservedTimes = apidatamock.map((reservation) => ({
    hour: reservation.hour,
    duration: reservation.duration,
    date: reservation.date,
  }));




  while (currentMinutes <= timeToMinutes(openedHours.value.finishing)) {
    if(currentMinutes === timeToMinutes(openedHours.value.finishing)){
      break;
    }
    let durationFound = null;

 

    const isReserved = reservedTimes.some(item => {  
      if (item.hour === minutesToTime(currentMinutes) && formData.value.date === item.date) {
        durationFound = item.duration;
        return true; 
      }
      return false; 
    });


    if (isReserved) {
      currentMinutes += durationFound;
    } else {
      
      times.push(minutesToTime(currentMinutes));
      currentMinutes += openedHours.value.minuterange;
    }
  }

  selectHour();
  availableTimes.value = times; // Actualiza el valor de availableTimes
 
};

// Observa el cambio en formData.date y llama a la función para calcular los tiempos disponibles
watch(() => formData.value.date, (newDate, oldDate) => {
  // Ejecuta availableTimes cuando formData.date cambia
  calculateAvailableTimes();
  selectedTime.value = '';
  document.querySelector('#appointment-time').value = -1;
});

// Llama a calculateAvailableTimes una vez al inicio para la fecha inicial
onMounted(() => {
  fetchAppointmentData();
  calculateAvailableTimes();

});


function timeToMinutes(time) {
  if (!time) return 0; // Si time es undefined, regresamos 0 minutos
  const [hours, minutes] = time.split(':').map(Number);
  return hours * 60 + minutes;
}
function minutesToTime(minutes) {
  const hours = Math.floor(minutes / 60);
  const remainingMinutes = minutes % 60;
  return `${String(hours).padStart(2, '0')}:${String(remainingMinutes).padStart(2, '0')}`;
}


const today = new Date();
const maxDate = new Date(today);
maxDate.setMonth(today.getMonth() + 3);

const minDate = computed(() => {
  const year = today.getFullYear();
  const month = String(today.getMonth() + 1).padStart(2, '0');
  const day = String(today.getDate()).padStart(2, '0');
  return `${year}-${month}-${day}`;
});

const maxDateValue = computed(() => {
  const year = maxDate.getFullYear();
  const month = String(maxDate.getMonth() + 1).padStart(2, '0');
  const day = String(maxDate.getDate()).padStart(2, '0');
  return `${year}-${month}-${day}`;
});

const selectHour = () => {
selectedTime.value = selectedTime.value;
formData.value.time = selectedTime.value;

calculatedValue.value = getSelectedHourTimeToNextReservation (availableTimes.value, selectedTime.value)

const filteredServices = services.filter(service => service.duration <= calculatedValue.value);

  availableServices.value = filteredServices
  


getLastAppointments(openedHours.value.maxservicetime, openedHours.value.minuterange);
}

 const getSelectedHourTimeToNextReservation = (times, timeValue) => {
 let counter = 0;
const mapped = times.map((time) => {
return timeToMinutes(time);
});
const ratio = 4;
const lastAppointments = mapped.slice(-ratio);
const lastAppointment = mapped.slice(-1);



for(let i = timeToMinutes(timeValue); i <= mapped[mapped.length - 1]; i+=openedHours.value.minuterange){
  if (mapped.includes(i) && !lastAppointments.includes(i)) {
      counter += 15;
      continue;
    }
    else if(mapped.includes(i) && lastAppointments.includes(i)){
      counter += 15;
       continue;
    }
    break;
  }
return counter;
 };

 const getLastAppointments = (max, min) => {
availableTimes.value

  return;

 }
 function generateRandomString(length) {
  const characters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
  let result = '';
  const charactersLength = characters.length;
  for (let i = 0; i < length; i++) {
    const randomIndex = Math.floor(Math.random() * charactersLength);
    result += characters.charAt(randomIndex);
  }
  return result;
}
const calculateServiceTime = () => {
  
  formData.value.selectedservicetime = services[formData.value.selectedservice].duration;

}
const handleSubmit = async () => {

  if(emailsent.value === true){
    errorlogger.value = "You have already made an appointment";
    return;
  }
  const nameRegex = /^[A-Za-zÁÉÍÓÚÜÑáéíóúüñ\s']{1,30}$/; 

  if(!nameRegex.test(formData.value.name)){
    errorlogger.value = "Please insert a valid name.";
    return;
  }


  const token = generateRandomString(10);
  const link = `https://nuxt-spa-template.vercel.app/confirmation/${token}`;
  const cancellation = `https://nuxt-spa-template.vercel.app/cancellation/${token}`
  const message = `Date: ${formData.value.date} Hour:${formData.value.time} Service:${services[formData.value.selectedservice].title} Duration:${formData.value.selectedservicetime} minutes`;



  const emailParams = {
    from_name: formData.value.name,
    to_email: formData.value.email,
    link: link,
    cancellation_link: cancellation,
    message: message,
  };

  // Envía el correo electrónico
  emailjs.send('service_87e2u3i', 'template_5d30ygk', emailParams, 'UJc6rUhAnbit4DOGE')
    .then(async (response) => {

   
      
      // Construye los datos para la solicitud POST
      const appointmentData = {
        date: formData.value.date,
        hour: formData.value.time,
        email: formData.value.email,
        name: formData.value.name,
        duration: formData.value.selectedservicetime,
        token: token, 
        serviceid: formData.value.selectedservice, 
        validated: false,
      };

   
      const responsePost = await fetch('https://spa-template-backend-express.vercel.app/api/v1/appointment', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(appointmentData),
      });

      if (responsePost.ok) {
    
        errorlogger.value = "Request sent. Please check your e-mail to confirm the appointment.";
        emailsent.value = true;
      
      } else {
        errorlogger.value = "An error has occurred while confirming the appointment.";
   
      }
    })
    .catch((error) => {
      console.error('Error al enviar el correo electrónico:', error);
      errorlogger.value = "An error has occurred while sending the confirmation e-mail.";
    });
};
 
const fetchAppointmentData = async () => {
  try {
    const response = await fetch('https://spa-template-backend-express.vercel.app/api/v1/appointment/');
    
    if (response.status === 200) {
      const jsonData = await response.json();
      
      // Mapea los datos al formato deseado y asigna a apidatamock
      apidatamock = jsonData.data.map((item) => ({
        reservationId: item.id.toString(),
        date: item.date,
        hour: item.hour,
        duration: parseInt(item.duration), // Convierte la duración a un número entero
        serviceId: item.serviceid
      }));
    } else {
      console.error('Error en la solicitud GET:', response.status);
    }
  } catch (error) {
    console.error('Error al realizar la solicitud GET:', error);
  }
};
</script>


<style scoped>
.service-carousel {
  display: flex;
  flex-wrap: nowrap;
  overflow: hidden;

}

.service-item {
  flex: 0 0 auto;
  position: relative;
  

}
.service-item::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 40px; 
  background-color: white; 
  z-index: 3;
}

.service-item:hover::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 0px; 
  background-color: white; 
  transition: 0.5s;
  }



.service-item, .service-text{
    background-color: rgba(33, 30, 28, 0.5);
}

.service-item:hover, .service-text:hover {
    background-color: rgba(33, 30, 28, 0.5);
  cursor: grab;
  transition: 0.5s;
  
}

.service-item img {
  max-width: 20vw;
  height: auto;

}
.service-item:hover img {
  max-width: 20vw;
  height: auto;

}


.service-item:hover > .service-text > .description-box {
  min-height: 130px !important;
  transition: 0.5s;
 }

.bg-appointment {
  background: linear-gradient(rgba(33, 30, 28, 0.7), rgba(33, 40, 28, 0.7)), url(/assets/img/carousel-1.jpg), no-repeat center center;
  background-size: cover;
}

@media (max-width: 753px) {
  .service-item img {
  max-width: 100vw;
  height: auto;
  
}
.service-item:hover img {
  max-width: 100vw;
  height: auto;
  
}
}
</style>