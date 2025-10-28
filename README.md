<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Zufällige Gruppen</title>
<style>
  body {
    font-family: Arial, sans-serif;
    background: #f5f5f5;
    margin: 0;
    padding: 20px;
  }
  h1 {
    text-align: center;
  }
  .panel {
    background: #fff;
    border-radius: 10px;
    padding: 15px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
    margin-bottom: 20px;
    max-width: 600px;
    margin-left: auto;
    margin-right: auto;
  }
  input, button {
    padding: 6px;
    margin: 4px 0;
    border: 1px solid #ccc;
    border-radius: 5px;
  }
  button {
    cursor: pointer;
    background-color: #2196F3;
    color: white;
    border: none;
  }
  button:hover {
    background-color: #1976D2;
  }
  .person {
    display: flex;
    justify-content: space-between;
    align-items: center;
    background: #eee;
    padding: 6px 10px;
    margin: 4px 0;
    border-radius: 5px;
  }
  .present {
    background: #c8f7c5;
  }
  .absent {
    background: #f7c5c5;
  }
  .groups {
    display: flex;
    flex-wrap: wrap;
    gap: 20px;
    justify-content: center;
  }
  .group {
    background: #fff;
    border-radius: 10px;
    padding: 10px;
    box-shadow: 0 1px 5px rgba(0,0,0,0.1);
    width: 250px;
  }
  .group h3 {
    text-align: center;
    border-bottom: 1px solid #ccc;
    padding-bottom: 6px;
  }
</style>
</head>
<body>

<h1>🎲 Zufällige Gruppeneinteilung</h1>

<div class="panel">
  <h3>➕ Person hinzufügen</h3>
  <input type="text" id="personName" placeholder="Name eingeben">
  <button onclick="addPerson()">Hinzufügen</button>
</div>

<div class="panel">
  <h3>👥 Alle Personen</h3>
  <div id="personList"></div>
</div>

<div class="panel">
  <h3>🧩 Zufällige Gruppen bilden</h3>
  <label>Wie viele Gruppen?</label>
  <input type="number" id="groupCount" min="2" value="2" style="width:60px;">
  <button onclick="shuffleGroups()">Gruppen zufällig bilden</button>
</div>

<div id="groupsContainer" class="groups"></div>

<script>
let persons = JSON.parse(localStorage.getItem("persons")) || [];

function save() {
  localStorage.setItem("persons", JSON.stringify(persons));
}

function renderPersons() {
  const list = document.getElementById("personList");
  list.innerHTML = "";
  persons.forEach((p, i) => {
    let div = document.createElement("div");
    div.className = "person " + (p.present ? "present" : "absent");
    div.innerHTML = `
      <span>${p.name}</span>
      <div>
        <button onclick="togglePresence(${i})">${p.present ? "Abwesend" : "Anwesend"}</button>
        <button onclick="removePerson(${i})" style="background:#e74c3c;">❌</button>
      </div>
    `;
    list.appendChild(div);
  });
}

function addPerson() {
  const name = document.getElementById("personName").value.trim();
  if (!name) return alert("Bitte einen Namen eingeben!");
  persons.push({ name, present: false });
  document.getElementById("personName").value = "";
  save();
  renderPersons();
}

function removePerson(i) {
  if (confirm("Person löschen?")) {
    persons.splice(i, 1);
    save();
    renderPersons();
  }
}

function togglePresence(i) {
  persons[i].present = !persons[i].present;
  save();
  renderPersons();
}

function shuffleGroups() {
  const count = parseInt(document.getElementById("groupCount").value);
  if (count < 1) return alert("Bitte eine gültige Gruppenzahl eingeben!");

  const active = persons.filter(p => p.present);
  if (active.length < count) return alert("Zu wenige anwesende Personen für so viele Gruppen!");

  // Shuffle mit Fisher-Yates
  let shuffled = [...active];
  for (let i = shuffled.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
  }

  // Aufteilen in Gruppen
  const groups = Array.from({ length: count }, () => []);
  shuffled.forEach((person, index) => {
    groups[index % count].push(person);
  });

  renderGroups(groups);
}

function renderGroups(groups) {
  const container = document.getElementById("groupsContainer");
  container.innerHTML = "";
  groups.forEach((g, i) => {
    const div = document.createElement("div");
    div.className = "group";
    div.innerHTML = `<h3>Gruppe ${i + 1}</h3>`;
    g.forEach(p => {
      const pDiv = document.createElement("div");
      pDiv.className = "person present";
      pDiv.textContent = p.name;
      div.appendChild(pDiv);
    });
    container.appendChild(div);
  });
}

renderPersons();
</script>

</body>
</html>
