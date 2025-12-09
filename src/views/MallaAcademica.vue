<template>
  <div class="contenedor-principal">
    <h1 class="titulo">Mi Malla Curricular</h1>

    <div v-if="error" class="error">
      <p>⚠️ No se pudo conectar con el servidor.</p>
    </div>

    <div v-else class="malla-grid">
      <div v-for="semestre in 9" :key="semestre" class="columna-semestre">
        <div class="encabezado-semestre">Semestre {{ semestre }}</div>
        
        <div class="lista-materias">
          <div 
            v-for="materia in filtrarPorSemestre(semestre)" 
            :key="materia.materia"
            class="tarjeta-materia"
            :class="{ 
              'aprobada': materia.estado === 'aprobado',
              'bloqueada': materia.estado === 'bloqueado' 
            }"
          >
            <div class="nombre-materia">
               <span v-if="materia.estado === 'bloqueado'">🔒 </span>
               {{ materia.materia }}
            </div>
            <div v-if="materia.estado === 'aprobado'" class="icon-check">✅</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'MallaAcademica',
  data() {
    return {
      materias: [], // Aquí guardamos los datos que vienen de la BD
      error: false
    }
  },
  mounted() {
    this.obtenerDatos();
  },
  methods: {
    async obtenerDatos() {
      try {
        // Petición a tu Backend (Asegúrate que el puerto 8080 sea correcto)
        const respuesta = await fetch('http://localhost:8080/api/estudiantes/1/malla');
        
        if (!respuesta.ok) throw new Error('Error en la red');
        
        this.materias = await respuesta.json();
        console.log("Datos cargados:", this.materias); // Para ver en la consola del navegador
      } catch (err) {
        console.error("Error cargando malla:", err);
        this.error = true;
      }
    },
    filtrarPorSemestre(numeroSemestre) {
      // Devuelve solo las materias que coinciden con el semestre de la columna
      return this.materias.filter(m => m.semestre === numeroSemestre);
    }
  }
}
</script>

<style scoped>
/* Estilos para que se vea bonito */
.contenedor-principal {
  padding: 20px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background-color: #f0f2f5;
  min-height: 100vh;
}

.titulo {
  text-align: center;
  color: #2c3e50;
  margin-bottom: 30px;
}

.malla-grid {
  display: flex;
  gap: 15px;
  overflow-x: auto; /* Permite scroll horizontal si no caben */
  padding-bottom: 20px;
}

.columna-semestre {
  min-width: 200px; /* Ancho fijo para cada columna */
  background: #e9ecef;
  border-radius: 8px;
  padding: 10px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.encabezado-semestre {
  font-weight: bold;
  text-align: center;
  color: #495057;
  text-transform: uppercase;
  border-bottom: 2px solid #dee2e6;
  padding-bottom: 5px;
}

.tarjeta-materia {
  background: white;
  padding: 12px;
  border-radius: 6px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.05);
  font-size: 0.9rem;
  border-left: 5px solid #ced4da; /* Borde gris por defecto */
  transition: all 0.3s ease;
}

/* === ESTILO CUANDO ESTÁ APROBADA === */
.aprobada {
  background-color: #d4edda; /* Verde claro */
  border-left-color: #28a745; /* Borde verde fuerte */
  color: #155724;
  text-decoration: line-through; /* Tachado */
  opacity: 0.8;
}

.nombre-materia {
  font-weight: 500;
}

.icon-check {
  float: right;
  font-size: 1.2em;
}

.error {
  color: red;
  text-align: center;
  font-weight: bold;
  background: #ffcccc;
  padding: 20px;
  border-radius: 10px;
}
/* === ESTADO: DISPONIBLE (Lo que te toca cursar) === */
.tarjeta-materia {
  /* Por defecto será 'disponible' (borde azul) */
  border-left-color: #007bff; 
  background-color: white;
}

/* === ESTADO: APROBADO (Verde) === */
.aprobada {
  border-left-color: #28a745;
  background-color: #e8f5e9;
  color: #155724;
  text-decoration: line-through;
  opacity: 0.7;
}

/* === ESTADO: BLOQUEADO (Gris/Candado) === */
/* Necesitas agregar esta clase en el <template> primero */
.bloqueada {
  border-left-color: #6c757d; /* Gris */
  background-color: #e2e3e5;
  color: #6c757d;
  cursor: not-allowed;
}
</style>