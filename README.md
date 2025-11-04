<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Controle de Gastos - Seliana Semijoia</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f0f0f0;
            color: #333;
            margin: 0;
            padding: 20px;
        }
        header {
            background-color: #6A0DAD;
            color: #FFD700;
            text-align: center;
            padding: 20px;
            border-radius: 10px;
        }
        .container {
            max-width: 1000px;
            margin: 20px auto;
            background-color: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(0,0,0,0.1);
        }
        form {
            margin-bottom: 20px;
        }
        label {
            display: block;
            margin: 10px 0 5px;
        }
        input, select, button {
            width: 100%;
            padding: 10px;
            margin-bottom: 10px;
            border: 1px solid #ccc;
            border-radius: 5px;
        }
        button {
            background-color: #6A0DAD;
            color: #FFD700;
            border: none;
            cursor: pointer;
        }
        button:hover {
            background-color: #FFD700;
            color: #6A0DAD;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 20px;
        }
        th, td {
            border: 1px solid #ddd;
            padding: 8px;
            text-align: left;
        }
        th {
            background-color: #6A0DAD;
            color: #FFD700;
        }
        .summary {
            margin-top: 20px;
            padding: 10px;
            border-radius: 5px;
            text-align: center;
            font-weight: bold;
        }
        .profit {
            background-color: #d4edda;
            color: #155724;
        }
        .loss {
            background-color: #f8d7da;
            color: #721c24;
        }
        .chart-container {
            width: 100%;
            margin: 20px 0;
        }
        #login {
            display: block;
        }
        #register {
            display: none;
        }
        #main {
            display: none;
        }
        .tab {
            display: inline-block;
            margin: 10px;
            cursor: pointer;
            padding: 10px;
            background-color: #6A0DAD;
            color: #FFD700;
            border-radius: 5px;
        }
        .tab.active {
            background-color: #FFD700;
            color: #6A0DAD;
        }
    </style>
