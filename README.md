<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Nursing Tool Matching Game</title>
  <style>
    :root {
      --primary: #9c27b0;      /* Ungu Utama */
      --secondary: #e91e63;    /* Pink Utama */
      --success: #2a9d8f;      /* Hijau lembut bila jawapan betul */
      --error: #e63946;        /* Merah bila jawapan salah */
      --bg: #fdf2f8;           /* Latar Belakang Pink/Ungu Pastel */
      --card-bg: #ffffff;
      --text: #4a148c;         /* Warna Teks Ungu Gelap */
    }

    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
    }

    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background-color: var(--bg);
      color: var(--text);
      display: flex;
      flex-direction: column;
      align-items: center;
      min-height: 100vh;
      padding: 20px;
    }

    header {
      text-align: center;
      margin-bottom: 20px;
    }

    h1 {
      color: var(--primary);
      font-size: 1.8rem;
      margin-bottom: 6px;
    }

    p {
      color: #6a1b9a;
      font-size: 0.95rem;
    }

    .stats-bar {
      display: flex;
      gap: 20px;
      margin-bottom: 20px;
      background: white;
      padding: 10px 24px;
      border-radius: 30px;
      box-shadow: 0 2px 8px rgba(156, 39, 176, 0.15);
      font-weight: 600;
    }

    .stats-bar span {
      color: var(--secondary);
    }

    .matching-container {
      display: flex;
      justify-content: center;
      gap: 30px;
      width: 100%;
      max-width: 800px;
    }

    .column {
      display: flex;
      flex-direction: column;
      gap: 12px;
      flex: 1;
    }

    .column-title {
      text-align: center;
      font-weight: 700;
      color: var(--primary);
      margin-bottom: 5px;
      font-size: 1.1rem;
    }

    .item-card {
      background: var(--card-bg);
      border: 2px solid #f3d5f7;
      border-radius: 12px;
      padding: 10px;
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      min-height: 80px;
      box-shadow: 0 2px 5px rgba(0,0,0,0.05);
      transition: all 0.2s ease;
      user-select: none;
    }

    .item-card:hover:not(.matched) {
      border-color: var(--secondary);
      transform: translateY(-2px);
    }

    .item-card.selected {
      border-color: var(--primary);
      background-color: #f3e8ff;
      box-shadow: 0 0 0 3px rgba(156, 39, 176, 0.3);
    }

    .item-card.matched {
      border-color: var(--success);
      background-color: #d1fae5;
      cursor: default;
      opacity: 0.85;
    }

    .item-card.incorrect {
      border-color: var(--error);
      background-color: #ffe4e6;
    }

    .item-card img {
      max-width: 100%;
      max-height: 70px;
      object-fit: contain;
    }

    .item-card .tool-name {
      font-weight: 600;
      font-size: 0.95rem;
      text-align: center;
    }

    .controls {
      margin-top: 25px;
    }

    button {
      background-color: var(--primary);
      color: white;
      border: none;
      padding: 10px 24px;
      font-size: 1rem;
      font-weight: 600;
      border-radius: 20px;
      cursor: pointer;
      box-shadow: 0 4px 6px rgba(156, 39, 176, 0.2);
      transition: background 0.2s, transform 0.1s;
    }

    button:hover {
      background-color: var(--secondary);
    }

    button:active {
      transform: scale(0.98);
    }

    .win-message {
      margin-top: 15px;
      font-weight: bold;
      color: var(--success);
      font-size: 1.2rem;
      min-height: 30px;
    }

    @media (max-width: 600px) {
      .matching-container {
        gap: 15px;
      }
      .item-card {
        min-height: 70px;
        padding: 6px;
      }
      .item-card img {
        max-height: 55px;
      }
      .item-card .tool-name {
        font-size: 0.8rem;
      }
    }
  </style>
