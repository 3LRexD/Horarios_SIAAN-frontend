<template>
  <div class="contenedor-malla-light">
    
    <div class="header-light">
      <div>
        <h2><i class="fa-solid fa-graduation-cap"></i> Malla Curricular Interactiva</h2>
        <p class="subtitulo">Selecciona las asignaturas para generar tu horario</p>
      </div>
      
      <div class="leyenda-light">
        <div class="badge-leyenda aprobado"><i class="fa-solid fa-check"></i> Aprobado</div>
        <div class="badge-leyenda disponible"><i class="fa-solid fa-circle"></i> Disponible</div>
        <div class="badge-leyenda seleccionado"><i class="fa-solid fa-star"></i> Seleccionado</div>
        <div class="badge-leyenda bloqueado"><i class="fa-solid fa-lock"></i> Bloqueado</div>
      </div>
    </div>

    <div class="contenedor-principal">
      <div v-if="cargando" class="loading-container">
        <div class="spinner-blue"></div>
        <p>Consultando historial académico...</p>
      </div>

      <div v-else class="malla-scroll-light">
        <div class="malla-grid">
          
          <div v-if="materiasIngles.length > 0" class="columna-light columna-ingles">
            <div class="header-semestre header-ingles">
              <i class="fa-solid fa-language"></i> IDIOMAS
            </div>
            
            <div class="nota-ingles">
              Requisito<br>Transversal
            </div>
            
            <div class="lista-tarjetas">
              <div v-for="materia in materiasIngles" :key="materia.id" 
                   class="card-materia"
                   :class="obtenerClaseEstado(materia)"
                   @click="toggleSeleccion(materia)"
                   :title="materia.nombre">
                   
                <div class="card-top">
                  <span class="sigla-tag sigla-ingles">{{ materia.sigla }}</span>
                  <i v-if="materia.aprobado" class="fa-solid fa-check-circle icon-status check"></i>
                  <i v-else-if="estaSeleccionada(materia.id)" class="fa-solid fa-star icon-status star"></i>
                  <i v-else-if="esMateriaBloqueada(materia)" class="fa-solid fa-lock icon-status lock"></i>
                  <div v-else class="circle-placeholder"></div>
                </div>
                
                <div class="card-body">
                  {{ materia.nombre }}
                </div>
              </div>
            </div>
          </div>
          
          <div v-for="(materias, semestre) in materiasPorSemestre" :key="semestre" class="columna-light">
            <div class="header-semestre">SEMESTRE {{ semestre }}</div>
            
            <div class="lista-tarjetas">
              <div v-for="materia in materias" :key="materia.id"
                   class="card-materia"
                   :class="obtenerClaseEstado(materia)"
                   @click="toggleSeleccion(materia)"
                   :title="materia.nombre">
                   
                <div class="card-top">
                  <span class="sigla-tag">{{ materia.sigla }}</span>
                  <i v-if="materia.aprobado" class="fa-solid fa-check-circle icon-status check"></i>
                  <i v-else-if="estaSeleccionada(materia.id)" class="fa-solid fa-star icon-status star"></i>
                  <i v-else-if="esMateriaBloqueada(materia)" class="fa-solid fa-lock icon-status lock"></i>
                  <div v-else class="circle-placeholder"></div>
                </div>
                
                <div class="card-body">
                  {{ materia.nombre }}
                </div>

              </div>
            </div>
          </div>

        </div>
      </div>

      <div class="panel-derecho" :class="{ 'visible': materiasSeleccionadas.length > 0 }">
        <div class="panel-contenido">
          <div class="panel-header">
            <i class="fa-solid fa-clipboard-list"></i>
            <h3>Selección</h3>
          </div>
          
          <div class="contador-materias">
            <span class="numero">{{ materiasSeleccionadas.length }}</span>
            <span class="texto">materias</span>
          </div>

          <button class="btn-generar" @click="manejarGeneracion" :disabled="cargandoGeneracion || materiasSeleccionadas.length === 0">
              <span v-if="cargandoGeneracion">
                  <i class="fa-solid fa-spinner fa-spin"></i> Calculando...
              </span>
              <span v-else>
                  <i class="fa-solid fa-file-excel"></i> Generar Horarios
              </span>
          </button>
        </div>
      </div>
    </div>

 

<VistaResultados 
    v-if="mostrarResultados"
    :opciones="opcionesGeneradas"
    @cerrar="mostrarResultados = false"
    @descargarExcel="descargarExcelGlobal"
    @descargarIndividual="descargarExcelIndividual"
    @descargarPdf="descargarPdfIndividual"