</head>
<body>
    <header>
        <h1>Seliana Semijoia - Controle de Gastos e Vendas</h1>
    </header>
    <div id="login" class="container">
        <div class="tab active" onclick="showTab('login')">Login</div>
        <div class="tab" onclick="showTab('register')">Registrar Vendedor</div>
        <div id="loginForm">
            <h2>Login</h2>
            <form id="loginFormEl">
                <label for="username">Usuário:</label>
                <input type="text" id="username" placeholder="Ex: vendedor1 ou admin">
                <label for="password">Senha:</label>
                <input type="password" id="password" placeholder="Ex: senha123 ou admin123">
                <button type="submit">Entrar</button>
            </form>
        </div>
        <div id="registerForm">
            <h2>Registrar Novo Vendedor</h2>
            <form id="registerFormEl">
                <label for="newUsername">Novo Usuário:</label>
                <input type="text" id="newUsername" placeholder="Ex: vendedor3">
                <label for="newPassword">Senha:</label>
                <input type="password" id="newPassword" placeholder="Ex: senha123">
                <button type="submit">Registrar</button>
            </form>
        </div>
    </div>
    <div id="main" class="container">
        <h2 id="panelTitle">Painel do Vendedor</h2>
        <form id="form">
            <label for="type">Tipo:</label>
            <select id="type">
                <option value="venda">Venda</option>
                <option value="gasto">Gasto (Compra/Conta)</option>
            </select>
            <label for="vendedor">Vendedor (para vendas):</label>
            <input type="text" id="vendedor" readonly value="">
            <label for="metodo">Método de Pagamento (para vendas):</label>
            <select id="metodo">
                <option value="dinheiro">Dinheiro</option>
                <option value="pix">PIX</option>
                <option value="debito">Débito</option>
                <option value="credito">Crédito</option>
            </select>
            <label for="valor">Valor (R$):</label>
            <input type="number" id="valor" step="0.01" placeholder="Ex: 100.50">
            <label for="data">Data:</label>
            <input type="date" id="data">
            <button type="submit">Registrar</button>
        </form>

        <h2>Gráficos Mensais</h2>
        <div class="chart-container">
            <canvas id="salesChart"></canvas>
        </div>
        <div class="chart-container">
            <canvas id="profitChart"></canvas>
        </div>

        <h2>Registros do Mês</h2>
        <table id="table">
            <thead>
                <tr>
                    <th>Data</th>
                    <th>Tipo</th>
                    <th>Vendedor</th>
                    <th>Método</th>
                    <th>Valor (R$)</th>
                </tr>
            </thead>
            <tbody id="tbody">
                <!-- Registros serão adicionados aqui -->
            </tbody>
        </table>

        <h2>Resumo Mensal</h2>
        <div id="summary" class="summary">
            <!-- Cálculos aparecerão aqui -->
        </div>
    </div>

    <script>
        const login = document.getElementById('login');
        const main = document.getElementById('main');
        const loginFormEl = document.getElementById('loginFormEl');
        const registerFormEl = document.getElementById('registerFormEl');
        const form = document.getElementById('form');
        const tbody = document.getElementById('tbody');
        const summary = document.getElementById('summary');
        const vendedorInput = document.getElementById('vendedor');
        const panelTitle = document.getElementById('panelTitle');
        let currentUser = null;
        let isAdmin = false;
        let users = JSON.parse(localStorage.getItem('users')) || { admin: 'admin123' }; // Admin pré-definido
        let registros = JSON.parse(localStorage.getItem('registros')) || [];
        const metaVendas = 8000;

        // Mostrar aba
        function showTab(tab) {
            document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
            document.querySelector(`[onclick="showTab('${tab}')"]`).classList.add('active');
            document.getElementById('loginForm').style.display = tab === 'login' ? 'block' : 'none';
            document.getElementById('registerForm').style.display = tab === 'register' ? 'block' : 'none';
        }

        // Registro
        registerFormEl.addEventListener('submit', (e) => {
            e.preventDefault();
            const newUsername = document.getElementById('newUsername').value;
            const newPassword = document.getElementById('newPassword').value;
            if (users[newUsername]) {
                alert('Usuário já existe!');
                return;
            }
            users[newUsername] = newPassword;
            localStorage.setItem('users', JSON.stringify(users));
            alert('Registrado com sucesso! Faça login.');
            showTab('login');
        });

        // Login
        loginFormEl.addEventListener('submit', (e) => {
            e.preventDefault();
            const username = document.getElementById('username').value;
            const password = document.getElementById('password').value;
            if (users[username] === password) {
                currentUser = username;
                isAdmin = username === 'admin';
                vendedorInput.value = isAdmin ? '' : currentUser;
                panelTitle.textContent = isAdmin ? 'Painel do Administrador' : 'Painel do Vendedor';
                login.style.display = 'none';
                main.style.display = 'block';
                loadRegistros();
            } else {
                alert('Usuário ou senha incorretos!');
            }
        });

        // Carregar registros
        function loadRegistros() {
            tbody.innerHTML = '';
            const filteredRegistros = isAdmin ? registros : registros.filter(r => r.vendedor === currentUser);
            filteredRegistros.forEach(reg => {
                const row = tbody.insertRow();
                row.insertCell(0).textContent = reg.data;
                row.insertCell(1).textContent = reg.type;
                row.insertCell(2).textContent = reg.vendedor || '-';
                row.insertCell(3).textContent = reg.metodo || '-';
                row.insertCell(4).textContent = reg.valor.toFixed(2);
            });
            calculateSummary(filteredRegistros);
            updateCharts(filteredRegistros);
        }

        // Calcular resumo
        function calculateSummary(filteredRegistros) {
            const now = new Date();
            const currentMonth = now.getMonth();
            const currentYear = now.getFullYear();
            let totalVendas = 0;
            let totalGastos = 0;
            let vendasPorVendedor = {};
            let vendasPorMetodo = { dinheiro: 0, pix: 0, debito: 0, credito: 0 };

            filteredRegistros.forEach(reg => {
                const regDate = new Date(reg.data);
                if (regDate.getMonth() === currentMonth && regDate.getFullYear() === currentYear) {
                    if (reg.type === 'venda') {
                        totalVendas += reg.valor;
                        if (reg.vendedor) {
                            vendasPorVendedor[reg.vendedor] = (vendasPorVendedor[reg.vendedor] || 0) + reg.valor;
                        }
                        if (reg.metodo) {
                            vendasPorMetodo[reg.metodo] += reg.valor;
                        }
                    } else {
                        totalGastos += reg.valor;
                    }
                }
            });

            const lucro = totalVendas - totalGastos;
            const metaAtingida = totalVendas >= metaVendas;
            const contasPagas = totalVendas >= totalGastos;
            const status = lucro >= 0 ? 'no verde' : 'no vermelho';
            const className = lucro >= 0 ? 'profit' : 'loss';

            summary.className = `summary ${className}`;
            summary.innerHTML = `
                <p>Total Vendas: R$ ${totalVendas.toFixed(2)} (Meta: R$ ${metaVendas.toFixed(2)} - ${metaAtingida ? 'Atingida' : 'Não Atingida'})</p>
                <p>Total Gastos (incluindo contas): R$ ${totalGastos.toFixed(2)}</p>
                <p>Lucro: R$ ${lucro.toFixed(2)} (${status})</p>
                <p>Contas Pagas: ${contasPagas ? 'Sim' : 'Não'} (Vendas >= Gastos)</p>
                ${isAdmin ? `<p>Vendas por Vendedor: ${Object.entries(vendasPorVendedor).map(([v, val]) => `${v}: R$ ${val.toFixed(2)}`).join(', ')}</p>` : ''}
                <p>Vendas por Método: Dinheiro: R$ ${vendasPorMetodo.dinheiro.toFixed(2)}, PIX: R$ ${vendasPorMetodo.pix.toFixed(2)}, Débito: R$ ${vendasPorMetodo.debito.toFixed(2)}, Crédito: R$ ${vendasPorMetodo.credito.toFixed(2)}</p>
            `;
        }

        // Gráficos
        let salesChart, profitChart;
        function updateCharts(filteredRegistros) {
            const now = new Date();
            const currentMonth = now.getMonth();
            const currentYear = now.getFullYear();
            const daysInMonth = new Date(currentYear, currentMonth + 1, 0).getDate();
            const salesData = Array(daysInMonth).fill(0);
            const profitData = Array(daysInMonth).fill(0);

            filteredRegistros.forEach(reg => {
                const regDate = new Date(reg.data);
                if (regDate.getMonth() === currentMonth && regDate.getFullYear() === currentYear) {
                    const day = regDate.getDate() - 1;
                    if (reg.type === 'venda') {
                        salesData[day] += reg.valor;
                        profitData[day] += reg.valor;
                    } else {
                        profitData[day] -= reg.valor;
                    }
                }
            });

            if (salesChart) salesChart.destroy();
            if (profitChart) profitChart.destroy();

            salesChart = new Chart(document.getElementById('salesChart'), {
                type: 'bar',
                data: {
                    labels: Array.from({length: daysInMonth}, (_, i) => i + 1),
                    datasets: [{
                        label: 'Vendas Diárias (R$)',
                        data: salesData,
                        backgroundColor: '#6A0DAD'
                    }]
                }
            });

            profitChart = new Chart(document.getElementById('profitChart'), {
                type: 'line',
                data: {
                    labels: Array.from({length: daysInMonth}, (_, i) => i + 1),
                    datasets: [{
                        label: 'Lucro Diário (R$)',
                        data: profitData,
                        borderColor: '#FFD700',
                        fill: false
                    }]
                }
            });
        }

        // Registrar novo item
        form.addEventListener('submit', (e) => {
            e.preventDefault();
            const type = document.getElementById('type').value;
            const vendedor = isAdmin ? document.getElementById('vendedor').value : currentUser;
            const metodo = document.getElementById('metodo').value;
            const valor = parseFloat(document.getElementById('valor').value);
            const data = document.getElementById('data').value;

            if (!valor || !data || (!isAdmin && !vendedor)) return alert('Preencha todos os campos obrigatórios!');

            registros.push({ type, vendedor, metodo, valor, data });
            localStorage.setItem('registros', JSON.stringify(registros));
            loadRegistros();
            form.reset();
        });
    </script>
</body>
</html>
