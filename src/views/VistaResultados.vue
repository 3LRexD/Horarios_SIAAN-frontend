<template>
  <div class="modal-overlay">
    <div class="modal-content">
      
      <div class="modal-header">
        <div class="titulo-con-icono">
            <i class="fa-solid fa-calendar-days"></i>
            <h2>Horarios Generados ({{ opciones.length }} opciones)</h2>
        </div>
        <button class="btn-volver" @click="emitirCerrar">
            <i class="fa-solid fa-arrow-left"></i> Volver a Selección
        </button>
      </div>

      <div class="modal-body-scroll">
        
        <div v-for="opcion in opciones" :key="opcion.idOpcion" class="opcion-card">
          <div class="opcion-header">
            <h3>Opción {{ opcion.idOpcion }}</h3>
            <div class="opcion-actions">
                <button class="btn-mini-excel" @click="emitirDescargaIndividual(opcion)" title="Descargar Excel esta opción">
                    <i class="fa-solid fa-file-excel"></i>
                </button>
                <button class="btn-mini-pdf" @click="emitirDescargaPdf(opcion)" title="Descargar PDF">
                    <i class="fa-solid fa-file-pdf"></i>
                </button>
            </div>
          </div>

          <div class="tabla-container">
            <table class="tabla-horario">
              <thead>
                <tr>
                  <th class="col-hora">Hora</th>
                  <th>Lun</th>
                  <th>Mar</th>
                  <th>Mié</th>
                  <th>Jue</th>
                  <th>Vie</th>
                  <th>Sáb</th> </tr>
              </thead>
              <tbody>
                <tr v-for="(bloque, index) in horariosFijos" :key="index">
                    <td class="hora-celda">{{ bloque.label }}</td>
                    
                    <td :style="estiloCelda(opcion.clases, 'LUNES', bloque)">
                        {{ textoCelda(opcion.clases, 'LUNES', bloque) }}
                    </td>
                    <td :style="estiloCelda(opcion.clases, 'MARTES', bloque)">
                        {{ textoCelda(opcion.clases, 'MARTES', bloque) }}
                    </td>
                    <td :style="estiloCelda(opcion.clases, 'MIERCOLES', bloque)">
                        {{ textoCelda(opcion.clases, 'MIERCOLES', bloque) }}
                    </td>
                    <td :style="estiloCelda(opcion.clases, 'JUEVES', bloque)">
                        {{ textoCelda(opcion.clases, 'JUEVES', bloque) }}
                    </td>
                    <td :style="estiloCelda(opcion.clases, 'VIERNES', bloque)">
                        {{ textoCelda(opcion.clases, 'VIERNES', bloque) }}
                    </td>
                     <td :style="estiloCelda(opcion.clases, 'SABADO', bloque)">
                        {{ textoCelda(opcion.clases, 'SABADO', bloque) }}
                    </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>

      </div>

      <div class="modal-footer">
        <span>¿Te gustan estos resultados?</span>
        <button class="btn-descargar-todo-excel" @click="emitirDescargaExcel">
          <i class="fa-solid fa-file-excel"></i> Descargar Todo (Excel)
        </button>
      </div>

    </div>
  </div>
</template>


<script setup>
import { defineProps, defineEmits } from 'vue';

const props = defineProps({
  opciones: { type: Array, required: true }
});

//const emit = defineEmits(['cerrar', 'descargarExcel']);
//const emit = defineEmits(['cerrar', 'descargarExcel', 'descargarIndividual']);
const emit = defineEmits(['cerrar', 'descargarExcel', 'descargarIndividual', 'descargarPdf']);
const emitirCerrar = () => emit('cerrar');
const emitirDescargaExcel = () => emit('descargarExcel');

// Estos son los intervalos exactos que usa tu universidad
const horariosFijos = [
    { label: "07:15 - 08:00", inicio: "07:15", fin: "08:00" },
    { label: "08:00 - 08:45", inicio: "08:00", fin: "08:45" },
    { label: "09:00 - 09:45", inicio: "09:00", fin: "09:45" },
    { label: "09:45 - 10:30", inicio: "09:45", fin: "10:30" },
    { label: "10:45 - 11:30", inicio: "10:45", fin: "11:30" },
    { label: "11:30 - 12:15", inicio: "11:30", fin: "12:15" },
    { label: "12:30 - 13:15", inicio: "12:30", fin: "13:15" },
    { label: "13:15 - 14:00", inicio: "13:15", fin: "14:00" },
    { label: "14:15 - 15:00", inicio: "14:15", fin: "15:00" },
    { label: "15:00 - 15:45", inicio: "15:00", fin: "15:45" },
    { label: "16:00 - 16:45", inicio: "16:00", fin: "16:45" },
    { label: "16:45 - 17:30", inicio: "16:45", fin: "17:30" },
    { label: "17:45 - 18:30", inicio: "17:45", fin: "18:30" },
    { label: "18:30 - 19:15", inicio: "18:30", fin: "19:15" },
    { label: "19:30 - 20:15", inicio: "19:30", fin: "20:15" },
    { label: "20:15 - 21:00", inicio: "20:15", fin: "21:00" }
];


