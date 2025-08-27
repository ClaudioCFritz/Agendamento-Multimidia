<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Agendamento de Tarefas</title>
  <style>
    * {
      box-sizing: border-box;
    }

    body {
      margin: 0;
      padding: 0;
      font-family: 'Segoe UI', sans-serif;
      background: #121212;
      color: #ffffff;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
    }

    .container {
      width: 100%;
      max-width: 500px;
      background: #1f1f1f;
      border-radius: 10px;
      box-shadow: 0 8px 20px rgba(0, 0, 0, 0.3);
      padding: 30px;
      margin: 20px;
    }

    h2 {
      text-align: center;
      color: #f9f9f9;
    }

    .tela {
      display: none;
    }

    .ativa {
      display: block;
    }

    button {
      width: 100%;
      padding: 12px;
      margin-top: 16px;
      border: none;
      border-radius: 8px;
      font-size: 1rem;
      background-color: #03dac5;
      color: #000;
      font-weight: bold;
      cursor: pointer;
    }

    button:hover {
      background-color: #018786;
    }

    ul {
      list-style: none;
      padding: 0;
      margin-top: 20px;
    }

    li {
      background: #2c2c2c;
      margin: 8px 0;
      padding: 10px;
      border-radius: 8px;
      display: flex;
      justify-content: space-between;
      align-items: center;
      flex-wrap: wrap;
    }

    .task-label {
      font-size: 1rem;
      font-weight: 500;
      text-align: left;
    }

    .task-checkbox {
      transform: scale(1.3);
    }

    .delete-icon {
      margin-left: 10px;
      cursor: pointer;
      color: #ff6b6b;
      font-weight: bold;
    }

    .delete-icon:hover {
      color: #ff4d4d;
    }

    .back-btn {
      background-color: #ff6b6b;
      color: white;
    }

    .back-btn:hover {
      background-color: #c0392b;
    }

    .form-agendar {
      margin-top: 10px;
      width: 100%;
    }

    .form-agendar input,
    .form-agendar select {
      width: 100%;
      padding: 8px;
      margin: 6px 0;
      border: 1px solid #ccc;
      border-radius: 6px;
      font-size: 0.9rem;
    }
  </style>
