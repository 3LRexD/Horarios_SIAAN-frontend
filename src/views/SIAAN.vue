<template>
  <div class="siaan-container">
    <!-- Topbar -->
    <header class="topbar">
      <div class="left-section">
        <button id="menu-btn" class="menu-btn" @click="toggleSidebar">
          <i class="fa-solid fa-bars"></i>
        </button>
        <h1>SISTEMA ACADÉMICO NACIONAL</h1>
        <div id="reloj">{{ currentTime }}</div>
      </div>

      <div class="user-info" :class="{ active: showDropdown }" @click="toggleDropdown">
        <div class="user-trigger">
          <img src="../assets/img/usuario-ucb.png" alt="Usuario" class="user-logo">
          <span>{{ userName }}</span>
          <i class="fa-solid fa-chevron-down arrow"></i>
        </div>

        <div class="dropdown-menu" :class="{ show: showDropdown }" @click.stop>
          <div class="dropdown-item" @click="handleDropdownClick('Editar datos personales')">
            <i class="fa-solid fa-user"></i> Editar datos personales
          </div>
          <div class="dropdown-item" @click="handleDropdownClick('Cambio de contraseña')">
            <i class="fa-solid fa-key"></i> Cambio de contraseña
          </div>
          <div class="dropdown-item logout" @click="handleDropdownClick('Terminar sesión')">
            <i class="fa-solid fa-right-from-bracket"></i> Terminar sesión
          </div>
        </div>
      </div>
    </header>

    <!-- Sidebar -->
    <aside id="sidebar" class="sidebar" :class="{ open: sidebarOpen }">
      <div class="logo-container">
        <img src="../assets/img/logo-ucb.png" alt="UCB Logo" class="ucb-logo">
      </div>

      <ul class="menu">
        <li class="menu-item" @click="toggleSubmenu($event)">
          <i class="fa-solid fa-user-graduate"></i> Estudiante
          <i class="fa-solid fa-chevron-right arrow"></i>
        </li>

        <ul class="submenu">
          <li><i class="fa-solid fa-credit-card"></i> Plataforma De Pagos En Línea</li>

          <li class="menu-item" @click="toggleSubmenu($event)">
            <i class="fa-solid fa-id-card"></i> Mis Datos Personales
            <i class="fa-solid fa-chevron-right arrow"></i>
          </li>
          <ul class="submenu">
            <li><i class="fa-solid fa-user"></i> Perfil</li>
            <li><i class="fa-solid fa-pen"></i> Actualizar Datos</li>
          </ul>

          <li class="menu-item" @click="toggleSubmenu($event)">
            <i class="fa-solid fa-folder-open"></i> File Académico
            <i class="fa-solid fa-chevron-right arrow"></i>
          </li>
          <ul class="submenu">
            <li><i class="fa-solid fa-dollar-sign"></i> Pagos Pendientes</li>
            <li><i class="fa-solid fa-graduation-cap"></i> Kardex De Estudiante</li>
            <li><i class="fa-solid fa-lines-leaning"></i> Oferta de Asignaturas</li>
            <li @click="navigateToHorario"><i class="fa-solid fa-calendar-plus"></i> Crear Horario</li>
            <li><i class="fa-solid fa-book"></i> Mis Asignaturas</li>
            <li><i class="fa-solid fa-people-line"></i> Programa De Profesionalización</li>

            <li class="menu-item" @click="toggleSubmenu($event)">
              <i class="fa-solid fa-chart-column"></i> Reportes
              <i class="fa-solid fa-chevron-right arrow"></i>
            </li>
            <ul class="submenu">
              <li><i class="fa-solid fa-layer-group"></i> Varios</li>
            </ul>
          </ul>

          <li><i class="fa-solid fa-list"></i> Inscripción De Asignaturas</li>
          <li><i class="fa-solid fa-file-circle-minus"></i> Abandono De Asignatura</li>
          <li><i class="fa-solid fa-clipboard-list"></i> Solicitudes Enviadas</li>
          <li><i class="fa-solid fa-book-open"></i> Biblioteca Virtual</li>
          <li><i class="fa-solid fa-id-badge"></i> Credencial Digital</li>
          <li><i class="fa-solid fa-circle-info"></i> Tutoriales Y Recursos De Apoyo</li>
          <li><i class="fa-solid fa-video"></i> Videos Estudiantes</li>
        </ul>
      </ul>
    </aside>

    <!-- Main Content con Carrusel -->
    <main class="content" :class="{ 'sidebar-open': sidebarOpen }">
      
      <div class="carousel-container" v-if="$route.path === '/siaan'">
        <div class="carousel" :style="{ transform: `translateX(-${currentIndex * 100}%)` }">
          
          <div 
            v-for="(image, index) in carouselImages" 
            :key="index" 
            class="carousel-slide"
          >
            <div class="card">
              <img :src="image.src" :alt="image.alt" class="carousel-image">
            </div>
          </div>
        </div>
        
        <div class="carousel-indicators">
          <button
            v-for="(image, index) in carouselImages"
            :key="index"
            :class="['indicator', { active: currentIndex === index }]"
            @click="goToSlide(index)"
          ></button>
        </div>

        <button class="carousel-control prev" @click="prevSlide">
          <i class="fa-solid fa-chevron-left"></i>
        </button>
        <button class="carousel-control next" @click="nextSlide">
          <i class="fa-solid fa-chevron-right"></i>
        </button>
      </div>

      <router-view></router-view>

    </main>

    <!-- Footer -->
    <footer class="footer-minimal">
      <p>© 2025 SIAAN UCB | Desarrollado por el equipo whendell</p>
    </footer>
  </div>
