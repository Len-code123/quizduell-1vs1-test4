<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Multiplayer Quiz</title>
  <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
  <script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.0/firebase-app.js";
    import { getDatabase, ref, set, onValue, update } from "https://www.gstatic.com/firebasejs/10.12.0/firebase-database.js";

    const firebaseConfig = {
      apiKey: "DEIN_API_KEY",
      authDomain: "DEIN_PROJEKT.firebaseapp.com",
      databaseURL: "https://DEIN_PROJEKT.firebaseio.com",
      projectId: "DEIN_PROJEKT",
      storageBucket: "DEIN_PROJEKT.appspot.com",
      messagingSenderId: "1234567890",
      appId: "1:1234567890:web:abc123"
    };

    const app = initializeApp(firebaseConfig);
    const db = getDatabase(app);

    const quizFragen = [
      { frage: "Was ist die Hauptstadt von Frankreich?", antworten: ["Paris", "Rom", "Berlin", "Madrid"], korrekt: 0 },
      { frage: "Wie viele Beine hat eine Spinne?", antworten: ["6", "8", "10", "12"], korrekt: 1 },
      { frage: "Was ist 5 × 6?", antworten: ["30", "25", "35", "20"], korrekt: 0 },
      { frage: "Welche Farbe ergibt Blau + Gelb?", antworten: ["Grün", "Orange", "Violett", "Braun"], korrekt: 0 },
      { frage: "Wie viele Kontinente gibt es?", antworten: ["5", "6", "7", "8"], korrekt: 2 },
      { frage: "Welcher Planet ist der Sonne am nächsten?", antworten: ["Merkur", "Venus", "Mars", "Erde"], korrekt: 0 },
      { frage: "Wie viele Tage hat ein Schaltjahr?", antworten: ["365", "366", "364", "367"], korrekt: 1 },
      { frage: "Was ist die chemische Formel für Wasser?", antworten: ["H2O", "CO2", "O2", "NaCl"], korrekt: 0 },
      { frage: "Welches Tier ist das größte auf der Erde?", antworten: ["Elefant", "Blauwal", "Giraffe", "Hai"], korrekt: 1 },
      { frage: "Wer malte die Mona Lisa?", antworten: ["Da Vinci", "Van Gogh", "Picasso", "Michelangelo"], korrekt: 0 },
      { frage: "Wie viele Zähne hat ein Erwachsener normalerweise?", antworten: ["28", "30", "32", "36"], korrekt: 2 },
      { frage: "Wie heißt der höchste Berg der Erde?", antworten: ["Mount Everest", "K2", "Zugspitze", "Mont Blanc"], korrekt: 0 }
    ];

    let spielCode = "";
    let aktuellerIndex = 0;
    let punkte = 0;
    let frageReihenfolge = [];
    let spielerID = "" + Math.floor(Math.random() * 1000000);

    function mischeFragen() {
      frageReihenfolge = [...Array(quizFragen.length).keys()].sort(() => Math.random() - 0.5).slice(0, 10);
    }

    document.getElementById("erstellenBtn").onclick = () => {
      spielCode = Math.floor(100000 + Math.random() * 900000).toString();
      document.getElementById("codeAnzeige").innerText = `Code: ${spielCode}`;
      mischeFragen();
      set(ref(db, `spiele/${spielCode}`), {
        fragenIndex: 0,
        reihenfolge: frageReihenfolge,
        spieler: {}
      });
      startQuiz();
    };

    document.getElementById("beitretenBtn").onclick = () => {
      spielCode = document.getElementById("codeInput").value;
      if (!spielCode.match(/^\d{6}$/)) {
        alert("Bitte gib einen gültigen 6-stelligen Code ein.");
        return;
      }
      onValue(ref(db, `spiele/${spielCode}`), snapshot => {
        const daten = snapshot.val();
        if (daten && typeof daten.fragenIndex === 'number') {
          aktuellerIndex = daten.fragenIndex;
          frageReihenfolge = daten.reihenfolge || [];
          zeigeFrage();
        }
        const punkteAnzeigen = daten.spieler || {};
        let resultHTML = '<h2 class="text-lg font-bold mb-2">Bestenliste</h2><ul>';
        for (let [spieler, p] of Object.entries(punkteAnzeigen)) {
          resultHTML += `<li>${spieler}: ${p} Punkte</li>`;
        }
        resultHTML += '</ul>';
        document.getElementById("ranking").innerHTML = resultHTML;
      });
      startQuiz();
    };

    function startQuiz() {
      document.getElementById("setup").style.display = "none";
      document.getElementById("quiz").style.display = "block";
      zeigeFrage();
    }

    function zeigeFrage() {
      const frageIndex = frageReihenfolge[aktuellerIndex];
      const frageObj = quizFragen[frageIndex];
      if (!frageObj) return;
      document.getElementById("frage").innerText = frageObj.frage;
      const container = document.getElementById("antworten");
      container.innerHTML = "";
      frageObj.antworten.forEach((a, i) => {
        const btn = document.createElement("button");
        btn.className = "bg-blue-500 text-white p-2 m-2 rounded hover:bg-blue-700";
        btn.innerText = a;
        btn.onclick = () => pruefeAntwort(i);
        container.appendChild(btn);
      });
    }

    function pruefeAntwort(index) {
      const frageIndex = frageReihenfolge[aktuellerIndex];
      if (index === quizFragen[frageIndex].korrekt) punkte++;
      aktuellerIndex++;
      update(ref(db, `spiele/${spielCode}/spieler`), { [spielerID]: punkte });
      if (aktuellerIndex < frageReihenfolge.length) {
        set(ref(db, `spiele/${spielCode}`), {
          fragenIndex: aktuellerIndex,
          reihenfolge: frageReihenfolge,
          spieler: { [spielerID]: punkte }
        });
        zeigeFrage();
      } else {
        document.getElementById("quiz").innerHTML = `<h2 class='text-xl font-bold mb-4'>Fertig! Du hast ${punkte} von ${frageReihenfolge.length} Punkten.</h2><div id='ranking'></div>`;
      }
    }
  </script>
</head>
<body class="bg-gray-100 min-h-screen flex items-center justify-center">
  <div class="w-full max-w-lg p-6 bg-white rounded-lg shadow text-center">
    <h1 class="text-2xl font-bold mb-4">Multiplayer Quiz</h1>
    <div id="setup">
      <button id="erstellenBtn" class="bg-purple-500 text-white px-4 py-2 rounded mb-2">Spiel erstellen</button>
      <p id="codeAnzeige" class="mb-4 font-semibold"></p>
      <input id="codeInput" class="border px-4 py-2 rounded w-full mb-2" placeholder="Code eingeben">
      <button id="beitretenBtn" class="bg-green-500 text-white px-4 py-2 rounded w-full">Beitreten</button>
    </div>
    <div id="quiz" style="display:none">
      <p id="frage" class="text-lg font-semibold"></p>
      <div id="antworten" class="mt-4"></div>
      <div id="ranking" class="mt-6"></div>
    </div>
  </div>
</body>
</html>