/>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue';
import axios from 'axios';
import VistaResultados from './VistaResultados.vue';


// --- ESTADO ---
const listaMaterias = ref([]);
const materiasSeleccionadas = ref([]);
const cargando = ref(true);
const ID_ESTUDIANTE = 1;

const mostrarResultados = ref(false);
const opcionesGeneradas = ref([]);
const cargandoGeneracion = ref(false);

// --- CARGA DE DATOS ---
onMounted(async () => {
  try {
    const res = await axios.get(`http://localhost:8080/api/academico/malla/${ID_ESTUDIANTE}`);
    
    listaMaterias.value = res.data.map(item => ({
      id: Number(item.idMateria),
      nombre: item.nombreMateria,
      sigla: item.sigla || '', // Asegurar que no sea null
      semestre: item.semestre,
      aprobado: String(item.estado) === 'true',
      requisitos: item.prerrequisitos ? String(item.prerrequisitos).split(',').map(Number) : []
    }));
    
  } catch (e) {
    console.error("Error cargando malla:", e);
  } finally {
    cargando.value = false;
  }
});

// --- LÓGICA DE AGRUPACIÓN (INGLÉS vs CARRERA) ---

// 1. Detectar Inglés
const esIngles = (m) => {
  // Criterio 1: Sigla empieza con IDM
  if (m.sigla && m.sigla.toUpperCase().startsWith('IDM')) return true;
  // Criterio 2: El nombre dice "Inglés" o "Idioma"
  const nombre = m.nombre.toLowerCase();
  return nombre.includes('inglés') || nombre.includes('ingles') || nombre.includes('idioma');
};

const materiasIngles = computed(() => {
  return listaMaterias.value
    .filter(m => esIngles(m))
    .sort((a, b) => a.nombre.localeCompare(b.nombre)); // Ordenar alfabéticamente
});

const materiasPorSemestre = computed(() => {
  const grupos = {};
  
  // Filtramos las que NO son inglés
  const carrera = listaMaterias.value.filter(m => !esIngles(m));
  
  carrera.forEach(m => {
    const sem = m.semestre || 0;
    if (!grupos[sem]) grupos[sem] = [];
    grupos[sem].push(m);
  });
  
  return grupos;
});

// --- FUNCIONES ---
const esMateriaBloqueada = (materia) => {
  if (materia.aprobado) return false;
  if (materia.requisitos.length === 0) return false;
  
  const requisitosCumplidos = materia.requisitos.every(reqId => {
    const matRequisito = listaMaterias.value.find(m => m.id === reqId);
    return matRequisito && matRequisito.aprobado;
  });

  return !requisitosCumplidos;
};

const obtenerClaseEstado = (materia) => {
  if (materia.aprobado) return 'status-aprobado';
  if (estaSeleccionada(materia.id)) return 'status-seleccionado';
  if (esMateriaBloqueada(materia)) return 'status-bloqueado';
  return 'status-disponible';
};

const estaSeleccionada = (id) => materiasSeleccionadas.value.includes(id);

const toggleSeleccion = (materia) => {
  if (materia.aprobado || esMateriaBloqueada(materia)) return;

  const idNum = Number(materia.id);
  
  // Buscamos el índice directamente en el valor actual
  const index = materiasSeleccionadas.value.indexOf(idNum);

  if (index > -1) {
    // DESELECCIONAR:
    // Quitamos solo ese elemento. Vue lo detecta y actualiza solo ese cuadrito.
    materiasSeleccionadas.value.splice(index, 1);
  } else {
    // SELECCIONAR:
    // Agregamos solo ese elemento.
    materiasSeleccionadas.value.push(idNum);
  }
  
  // NO hace falta la línea "materiasSeleccionadas.value = ...", Vue 3 ya es reactivo con push y splice.
};

const manejarGeneracion = async () => {
    if (materiasSeleccionadas.value.length === 0) return;

    cargandoGeneracion.value = true;
    try {
        // Pedimos el JSON para la vista previa
        const res = await axios.post('http://localhost:8080/api/generador/generar', materiasSeleccionadas.value);
        
        if (res.data.mensaje && !res.data.data) {
            alert(res.data.mensaje); // "No hay combinaciones"
        } else {
            opcionesGeneradas.value = res.data; // Guardamos los datos
            mostrarResultados.value = true;     // Abrimos el componente hijo
        }
    } catch (e) {
        console.error(e);
        alert("Error al conectar con el servidor");
    } finally {
        cargandoGeneracion.value = false;
    }
};