</template>

<script>

import appUcb from '../assets/img/app-ucb.png'
import frase1 from '../assets/img/frase1.png'
import frase2 from '../assets/img/frase2.png'

export default {
  name: 'SIAAN',
  data() {
    return {
      sidebarOpen: false,
      showDropdown: false,
      currentTime: '',
      userName: 'Usuario',
      timer: null,
      carouselInterval: null,
      currentIndex: 0,
      carouselImages: [
        { src: appUcb, alt: 'UCB App' },
        { src: frase1, alt: 'Frase motivacional 1' },
        { src: frase2, alt: 'Frase motivacional 2' }
      ]
    }
  },
  mounted() {
    this.verifyAuthentication();
    this.startClock();
    this.setupClickOutside();
    this.startCarousel();
  },
  beforeUnmount() {
    if (this.timer) {
      clearInterval(this.timer);
    }
    if (this.carouselInterval) {
      clearInterval(this.carouselInterval);
    }
  },
  methods: {
    verifyAuthentication() {
      const userData = JSON.parse(localStorage.getItem("usuario"));
      if (userData && userData.nombre) {
        this.userName = userData.nombre;
      } else {
        this.$router.push('/login');
      }
    },

    toggleSidebar() {
      this.sidebarOpen = !this.sidebarOpen;
    },

    toggleDropdown(event) {
      event.stopPropagation();
      this.showDropdown = !this.showDropdown;
    },

    toggleSubmenu(event) {
      event.stopPropagation();
      const menuItem = event.currentTarget;
      const submenu = menuItem.nextElementSibling;
      const arrow = menuItem.querySelector('.arrow');

      if (submenu.style.display === 'flex') {
        submenu.style.display = 'none';
        arrow.classList.remove('fa-chevron-down');
        arrow.classList.add('fa-chevron-right');
      } else {
        submenu.style.display = 'flex';
        arrow.classList.remove('fa-chevron-right');
        arrow.classList.add('fa-chevron-down');
      }
    },

    handleDropdownClick(option) {
      switch (option) {
        case "Editar datos personales":
          alert("Editar datos personales próximamente....");
          break;
        case "Cambio de contraseña":
          alert("Cambio de contraseña próximamente....");
          break;
        case "Terminar sesión":
          this.logout();
          break;
        default:
          console.log("Opción no reconocida:", option);
      }
      this.showDropdown = false;
    },

    logout() {
      localStorage.clear();
      sessionStorage.clear();
      this.$router.push('/login');
    },

    navigateToHorario() {
  // Asegúrate de que apunte a la ruta hija
      this.$router.push('/siaan/crear-horario');
    },

    startClock() {
      this.updateTime();
      this.timer = setInterval(this.updateTime, 1000);
    },

    updateTime() {
      const fecha = new Date();
      const opciones = { 
        hour: '2-digit', 
        minute: '2-digit', 
        second: '2-digit',
        hour12: false
      };
      
      const horaActual = fecha.toLocaleTimeString('es-BO', opciones);
      const fechaActual = fecha.toLocaleDateString('es-BO');
      this.currentTime = `${fechaActual} | ${horaActual}`;
    },

    setupClickOutside() {
      document.addEventListener('click', () => {
        this.showDropdown = false;
      });
    },

    // Métodos del carrusel
    startCarousel() {
      this.carouselInterval = setInterval(() => {
        this.nextSlide();
      }, 3000); 
    },

    nextSlide() {
      this.currentIndex = (this.currentIndex + 1) % this.carouselImages.length;
    },

    prevSlide() {
      this.currentIndex = this.currentIndex === 0 
        ? this.carouselImages.length - 1 
        : this.currentIndex - 1;
    },

    goToSlide(index) {
      this.currentIndex = index;
      this.restartCarousel();
    },

    restartCarousel() {
      if (this.carouselInterval) {
        clearInterval(this.carouselInterval);
      }
      this.startCarousel();
    }
  }
}
</script>

