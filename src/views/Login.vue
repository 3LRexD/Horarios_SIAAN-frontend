<template>
  <div class="login-background">
    <div class="login-box">
      <h2>Iniciar sesión</h2>

      <form @submit.prevent="handleLogin" novalidate>
        <div class="input-container" :class="{ error: errors.usuario }">
          <input 
            type="text" 
            id="usuario" 
            v-model="form.usuario" 
            placeholder=" " 
            required
          >
          <label for="usuario">Ingrese su usuario *</label>
          <small class="error-text">{{ errors.usuario }}</small>
        </div>

        <div class="input-container password-container" :class="{ error: errors.password }">
          <input 
            :type="showPassword ? 'text' : 'password'" 
            id="password" 
            v-model="form.password" 
            placeholder=" " 
            required
          >
          <label for="password">Ingrese su contraseña *</label>
          <span class="toggle-password" @click="togglePassword">👁️</span>
          <small class="error-text">{{ errors.password }}</small>
        </div>

        <button type="submit" class="btn-enviar">ENVIAR</button>
      </form>

      <div class="divider">o</div>

      <!-- Contenedor para el botón de Google -->
      <div id="googleButton"></div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Login',
  data() {
    return {
      form: {
        usuario: '',
        password: ''
      },
      errors: {
        usuario: '',
        password: ''
      },
      showPassword: false,
      usuariosValidos: [
        { correo: "jose.rodriguez.p", contrasena: "12345", nombre: "RODRIGUEZ PORCO JOSE FABIAN" },
        { correo: "ian.coaquira", contrasena: "67890", nombre: "COAQUIRA URIARTE IAN JAMID" },
        { correo: "regina.maldonado", contrasena: "54321", nombre: "MALDONADO CAMARGO REGINA ANDREA" },
        { correo: "josue.nina.r", contrasena: "09876", nombre: "NINA ROJAS JOSUE EMMANUEL" },
        { correo: "joseph.cortez", contrasena: "69696", nombre: "CORTEZ CHURA JOSEPH HANS WENDELL" }
      ]
    }
  },
  mounted() {
    this.loadGoogleScript();
  },
  methods: {
    togglePassword() {
      this.showPassword = !this.showPassword;
    },
    
    handleLogin() {
      this.errors = { usuario: '', password: '' };
      
      let valid = true;

      if (!this.form.usuario.trim()) {
        this.errors.usuario = "Debes ingresar un usuario.";
        valid = false;
      }

      if (!this.form.password.trim()) {
        this.errors.password = "Debes ingresar una contraseña.";
        valid = false;
      }

      if (!valid) return;

      const usuario = this.form.usuario.trim().toLowerCase();
      const password = this.form.password.trim();

      const usuarioEncontrado = this.usuariosValidos.find(
        u => u.correo === usuario && u.contrasena === password
      );

      if (usuarioEncontrado) {
        localStorage.setItem("usuario", JSON.stringify({
          correo: usuario,
          nombre: usuarioEncontrado.nombre
        }));
        
        this.$router.push('/siaan');
      } else {
        this.errors.password = "Usuario o contraseña incorrectos.";
      }
    },

    loadGoogleScript() {
      if (document.querySelector('script[src*="accounts.google.com"]')) {
        this.initializeGoogleSignIn();
        return;
      }

      const script = document.createElement('script');
      script.src = 'https://accounts.google.com/gsi/client';
      script.async = true;
      script.defer = true;
      script.onload = () => {
        this.initializeGoogleSignIn();
      };
      document.head.appendChild(script);
    },

    initializeGoogleSignIn() {
      const checkGoogle = () => {
        if (window.google && window.google.accounts) {
          const CLIENT_ID = "590026680933-n09ornjl6d84vhtg90njbv4897ebnh2k.apps.googleusercontent.com";
          
          window.google.accounts.id.initialize({
            client_id: CLIENT_ID,
            callback: this.handleCredentialResponse,
          });

          window.google.accounts.id.renderButton(
            document.getElementById("googleButton"),
            { 
              theme: "outline", 
              size: "large", 
              text: "signin_with"
            }
          );
        } else {
          setTimeout(checkGoogle, 100);
        }
      };
      
      checkGoogle();
    },

    handleCredentialResponse(response) {
      try {
        const payload = JSON.parse(atob(response.credential.split(".")[1]));
        const email = payload.email;

        if (email.endsWith("@ucb.edu.bo")) {
          localStorage.setItem("usuario", JSON.stringify({
            correo: email,
            nombre: payload.name || "Usuario UCB",
            metodo: "google"
          }));
          
          this.$router.push('/siaan');
        } else {
          alert("Solo cuentas @ucb.edu.bo pueden acceder.");
        }
      } catch (error) {
        alert("Error al procesar el inicio de sesión con Google.");
      }
    }
  }
}
</script>

