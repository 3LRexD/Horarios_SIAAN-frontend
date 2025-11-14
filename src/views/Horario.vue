<template>
  <div class="horario-container">
    <header>
      <h1>Malla Curricular Ingeniería de Sistemas</h1>
    </header>

    <main>
      <div id="board" class="board">
        <div 
          v-for="semester in semesters" 
          :key="semester.number"
          class="semester"
          :style="{ background: semester.color }"
        >
          <div class="semester-header">
            Semestre {{ semester.number }}
          </div>
          
          <div class="courses">
            <div 
              v-if="semester.courses.length === 0"
              class="empty-state"
            >
              Sin materias
            </div>
            
            <div
              v-for="course in semester.courses"
              :key="course.code"
              class="course"
              :class="{ blocked: semester.number > CURRENT_SEMESTER }"
              @click="handleCourseClick(course, semester.number)"
            >
              <div class="course-name">{{ course.name }}</div>
              <div class="course-code">{{ course.code }}</div>
              <div class="course-prereq">Requisito: {{ course.prereq || 'Ninguno' }}</div>
            </div>
          </div>
        </div>
      </div>
    </main>

    <footer>
      Malla curricular académica
    </footer>
  </div>
</template>

<script>

const SEM_COUNT = 9;
const CURRENT_SEMESTER = 6;

const SEMESTER_COLORS = [
  "#E0BBE4",
  "#FFDFD3",
  "#D4F1F4",
  "#FFF4C2",
  "#C9E4CA",
  "#FFDEE9",
  "#D5AAFF",
  "#FFE5B4",
  "#B4E7FF"
];

const data = {
  semesters: {
    1: [
      { name: "Algebra Lineal", code: "MAT-123", prereq: "Ninguno" },
      { name: "Matemáticas Discretas", code: "MAT-122", prereq: "Ninguno" },
      { name: "Introducción a la Programación", code: "SIS-111", prereq: "Ninguno" },
      { name: "Ingeniería de Sistemas", code: "SIS-123", prereq: "Ninguno" },
      { name: "Pensamiento Crítico", code: "FIL-107", prereq: "Ninguno" }
    ],
    2: [
      { name: "Cálculo I", code: "MAT-132", prereq: "MAT-030" },
      { name: "Investigación Operativa I", code: "MAT-251", prereq: "MAT-123" },
      { name: "Probabilidad y Estadística I", code: "MAT-142", prereq: "MAT-123" },
      { name: "Programación I", code: "SIS-112", prereq: "SIS-111" },
      { name: "Arquitectura Computacional y Sistemas Operativos", code: "SIS-133", prereq: "MAT-122 / SIS-123" },
      { name: "Formación Humano Cristiana I", code: "FHC-102", prereq: "Ninguno" }
    ],
    3: [
      { name: "Cálculo II", code: "MAT-133", prereq: "MAT-132" },
      { name: "Física I y Laboratorio", code: "FIS-111", prereq: "MAT-132" },
      { name: "Probabilidad y Estadística II", code: "MAT-143", prereq: "MAT-142" },
      { name: "Programación II", code: "SIS-113", prereq: "SIS-112" },
      { name: "Bases de Datos I", code: "SIS-122", prereq: "SIS-112" },
      { name: "Metodología de la Investigación", code: "IND-260", prereq: "Ninguno" },
      { name: "Epistemología", code: "FIL-207", prereq: "Ninguno" }
    ],
    4: [
      { name: "Investigación Operativa II y Laboratorio", code: "MAT-252", prereq: "MAT-251" },
      { name: "Estructuras de Datos", code: "SIS-211", prereq: "SIS-113" },
      { name: "Tecnologias Web I", code: "SIS-214", prereq: "SIS-113 / SIS-122" },
      { name: "Bases de Datos II", code: "SIS-221", prereq: "SIS-122" },
      { name: "Sistemas de Información", code: "SIS-225", prereq: "SIS-122" },
      { name: "Formación Humano Cristiana II", code: "FHC-202", prereq: "FHC-102" }
    ],
    5: [
      { name: "Analisis de Algoritmos ", code: "MAT-361", prereq: "SIS-211" },
      { name: "Programación de Dispositivos Móviles", code: "SIS-212", prereq: "SIS-113 / SIS-122" },
      { name: "Tecnologias Web II", code: "SIS-215", prereq: "SIS-214 / SIS-221" },
      { name: "Ingenieria de Datos", code: "SIS-251", prereq: "MAT-133 / MAT-123" },
      { name: "Sistemas de Soporte a las Decisiones Gerenciales", code: "SIS-226", prereq: "SIS-225 / SIS-221" },
      { name: "Redes de Computadoras I", code: "SIS-231", prereq: "SIS-133" },
      { name: "LIBRE I", code: " ", prereq: "  " }
    ],
    6: [
      { name: "Ingenieria de Software ", code: "SIS-213", prereq: "SIS-215 / SIS-225" },
      { name: "Modelado, Dinamica de Sistemas y Simulación", code: "SIS-252 / MAT-143", prereq: "SIS-113 / SIS-122" },
      { name: "Inteligencia Artificial", code: "SIS-216", prereq: "SIS-214 / SIS-221" },
      { name: "Derecho de las TIC", code: "SIS-251", prereq: "MAT-133 / MAT-123" },
      { name: "Taller de Sistemas de Información", code: "SIS-226", prereq: "SIS-225 / SIS-221" },
      { name: "Redes de Computadoras II", code: "SIS-231", prereq: "SIS-133" },
      { name: "LIBRE II", code: " ", prereq: "  " }
    ],
    7: [
      { name: "Arquitectura de Software", code: "SIS-311", prereq: "SIS-213" },
      { name: "Gestión de Calidad de Sistemas", code: "SIS-312", prereq: "SIS-252" },
      { name: "Machine Learning", code: "SIS-312", prereq: "SIS-252" },
      { name: "Ética y Seguridad de Sistemas", code: "SIS-325", prereq: "SIS-323" },
      { name: "Computación en la Nube", code: "SIS-331", prereq: "SIS-233" },
      { name: "Internet de las Cosas", code: "SIS-234", prereq: "SIS-233" },
      { name: "Formación Humano Cristiana III", code: "FHC-202", prereq: "FHC-202" }
    ],
    8: [
      { name: "Gerencia de las TIC", code: "SIS-313", prereq: "SIS-226" },
      { name: "Taller de Sistemas Inteligentes", code: "SIS-352", prereq: "SIS-351" },
      { name: "Auditoria de Sistemas", code: "SIS-324", prereq: "SIS-312" },
      { name: "Práctica Preprofecional", code: "SIS-361", prereq: "SIS-312 / SIS-331" },
      { name: "Taller de Grado I", code: "SIS-362", prereq: "SIS-312 / SIS-331" }
    ],
    9: [
      { name: "Taller de Grado II", code: "SIS-363", prereq: "SIS-362" }
    ]
  }
};

