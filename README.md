<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<title>El Impostor</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
body {
  margin: 0;
  font-family: Arial, Helvetica, sans-serif;
  background: #111;
  color: #f2f2f2;
  text-align: center;
}

.screen {
  display: none;
  padding: 25px;
}

.active {
  display: block;
}

.logo {
  max-width: 120px;
  margin: 20px auto;
}

h1, h2 {
  letter-spacing: 1px;
}

button {
  background: #c62828;
  border: none;
  color: white;
  padding: 15px 25px;
  margin: 10px;
  font-size: 16px;
  cursor: pointer;
  border-radius: 6px;
}

button:hover {
  background: #b71c1c;
}

input, select {
  padding: 10px;
  font-size: 16px;
  width: 80%;
  margin: 10px 0;
}

.small {
  opacity: 0.7;
  font-size: 14px;
}

.word {
  font-size: 28px;
  margin: 40px 0;
}

.impostor {
  color: #ff5252;
  font-size: 30px;
  font-weight: bold;
}
</style>
</head>

<body>

<img src="logo.png" class="logo">

<!-- REGLAS -->
<div class="screen active" id="rules">
  <h1>EL IMPOSTOR</h1>
  <p>Todos saben la palabra.<br>Uno (o dos) no.<br>Mentí bien o pagá un trago.</p>
  <p class="small">No mires cuando no te toca, botón.</p>
  <button onclick="show('setup')">ENTENDÍ, ARRANCAR</button>
</div>

<!-- SETUP -->
<div class="screen" id="setup">
  <h2>CONFIGURACIÓN</h2>
  <input type="number" id="table" placeholder="Número de mesa">
  <input type="number" id="players" placeholder="Cantidad de jugadores (3 a 8)">
  <button onclick="goCategory()">SEGUIR</button>
</div>

<!-- CATEGORÍA -->
<div class="screen" id="category">
  <h2>CATEGORÍA</h2>
  <button onclick="startGame('futbol')">⚽ FÚTBOL</button>
  <button onclick="startGame('tv')">📺 TV ARGENTINA</button>
  <button onclick="startGame('hogar')">🏠 COSAS DEL HOGAR</button>
  <button onclick="startGame('disney')">🐭 DISNEY</button>
  <button onclick="startGame('general')">🎲 GENERALES</button>
  <button onclick="startGame('picante')">🔥 +18 PICANTE</button>
</div>

<!-- PASAR -->
<div class="screen" id="pass">
  <h2 id="passText"></h2>
  <p class="small">No seas chusma.</p>
  <button onclick="countdown()">PASAR AL SIGUIENTE JUGADOR</button>
</div>

<!-- CUENTA -->
<div class="screen" id="count">
  <h2>PREPARATE…</h2>
  <div id="countNum" class="word"></div>
</div>

<!-- SECRETO -->
<div class="screen" id="secret">
  <div id="secretWord" class="word"></div>
  <button onclick="continueGame()">CONTINUAR</button>
</div>

<!-- FINAL -->
<div class="screen" id="end">
  <h2>FIN DEL JUEGO</h2>
  <p>¿Qué pasó?</p>
  <button onclick="alert('El impostor ganó. Vergüenza ajena.')">GANÓ EL IMPOSTOR</button>
  <button onclick="alert('Bien ahí. Alguien paga algo.')">ENCONTRARON AL IMPOSTOR</button>
  <button onclick="location.reload()">JUGAR DE NUEVO</button>
</div>

<script>
let totalPlayers = 0;
let currentPlayer = 1;
let impostors = [];
let secretWord = "";

const words = {
  futbol: [
    "Messi","Maradona","Ronaldo","Mbappé","Neymar","Benzema","Haaland","Lewandowski",
    "Iniesta","Xavi","Modric","Zidane","Ronaldinho","Kane","Suárez","Griezmann",
    "De Bruyne","Salah","Kroos","Dybala"
  ],
  tv: [
    "Moria Casán","Zulma Lobato","Ricardo Fort","Susana Giménez","Marcelo Tinelli",
    "Vicky Xipolitakis","Alex Caniggia","Charlotte Caniggia","Aníbal Pachano",
    "Flavio Mendoza","Carmen Barbieri","Morena Rial","Wanda Nara","L-Gante",
    "Samanta Farjat","Alfa GH","Beto Casella","Yanina Latorre","Marley","Cacho Castaña"
  ],
  hogar: [
    "Escoba","Heladera","Microondas","Cuchillo","Sillón","Almohada","Ventilador",
    "Plancha","Tender","Espejo","Vaso","Plato","Tenedor","Sartén","Cafetera",
    "Licuadora","Toalla","Cepillo","Mesa","Silla"
  ],
  disney: [
    "Mickey","Minnie","Goofy","Donald","Simba","Scar","Elsa","Anna","Buzz","Woody"
  ],
  general: [
    "Pizza","Asado","Mate","Playa","Fiesta","Noche","Viaje","Birra","Amigos",
    "Música","Baile","Celular","Auto","Trabajo","Sueño","Lluvia","Sol","Calor",
    "Frío","Domingo"
  ],
  picante: [
    "Lencería","Besos intensos","Roleplay","Aceite corporal","Esposas",
    "Juguete íntimo","Strip tease","Masaje lento","Ducha compartida",
    "Mordidas","Susurros","Cama desordenada","Miradas incómodas",
    "Tacones","Camisa abierta","Juego previo","Luces bajas",
    "Sábanas","Perfume","Prohibido"
  ]
};

function show(id) {
  document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
  document.getElementById(id).classList.add('active');
}

function goCategory() {
  totalPlayers = parseInt(document.getElementById('players').value);
  if (totalPlayers < 3 || totalPlayers > 8) {
    alert("De 3 a 8 jugadores, no seas vivo.");
    return;
  }
  show('category');
}

function startGame(cat) {
  currentPlayer = 1;
  impostors = [];
  secretWord = words[cat][Math.floor(Math.random() * words[cat].length)];

  let impostorCount = totalPlayers > 5 ? 2 : 1;
  while (impostors.length < impostorCount) {
    let r = Math.floor(Math.random() * totalPlayers) + 1;
    if (!impostors.includes(r)) impostors.push(r);
  }

  showPass();
}

function showPass() {
  document.getElementById('passText').innerText =
    "Pasá el teléfono al jugador " + currentPlayer;
  show('pass');
}

function countdown() {
  show('count');
  let c = 3;
  document.getElementById('countNum').innerText = c;
  let i = setInterval(() => {
    c--;
    document.getElementById('countNum').innerText = c;
    if (c === 0) {
      clearInterval(i);
      showSecret();
    }
  }, 1000);
}

function showSecret() {
  let el = document.getElementById('secretWord');
  if (impostors.includes(currentPlayer)) {
    el.innerHTML = "<div class='impostor'>SOS EL IMPOSTOR</div>";
  } else {
    el.innerText = secretWord;
  }
  show('secret');
}

function continueGame() {
  currentPlayer++;
  if (currentPlayer > totalPlayers) {
    show('end');
  } else {
    showPass();
  }
}
</script>

</body>
</html>