<style scoped>
.login-background {
  background: url('../assets/img/fondo-ucb.png') no-repeat center center fixed;
  background-size: cover;
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  margin: 0;
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
}

.login-box {  
  background: white;
  padding: 30px 35px;
  border-radius: 5px;
  box-shadow: 0 6px 25px rgba(0,0,0,0.25);
  width: 270px;
  height: auto;
  min-height: 350px;
  text-align: center;  
  backdrop-filter: blur(8px);
  animation: fadeIn 0.7s ease;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(15px); }
  to { opacity: 1; transform: translateY(0); }
}

.login-box h2 {
  margin-bottom: 20px;
  margin-top: 0;
  color: #001f3f;
  font-weight: 500;
  text-align: start;
}

.input-container {
  position: relative;
  margin-bottom: 20px;
  width: 100%;
}

.input-container input {
  width: 100%;
  padding: 12px 10px;
  border: 1.5px solid #ccc;
  border-radius: 6px;
  font-size: 14px;
  outline: none;
  background: transparent;
  transition: all 0.3s ease;
}

.input-container label {
  position: absolute;
  left: 10px;
  top: 50%;
  transform: translateY(-50%);
  color: #666;
  font-size: 14px;
  pointer-events: none;
  transition: all 0.25s ease;
  background: transparent;
  padding: 0 4px;
}

.input-container input:focus + label,
.input-container input:not(:placeholder-shown) + label {
  top: 0;
  left: 8px;
  font-size: 12px;
  color: #00244d;
  background: rgb(255, 255, 255);
  border-radius: 4px;
  padding: 0 6px;
}

.password-container {
  position: relative;
  margin-bottom: 20px;
}

.toggle-password {
  position: absolute;
  right: 12px;
  top: 50%;
  transform: translateY(-50%);
  cursor: pointer;
  user-select: none;
  font-size: 18px;
  opacity: 0.7;
  transition: opacity 0.3s;
  z-index: 2;
}

.toggle-password:hover {
  opacity: 1;
}

.btn-enviar {
  width: 100%;
  background-color: #00244d;
  color: white;
  padding: 12px;
  border: none;
  border-radius: 6px;
  font-size: 14px;
  cursor: pointer;
  font-weight: 500;
  transition: all 0.3s ease;
  margin-top: 10px;
}

.btn-enviar:hover {
  background-color: #003a7a;
  transform: translateY(-2px);
  box-shadow: 0 3px 10px rgba(0,0,0,0.15);
}

.divider {
  margin: 20px 0;
  color: #999;
  position: relative;
  font-size: 14px;
}

.divider::before,
.divider::after {
  content: "";
  position: absolute;
  top: 50%;
  width: 40%;
  height: 1px;
  background: #ddd;
}

.divider::before { 
  left: 0; 
}

.divider::after { 
  right: 0; 
}

.input-container.error input {
  border-color: #ff4444;
}

.error-text {
  color: #ff4444;
  font-size: 12px;
  display: block;
  text-align: left;
  margin-top: 5px;
  margin-left: 2px;
}

#googleButton {
  display: flex;
  justify-content: center;
  margin-top: 10px;
  min-height: 40px;
}

@media (max-width: 480px) {
  .login-box {
    width: 90%;
    max-width: 300px;
    padding: 25px 20px;
  }
  
  .login-box h2 {
    font-size: 1.3rem;
  }
}
</style>