<style scoped>

* {
  font-family: "Arial", sans-serif;
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  background: #f3f4f6;
  height: 100vh;
  overflow: hidden;
}

.siaan-container {
  min-height: 100vh;
  position: relative;
}

/* Topbar */
.topbar {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 60px;
  background: #002b55;   
  color: white;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 20px;
  z-index: 1000;         
}

.left-section {
  display: flex;
  align-items: center;
  gap: 15px;
}

.left-section h1,
.topbar h1 {
  font-size: 18px;
  font-weight: bold;
  letter-spacing: 0.5px;
}

.menu-btn {
  font-size: 18px;
  background: none;
  border: none;
  color: white;
  cursor: pointer;
}

.user-info {
  display: flex;
  align-items: center;
  gap: 10px;
  position: relative;
  cursor: pointer;
  user-select: none;
}

.user-logo {
  width: 35px;
  height: 35px;
  border-radius: 50%;
}

.user-trigger {
  display: flex;
  align-items: center;
  gap: 8px;
}

.user-info .arrow {
  font-size: 12px;
  transition: transform 0.25s ease;
}

.user-info.active .arrow {
  transform: rotate(180deg);
}

.dropdown-menu {
  position: absolute;
  top: 48px;
  right: 0;
  background: #fff;
  color: #333;
  border-radius: 8px;
  box-shadow: 0 3px 10px rgba(0,0,0,0.15);
  width: 220px;
  padding: 8px 0;
  display: none;
  flex-direction: column;
  z-index: 1100;
  animation: fadeIn 0.2s ease;
}

.dropdown-menu.show {
  display: flex;
}

.dropdown-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px 18px;
  font-size: 14px;
  color: #333;
  transition: background 0.2s;
  cursor: pointer;
}

.dropdown-item i {
  color: #002b55;
  width: 18px;
}

.dropdown-item:hover {
  background: #eef4fb;
}

.dropdown-item.logout {
  border-top: 1px solid #eee;
  color: #c0392b;
}

.dropdown-item.logout i {
  color: #c0392b;
}

#reloj {
  font-size: 14px;
  font-weight: bold;
  color: white;
  margin-left: auto;
  margin-right: 15px;
}