export default {
  name: 'Horario',
  data() {
    return {
      SEM_COUNT,
      CURRENT_SEMESTER,
      semesters: []
    }
  },
  mounted() {
    this.initializeSemesters();
  },
  methods: {
    initializeSemesters() {

      this.semesters = Array.from({ length: SEM_COUNT }, (_, index) => {
        const semesterNumber = index + 1;
        return {
          number: semesterNumber,
          color: SEMESTER_COLORS[index],
          courses: data.semesters[semesterNumber] || []
        };
      });
    },
    
    handleCourseClick(course, semesterNumber) {
      if (semesterNumber > this.CURRENT_SEMESTER) {

        console.log(`Curso ${course.name} está bloqueado para el semestre ${semesterNumber}`);

        return;
      }
      
      console.log(`Curso seleccionado: ${course.name} (${course.code})`);

    }
  }
}
</script>

<style scoped>

:root {
  --gap: 12px;
  --radius: 8px;
}

* {
  box-sizing: border-box;
}

body {
  margin: 0;
  padding: 20px;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Inter, sans-serif;
  background: linear-gradient(180deg, #f8f9fa, #e9ecef);
  color: #2d3748;
  min-height: 100vh;
}

.horario-container {
  min-height: 100vh;
  padding: 20px;
}

header {
  text-align: center;
  margin-bottom: 30px;
}

h1 {
  font-size: 28px;
  margin: 0;
  color: #4a5568;
  font-weight: 600;
}

.board {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: var(--gap);
  max-width: 1600px;
  margin: 0 auto;
}

.semester {
  border-radius: 12px;
  padding: 16px;
  display: flex;
  flex-direction: column;
  gap: 10px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.semester:hover {
  transform: translateY(-4px);
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.12);
}

.semester-header {
  font-weight: 700;
  text-align: center;
  font-size: 15px;
  background: rgba(255, 255, 255, 0.5);
  padding: 10px;
  border-radius: 8px;
  color: #2d3748;
}

.courses {
  display: flex;
  flex-direction: column;
  gap: 8px;
  min-height: 50px;
}

.course {
  background: rgba(255, 255, 255, 0.7);
  border-radius: var(--radius);
  padding: 12px;
  text-align: center;
  transition: all 0.2s ease;
  border: 2px solid rgba(255, 255, 255, 0.8);
  cursor: pointer;
}

.course:hover {
  transform: scale(1.03);
  background: rgba(255, 255, 255, 0.9);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.course.blocked {
  cursor: not-allowed;
  opacity: 0.7;
}

.course.blocked:hover {
  background: #ffcdd2;
  border-color: #ef5350;
  box-shadow: 0 2px 8px rgba(239, 83, 80, 0.3);
}

.course-name {
  font-weight: 600;
  font-size: 14px;
  margin-bottom: 4px;
  color: #2d3748;
}

.course-code {
  font-size: 12px;
  color: #718096;
  margin-bottom: 6px;
}

.course-prereq {
  font-size: 11px;
  color: #a0aec0;
  font-style: italic;
  margin-top: 4px;
  padding-top: 6px;
  border-top: 1px solid rgba(0, 0, 0, 0.1);
}

.empty-state {
  text-align: center;
  padding: 20px;
  color: #a0aec0;
  font-size: 13px;
  font-style: italic;
}

footer {
  margin-top: 30px;
  padding-top: 20px;
  border-top: 1px solid rgba(0, 0, 0, 0.1);
  color: #718096;
  font-size: 13px;
  text-align: center;
}

@media (min-width: 1400px) {
  .board {
    grid-template-columns: repeat(9, 1fr);
  }
}

@media (max-width: 600px) {
  h1 {
    font-size: 22px;
  }
  .board {
    gap: 10px;
  }
}
</style>