</head>
<body>

  <header>
    <h1>Nursing Fundamentals: Tool Matcher</h1>
    <p>Click a picture on the left, then click its matching name on the right.</p>
  </header>

  <section class="stats-bar">
    <div>Matches: <span id="matches-count">0</span> / 10</div>
    <div>Attempts: <span id="attempts-count">0</span></div>
  </section>

  <main class="matching-container">
    <div class="column">
      <div class="column-title">Medical Picture</div>
      <div id="images-column" class="column"></div>
    </div>
    
    <div class="column">
      <div class="column-title">Instrument Name</div>
      <div id="names-column" class="column"></div>
    </div>
  </main>

  <div class="win-message" id="win-message"></div>

  <div class="controls">
    <button id="reset-btn">Restart Game</button>
  </div>

  <script>
    const instruments = [
      { id: 'sphygmomanometer', name: 'Aneroid Sphygmomanometer', image: 'aneroid sphygmomanometer.jpg' },
      { id: 'artery_forceps', name: 'Artery Forceps', image: 'artery forcep.jpg' },
      { id: 'branula', name: 'Branula (IV Cannula)', image: 'branula.jpg' },
      { id: 'bryant_forceps', name: 'Bryant Dressing Forceps', image: 'bryant dressing forceps.jpg' },
      { id: 'cotton_bandage', name: 'Cotton Bandage', image: 'cotton bandage.jpg' },
      { id: 'cotton_balls', name: 'Cotton Balls', image: 'cotton.jpg' },
      { id: 'crepe_bandage', name: 'Crepe Bandage', image: 'crepe bandage.jpg' },
      { id: 'durapore', name: 'Durapore Surgical Tape', image: 'durapore,jpg.jpg' },
      { id: 'gallipot', name: 'Gallipot', image: 'gallipot.jpg' },
      { id: 'gauze', name: 'Gauze Swabs', image: 'gauze.jpg' }
    ];

    let selectedImgCard = null;
    let selectedNameCard = null;
    let matchedCount = 0;
    let attemptsCount = 0;

    const imagesCol = document.getElementById('images-column');
    const namesCol = document.getElementById('names-column');
    const matchesEl = document.getElementById('matches-count');
    const attemptsEl = document.getElementById('attempts-count');
    const winMessageEl = document.getElementById('win-message');
    const resetBtn = document.getElementById('reset-btn');

    function shuffle(array) {
      return [...array].sort(() => Math.random() - 0.5);
    }

    function initGame() {
      imagesCol.innerHTML = '';
      namesCol.innerHTML = '';
      winMessageEl.textContent = '';
      selectedImgCard = null;
      selectedNameCard = null;
      matchedCount = 0;
      attemptsCount = 0;
      matchesEl.textContent = matchedCount;
      attemptsEl.textContent = attemptsCount;

      const shuffledImages = shuffle(instruments);
      const shuffledNames = shuffle(instruments);

      shuffledImages.forEach(item => {
        const card = document.createElement('div');
        card.classList.add('item-card');
        card.dataset.id = item.id;
        card.innerHTML = `<img src="${item.image}" alt="${item.name}">`;
        card.addEventListener('click', () => selectImage(card));
        imagesCol.appendChild(card);
      });

      shuffledNames.forEach(item => {
        const card = document.createElement('div');
        card.classList.add('item-card');
        card.dataset.id = item.id;
        card.innerHTML = `<div class="tool-name">${item.name}</div>`;
        card.addEventListener('click', () => selectName(card));
        namesCol.appendChild(card);
      });
    }

    function selectImage(card) {
      if (card.classList.contains('matched')) return;
      if (selectedImgCard) selectedImgCard.classList.remove('selected');
      selectedImgCard = card;
      selectedImgCard.classList.add('selected');
      checkPair();
    }

    function selectName(card) {
      if (card.classList.contains('matched')) return;
      if (selectedNameCard) selectedNameCard.classList.remove('selected');
      selectedNameCard = card;
      selectedNameCard.classList.add('selected');
      checkPair();
    }

    function checkPair() {
      if (!selectedImgCard || !selectedNameCard) return;

      attemptsCount++;
      attemptsEl.textContent = attemptsCount;

      const imgId = selectedImgCard.dataset.id;
      const nameId = selectedNameCard.dataset.id;

      if (imgId === nameId) {
        selectedImgCard.classList.remove('selected');
        selectedNameCard.classList.remove('selected');
        selectedImgCard.classList.add('matched');
        selectedNameCard.classList.add('matched');
        selectedImgCard = null;
        selectedNameCard = null;

        matchedCount++;
        matchesEl.textContent = matchedCount;

        if (matchedCount === instruments.length) {
          winMessageEl.textContent = '🎉 Congratulations! You successfully matched all tools!';
        }
      } else {
        const tempImg = selectedImgCard;
        const tempName = selectedNameCard;
        tempImg.classList.add('incorrect');
        tempName.classList.add('incorrect');
        selectedImgCard = null;
        selectedNameCard = null;

        setTimeout(() => {
          tempImg.classList.remove('selected', 'incorrect');
          tempName.classList.remove('selected', 'incorrect');
        }, 600);
      }
    }

    resetBtn.addEventListener('click', initGame);
    initGame();
  </script>
</body>
</html>
