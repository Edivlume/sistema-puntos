<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Sistema de Puntos - Teatro Musical</title>
  <link href="https://fonts.googleapis.com/css2?family=Montserrat:wght@400;700&display=swap" rel="stylesheet">
  <style>
    body { font-family: 'Montserrat', sans-serif; margin: 0; padding: 0; background: linear-gradient(135deg, #f5f7fa, #c3cfe2); }
    nav {
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      z-index: 1000; background-color: #2c2c2c; padding: 10px; text-align: center; }
    nav button { background-color: #f4f4f4; border: none; padding: 10px 20px; margin: 0 5px; cursor: pointer; font-weight: bold; border-radius: 5px; transition: background-color 0.3s ease; }
    nav button:hover { background-color: #ddd; }
    section { display: none; margin-top: 70px; padding: 20px; }
    section.active { display: block; background-color: white; margin: 20px; border-radius: 10px; box-shadow: 0 4px 8px rgba(0,0,0,0.1); }
    h1 { text-align: center; color: #333; }
    table { width: 100%; border-collapse: collapse; }
    th, td { padding: 10px; border: 1px solid #ccc; text-align: left; }
    th { background-color: #e0e0e0; }
    .alerta-verde { background-color: #C6EFCE; }
    .alerta-amarilla { background-color: #FFEB9C; }
    .alerta-roja { background-color: #F4CCCC; }
    label, select, input, textarea { display: block; margin-bottom: 10px; width: 100%; padding: 10px; border: 1px solid #ccc; border-radius: 5px; }
    button.submit { background-color: #28a745; color: white; border: none; padding: 10px; border-radius: 5px; cursor: pointer; font-weight: bold; width: 100%; }
    #passwordPrompt { display: none; padding: 20px; text-align: center; }
    #passwordPrompt input { padding: 10px; width: 250px; margin-bottom: 10px; }
  </style>
</head>
<body>
  <section id="puntajes" class="active">
    <h1>Resumen de Puntajes</h1>
    <div style="display: flex; flex-wrap: wrap; gap: 10px; justify-content: space-between; margin-bottom: 20px;">
      <div style="flex: 1; min-width: 200px;"><strong>Total de alumnos:</strong> <span id="totalAlumnos">0</span></div>
      <div style="flex: 1; min-width: 200px;"><strong>Promedio general:</strong> <span id="promedioGeneral">0</span></div>
      <div style="flex: 1; min-width: 200px;"><strong>Alumnos con alerta amarilla:</strong> <span id="amarillos">0</span></div>
      <div style="flex: 1; min-width: 200px;"><strong>Alumnos con alerta roja:</strong> <span id="rojos">0</span></div>
    </div>
    <table id="tablaPuntajes">
      <thead>
        <tr>
          <th>#</th>
          <th>Nombre del Alumno</th>
          <th>Puntaje Actual</th>
          <th>Rango de Alerta</th>
          <th>Puntos Restados</th>
          <th>Última Fecha</th>
          <th>Observaciones</th>
        </tr>
      </thead>
      <tbody></tbody>
    </table>
  </section>

  <section id="registro">
    <h1>Registro de Faltas</h1>
    <form onsubmit="registrarFalta(event)">
      <label for="fecha">Fecha:</label>
      <input type="date" id="fecha" required>
      <label for="alumno">Alumno:</label>
      <select id="alumno" required></select>
      <label for="falta">Tipo de falta:</label>
      <select id="falta" required>
        <option value="No saber trazo montado y grabado (reincidente)">No saber trazo montado y grabado (reincidente)</option>
        <option value="Retardo (tolerancia 15 min)">Retardo (tolerancia 15 min)</option>
        <option value="Falta no justificada">Falta no justificada</option>
        <option value="Falta sin aviso previo">Falta sin aviso previo</option>
      </select>
      <label for="observaciones">Observaciones:</label>
      <textarea id="observaciones"></textarea>
      <button class="submit" type="submit">Registrar</button>
    </form>
  </section>

  <section id="admin">
    <h1>Historial de Faltas</h1>
    <table id="tablaHistorial">
      <thead>
        <tr><th>Fecha</th><th>Alumno</th><th>Falta</th><th>Puntos</th><th>Observaciones</th><th>Acciones</th></tr>
      </thead>
      <tbody></tbody>
    </table>
  </section>

  <div id="passwordPrompt">
    <h3>Ingresa la contraseña de administrador</h3>
    <input type="password" id="adminPassword" placeholder="Contraseña">
    <br>
    <button onclick="validatePassword()">Acceder</button>
  </div>

  <nav>
    <button onclick="showTab('puntajes')">Resumen de Puntajes</button>
    <button onclick="requestAccess('registro')">Registro de Faltas</button>
    <button onclick="requestAccess('admin')">Historial (Admin)</button>
    <button onclick="logout()">Cerrar sesión</button>
  </nav>

  <script type="module">
    import { initializeApp } from 'https://www.gstatic.com/firebasejs/10.12.3/firebase-app.js';
import { getAuth, signInAnonymously } from 'https://www.gstatic.com/firebasejs/10.12.3/firebase-auth.js';
import { getFirestore, doc, getDoc, setDoc, onSnapshot } from 'https://www.gstatic.com/firebasejs/10.12.3/firebase-firestore.js';

    const alumnos = ["Areli Flores Salinas", "Emilia Cárdenas Navarro", "Jose Raymundo Rivas Colin", "Juan Carlos Nuñez", "Ana Belén Ortiz Villeda", "Renzo Luces", "Milton Fabricio Aguirre Duarte", "Triana Huerta Torres Landa", "Mariana Piñera Barreda", "Bárbara Sánchez", "Luis Pablo Perez Torrescano", "Ximena Carrero", "Fernando Roman Geronimo", "Danae Jasel Botello Lopez", "Gabriela Lozano Pedroza", "Karla Lizbeth Pérez Morales", "Zeltzin Citlali Feregrino Velázquez", "Edgar Iván Lugo Meza", "Diego Ramirez Torres", "Victoria Martinez"];
    const faltas = {
      "No saber trazo montado y grabado (reincidente)": 3,
      "Retardo (tolerancia 15 min)": 1,
      "Falta no justificada": 2,
      "Falta sin aviso previo": 3
    };

// === Firebase (integración) ===

// 1) Configuración (la pegaremos en el siguiente paso)
const firebaseConfig = {
  apiKey: "AIzaSyDNXoUT2QDtrHJNbrMAQMIPZ5IIaAKjdsw",
  authDomain: "sistema-de-puntos-ba2f9.firebaseapp.com",
  projectId: "sistema-de-puntos-ba2f9",
  storageBucket: "sistema-de-puntos-ba2f9.firebasestorage.app",
  messagingSenderId: "267134069783",
  appId: "1:267134069783:web:f95231a04e6189820fc052"
};

// 2) Arranque + auth anónima
const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
await signInAnonymously(auth);

// 3) Firestore + refs
const db = getFirestore(app);
const GRUPO_ID = 'default'; // luego podemos usar ?g=... en la URL
const puntajesRef  = doc(db, 'grupos', GRUPO_ID, 'datos', 'puntajes');
const historialRef = doc(db, 'grupos', GRUPO_ID, 'datos', 'historial');

// 4) Estado en memoria + defaults (se mantiene igual que antes)
const defaultPuntajes = alumnos.reduce((acc, nombre) => (acc[nombre] = 10, acc), {});
let puntajes  = JSON.parse(localStorage.getItem('puntajes'))  || { ...defaultPuntajes };
let historial = JSON.parse(localStorage.getItem('historial')) || [];

// Sincroniza con lista de alumnos (por si cambian)
let changed = false;
alumnos.forEach(nombre => { if (!(nombre in puntajes)) { puntajes[nombre] = 10; changed = true; } });
Object.keys(puntajes).forEach(nombre => { if (!alumnos.includes(nombre)) { delete puntajes[nombre]; changed = true; } });

// 5) Carga inicial desde Firestore (si no existe, siembra con local)
{
  const [pSnap, hSnap] = await Promise.all([getDoc(puntajesRef), getDoc(historialRef)]);

  if (pSnap.exists()) puntajes  = pSnap.data();
  else await setDoc(puntajesRef, puntajes);

  if (hSnap.exists()) historial = hSnap.data().items || [];
  else await setDoc(historialRef, { items: historial });

  // cache local y primer render
  localStorage.setItem('puntajes', JSON.stringify(puntajes));
  localStorage.setItem('historial', JSON.stringify(historial));
  renderPuntajes();
}

// 6) Escucha en tiempo real
onSnapshot(puntajesRef, (snap) => {
  if (!snap.exists()) return;
  puntajes = snap.data();
  localStorage.setItem('puntajes', JSON.stringify(puntajes));
  renderPuntajes();
});

onSnapshot(historialRef, (snap) => {
  if (!snap.exists()) return;
  historial = snap.data().items || [];
  localStorage.setItem('historial', JSON.stringify(historial));
  if (typeof renderHistorial === 'function') renderHistorial();
});

// 7) Funciones para guardar (las usaremos en tus handlers)
async function guardarPuntajes() {
  await setDoc(puntajesRef, puntajes, { merge: true });
}
async function pushHistorial(entry) {
  historial.push(entry);
  await setDoc(historialRef, { items: historial }, { merge: true });
}

    let protectedTab = null;

    function showTab(tabId) {
      document.querySelectorAll("section").forEach(s => s.classList.remove("active"));
      document.getElementById("passwordPrompt").style.display = "none";
      document.getElementById(tabId).classList.add("active");
      if (tabId === 'puntajes') renderPuntajes();
      if (tabId === 'admin') renderHistorial();
    }

    function requestAccess(tab) {
      if (isAdmin) {
        showTab(tab);
      } else {
        protectedTab = tab;
        document.getElementById("passwordPrompt").style.display = "block";
      }
    }

    function validatePassword() {
      const input = document.getElementById("adminPassword").value;
      if (input === "admin123") {
        isAdmin = true;
        document.getElementById("adminPassword").value = "";
        showTab(protectedTab);
      } else {
        alert("Contraseña incorrecta");
      }
    }

    function logout() {
      isAdmin = false;
      protectedTab = null;
      showTab('puntajes');
      alert("Sesión cerrada.");
    }

    function registrarFalta(event) {
  event.preventDefault();

  const alumno = document.getElementById("alumno").value;
  const falta = document.getElementById("falta").value;
  const observaciones = document.getElementById("observaciones").value;
  const puntos = faltas[falta];
  const fechaInput = document.getElementById("fecha").value;
  const fecha = fechaInput ? new Date(fechaInput).toLocaleDateString() : new Date().toLocaleDateString();

  // actualiza en memoria
  puntajes[alumno] -= puntos;
  // guarda en nube
  guardarPuntajes();
  pushHistorial({ fecha, alumno, falta, puntos: -puntos, observaciones });

  document.querySelector("form").reset();
  alert("Falta registrada con éxito");
}

    function renderPuntajes() {
      const tbody = document.querySelector("#tablaPuntajes tbody");
      const totalSpan = document.getElementById("totalAlumnos");
      const promedioSpan = document.getElementById("promedioGeneral");
      const amarillosSpan = document.getElementById("amarillos");
      const rojosSpan = document.getElementById("rojos");

      let total = alumnos.length;
      let suma = 0;
      let amarillos = 0;
      let rojos = 0;

      tbody.innerHTML = "";
      alumnos.forEach(alumno => {
        const puntos = puntajes[alumno];
        suma += puntos;
        if (puntos <= 7 && puntos > 4) amarillos++;
        if (puntos <= 4) rojos++;
        const registrosAlumno = historial.filter(h => h.alumno === alumno);
        const ultimaFalta = registrosAlumno.slice(-1)[0] || {};
        const puntosRestados = registrosAlumno.reduce((acc, r) => acc + Math.abs(r.puntos), 0);
        const observacion = ultimaFalta.observaciones || '';
        const ultimaFecha = ultimaFalta.fecha || '';
        const row = document.createElement("tr");
        row.className = puntos > 7 ? "alerta-verde" : puntos > 4 ? "alerta-amarilla" : "alerta-roja";
        const estadoTexto = puntos > 7 ? '✔️ Bien' : puntos > 4 ? '⚠️ Alerta Amarilla' : '❌ Alerta Roja';
        row.innerHTML = `<td>${alumnos.indexOf(alumno)+1}</td><td>${alumno}</td><td>${puntos}</td><td>${estadoTexto}</td><td>${puntosRestados}</td><td>${ultimaFecha}</td><td>${observacion}</td>`;
        tbody.appendChild(row);
      });

      totalSpan.textContent = total;
      promedioSpan.textContent = (suma / total).toFixed(2);
      amarillosSpan.textContent = amarillos;
      rojosSpan.textContent = rojos;
    }

    function renderHistorial() {
      const tbody = document.querySelector("#tablaHistorial tbody");
      tbody.innerHTML = "";
      historial.forEach(reg => {
        const row = document.createElement("tr");
        const index = historial.indexOf(reg);
        const editarBtn = `<button onclick=\"editarRegistro(${index})\">✏️</button>`;
        const eliminarBtn = `<button onclick=\"eliminarRegistro(${index})\">🗑️</button>`;
        row.innerHTML = `<td>${reg.fecha}</td><td>${reg.alumno}</td><td>${reg.falta}</td><td>${reg.puntos}</td><td>${reg.observaciones}</td><td>${editarBtn} ${eliminarBtn}</td>`;
        tbody.appendChild(row);
      });
    }

    const selectAlumno = document.getElementById("alumno");
    alumnos.forEach(nombre => {
      const opt = document.createElement("option");
      opt.value = nombre;
      opt.textContent = nombre;
      selectAlumno.appendChild(opt);
    });

function editarRegistro(index) {
  const reg = historial[index];

  // repone valores en el formulario
  const iso = new Date(reg.fecha).toISOString().split('T')[0];
  document.getElementById("fecha").value = iso;
  document.getElementById("alumno").value = reg.alumno;
  document.getElementById("falta").value = reg.falta;
  document.getElementById("observaciones").value = reg.observaciones;

  // revertir efecto del registro en memoria
  puntajes[reg.alumno] += Math.abs(reg.puntos);
  historial.splice(index, 1);

  // guardar en nube
  guardarPuntajes();
  setDoc(historialRef, { items: historial }, { merge: true });

  renderHistorial();
  renderPuntajes();
}

function eliminarRegistro(index) {
  if (!confirm("¿Estás seguro de que deseas eliminar este registro?")) return;

  const reg = historial[index];
  // revertir puntaje en memoria
  puntajes[reg.alumno] += Math.abs(reg.puntos);
  historial.splice(index, 1);

  // guardar en nube
  guardarPuntajes();
  setDoc(historialRef, { items: historial }, { merge: true });

  renderHistorial();
  renderPuntajes();
}

</script>
</body>
</html>
