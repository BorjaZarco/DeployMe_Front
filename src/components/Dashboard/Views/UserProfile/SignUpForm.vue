<template>
  <card>
    <h4 slot="header" class="text-center">Registrate!</h4>
    <form class="justify-content-center">
      <div class="row justify-content-center">
        <div class="col-md-10">
          <fg-input type="text"
                    label="Username"
                    placeholder=""
                    v-model="username">
          </fg-input>
        </div>
      </div>
      
      <div class="row justify-content-center">
        <div class="col-md-10">
          <fg-input type="text"
                    label="Email"
                    placeholder=""
                    v-model="email">
          </fg-input>
        </div>
      </div>
      
      <div class="row justify-content-center" v-if="badEmail">
        <p class="text-danger">Introduce un email válido</p>
      </div>      

      <div class="row justify-content-center">
        <div class="col-md-10">
          <fg-input type="password"
                    label="Contraseña"
                    placeholder=""
                    v-model="password">
          </fg-input>
        </div>
      </div>

      <div class="row justify-content-center" v-if="shortPassword">
        <p class="text-danger">La contraseña ha de tener un mínimo de 8 caracteres</p>
      </div>  

      <div class="row justify-content-center">
        <div class="col-md-10">
          <fg-input type="password"
                    label="Repite Contraseña"
                    placeholder=""
                    v-model="doubleP">
          </fg-input>
        </div>
      </div>

      <div class="text-center">
        <p>Ya tienes cuenta? <router-link to="/login">Inicia sesión</router-link></p>
      </div>

      <div class="row justify-content-center" v-if="badPassword">
        <p class="text-danger">Las contraseñas no coinciden</p>
      </div>      

      <div class="text-center">
        <button type="submit" 
                class="btn btn-info btn-fill float-center" 
                :disabled="badPassword || shortPassword || badEmail"
                @click="signup">
          Sign Up
        </button>
      </div>
      <div class="clearfix"></div>
    </form>
  </card>
</template>
<script>
  import { post } from 'axios';
  import Card from 'src/components/UIComponents/Cards/Card.vue'

  export default {
    components: {
      Card
    },
    data () {
      return {
        username: "",
        password: "",
        doubleP: "",
        email: "",
        badPassword: false,
        shortPassword: false, 
        badEmail: false
      }
    },
    methods: {
      signup () {
        if (this.badPassword || !this.username || !this.password || !this.email) {
          this.showNotification('Warn:','Rellena todos los campos', 5, 'warn')
        } else {
          this.showNotification('Info:', 'Creando cuenta...', 3, 'info');
          post('http://localhost:5000/api/users', { username: this.username, password: this.password, email: this.email })
          .then (res => {
            post('http://localhost:5000/api/login', {username: this.username, password: this.password})
            .then (res => {
              localStorage.token = res.data;
              localStorage.username = this.username;
              this.$router.push('home'); 
              this.showNotification('Info:', `Bienvenido ${localStorage.username}!, tu sesión durará 30 minutos`, 3, 'info');
            })
            .catch ( error => {
              this.showNotification('Error:', 'Se ha producido un error interno, intentalo loguearse manualmente', 3, 'error');
              this.$router.push('login');
            })
          })
          .catch ( error => {
            console.log(error);
            this.showNotification('Error:','Usuario ya registrado', 5, 'error')
          })      
        }
      },
      showNotification(title, body, duration, type){
        this.$notify({
            group: 'notification-group',
            title: title,
            text: body,
            type: type,
            duration: duration * 1000
        });
      },
      hideNotifications(){
          this.$notify({
              group: 'notification-group',
              clean: true
          })
      },
    },
    watch: {
      password: function () {
        this.badPassword = this.password !== this.doubleP;
        this.shortPassword = this.password.length < 8 ;
      },
      doubleP: function () {
        this.badPassword = this.password !== this.doubleP;
      }, 
      email: function () {
        var re = /^(?:[a-z0-9!#$%&amp;'*+/=?^_`{|}~-]+(?:\.[a-z0-9!#$%&amp;'*+/=?^_`{|}~-]+)*|"(?:[\x01-\x08\x0b\x0c\x0e-\x1f\x21\x23-\x5b\x5d-\x7f]|\\[\x01-\x09\x0b\x0c\x0e-\x7f])*")@(?:(?:[a-z0-9](?:[a-z0-9-]*[a-z0-9])?\.)+[a-z0-9](?:[a-z0-9-]*[a-z0-9])?|\[(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?|[a-z0-9-]*[a-z0-9]:(?:[\x01-\x08\x0b\x0c\x0e-\x1f\x21-\x5a\x53-\x7f]|\\[\x01-\x09\x0b\x0c\x0e-\x7f])+)\])$/;
        this.badEmail =  !(re.test(this.email));   
      }
    }
}
</script>
<style>
</style>
