<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Money Moves with Miller</title>
  <style>
    body {
      font-family: 'Fredoka', sans-serif;
      background: #d0f0c0;
      margin: 0;
      padding: 0;
      text-align: center;
      color: #3d1a40;
    }
    h1 {
      background: #ffcad4;
      margin: 0;
      padding: 1rem;
      font-size: 2rem;
      color: #6a0572;
    }
    .board {
      display: grid;
      grid-template-columns: repeat(11, 80px);
      grid-template-rows: repeat(11, 80px);
      width: 880px;
      height: 880px;
      margin: 2rem auto;
      position: relative;
      background-color: #fff;
      border: 8px solid #6a0572;
    }
    .tile {
      border: 1px solid #ccc;
      font-size: 0.7rem;
      font-weight: bold;
      display: flex;
      justify-content: center;
      align-items: center;
      text-align: center;
      box-sizing: border-box;
    }
    .center {
      grid-column: 5 / span 3;
      grid-row: 5 / span 3;
      background-color: #ffeb3b;
      z-index: 1;
      border-radius: 12px;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      font-size: 1rem;
    }
    .dashboard {
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      gap: 1rem;
      margin: 1rem;
    }
    .player-card {
      border: 2px dashed #f48fb1;
      border-radius: 12px;
      padding: 1rem;
      background: #ffffff;
      width: 220px;
      box-shadow: 0 4px 8px rgba(0,0,0,0.08);
    }
    .property-drop {
      min-height: 40px;
      border: 1px dashed #ccc;
      margin-top: 0.5rem;
      padding: 5px;
    }
    .token {
      font-size: 1.2rem;
      cursor: grab;
    }
    .property-card {
      display: inline-block;
      margin: 0.5rem;
      padding: 0.5rem 1rem;
      font-weight: bold;
      border: 2px solid #333;
      border-radius: 6px;
      cursor: grab;
      background-color: #fff;
    }
  </style>
</head>
<body>
  <h1>🎲💸 Money Moves with Miller 💸🎲</h1>
  <div class="board" id="gameBoard"></div>
  <div class="center">
    💰 Free Parking<br>
    $<input id="freeParking" type="number" value="0" style="width: 80px;">
  </div>
  <h2>🏦 Player Banks</h2>
  <div class="dashboard" id="dashboard"></div>
  <h2>🏷️ Property Cards</h2>
  <div class="dashboard" id="propertyCards"></div>
  <script>
    const tileNames = [
      "Go", "Budget Blvd", "Class Chest", "Debt Drive", "Snack Tax", "Director Diamond Rail", "Penny Lane", "Pop Quiz Panic", "Credit Court", "Nickel Nook", "Jail / Just Visiting",
      "Jazzlynn Junction", "Utility Hub", "Braelyn Bend", "Theo Theater", "Wojeck Station", "Amor Alley", "Class Chest", "Kyle Court", "Emma Estates", "Free Parking",
      "Avah Avenue", "Pop Quiz Panic", "Angel Alley", "Lydia Lane", "Ryah Rail", "Ari Arcade", "Snack Stand", "Utility Co.", "Benaiah Blvd", "Go to Budget Bootcamp",
      "Kiona Keys", "Class Chest", "William Way", "Principal Parkway", "Final Exam Express", "Chance Challenge", "Capstone Court", "Snack Tax II", "Graduation Gardens"
    ];

    const propertyColors = [
      "", "brown", "community", "brown", "tax", "railroad", "light-blue", "chance", "light-blue", "light-blue", "",
      "pink", "utility", "pink", "pink", "railroad", "orange", "community", "orange", "orange", "",
      "red", "chance", "red", "red", "railroad", "yellow", "yellow", "utility", "yellow", "",
      "green", "community", "green", "green", "railroad", "chance", "dark-blue", "tax", "dark-blue"
    ];

    function buildBoard() {
      const board = document.getElementById("gameBoard");
      for (let i = 0; i < 121; i++) {
        const tile = document.createElement("div");
        tile.className = "tile";
        board.appendChild(tile);
      }

      const outerTiles = [...Array(11).keys(),
        ...Array.from({length: 9}, (_, i) => (i+1)*11 + 10),
        ...Array.from({length: 11}, (_, i) => 120 - i),
        ...Array.from({length: 9}, (_, i) => (9 - i)*11)
      ];

      outerTiles.forEach((index, i) => {
        const tile = board.children[index];
        tile.textContent = tileNames[i] || `Tile ${i}`;
        if (propertyColors[i]) tile.classList.add(propertyColors[i]);
        tile.id = `tile-${i}`;
        tile.ondrop = drop;
        tile.ondragover = allowDrop;
      });
    }

    const students = [
      { name: 'Jazzlynn', token: '🦋' }, { name: 'Braelyn', token: '🌈' }, { name: 'Josh', token: '🐢' }, { name: 'Angel', token: '🦅' },
      { name: 'Ari', token: '🎧' }, { name: 'Kyle', token: '🚀' }, { name: 'Emma', token: '🐝' }, { name: 'Amor', token: '🌻' },
      { name: 'Lydia', token: '🐱' }, { name: 'Theo', token: '🧸' }, { name: 'William', token: '🐧' }, { name: 'Avah', token: '🧁' },
      { name: 'Ryah', token: '🦄' }, { name: 'Kiona', token: '✨' }
    ];

    function setupPlayers() {
      const dashboard = document.getElementById("dashboard");
      const cards = document.getElementById("propertyCards");
      students.forEach((student, index) => {
        const card = document.createElement("div");
        card.classList.add("player-card");
        card.innerHTML = `
          <h3>${student.name} ${student.token}</h3>
          <p><strong>Money:</strong> $<input type="number" value="1000"></p>
          <div><strong>Properties:</strong>
            <div class="property-drop" ondrop="drop(event)" ondragover="allowDrop(event)"></div>
          </div>`;
        dashboard.appendChild(card);

        const token = document.createElement("div");
        token.className = "token";
        token.draggable = true;
        token.textContent = student.token;
        token.id = `token-${student.name}`;
        token.ondragstart = drag;
        const startTile = document.getElementById("tile-0");
        startTile.appendChild(token);
      });

      tileNames.forEach((name, i) => {
        if (["brown","light-blue","pink","orange","red","yellow","green","dark-blue"].includes(propertyColors[i])) {
          const propCard = document.createElement("div");
          propCard.classList.add("property-card", `card-${propertyColors[i]}`);
          propCard.draggable = true;
          propCard.id = `card-${name}`;
          propCard.textContent = name;
          propCard.ondragstart = drag;
          cards.appendChild(propCard);
        }
      });
    }

    function allowDrop(ev) { ev.preventDefault(); }
    function drag(ev) { ev.dataTransfer.setData("text", ev.target.id); }
    function drop(ev) {
      ev.preventDefault();
      const data = ev.dataTransfer.getData("text");
      const token = document.getElementById(data);
      ev.target.appendChild(token);
    }

    window.onload = () => {
      buildBoard();
      setupPlayers();
    };
  </script>
</body>
</html>

