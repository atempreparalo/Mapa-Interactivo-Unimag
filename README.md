# Mapa-arboles-Unimag
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Inventario de árboles — Universidad del Magdalena</title>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/leaflet.min.css">
<style>
  :root{
    --ink:#16241C;
    --canopy:#2F6B4F;
    --bark:#6B4F3A;
    --sage:#E4E9DE;
    --lime:#C6E36B;
    --paper:#F7F8F4;
    /* Colores oficiales de la Lista Roja de la UICN */
    --uicn-lc:#60C659;   /* Preocupación menor */
    --uicn-nt:#CCE226;   /* Casi amenazada */
    --uicn-vu:#F9E814;   /* Vulnerable */
    --uicn-en:#FC7F3F;   /* En peligro */
    --uicn-cr:#D81E05;   /* En peligro crítico */
    --uicn-dd:#D1D1C6;   /* Datos insuficientes */
  }
  *{box-sizing:border-box}
  html,body{height:100%;margin:0}
  body{
    font-family:"Helvetica Neue",Helvetica,Arial,sans-serif;
    color:var(--ink);
    background:var(--ink);
    display:grid;
    grid-template-columns:340px 1fr;
    height:100vh;
    overflow:hidden;
  }

  /* ---------- Panel lateral ---------- */
  aside{
    background:var(--ink);
    color:var(--sage);
    display:flex;
    flex-direction:column;
    min-height:0;
    border-right:1px solid #0d160f;
  }
  .titulo{padding:22px 22px 16px;border-bottom:1px solid #24382c}
  .titulo h1{
    margin:0 0 6px;
    font-family:Georgia,"Iowan Old Style","Times New Roman",serif;
    font-size:25px;
    font-weight:400;
    letter-spacing:.2px;
  }
  .titulo p{margin:0;font-size:13px;line-height:1.5;color:#9db19f}

  .buscador{padding:14px 22px 10px}
  .buscador input{
    width:100%;padding:9px 11px;border-radius:6px;
    border:1px solid #33503f;background:#0f1d15;color:var(--sage);
    font-size:14px;font-family:inherit;
  }
  .buscador input::placeholder{color:#6f846f}
  .buscador input:focus{outline:2px solid var(--lime);outline-offset:1px}

  .lista{flex:1;overflow-y:auto;padding:4px 12px 12px;min-height:0}
  .arbol{
    width:100%;text-align:left;cursor:pointer;
    background:transparent;border:0;border-bottom:1px solid #23372b;
    padding:12px 10px;color:inherit;font:inherit;
    display:grid;grid-template-columns:12px 1fr;gap:11px;align-items:start;
  }
  .arbol:hover,.arbol:focus-visible{background:#1e3125}
  .arbol:focus-visible{outline:2px solid var(--lime);outline-offset:-2px}
  .arbol[aria-current="true"]{background:#233a2b}
  .punto{width:11px;height:11px;border-radius:50%;margin-top:5px}
  .arbol .cientifico{
    font-family:Georgia,serif;font-style:italic;font-size:15px;display:block;
  }
  .arbol .comun{font-size:12.5px;color:#9db19f;display:block;margin-top:2px}

  .leyenda{padding:14px 22px 18px;border-top:1px solid #24382c;font-size:12.5px;color:#9db19f}
  .leyenda div{display:flex;align-items:center;gap:8px;margin-top:7px}
  .leyenda span.punto{margin:0;flex:none}
  .leyenda span.punto.claro{background:var(--uicn-vu);border:1px solid #6f6f5a}
  .leyenda span.punto.dd{background:var(--uicn-dd);border:1px solid #6f6f5a}
  .leyenda .pie{margin:10px 0 0;font-size:11.5px;line-height:1.5;color:#7e927f;display:block}
  .leyenda .pie b{color:#9db19f;font-weight:600}

  .acciones{padding:0 22px 20px;display:flex;gap:8px;flex-wrap:wrap}
  button.accion{
    font:inherit;font-size:13px;cursor:pointer;
    padding:8px 12px;border-radius:6px;
    border:1px solid #3d5c48;background:#1c2e22;color:var(--sage);
  }
  button.accion:hover{background:#264034}
  button.accion[aria-pressed="true"]{background:var(--lime);color:var(--ink);border-color:var(--lime)}

  /* ---------- Mapa ---------- */
  main{position:relative;min-width:0}
  #mapa{position:absolute;inset:0;background:#dfe3d8}
  .coords{
    position:absolute;left:0;bottom:0;z-index:500;
    background:rgba(22,36,28,.86);color:var(--sage);
    font-size:12px;padding:5px 10px;font-variant-numeric:tabular-nums;
  }
  .aviso{
    position:absolute;z-index:600;top:12px;left:50%;transform:translateX(-50%);
    background:var(--lime);color:var(--ink);padding:9px 15px;border-radius:20px;
    font-size:13.5px;box-shadow:0 2px 10px rgba(0,0,0,.25);display:none;
  }
  .aviso.visible{display:block}

  /* ---------- Popup ---------- */
  .leaflet-popup-content-wrap{border-radius:8px;padding:0;overflow:hidden}
  .leaflet-popup-content{margin:0;width:262px !important;line-height:1.45}
  .ficha img{display:block;width:100%;height:150px;object-fit:cover;background:var(--sage)}
  .ficha .cuerpo{padding:12px 14px 14px}
  .ficha h2{
    margin:0;font-family:Georgia,serif;font-style:italic;font-weight:400;font-size:18px;
  }
  .ficha .comun{margin:2px 0 10px;font-size:13px;color:#5e6d60}
  .ficha dl{margin:0;display:grid;grid-template-columns:auto 1fr;gap:4px 12px;font-size:13px}
  .ficha dt{color:#5e6d60}
  .ficha dd{margin:0;font-variant-numeric:tabular-nums}
  .estado{display:inline-flex;align-items:center;gap:6px}
  .ficha .nota{margin:10px 0 0;font-size:12.5px;color:#5e6d60;border-top:1px solid #e3e7de;padding-top:8px}

  .marcador{
    width:16px;height:16px;border-radius:50%;
    border:2.5px solid #fff;box-shadow:0 1px 4px rgba(0,0,0,.45);
  }
  /* VU, NT, DD y NE son colores muy claros: sin este borde interior
     desaparecen sobre el satelital o sobre fondos claros. */
  .marcador.claro{box-shadow:0 0 0 1px #4a4a3c inset, 0 1px 4px rgba(0,0,0,.45)}
  .marcador.activo{transform:scale(1.45);transition:transform .18s ease}

  @media (max-width:820px){
    body{grid-template-columns:1fr;grid-template-rows:44vh 56vh}
    aside{order:2;border-right:0;border-top:1px solid #0d160f}
    main{order:1}
    .titulo{padding:14px 18px 10px}
    .titulo h1{font-size:20px}
    .titulo p{display:none}
  }
  @media (prefers-reduced-motion:reduce){*{transition:none !important}}
</style>
</head>
<body>

<aside>
  <div class="titulo">
    <h1>Inventario de árboles</h1>
    <p>Campus de la Universidad del Magdalena, Santa Marta. Haz clic en un punto del mapa
       o en la lista para ver la ficha del árbol. Las coordenadas y las medidas son de
       ejemplo, para probar el funcionamiento.</p>
  </div>

  <div class="buscador">
    <input id="buscar" type="search" placeholder="Buscar por especie o nombre común" aria-label="Buscar árbol">
  </div>

  <div class="lista" id="lista"></div>

  <div class="leyenda">
    Estado de conservación (Lista Roja de la UICN)
    <div><span class="punto" style="background:var(--uicn-cr)"></span> En peligro crítico (CR)</div>
    <div><span class="punto" style="background:var(--uicn-en)"></span> En peligro (EN)</div>
    <div><span class="punto claro"></span> Vulnerable (VU)</div>
    <div><span class="punto" style="background:var(--uicn-nt)"></span> Casi amenazada (NT)</div>
    <div><span class="punto" style="background:var(--uicn-lc)"></span> Preocupación menor (LC)</div>
    <div><span class="punto dd"></span> Datos insuficientes (DD)</div>
    <p class="pie">La categoría es de la <b>especie</b>, no del ejemplar.
       Un árbol sano puede pertenecer a una especie amenazada.</p>
  </div>

  <div class="acciones">
    <button class="accion" id="btn-importar">Importar CSV</button>
    <button class="accion" id="btn-agregar" aria-pressed="false">Agregar árbol</button>
    <button class="accion" id="btn-exportar">Descargar CSV</button>
    <button class="accion" id="btn-ubicacion">Mi ubicación</button>
    <input type="file" id="archivo-csv" accept=".csv" hidden>
  </div>
</aside>

<main>
  <div id="mapa"></div>
  <div class="aviso" id="aviso" role="status"></div>
  <div class="coords" id="coords">Mueve el cursor sobre el mapa</div>
</main>

<script src="https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/leaflet.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.4.1/papaparse.min.js"></script>
<script>
/* =====================================================================
   1) TUS DATOS

   ESTADO DE CONSERVACIÓN — procedencia y advertencias
   ---------------------------------------------------------------------
   La categoría del campo `uicn` es la de la Lista Roja GLOBAL de la UICN
   para cada ESPECIE, consultada en iucnredlist.org y contrastada con
   Plants of the World Online (Kew) y las fichas de iNaturalist.

   Categorías de este inventario:
     Handroanthus chrysanthus ... VU (2020)   <- la única amenazada
     Mangifera indica ........... DD (2021)
     Las ocho restantes ......... LC

   TRES ADVERTENCIAS QUE IMPORTAN:

   1. La categoría es de la ESPECIE, no del ejemplar. Un árbol enfermo
      puede ser LC y uno impecable puede ser VU. Si necesitas seguir la
      salud de cada individuo, eso es un campo aparte: no lo sustituye.

   2. Es la lista GLOBAL. Colombia tiene su propia evaluación nacional
      (Libro Rojo de Plantas y Resolución 1912 de 2017 del Ministerio de
      Ambiente), y no siempre coincide: una especie puede ser LC en el
      mundo y estar amenazada aquí. Para un trabajo formal en Colombia
      conviene declarar las dos.

   3. Las categorías se revisan. Anota siempre el año de evaluación
      (campo uicn_anio) y vuelve a verificar antes de publicar.

   NE (No evaluada) no significa "sin riesgo": significa que nadie la ha
   evaluado. No la trates como equivalente a LC.
   ---------------------------------------------------------------------

   Puedes editar este arreglo a mano, o (más fácil) usar el botón
   "Importar CSV" de la página y no tocar código para nada.

   Si importas un CSV (por ejemplo exportado desde Excel), usa estos
   encabezados de columna, en cualquier orden:
     lat, lon, cientifico, comun, altura, dap, plantado, uicn, uicn_anio,
     nota, foto
   - lat / lon: en grados decimales (WGS84), obligatorios.
   - uicn: categoría de la Lista Roja. Acepta LC, NT, VU, EN, CR, DD y NE.
     Lo que no reconozca se guarda como NE, nunca se inventa una categoría.
   - uicn_anio: año de la evaluación, opcional pero recomendable.
   - foto: URL de la imagen (déjala vacía para usar la ilustración automática).
   Al importar, los árboles de ejemplo de abajo se reemplazan por los tuyos.
   ===================================================================== */
const ARBOLES = [
  { id:1, lat:11.22318, lon:-74.18775, cientifico:"Handroanthus chrysanthus", comun:"Cañaguate",
    altura:11.2, dap:38.5, plantado:"2007", uicn:"VU", uicn_anio:"2020",
    nota:"Pierde todas las hojas antes de florecer y se cubre de flores amarillas en plena sequía.", foto:null },
  { id:2, lat:11.22245, lon:-74.18512, cientifico:"Bursera simaruba", comun:"Indio desnudo",
    altura:8.4, dap:27, plantado:"2011", uicn:"LC", uicn_anio:"2019",
    nota:"Corteza cobriza que se desprende en láminas. La capa interna es verde y hace fotosíntesis.", foto:null },
  { id:3, lat:11.22085, lon:-74.18688, cientifico:"Enterolobium cyclocarpum", comun:"Orejero",
    altura:19.6, dap:74.2, plantado:"1994", uicn:"LC", uicn_anio:"2019",
    nota:"Fruto en legumbre enroscada con forma de oreja. Copa más ancha que alta.", foto:null },
  { id:4, lat:11.21992, lon:-74.18455, cientifico:"Samanea saman", comun:"Campano",
    altura:16.3, dap:96.8, plantado:"1985", uicn:"LC", uicn_anio:"2021",
    nota:"Copa aparasolada. Las hojas se pliegan de noche y con cielo cubierto.", foto:null },
  { id:5, lat:11.22412, lon:-74.18592, cientifico:"Crescentia cujete", comun:"Totumo",
    altura:5.1, dap:16.4, plantado:"2015", uicn:"LC", uicn_anio:"2019",
    nota:"Cauliflora: las flores y los frutos nacen pegados al tronco, no en las ramas finas.", foto:null },
  { id:6, lat:11.22135, lon:-74.18885, cientifico:"Mangifera indica", comun:"Mango",
    altura:13.8, dap:52.7, plantado:"1996", uicn:"DD", uicn_anio:"2021",
    nota:"Especie introducida desde Asia. Ramas bajas sobre el andén, requiere poda de realce.", foto:null },
  { id:7, lat:11.22052, lon:-74.18842, cientifico:"Tabebuia rosea", comun:"Roble morado",
    altura:12.4, dap:48, plantado:"2009", uicn:"LC", uicn_anio:"2019",
    nota:"Floración rosada entre febrero y marzo.", foto:null },
  { id:8, lat:11.22380, lon:-74.18398, cientifico:"Ceiba pentandra", comun:"Ceiba",
    altura:21.0, dap:135, plantado:"1978", uicn:"LC", uicn_anio:"2018",
    nota:"Ejemplar patrimonial, raíces tabulares visibles.", foto:null },
  { id:9, lat:11.21925, lon:-74.18670, cientifico:"Roystonea regia", comun:"Palma real",
    altura:16.2, dap:40, plantado:"2013", uicn:"LC", uicn_anio:"2019",
    nota:"Alineamiento de seis palmas en el andén interno del campus.", foto:null },
  { id:10, lat:11.22290, lon:-74.18645, cientifico:"Pithecellobium dulce", comun:"Payandé",
    altura:6.1, dap:33, plantado:"2016", uicn:"LC", uicn_anio:"2019",
    nota:"Herida en el fuste con pudrición, evaluar por un ingeniero forestal.", foto:null }
];

/* Categorías de la Lista Roja de la UICN con sus colores oficiales.
   Están las nueve, aunque el inventario solo use tres, para que cualquier
   especie que agregues encuentre la suya. */
const COLORES = {
  EX:"#000000",   // Extinta
  EW:"#3D2444",   // Extinta en estado silvestre
  CR:"#D81E05",   // En peligro crítico
  EN:"#FC7F3F",   // En peligro
  VU:"#F9E814",   // Vulnerable
  NT:"#CCE226",   // Casi amenazada
  LC:"#60C659",   // Preocupación menor
  DD:"#D1D1C6",   // Datos insuficientes
  NE:"#FFFFFF"    // No evaluada
};
const ETIQUETA = {
  EX:"Extinta (EX)",
  EW:"Extinta en estado silvestre (EW)",
  CR:"En peligro crítico (CR)",
  EN:"En peligro (EN)",
  VU:"Vulnerable (VU)",
  NT:"Casi amenazada (NT)",
  LC:"Preocupación menor (LC)",
  DD:"Datos insuficientes (DD)",
  NE:"No evaluada (NE)"
};
/* VU, NT, DD y NE son colores muy claros: estos marcadores llevan un
   borde interior oscuro para no desaparecer sobre el satelital. */
const BORDE_OSCURO = ["VU","NT","DD","NE"];

/* =====================================================================
   1b) QUÉ HERRAMIENTAS VE EL PÚBLICO
   ---------------------------------------------------------------------
   Con false, el botón se elimina del HTML y su código queda inerte: nadie
   que abra la página puede importar, descargar ni agregar árboles.
   El código sigue completo más abajo, así que para trabajar sobre el
   inventario basta poner true aquí, guardar, y volver a false antes de
   publicar. Mantén siempre dos copias: esta para editar y otra publicada.

   Nota: esto controla la interfaz, no la seguridad. Cualquiera puede leer
   el arreglo ARBOLES en el código fuente de la página. Si alguna vez
   manejas ubicaciones sensibles, redondea las coordenadas antes de
   publicar en lugar de confiar en ocultar botones.
   ===================================================================== */
const HERRAMIENTAS = {
  importarCSV:   false,   // botón "Importar CSV"
  descargarCSV:  false,   // botón "Descargar CSV"
  agregarArbol:  false,   // botón "Agregar árbol" y el clic en el mapa
  miUbicacion:   true     // botón "Mi ubicación"
};

/* Quita del documento los botones desactivados */
(function aplicarHerramientas(){
  const mapaBotones = {
    "btn-importar":  HERRAMIENTAS.importarCSV,
    "btn-exportar":  HERRAMIENTAS.descargarCSV,
    "btn-agregar":   HERRAMIENTAS.agregarArbol,
    "btn-ubicacion": HERRAMIENTAS.miUbicacion
  };
  Object.keys(mapaBotones).forEach(id => {
    if (mapaBotones[id]) return;
    const el = document.getElementById(id);
    if (el) el.remove();
  });
  if (!HERRAMIENTAS.importarCSV){
    const inp = document.getElementById("archivo-csv");
    if (inp) inp.remove();
  }
  const barra = document.querySelector(".acciones");
  if (barra && !barra.querySelector("button")) barra.style.display = "none";
})();

/* Registra un manejador solo si el botón sigue existiendo */
function alPulsar(id, manejador){
  const el = document.getElementById(id);
  if (el) el.addEventListener("click", manejador);
  return el;
}

/* =====================================================================
   2) MAPA Y CAPAS BASE REALES
   ===================================================================== */
const mapa = L.map("mapa", { zoomControl:true }).setView([11.2215, -74.1862], 16);

/* ---- CAPAS BASE ---------------------------------------------------------
   Solo dos, tal como se pidió: OpenStreetMap y Satélite.

   PARA AÑADIR O QUITAR CAPAS hay que tocar dos sitios, siempre los dos:
     1) declarar (o borrar) la constante L.tileLayer, aquí abajo;
     2) añadirla (o borrarla) del objeto capasBase, más abajo.
   Si declaras una capa y no la metes en capasBase, no aparece en el
   selector. Si la metes en capasBase sin declararla, la página falla.

   La capa que lleva .addTo(mapa) es la que se ve al abrir. Ahora es
   OpenStreetMap; para arrancar en satélite, mueve el .addTo(mapa) de una
   constante a la otra.

   Otras opciones por si las quieres de vuelta, todas sin nombres de calles
   ni de lugares (pega la URL en un nuevo L.tileLayer):
     Callejero limpio  https://{s}.basemaps.cartocdn.com/rastertiles/voyager_nolabels/{z}/{x}/{y}{r}.png
     Claro             https://{s}.basemaps.cartocdn.com/light_nolabels/{z}/{x}/{y}{r}.png
     Oscuro            https://{s}.basemaps.cartocdn.com/dark_nolabels/{z}/{x}/{y}{r}.png
   --------------------------------------------------------------------- */
const callejero = L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
  maxZoom:19, attribution:"&copy; OpenStreetMap"
}).addTo(mapa);

const satelital = L.tileLayer(
  "https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}", {
  maxZoom:19, attribution:"Im&aacute;genes &copy; Esri, Maxar, Earthstar Geographics"
});

/* ---- TU CARTOGRAFÍA DE QGIS (opcional) --------------------------------
   Dos formas de engancharla. En ambos casos el proyecto de QGIS debe estar
   en EPSG:3857 y con el etiquetado apagado en TODAS las capas
   (Propiedades de capa ▸ Etiquetas ▸ Sin etiquetas).

   A) tipo:"teselas"  — la recomendada, porque es multiescala.
      En QGIS: Procesos ▸ Herramientas ráster ▸ Generar teselas XYZ (Directorio).
        Extensión .......... el campus más un margen del 15%
        Zoom ............... 13 a 19
        DPI ................ 96 (192 si quieres nitidez en pantallas retina)
        Formato ............ PNG si necesitas transparencia, si no JPG
        Metatesela ......... 4, para que no se corten los símbolos
        Eje Y invertido .... DESMARCADO (Leaflet usa convención XYZ)
      Copia la carpeta resultante junto a este archivo y deja
      plantilla:"tiles/{z}/{x}/{y}.png". Para un campus de 2x2 km salen
      unas 1000 teselas, entre 5 y 20 MB.

   B) tipo:"imagen"   — rápida, pero de una sola escala.
      Diseño de impresión con SOLO el marco de mapa: sin escala gráfica,
      sin flecha de norte, sin título, sin márgenes. Al exportar marca
      "Generar archivo de georreferenciación (world file)". Obtienes .png
      y .pgw; convierte las esquinas del .pgw a lat/lon y ponlas abajo.

   Cambia activo a true y la capa aparece en el selector, arriba a la
   derecha, como capa base predeterminada.

   Ojo con la atribución: aunque generes las teselas tú, la licencia de los
   datos de origen te sigue. Si construiste la base sobre OSM, la mención
   ODbL es obligatoria; lo mismo con IGAC, Esri o NASA.
   ------------------------------------------------------------------- */
const MI_PLANO = {
  activo:     false,
  tipo:       "teselas",              // "teselas" o "imagen"

  // --- si tipo === "teselas" ---
  plantilla:  "tiles/{z}/{x}/{y}.png",
  zoomMin:    13,
  zoomMax:    19,

  // --- si tipo === "imagen" ---
  url:        "plano.png",
  suroeste:   [11.2180, -74.1900],    // [lat, lon] esquina inferior izquierda
  noreste:    [11.2250, -74.1820],    // [lat, lon] esquina superior derecha

  opacidad:   1,
  atribucion: "Cartograf&iacute;a propia (QGIS) &middot; Datos: OSM (ODbL)"
};

const capasBase = {
  "OpenStreetMap": callejero,
  "Satelital":     satelital
};

if (MI_PLANO.activo){
  const miPlano = MI_PLANO.tipo === "teselas"
    ? L.tileLayer(MI_PLANO.plantilla, {
        minZoom:MI_PLANO.zoomMin,
        maxZoom:MI_PLANO.zoomMax,
        maxNativeZoom:MI_PLANO.zoomMax,   // permite acercar más sin pedir teselas inexistentes
        opacity:MI_PLANO.opacidad,
        attribution:MI_PLANO.atribucion })
    : L.imageOverlay(MI_PLANO.url, [MI_PLANO.suroeste, MI_PLANO.noreste], {
        opacity:MI_PLANO.opacidad,
        attribution:MI_PLANO.atribucion });

  mapa.removeLayer(callejero);
  miPlano.addTo(mapa);
  // Se inserta de primera para que quede como opción predeterminada del selector
  const reordenadas = { "Cartograf\u00eda del proyecto (QGIS)": miPlano };
  Object.keys(capasBase).forEach(k => { reordenadas[k] = capasBase[k]; });
  Object.keys(capasBase).forEach(k => { delete capasBase[k]; });
  Object.keys(reordenadas).forEach(k => { capasBase[k] = reordenadas[k]; });
}

L.control.layers(capasBase, {}, { position:"topright" }).addTo(mapa);

L.control.scale({ imperial:false, position:"bottomright" }).addTo(mapa);

/* =====================================================================
   3) MARCADORES Y FICHAS
   ===================================================================== */
const capaArboles = L.layerGroup().addTo(mapa);
const registro = new Map();   // id -> { marcador, datos }

function ilustracion(arbol){
  const copa = COLORES[arbol.uicn] || COLORES.NE;
  const svg = `<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 260 150">
    <rect width="260" height="150" fill="#E4E9DE"/>
    <rect x="0" y="118" width="260" height="32" fill="#D3D9CA"/>
    <rect x="123" y="72" width="14" height="48" fill="#6B4F3A"/>
    <path d="M130 78 L108 58 M130 88 L152 66" stroke="#6B4F3A" stroke-width="6" fill="none"/>
    <circle cx="130" cy="60" r="34" fill="${copa}"/>
    <circle cx="100" cy="72" r="24" fill="${copa}" opacity=".88"/>
    <circle cx="160" cy="70" r="26" fill="${copa}" opacity=".82"/>
    <text x="130" y="140" font-family="Georgia,serif" font-style="italic" font-size="13"
      text-anchor="middle" fill="#5e6d60">${arbol.cientifico}</text>
  </svg>`;
  return "data:image/svg+xml;charset=utf-8," + encodeURIComponent(svg);
}

function ficha(a){
  const src = a.foto || ilustracion(a);
  return `<div class="ficha">
    <img src="${src}" alt="Fotografía de ${a.comun}">
    <div class="cuerpo">
      <h2>${a.cientifico}</h2>
      <p class="comun">${a.comun}</p>
      <dl>
        <dt>Estado UICN</dt>
        <dd class="estado"><span class="punto" style="width:9px;height:9px;border-radius:50%;border:1px solid rgba(0,0,0,.35);background:${COLORES[a.uicn] || COLORES.NE}"></span>${ETIQUETA[a.uicn] || ETIQUETA.NE}</dd>
        <dt>Evaluada en</dt><dd>${a.uicn_anio || "—"}</dd>
        <dt>Altura</dt><dd>${a.altura} m</dd>
        <dt>DAP</dt><dd>${a.dap} cm</dd>
        <dt>Plantado</dt><dd>${a.plantado}</dd>
        <dt>Coordenadas</dt><dd>${a.lat.toFixed(5)}, ${a.lon.toFixed(5)}</dd>
      </dl>
      ${a.nota ? `<p class="nota">${a.nota}</p>` : ""}
    </div>
  </div>`;
}

function dibujar(a){
  const icono = L.divIcon({
    className:"",
    html:`<div class="marcador${BORDE_OSCURO.includes(a.uicn) ? " claro" : ""}" style="background:${COLORES[a.uicn] || COLORES.NE}"></div>`,
    iconSize:[16,16], iconAnchor:[8,8], popupAnchor:[0,-8]
  });
  const m = L.marker([a.lat, a.lon], { icon:icono, title:`${a.comun} — ${a.cientifico}`, riseOnHover:true })
    .bindPopup(ficha(a), { maxWidth:262, minWidth:262 })
    .addTo(capaArboles);
  m.on("popupopen", () => marcarActivo(a.id));
  registro.set(a.id, { marcador:m, datos:a });
}

ARBOLES.forEach(dibujar);
/* maxZoom 16 deja ver las vías del entorno del campus. Sin ese tope el
   encuadre se cierra tanto sobre los árboles que la base parece vacía. */
mapa.fitBounds(L.latLngBounds(ARBOLES.map(a => [a.lat, a.lon])).pad(0.45), { maxZoom:16 });

/* =====================================================================
   4) LISTA LATERAL SINCRONIZADA CON EL MAPA
   ===================================================================== */
const lista = document.getElementById("lista");

function pintarLista(filtro = ""){
  const q = filtro.trim().toLowerCase();
  const visibles = [...registro.values()]
    .map(r => r.datos)
    .filter(a => !q || a.cientifico.toLowerCase().includes(q) || a.comun.toLowerCase().includes(q));

  if (!visibles.length){
    lista.innerHTML = `<p style="padding:16px 10px;font-size:13px;color:#9db19f">
      Ningún árbol coincide con “${filtro}”. Prueba con otro nombre o borra la búsqueda.</p>`;
    return;
  }

  lista.innerHTML = visibles.map(a => `
    <button class="arbol" data-id="${a.id}" aria-current="false">
      <span class="punto" style="background:${COLORES[a.uicn] || COLORES.NE}"></span>
      <span>
        <span class="cientifico">${a.cientifico}</span>
        <span class="comun">${a.comun} · ${a.altura} m · ${a.lat.toFixed(4)}, ${a.lon.toFixed(4)}</span>
      </span>
    </button>`).join("");

  lista.querySelectorAll(".arbol").forEach(b => {
    b.addEventListener("click", () => irA(Number(b.dataset.id)));
  });
}

function irA(id){
  const r = registro.get(id);
  if (!r) return;
  mapa.flyTo(r.marcador.getLatLng(), Math.max(mapa.getZoom(), 17), { duration:.7 });
  r.marcador.openPopup();
}

function marcarActivo(id){
  lista.querySelectorAll(".arbol").forEach(b => {
    b.setAttribute("aria-current", String(Number(b.dataset.id) === id));
  });
  registro.forEach((r, key) => {
    const el = r.marcador.getElement()?.querySelector(".marcador");
    if (el) el.classList.toggle("activo", key === id);
  });
}

mapa.on("popupclose", () => marcarActivo(null));
document.getElementById("buscar").addEventListener("input", e => pintarLista(e.target.value));
pintarLista();

/* =====================================================================
   5) COORDENADAS EN VIVO
   ===================================================================== */
const coords = document.getElementById("coords");
mapa.on("mousemove", e => {
  coords.textContent = `Lat ${e.latlng.lat.toFixed(5)}  ·  Lon ${e.latlng.lng.toFixed(5)}`;
});

/* =====================================================================
   6) AGREGAR ÁRBOL CON UN CLIC
   ===================================================================== */
const aviso = document.getElementById("aviso");
let modoAgregar = false;

function mostrarAviso(texto, ms = 3200){
  aviso.textContent = texto;
  aviso.classList.add("visible");
  clearTimeout(mostrarAviso._t);
  mostrarAviso._t = setTimeout(() => aviso.classList.remove("visible"), ms);
}

const btnAgregar = alPulsar("btn-agregar", () => {
  modoAgregar = !modoAgregar;
  btnAgregar.setAttribute("aria-pressed", String(modoAgregar));
  btnAgregar.textContent = modoAgregar ? "Cancelar" : "Agregar árbol";
  mapa.getContainer().style.cursor = modoAgregar ? "crosshair" : "";
  if (modoAgregar) mostrarAviso("Haz clic en el mapa donde está el árbol");
});

mapa.on("click", e => {
  if (!HERRAMIENTAS.agregarArbol || !modoAgregar) return;
  const comun = prompt("Nombre común del árbol:", "Roble morado");
  if (comun === null) return;
  const cientifico = prompt("Nombre científico:", "Tabebuia rosea") || "Sin identificar";
  const altura = parseFloat(prompt("Altura en metros:", "8")) || 0;
  const uicn = (prompt("Categoría UICN (LC, NT, VU, EN, CR, DD, NE):", "NE") || "NE").toUpperCase();

  const nuevo = {
    id: Date.now(),
    lat: e.latlng.lat, lon: e.latlng.lng,
    cientifico, comun: comun || "Sin nombre",
    altura, dap: 0, plantado: new Date().getFullYear().toString(),
    uicn: COLORES[uicn] ? uicn : "NE",
    uicn_anio: "",
    nota: "Registrado desde el mapa.", foto: null
  };

  dibujar(nuevo);
  pintarLista(document.getElementById("buscar").value);
  irA(nuevo.id);
  modoAgregar = false;
  if (btnAgregar){
    btnAgregar.setAttribute("aria-pressed","false");
    btnAgregar.textContent = "Agregar árbol";
  }
  mapa.getContainer().style.cursor = "";
});

/* =====================================================================
   7) IMPORTAR Y EXPORTAR CSV, IR A MI UBICACIÓN
   ===================================================================== */
alPulsar("btn-importar", () => {
  if (typeof Papa === "undefined"){
    mostrarAviso("El lector de CSV no se cargó. Revisa tu conexión y recarga la página.");
    return;
  }
  document.getElementById("archivo-csv").click();
});

const entradaCSV = document.getElementById("archivo-csv");
if (entradaCSV) entradaCSV.addEventListener("change", e => {
  const archivo = e.target.files[0];
  if (!archivo) return;

  Papa.parse(archivo, {
    header:true,
    skipEmptyLines:true,
    transformHeader: h => h.trim().toLowerCase(),
    complete: resultado => {
      const nuevos = resultado.data.map((f, i) => ({
        id: Date.now() + i,
        lat: parseFloat(f.lat ?? f.latitud),
        lon: parseFloat(f.lon ?? f.lng ?? f.longitud),
        cientifico: (f.cientifico || f.especie || "Sin identificar").trim(),
        comun: (f.comun || f.nombre_comun || "Sin nombre").trim(),
        altura: parseFloat(f.altura) || 0,
        dap: parseFloat(f.dap) || 0,
        plantado: f.plantado || f.año || "",
        uicn: COLORES[String(f.uicn || f.categoria || f.estado || "NE").toUpperCase()]
                ? String(f.uicn || f.categoria || f.estado).toUpperCase() : "NE",
        uicn_anio: f.uicn_anio || f.anio_evaluacion || "",
        nota: f.nota || f.observaciones || "",
        foto: f.foto || f.url_foto || null
      })).filter(a => !isNaN(a.lat) && !isNaN(a.lon));

      if (!nuevos.length){
        mostrarAviso("No se encontraron coordenadas válidas. Revisa que existan columnas lat y lon.");
        return;
      }

      capaArboles.clearLayers();
      registro.clear();
      nuevos.forEach(dibujar);
      pintarLista(document.getElementById("buscar").value);
      mapa.fitBounds(L.latLngBounds(nuevos.map(a => [a.lat, a.lon])).pad(0.25));
      mostrarAviso(`Se cargaron ${nuevos.length} árboles desde ${archivo.name}`);
      e.target.value = "";
    },
    error: () => mostrarAviso("No se pudo leer el archivo. Revisa que sea un CSV válido.")
  });
});

alPulsar("btn-exportar", () => {
  const datos = [...registro.values()].map(r => r.datos);
  const csv = Papa.unparse(datos);
  const blob = new Blob([csv], { type:"text/csv;charset=utf-8" });
  const url = URL.createObjectURL(blob);
  const a = document.createElement("a");
  a.href = url; a.download = "arboles.csv"; a.click();
  URL.revokeObjectURL(url);
  mostrarAviso(`Se descargaron ${datos.length} árboles en arboles.csv`);
});

alPulsar("btn-ubicacion", () => {
  if (!navigator.geolocation){
    mostrarAviso("Este navegador no entrega tu ubicación");
    return;
  }
  navigator.geolocation.getCurrentPosition(
    p => mapa.flyTo([p.coords.latitude, p.coords.longitude], 18),
    () => mostrarAviso("No se pudo obtener tu ubicación. Revisa los permisos del navegador.")
  );
});
</script>
</body>
</html>