const descargarExcelGlobal = async () => {
    try {
        const response = await axios.post(
            'http://localhost:8080/api/generador/exportar-excel', 
            materiasSeleccionadas.value,
            { responseType: 'blob' }
        );
        const url = window.URL.createObjectURL(new Blob([response.data]));
        const link = document.createElement('a');
        link.href = url;
        link.setAttribute('download', 'mis_horarios.xlsx');
        document.body.appendChild(link);
        link.click();
    } catch (error) {
        alert("Error al descargar el archivo");
    }
};

const descargarHorario = async () => {
  if (materiasSeleccionadas.value.length === 0) {
    alert("⚠️ Selecciona al menos una materia");
    return;
  }
  try {
    const response = await axios.post(
      'http://localhost:8080/api/generador/exportar-excel', 
      materiasSeleccionadas.value, 
      { responseType: 'blob' }
    );
    const url = window.URL.createObjectURL(new Blob([response.data]));
    const link = document.createElement('a');
    link.href = url;
    link.setAttribute('download', 'Horario_SIAAN.xlsx');
    document.body.appendChild(link);
    link.click();
    setTimeout(() => {
      document.body.removeChild(link);
      window.URL.revokeObjectURL(url);
    }, 200);
  } catch (e) { 
    console.error("Error descarga:", e);
    alert("Error al generar Excel.");
  }
};

const descargarExcelIndividual = async (listaIdsParalelos) => {
    try {
        const response = await axios.post(
            'http://localhost:8080/api/generador/exportar-individual', 
            listaIdsParalelos, // Enviamos solo los IDs de esa opción
            { responseType: 'blob' }
        );
        
        const url = window.URL.createObjectURL(new Blob([response.data]));
        const link = document.createElement('a');
        link.href = url;
        link.setAttribute('download', 'horario_seleccionado.xlsx');
        document.body.appendChild(link);
        link.click();
    } catch (error) {
        console.error(error);
        alert("Error al descargar el horario individual.");
    }
};

// Función para descargar PDF Individual
const descargarPdfIndividual = async (listaIdsParalelos) => {
    try {
        const response = await axios.post(
            'http://localhost:8080/api/generador/exportar-pdf-individual', 
            listaIdsParalelos,
            { responseType: 'blob' } // Importante
        );
        
        const url = window.URL.createObjectURL(new Blob([response.data]));
        const link = document.createElement('a');
        link.href = url;
        link.setAttribute('download', 'mi_horario.pdf'); // Extensión .pdf
        document.body.appendChild(link);
        link.click();
    } catch (error) {
        console.error(error);
        alert("Error al descargar el PDF.");
    }
};

</script>

<style scoped>
* { box-sizing: border-box; }

.contenedor-malla-light {
  width: 100%;
  height: 100%; /* Ocupa el contenedor padre */
  background-color: #f3f4f6;
  display: flex;
  flex-direction: column;
  font-family: 'Segoe UI', sans-serif;
  overflow: hidden; /* Evita doble scroll */
}

/* HEADER */
.header-light {
  background: white;
  padding: 15px 25px;
  border-bottom: 2px solid #e5e7eb;
  display: flex;
  justify-content: space-between;
  align-items: center;
  box-shadow: 0 2px 4px rgba(0,0,0,0.05);
  flex-shrink: 0; /* No se encoge */
}

