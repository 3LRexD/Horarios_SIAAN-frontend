<template>
  <div class="horario-container">
    <header class="topbar">
      <div class="left-section">
        <button class="menu-btn" @click="$router.push('/siaan')">
          <i class="fa-solid fa-arrow-left"></i>
        </button>
        <h1>HORARIO ACADÉMICO</h1>
      </div>
      <div class="user-info">
        <img src="../assets/img/usuario-ucb.png" alt="Usuario" class="user-logo">
        <span>{{ userName }}</span>
      </div>
    </header>

    <main class="horario-content">
      <div class="horario-card">
        <h2>Horario en Desarrollo</h2>
        <p>Esta funcionalidad estará disponible próximamente.</p>
        <button @click="$router.push('/siaan')" class="btn-volver">
          Volver al SIAAN
        </button>
      </div>
    </main>
  </div>
</template>

<script>
export default {
  name: 'Horario',
  data() {
    return {
      userName: 'Usuario'
    }
  },
  mounted() {
    this.verifyAuthentication();
  },
  methods: {
    verifyAuthentication() {
      const userData = JSON.parse(localStorage.getItem("usuario"));
      if (userData && userData.nombre) {
        this.userName = userData.nombre;
      } else {
        this.$router.push('/login');
      }
    }
  }
}
</script>

<style scoped>
.horario-container {
  min-height: 100vh;
  background: #f3f4f6;
}

.topbar {
  background: #002b55;
  color: white;
  padding: 0 20px;
  height: 60px;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.left-section {
  display: flex;
  align-items: center;
  gap: 15px;
}

.menu-btn {
  background: none;
  border: none;
  color: white;
  font-size: 18px;
  cursor: pointer;
  padding: 8px;
}

.user-info {
  display: flex;
  align-items: center;
  gap: 10px;
}

.user-logo {
  width: 35px;
  height: 35px;
  border-radius: 50%;
}

.horario-content {
  padding: 40px 20px;
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: calc(100vh - 60px);
}

.horario-card {
  background: white;
  padding: 40px;
  border-radius: 10px;
  box-shadow: 0 4px 15px rgba(0,0,0,0.1);
  text-align: center;
  max-width: 500px;
  width: 100%;
}

.horario-card h2 {
  color: #002b55;
  margin-bottom: 15px;
}

.horario-card p {
  color: #666;
  margin-bottom: 25px;
  line-height: 1.5;
}

.btn-volver {
  background: #002b55;
  color: white;
  padding: 12px 30px;
  border: none;
  border-radius: 6px;
  font-size: 14px;
  cursor: pointer;
  transition: background 0.3s;
}

.btn-volver:hover {
  background: #003a7a;
}

@media (max-width: 768px) {
  .horario-card {
    padding: 30px 20px;
    margin: 20px;
  }
  
  .user-info span {
    display: none;
  }
}
</style>