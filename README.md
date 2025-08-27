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
      <button class="logout-btn" onclick="sairApp()">Sair do App</button>
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

    function sairApp() {
      location.reload(); // Ou window.location.href = "https://sua-pagina-inicial.com";
    }

    carregarAgendamentos();
    atualizarListaTarefas();
  </script>
</body>
</html>
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Agendamento de Tarefas</title>
  <style>
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }

    body {
      font-family: 'Segoe UI', sans-serif;
      background: #121212;
      color: #ffffff;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
      text-align: center;
    }

    h1 {
      font-size: 2rem;
      font-weight: bold;
      color: #f9f9f9;
    }
  </style>
</head>
<body>
  <h1>Agendamento de Tarefas</h1>
</body>
</html>