// FUNCIÓN PARA BUSCAR SI HAY CLASE EN ESE BLOQUE ---
const buscarClase = (listaClases, diaColumna, bloque) => {
    if (!listaClases) return null;

    // Normalizar día (quitar tildes y mayúsculas)
    const diaBuscado = diaColumna.toUpperCase().replace('É', 'E').replace('Á', 'A');

    return listaClases.find(clase => {
        if (!clase.dia) return false;
        const diaClase = clase.dia.toUpperCase().replace('É', 'E').replace('Á', 'A');
        
        //Coincidir el día
        if (diaClase !== diaBuscado) return false;

        // Coincidir la hora
        // Verificamos si el bloque actual está "dentro" del horario de la clase.
        // Convertimos strings "07:15" a comparables si es necesario, pero 
        // string comparison suele funcionar bien con formato HH:MM militar.
        return (bloque.inicio >= clase.horaInicio && bloque.inicio < clase.horaFin);
    });
};

// --- HELPERS VISUALES ---
const estiloCelda = (listaClases, dia, bloque) => {
    const claseEncontrada = buscarClase(listaClases, dia, bloque);
    
    if (claseEncontrada) {
        return {
            backgroundColor: claseEncontrada.color,
            color: '#000', 
            fontWeight: 'bold',
            fontSize: '10px', 
            border: '1px solid #ccc',
            textAlign: 'center',
            verticalAlign: 'middle',
            padding: '2px'
        };
    }
    return {}; 
};

const textoCelda = (listaClases, dia, bloque) => {
    const claseEncontrada = buscarClase(listaClases, dia, bloque);
    if (claseEncontrada) {
        return `${claseEncontrada.nombreMateria}`;
    }
    return '';
};

const emitirDescargaIndividual = (opcion) => {
    // Truco: Una opción tiene muchas "Clases" (DTOs).
    // Muchas clases pueden pertenecer al mismo Paralelo (ej: Lun y Mie).
    // Usamos 'Set' para sacar los IDs únicos de los paralelos de esta opción.
    
    const idsUnicos = [...new Set(opcion.clases.map(c => c.idParalelo))];
    
    // Emitimos la lista de IDs al padre
    emit('descargarIndividual', idsUnicos);
};

const emitirDescargaPdf = (opcion) => {
    const idsUnicos = [...new Set(opcion.clases.map(c => c.idParalelo))];
    emit('descargarPdf', idsUnicos);
};

</script>

<style scoped>

.modal-overlay {
  position: fixed; top: 0; left: 0; width: 100%; height: 100%;
  background: rgba(0, 0, 0, 0.85);
  display: flex; justify-content: center; align-items: center; z-index: 9999;
}

.modal-content {
  background: #f0f2f5; width: 95%; height: 95%; 
  border-radius: 12px; display: flex; flex-direction: column;
  overflow: hidden; box-shadow: 0 10px 25px rgba(0,0,0,0.5);
}

.modal-header {
  background: #003366; color: white; padding: 15px 25px;
  display: flex; justify-content: space-between; align-items: center;
}
.titulo-con-icono { display: flex; gap: 10px; align-items: center; }

.btn-volver {
  background: transparent; border: 1px solid white; color: white;
  padding: 8px 15px; border-radius: 5px; cursor: pointer; transition: 0.2s;
}
.btn-volver:hover { background: rgba(255,255,255,0.2); }

.modal-body-scroll {
  flex: 1; padding: 20px; overflow-x: auto; overflow-y: hidden;
  display: flex; gap: 20px; align-items: flex-start; 
}

.opcion-card {
  min-width: 550px;
  max-width: 600px;
  background: white; border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0,0,0,0.1);
  display: flex; flex-direction: column;
  height: 100%; 
}

.opcion-header {
  padding: 10px 15px; border-bottom: 1px solid #eee;
  display: flex; justify-content: space-between; align-items: center;
  background: #fafafa; border-radius: 8px 8px 0 0;
}

.btn-mini-excel { 
border: none; padding: 5px 10px; 
border-radius: 4px; cursor: pointer; 
color: white; background: #28a745; 
}

.btn-mini-pdf { 
background: #dc3545; 
color: white; 
cursor: pointer; 
}

.btn-mini-pdf:hover { 
background: #c82333; 
}

.tabla-container {
  padding: 5px; overflow-y: auto; flex: 1;
}

.tabla-horario { width: 100%; border-collapse: collapse; font-size: 11px; table-layout: fixed; }
.tabla-horario th { background: #003366; color: white; padding: 4px; position: sticky; top: 0; z-index: 1;}
.col-hora { width: 80px; } 

.tabla-horario td { border: 1px solid #ddd; padding: 2px; text-align: center; height: 30px; overflow: hidden; white-space: nowrap; text-overflow: ellipsis;}
.hora-celda { background: #eee; font-weight: bold; font-size: 10px; color: #333;}

.modal-footer {
  padding: 15px; background: white; border-top: 1px solid #ddd;
  display: flex; justify-content: flex-end; align-items: center; gap: 15px;
}
.btn-descargar-todo-excel {
  background: #217346; color: white; border: none; padding: 10px 20px;
  border-radius: 6px; font-weight: bold; cursor: pointer; font-size: 14px;
}
</style>