.sidebar {
  position: fixed;
  top: 60px;
  left: -260px;
  width: 260px;
  height: calc(100% - 60px);
  background: #ffffff;
  box-shadow: 2px 0 5px rgba(0,0,0,0.15);
  transition: left 0.3s ease;
  overflow-y: auto;
  z-index: 900;
}

.sidebar.open {
  left: 0;
}

.logo-container {
  text-align: center;
  padding: 15px 0;
  border-bottom: 1px solid #e5e5e5;
}

.ucb-logo {
  width: 200px;
}

.menu {
  list-style: none;
  padding: 10px 0;
}

.menu-item {
  padding: 12px 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: #002b55;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.3s;
}

.menu-item:hover {
  background: #e5ecf7;
}

.menu-item i {
  margin-right: 10px;
}

.submenu {
  list-style: none;
  display: none;
  flex-direction: column;
  background: #f7f9fc;
}

.submenu li {
  padding: 10px 35px;
  font-size: 14px;
  color: #002b55;
  cursor: pointer;
  transition: background 0.3s;
}

.submenu li:hover {
  background: #dce6f3;
}

.content {
  margin-top: 60px;
  margin-left: 0;
  padding: 25px;
  display: flex;
  justify-content: center;
  align-items: center;
  transition: margin-left 0.3s;
  min-height: calc(100vh - 100px);
}

.sidebar.open ~ .content {
  margin-left: 260px;
}

.carousel-container {
  position: relative;
  width: 100%;
  max-width: 600px;
  height: auto;
  overflow: hidden;
  border-radius: 10px;
}

.carousel {
  display: flex;
  transition: transform 0.5s ease-in-out;
  height: 100%;
}

.carousel-slide {
  min-width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 0 10px;
  box-sizing: border-box;
}

.card {
  background: white;
  border-radius: 10px;
  box-shadow: 0 2px 6px rgba(0,0,0,0.1);
  overflow: hidden;
  width: 550px;
  transition: transform 0.2s ease;
}

.card:hover {
  transform: translateY(-4px);
}

.carousel-image {
  width: 100%;
  height: auto;
  display: block;
}

.carousel-indicators {
  position: absolute;
  bottom: 15px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  gap: 8px;
  z-index: 10;
}

.indicator {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  border: none;
  background: rgba(255, 255, 255, 0.5);
  cursor: pointer;
  transition: all 0.3s ease;
}

.indicator.active {
  background: white;
  transform: scale(1.2);
}

.indicator:hover {
  background: rgba(255, 255, 255, 0.8);
}

.carousel-control {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  background: rgba(0, 43, 85, 0.7);
  color: white;
  border: none;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  transition: all 0.3s ease;
  z-index: 10;
}

.carousel-control:hover {
  background: rgba(0, 43, 85, 0.9);
  transform: translateY(-50%) scale(1.1);
}

.carousel-control.prev {
  left: 15px;
}

.carousel-control.next {
  right: 15px;
}

.carousel-control i {
  font-size: 16px;
}

/* Footer */
.footer-minimal {
  background: #002b55;
  color: #fff;
  text-align: center;
  padding: 10px;
  font-size: 13px;
  position: fixed;
  bottom: 0;
  width: 100%;
}

/* Animations */
@keyframes fadeIn {
  from { opacity: 0; transform: translateY(-5px); }
  to { opacity: 1; transform: translateY(0); }
}

/* Responsive */
@media (max-width: 780px) {
  .sidebar.open ~ .content {
    margin-left: 0;
  }

  .user-info span {
    display: none;
  }
  
  .card {
    width: 100%;
    max-width: 400px;
  }
  
  .carousel-container {
    max-width: 90%;
  }
  
  .carousel-control {
    width: 35px;
    height: 35px;
  }
  
  .carousel-control i {
    font-size: 14px;
  }
}

@media (max-width: 480px) {
  .carousel-slide {
    padding: 0 5px;
  }
}
</style>