.header-light h2 { margin: 0; font-size: 20px; color: #002b55; display: flex; align-items: center; gap: 10px; }
.subtitulo { color: #6b7280; font-size: 13px; margin: 4px 0 0 0; }

.leyenda-light { display: flex; gap: 15px; }
.badge-leyenda { font-size: 12px; display: flex; align-items: center; gap: 6px; padding: 4px 10px; border-radius: 15px; font-weight: 600; }
.aprobado { color: #059669; background: #d1fae5; }
.disponible { color: #002b55; background: #dbeafe; }
.seleccionado { color: #d97706; background: #fef3c7; }
.bloqueado { color: #9ca3af; background: #f3f4f6; }

/* CONTENEDOR PRINCIPAL */
.contenedor-principal { flex: 1; display: flex; overflow: hidden; position: relative; }

.malla-scroll-light {
  flex: 1;
  overflow-x: auto;
  overflow-y: hidden;
  padding: 20px;
}

.malla-grid { display: flex; gap: 15px; height: 100%; }

/* COLUMNAS */
.columna-light {
  min-width: 140px;
  max-width: 140px;
  background: white;
  border-radius: 8px;
  display: flex;
  flex-direction: column;
  height: 100%;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.05);
  border: 1px solid #e5e7eb;
}

/* ESTILO ESPECIAL INGLÉS */
.columna-ingles {
  border: 2px solid #8b5cf6; /* Borde morado */
  background: #faf5ff;
}
.header-ingles { color: #7c3aed !important; background: #f3e8ff !important; }
.nota-ingles { font-size: 9px; color: #7c3aed; text-align: center; padding: 5px; background: #f3e8ff; border-bottom: 1px solid #e9d5ff; font-weight: bold; }
.sigla-ingles { background: #f3e8ff !important; color: #7c3aed !important; }

.header-semestre {
  text-align: center;
  font-size: 11px;
  font-weight: 800;
  color: #6b7280;
  padding: 10px 0;
  border-bottom: 1px solid #f3f4f6;
}

.lista-tarjetas {
  padding: 10px;
  display: flex;
  flex-direction: column;
  gap: 10px;
  overflow-y: auto;
  /* Scrollbar fina */
  scrollbar-width: thin;
  scrollbar-color: #cbd5e1 transparent;
}

/* TARJETAS */
.card-materia {
  background: white;
  border-radius: 6px;
  padding: 8px;
  min-height: 85px;
  border: 1px solid #e5e7eb;
  cursor: pointer;
  transition: all 0.2s;
  display: flex;
  flex-direction: column;
  box-shadow: 0 1px 2px rgba(0,0,0,0.05);
}
.card-materia:hover { transform: translateY(-3px); box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1); z-index: 5; }

.card-top { display: flex; justify-content: space-between; margin-bottom: 5px; }
.sigla-tag { font-size: 10px; font-weight: bold; color: #6b7280; background: #f3f4f6; padding: 2px 5px; border-radius: 4px; }
.icon-status { font-size: 14px; }
.circle-placeholder { width: 14px; height: 14px; border: 1px solid #d1d5db; border-radius: 50%; }

.card-body {
    /* ... lo que ya tienes ... */
    display: -webkit-box;
    -webkit-line-clamp: 3;
    line-clamp: 3;            /* <--- Agrega esta línea */
    -webkit-box-orient: vertical;
    overflow: hidden;
}

/* ESTADOS */
.status-aprobado { border-left: 4px solid #10b981; opacity: 0.6; background: #f9fafb; cursor: default; }
.status-aprobado .check { color: #10b981; }

.status-disponible { border-left: 4px solid #002b55; }
.status-disponible:hover { border-color: #2563eb; }

.status-seleccionado { border: 2px solid #f59e0b; background: #fffbeb; }
.status-seleccionado .star { color: #f59e0b; animation: pop 0.3s ease; }

.status-bloqueado { background: #f9fafb; border: 1px dashed #d1d5db; color: #9ca3af; cursor: not-allowed; }

/* PANEL DERECHO */
.panel-derecho {
  width: 0;
  background: white;
  border-left: 2px solid #e5e7eb;
  transition: width 0.3s ease;
  overflow: hidden;
  box-shadow: -4px 0 6px -1px rgba(0, 0, 0, 0.05);
}
.panel-derecho.visible { width: 260px; }

.panel-contenido { width: 260px; padding: 20px; display: flex; flex-direction: column; gap: 20px; }
.panel-header { display: flex; align-items: center; gap: 10px; color: #002b55; border-bottom: 2px solid #e5e7eb; padding-bottom: 10px; }
.panel-header h3 { margin: 0; font-size: 16px; }

.contador-materias {
  background: linear-gradient(135deg, #002b55 0%, #1e40af 100%);
  padding: 15px;
  border-radius: 12px;
  text-align: center;
  color: white;
}
.contador-materias .numero { display: block; font-size: 40px; font-weight: 800; color: #fbbf24; line-height: 1; margin-bottom: 5px; }
.contador-materias .texto { font-size: 12px; text-transform: uppercase; letter-spacing: 1px; opacity: 0.9; }

.btn-generar {
  background: #002b55;
  color: white;
  border: none;
  padding: 12px;
  border-radius: 8px;
  font-weight: 600;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  transition: all 0.2s;
}
.btn-generar:hover { background: #1e40af; transform: translateY(-2px); }

/* LOADING */
.loading-container { display: flex; flex-direction: column; align-items: center; justify-content: center; height: 100%; gap: 15px; color: #6b7280; flex: 1; }
.spinner-blue { width: 40px; height: 40px; border: 4px solid #e5e7eb; border-top-color: #002b55; border-radius: 50%; animation: spin 1s linear infinite; }

@keyframes spin { to { transform: rotate(360deg); } }
@keyframes pop { 0% { transform: scale(1); } 50% { transform: scale(1.4); } 100% { transform: scale(1); } }
</style>