</head>
<body>
  <div class="container">
    <!-- Tela principal -->
    <div id="tela1" class="tela ativa">
      <h2>Agendar Tarefas</h2>
      <ul id="listaTarefas"></ul>
      <button onclick="mostrarAgendamentos(); mudarTela('tela2')">Ver Agendamentos</button>
    </div>

    <!-- Tela de agendamentos -->
    <div id="tela2" class="tela">
      <h2>Agendamentos Atuais</h2>
      <ul id="listaAgendamentos"></ul>
      <button class="back-btn" onclick="mudarTela('tela1')">Voltar ao Início</button>
    </div>
  </div>

  <script>
    const tarefas = ["DATASHOW", "LIVE", "MESA DE SOM", "APOIO"];
    const diasValidos = ["Domingo","Segunda","Terça","Quarta","Quinta","Sexta","Sábado"];
    let agendamentos = [];

    function carregarAgendamentos() {
      const data = localStorage.getItem("agendamentos");
      agendamentos = data ? JSON.parse(data) : [];
    }

    function salvarAgendamentos() {
      localStorage.setItem("agendamentos", JSON.stringify(agendamentos));
    }

    function limparAgendamentosExpirados() {
      const agora = Date.now();
      agendamentos = agendamentos.filter(ag => {
        const agTime = new Date(ag.timestamp).getTime();
        return agora < agTime + 4 * 60 * 60 * 1000;
      });
      salvarAgendamentos();
    }

    function atualizarListaTarefas() {
      limparAgendamentosExpirados();
      const lista = document.getElementById("listaTarefas");
      lista.innerHTML = "";
      tarefas.forEach((nomeTarefa) => {
        const li = document.createElement("li");
        const label = document.createElement("span");
        label.className = "task-label";
        label.textContent = nomeTarefa;
        const checkbox = document.createElement("input");
        checkbox.type = "checkbox";
        checkbox.className = "task-checkbox";
        const estaAgendada = agendamentos.some(ag => ag.tarefa === nomeTarefa);
        checkbox.checked = estaAgendada;
        checkbox.disabled = estaAgendada;
        li.appendChild(label);
        li.appendChild(checkbox);
        lista.appendChild(li);

        checkbox.onchange = function () {
          if (checkbox.checked) {
            const form = criarFormAgendamento(nomeTarefa, checkbox);
            li.appendChild(form);
          }
        };
      });
    }

    function criarFormAgendamento(nomeTarefa, checkbox) {
      const form = document.createElement("div");
      form.className = "form-agendar";
      const inputNome = document.createElement("input");
      inputNome.placeholder = "Seu nome";

      const selectDia = document.createElement("select");
      diasValidos.forEach(dia => {
        const opt = document.createElement("option");
        opt.value = dia;
        opt.textContent = dia;
        selectDia.appendChild(opt);
      });

      const selectPeriodo = document.createElement("select");
      ["Manhã", "Noite"].forEach(p => {
        const opt = document.createElement("option");
        opt.value = p;
        opt.textContent = p;
        selectPeriodo.appendChild(opt);
      });

      const btnConfirmar = document.createElement("button");
      btnConfirmar.textContent = "Confirmar Agendamento";
      btnConfirmar.onclick = function () {
        if (!inputNome.value) {
          alert("Digite seu nome!");
          return;
        }

        const diaSemana = selectDia.value;
        const periodo = selectPeriodo.value;
        const hoje = new Date();
        const indiceHoje = hoje.getDay();
        const indiceEscolhido = diasValidos.indexOf(diaSemana);
        let diasParaSomar = indiceEscolhido - indiceHoje;
        if (diasParaSomar <= 0) diasParaSomar += 7;

        const dataAgendada = new Date();
        dataAgendada.setDate(hoje.getDate() + diasParaSomar);
        if (periodo === "Manhã") {
          dataAgendada.setHours(9, 0, 0);
        } else {
          dataAgendada.setHours(diaSemana === "Domingo" ? 18 : 19, 0, 0);
        }

        const novoAgendamento = {
          tarefa: nomeTarefa,
          diaSemana,
          periodo,
          nome: inputNome.value,
          dataHora: dataAgendada.toLocaleString("pt-BR"),
          timestamp: dataAgendada.getTime()
        };

        agendamentos.push(novoAgendamento);
        salvarAgendamentos();
        alert("Tarefa agendada com sucesso!");
        atualizarListaTarefas();
      };

      form.appendChild(inputNome);
      form.appendChild(selectDia);
      form.appendChild(selectPeriodo);
      form.appendChild(btnConfirmar);
      return form;
    }

    function mostrarAgendamentos() {
      limparAgendamentosExpirados();
      const lista = document.getElementById("listaAgendamentos");
      lista.innerHTML = "";

      if (agendamentos.length === 0) {
        lista.innerHTML = "<li>Nenhum agendamento disponível.</li>";
        return;
      }

      agendamentos.forEach((ag, index) => {
        const li = document.createElement("li");
        const linha1 = document.createElement("div");
        linha1.textContent = `${ag.nome} – ${ag.tarefa}`;
        const excluir = document.createElement("span");
        excluir.className = "delete-icon";
        excluir.innerHTML = "🗑️";
        excluir.title = "Excluir agendamento";
        excluir.onclick = function () {
          if (confirm("Tem certeza que deseja excluir este agendamento?")) {
            agendamentos.splice(index, 1);
            salvarAgendamentos();
            mostrarAgendamentos();
            atualizarListaTarefas();
          }
        };
        linha1.appendChild(excluir);
        const linha2 = document.createElement("div");
        linha2.textContent = `📅 ${ag.diaSemana} (${ag.periodo}) – 🕒 ${ag.dataHora}`;
        li.appendChild(linha1);
        li.appendChild(linha2);
        lista.appendChild(li);
      });
    }

    function mudarTela(telaId) {
      document.querySelectorAll(".tela").forEach(tela => tela.classList.remove("ativa"));
      document.getElementById(telaId).classList.add("ativa");
    }

    carregarAgendamentos();
    atualizarListaTarefas();
  </script>
</body>
</html>
