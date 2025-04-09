# Alice-story
from zipfile import ZipFile
import os

# Создание структуры проекта
project_dir = "/mnt/data/alisa_story"
os.makedirs(project_dir, exist_ok=True)

html_code = '''<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <title>История Алисы</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #f8f8ff;
      max-width: 700px;
      margin: 40px auto;
      padding: 20px;
      background: white;
      border-radius: 10px;
      box-shadow: 0 0 10px rgba(0,0,0,0.1);
    }
    button {
      margin: 5px;
      padding: 10px;
      font-size: 16px;
      background: #d7b0ff;
      border: none;
      border-radius: 5px;
      cursor: pointer;
    }
    button:hover {
      background: #c69aff;
    }
    #stats {
      font-size: 14px;
      margin-top: 10px;
    }
    #diary {
      background: #f1eaff;
      padding: 10px;
      margin-top: 20px;
      border-radius: 5px;
      font-style: italic;
    }
  </style>
</head>
<body>
  <h1>История Алисы</h1>
  <div id="gameText"></div>
  <div id="choices"></div>

  <button onclick="saveGame()">Сохранить</button>
  <button onclick="loadGame()">Загрузить</button>

  <div id="stats"></div>
  <div id="diary"><strong>Дневник Алисы:</strong><br><span id="diaryText">Пусто...</span></div>

  <script>
    let stats = {
      vikaTrust: 0,
      kiraAffinity: 0,
      stress: 0
    };

    let diary = "Пусто...";
    let currentIndex = 0;

    const story = [
      {
        text: "Алиса снова видит, как Вика обнимается с Артёмом возле школы. Больно. Что делать?",
        choices: [
          { text: "Отвернуться и уйти", next: 1, effect: () => { stats.stress += 1; updateDiary("Иногда мне кажется, что я ей никто..."); } },
          { text: "Подойти и поздороваться", next: 2, effect: () => { stats.stress += 2; updateDiary("Я пыталась улыбнуться, но сердце сжималось..."); } }
        ]
      },
      {
        text: "Ты ускользаешь, но слёзы наворачиваются. Позже тебя находит Кира — тихая девочка из параллели.",
        choices: [
          { text: "Поговорить с Кирой", next: 3, effect: () => { stats.kiraAffinity += 1; } },
          { text: "Сказать, что не хочешь говорить", next: 4 }
        ]
      },
      {
        text: "Артём смотрит на тебя с подозрением, но Вика улыбается. «Привет, Алис».",
        choices: [
          { text: "Сделать вид, что всё нормально", next: 5 },
          { text: "Посмотреть Вике в глаза и уйти", next: 1 }
        ]
      },
      {
        text: "Кира говорит: «Ты кажешься грустной. Я рядом, если что».",
        choices: [
          { text: "Поблагодарить её", next: 6, effect: () => { stats.kiraAffinity += 2; updateDiary("Кира... возможно, я недооценила её."); } },
          { text: "Спросить, не хочет ли она прогуляться", next: 7, effect: () => { stats.kiraAffinity += 3; } }
        ]
      },
      {
        text: "Ты осталась одна. Мысли путаются.",
        choices: [
          { text: "Записать всё в дневник", next: 6, effect: () => { updateDiary("Я боюсь. Боюсь потерять Вику. Боюсь сказать правду."); } }
        ]
      },
      {
        text: "Позже Вика находит тебя. Она говорит: «Ты всё хорошо?»",
        choices: [
          { text: "Сказать, что ты устала", next: 6 },
          { text: "Признаться ей", next: 8, effect: () => { stats.vikaTrust += 2; updateDiary("Я сказала ей... всё. Не знаю, что будет дальше."); } }
        ]
      },
      {
        text: "Проходит день. Ты чувствуешь, как меняется атмосфера. Что ты хочешь?",
        choices: [
          { text: "Пойти на прогулку с Кирой", next: 9 },
          { text: "Позвонить Вике", next: 10 }
        ]
      },
      {
        text: "Кира улыбается и берёт тебя за руку. Вы гуляете под вечерним небом.",
        choices: [
          { text: "Поцеловать её", next: 11, effect: () => { stats.kiraAffinity += 5; updateDiary("Я сделала это. Кира — она особенная."); } },
          { text: "Просто быть рядом", next: 6 }
        ]
      },
      {
        text: "Вика молчит. Потом тихо говорит: «Мне тоже бывает сложно. Я не знаю, что делать с Артёмом...»",
        choices: [
          { text: "Спросить, любит ли она его", next: 12 },
          { text: "Сказать, что любишь её", next: 13, effect: () => { stats.vikaTrust += 3; updateDiary("Я не могу больше молчать. Я люблю её."); } }
        ]
      },
      {
        text: "Вы смеётесь с Кирой. В её глазах — тепло. Возможно, это начало чего-то настоящего.",
        choices: [
          { text: "Конец. Начать заново?", next: 0 }
        ]
      },
      {
        text: "Кира смотрит на тебя в ответ. Всё замедляется.",
        choices: [
          { text: "Конец. Начать заново?", next: 0 }
        ]
      },
      {
        text: "Вика смотрит в сторону. «Я... не уверена, что люблю его»",
        choices: [
          { text: "Сказать, что ты будешь рядом", next: 6 },
          { text: "Предложить расстаться с Артёмом", next: 14 }
        ]
      },
      {
        text: "Она смотрит прямо на тебя. «Ты правда это чувствуешь?»",
        choices: [
          { text: "Кивнуть", next: 15 },
          { text: "Сказать, что не ждёшь ответа", next: 6 }
        ]
      },
      {
        text: "Вика говорит, что ей нужно подумать. Но ты чувствуешь: вы теперь ближе.",
        choices: [
          { text: "Конец. Начать заново?", next: 0 }
        ]
      },
      {
        text: "Вика сдерживает слёзы. «Я тоже… чувствую это. Только с тобой — по-настоящему.»",
        choices: [
          { text: "Обнять её", next: 16 },
          { text: "Взять за руку", next: 16 }
        ]
      },
      {
        text: "Вы стоите, обнявшись. Мир замер. Всё остальное неважно.",
        choices: [
          { text: "Конец. Начать заново?", next: 0 }
        ]
      }
    ];

    function updateDiary(text) {
      diary = text;
      document.getElementById("diaryText").innerText = diary;
    }

    function showScene(index) {
      currentIndex = index;
      const scene = story[index];
      document.getElementById("gameText").innerHTML = `<p>${scene.text}</p>`;
      const choicesDiv = document.getElementById("choices");
      choicesDiv.innerHTML = '';
      scene.choices.forEach(choice => {
        const btn = document.createElement('button');
        btn.textContent = choice.text;
        btn.onclick = () => {
          if (choice.effect) choice.effect();
          showScene(choice.next);
          updateStats();
        };
        choicesDiv.appendChild(btn);
      });
    }

    function updateStats() {
      document.getElementById("stats").innerHTML = `
        <strong>Доверие Вики:</strong> ${stats.vikaTrust} | 
        <strong>Симпатия Киры:</strong> ${stats.kiraAffinity} | 
        <strong>Стресс:</strong> ${stats.stress}
      `;
    }

    function saveGame() {
      localStorage.setItem('saveIndex', currentIndex);
      localStorage.setItem('saveStats', JSON.stringify(stats));
      localStorage.setItem('saveDiary', diary);
      alert("Игра сохранена!");
    }

    function loadGame() {
      const savedIndex = localStorage.getItem('saveIndex');
      const savedStats = localStorage.getItem('saveStats');
      const savedDiary = localStorage.getItem('saveDiary');
      if (savedIndex !== null && savedStats && savedDiary) {
        currentIndex = parseInt(savedIndex);
        stats = JSON.parse(savedStats);
        diary = savedDiary;
        showScene(currentIndex);
        updateStats();
        document.getElementById("diaryText").innerText = diary;
      } else {
        alert("Сохранение не найдено.");
      }
    }

    showScene(currentIndex);
    updateStats();
  </script>
</body>
</html>
'''

# Сохраняем HTML файл
html_path = os.path.join(project_dir, "index.html")
with open(html_path, "w", encoding="utf-8") as f:
    f.write(html_code)

# Создаём ZIP
zip_path = "/mnt/data/alisa_story_game.zip"
with ZipFile(zip_path, 'w') as zipf:
    zipf.write(html_path, arcname="index.html")

zip_path
