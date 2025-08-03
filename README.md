# painel-passe-booyah
<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Painel Passe Booyah - ABG Distribuidora</title>
<style>
  body {
    background: #111;
    color: #fff;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    margin: 0;
    padding: 20px;
    text-align: center;
  }
  h1 {
    color: #ffcc00;
    margin-bottom: 15px;
  }
  input, select, button {
    width: 90%;
    max-width: 400px;
    padding: 12px;
    margin: 8px auto;
    border-radius: 8px;
    border: none;
    font-size: 1em;
    display: block;
  }
  button {
    background: #ffcc00;
    color: #111;
    font-weight: bold;
    cursor: pointer;
    transition: background 0.3s ease;
  }
  button:hover {
    background: #ffaa00;
  }
  .box {
    background: #222;
    padding: 20px;
    border-radius: 15px;
    max-width: 450px;
    margin: 0 auto 20px auto;
  }
  pre {
    background: #000;
    color: #0f0;
    text-align: left;
    max-width: 450px;
    margin: 0 auto;
    padding: 15px;
    border-radius: 10px;
    overflow-x: auto;
    height: 250px;
  }
  label {
    display: block;
    margin-top: 15px;
    font-weight: bold;
    color: #ffcc00;
  }
  @media (max-width: 500px) {
    body {
      padding: 10px;
    }
    input, select, button {
      width: 100%;
      max-width: 100%;
    }
    pre {
      height: 200px;
    }
  }
</style>
</head>
<body>

<h1>📦 Painel Passe Booyah</h1>

<div class="box">
  <label for="resellerKey">Reseller Key:</label>
  <input type="text" id="resellerKey" placeholder="Digite sua chave" autocomplete="off" />

  <label for="acao">Ação:</label>
  <select id="acao">
    <option value="enviar">Enviar Passe</option>
    <option value="verificar">Verificar Contas</option>
    <option value="adicionar">Adicionar Conta</option>
    <option value="remover">Remover Conta</option>
  </select>

  <div id="camposExtras"></div>

  <button onclick="executarAcao()">Executar</button>
</div>

<h3>Resultado:</h3>
<pre id="resultado">Aguardando ação...</pre>

<script>
  const API_BASE = "https://fwxpasses-freefire.squareweb.app";

  function salvarKey(key) {
    if (key) {
      localStorage.setItem('reseller_key', key);
    }
  }
  function carregarKey() {
    return localStorage.getItem('reseller_key') || '';
  }

  function camposExtras() {
    const acao = document.getElementById("acao").value;
    let html = '';

    if (acao === "enviar") {
      html += '<label for="clientID">ClientID do jogador:</label>';
      html += '<input type="text" id="clientID" placeholder="ClientID do jogador" autocomplete="off" />';
      html += '<label for="mensagem">Mensagem (opcional):</label>';
      html += '<input type="text" id="mensagem" placeholder="Mensagem personalizada" autocomplete="off" />';
    } else if (acao === "adicionar") {
      html += '<label for="uid">UID da conta Garena:</label>';
      html += '<input type="text" id="uid" placeholder="UID da conta" autocomplete="off" />';
      html += '<label for="senha">Senha da conta:</label>';
      html += '<input type="password" id="senha" placeholder="Senha da conta" autocomplete="off" />';
    } else if (acao === "remover") {
      html += '<label for="uid">UID da conta:</label>';
      html += '<input type="text" id="uid" placeholder="UID da conta" autocomplete="off" />';
      html += '<label for="senha">Senha (opcional):</label>';
      html += '<input type="password" id="senha" placeholder="Senha (opcional)" autocomplete="off" />';
    }
    document.getElementById("camposExtras").innerHTML = html;
  }

  document.getElementById("acao").addEventListener("change", camposExtras);

  function executarAcao() {
    const key = document.getElementById("resellerKey").value.trim();
    if (!key) {
      alert("Por favor, informe a Reseller Key.");
      return;
    }
    salvarKey(key);

    const acao = document.getElementById("acao").value;
    let url = "";
    let method = "POST";
    let data = {};

    if (acao === "enviar") {
      url = `${API_BASE}/api/enviar/passe`;
      const clientID = document.getElementById("clientID").value.trim();
      if (!clientID) {
        alert("Informe o ClientID do jogador.");
        return;
      }
      const mensagem = document.getElementById("mensagem").value.trim() || "Bom aproveito ao Passe Booyah!";
      data = { reseller_key: key, clientID, mensagem };
    } else if (acao === "verificar") {
      url = `${API_BASE}/api/contas/verificar?reseller_key=${encodeURIComponent(key)}`;
      method = "GET";
    } else if (acao === "adicionar") {
      url = `${API_BASE}/api/adicionar_conta`;
      const uid = document.getElementById("uid").value.trim();
      const senha = document.getElementById("senha").value.trim();
      if (!uid || !senha) {
        alert("Informe UID e senha para adicionar conta.");
        return;
      }
      data = { reseller_key: key, uid, password: senha };
    } else if (acao === "remover") {
      url = `${API_BASE}/api/remover_conta`;
      const uid = document.getElementById("uid").value.trim();
      if (!uid) {
        alert("Informe o UID para remover conta.");
        return;
      }
      const senha = document.getElementById("senha").value.trim();
      data = { reseller_key: key, uid };
      if (senha) data.password = senha;
    }

    document.getElementById("resultado").textContent = "Carregando...";

    fetch(url, {
      method,
      headers: { "Content-Type": "application/x-www-form-urlencoded" },
      body: method === "POST" ? new URLSearchParams(data) : null
    })
    .then(response => response.json())
    .then(json => {
      document.getElementById("resultado").textContent = JSON.stringify(json, null, 4);
    })
    .catch(err => {
      document.getElementById("resultado").textContent = "Erro: " + err.message;
    });
  }

  window.onload = () => {
    document.getElementById("resellerKey").value = carregarKey();
    camposExtras();
  };
</script>

</body>
</html>
