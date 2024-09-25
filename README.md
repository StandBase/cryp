<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Генераторы</title>
    <style>
        body { font-family: Arial, sans-serif; background-color: #f0f0f0; text-align: center; margin-top: 50px; }
        button, input { padding: 8px 16px; margin-top: 10px; border: none; border-radius: 5px; cursor: pointer; }
        .gradient-button { background: linear-gradient(90deg, #ff7e5f, #feb47b); color: white; }
        #authSection, #gameSection, #shopSection, #inventorySection { display: none; margin-top: 20px; background: white; padding: 20px; border-radius: 10px; }
    </style>
</head>
<body>
    <div id="authSection">
        <h2>Регистрация</h2>
        <input type="text" id="regLogin" placeholder="Логин" required>
        <input type="email" id="regEmail" placeholder="Почта" required>
        <input type="password" id="regPassword" placeholder="Пароль" required>
        <button class="gradient-button" id="registerButton">Зарегистрироваться</button>
        <h2>Вход</h2>
        <input type="text" id="loginEmail" placeholder="Логин/Почта" required>
        <input type="password" id="loginPassword" placeholder="Пароль" required>
        <button class="gradient-button" id="loginButton">Войти</button>
        <p id="authMessage" style="color: red;"></p>
    </div>

    <div id="gameSection">
        <h1>Ваш баланс: <span id="coinBalance">0</span></h1>
        <button class="gradient-button" id="increaseButton">Клик</button>
        <button class="gradient-button" id="shopButton">Перейти в магазин</button>
        <button class="gradient-button" id="inventoryButton">Инвентарь</button>
        <button class="gradient-button" id="logoutButton">Выйти</button>
    </div>

    <div id="shopSection">
        <h2>Магазин</h2>
        <div id="generatorList"></div>
        <button class="gradient-button" id="backToGameFromShopButton">Назад в игру</button>
    </div>

    <div id="inventorySection">
        <h2>Инвентарь</h2>
        <div id="inventoryList"></div>
        <button class="gradient-button" id="backToGameFromInventoryButton">Назад в игру</button>
    </div>

    <script>
        let currentUser;

        function showGameSection() {
            document.getElementById('authSection').style.display = 'none';
            document.getElementById('gameSection').style.display = 'block';
            loadGameData();
        }

        function loadGameData() {
            currentUser = JSON.parse(localStorage.getItem(localStorage.getItem('currentUser')));
            document.getElementById('coinBalance').textContent = currentUser.coinBalance;
        }

        function saveGameData() {
            localStorage.setItem(currentUser.email, JSON.stringify(currentUser));
        }

        function generateCoins() {
            currentUser.coinBalance += 1; // Генерация 1 коин каждую минуту
            document.getElementById('coinBalance').textContent = currentUser.coinBalance;
            saveGameData();
        }

        document.getElementById('registerButton').addEventListener('click', function() {
            const login = document.getElementById('regLogin').value;
            const email = document.getElementById('regEmail').value;
            const password = document.getElementById('regPassword').value;

            if (login && email && password) {
                if (localStorage.getItem(email)) {
                    document.getElementById('authMessage').textContent = 'Пользователь с такой почтой уже существует!';
                } else {
                    const user = {
                        login: login,
                        email: email,
                        password: password,
                        coinBalance: 0,
                        generators: [{ name: 'Базовый Генератор', income: 1 }]
                    };
                    localStorage.setItem(email, JSON.stringify(user));
                    document.getElementById('authMessage').textContent = 'Регистрация успешна! Войдите в систему.';
                }
            } else {
                document.getElementById('authMessage').textContent = 'Заполните все поля!';
            }
        });

        document.getElementById('loginButton').addEventListener('click', function() {
            const loginEmail = document.getElementById('loginEmail').value;
            const password = document.getElementById('loginPassword').value;
            const user = JSON.parse(localStorage.getItem(loginEmail));

            if (user && user.password === password) {
                localStorage.setItem('currentUser', user.email);
                showGameSection();
            } else {
                document.getElementById('authMessage').textContent = 'Неверный логин или пароль!';
            }
        });

        document.getElementById('increaseButton').addEventListener('click', function() {
            currentUser.coinBalance += 1;
            document.getElementById('coinBalance').textContent = currentUser.coinBalance;
            saveGameData();
        });

        document.getElementById('shopButton').addEventListener('click', function() {
            document.getElementById('gameSection').style.display = 'none';
            document.getElementById('shopSection').style.display = 'block';
            loadGenerators();
        });

        document.getElementById('inventoryButton').addEventListener('click', function() {
            document.getElementById('gameSection').style.display = 'none';
            document.getElementById('inventorySection').style.display = 'block';
            loadInventory();
        });

        function loadGenerators() {
            const generatorList = document.getElementById('generatorList');
            generatorList.innerHTML = '';

            const generators = [
                { name: 'Генератор 1', cost: 10, income: 1 },
                { name: 'Генератор 2', cost: 50, income: 5 },
                { name: 'Генератор 3', cost: 100, income: 10 }
            ];

            generators.forEach((gen) => {
                const button = document.createElement('button');
                button.textContent = `${gen.name} - ${gen.cost} монет`;
                button.className = 'gradient-button';
                button.addEventListener('click', function() {
                    if (currentUser.coinBalance >= gen.cost) {
                        currentUser.coinBalance -= gen.cost;
                        currentUser.generators.push(gen);
                        document.getElementById('coinBalance').textContent = currentUser.coinBalance;
                        saveGameData();
                        alert(`Вы купили ${gen.name}!`);
                    } else {
                        alert('Недостаточно коина для покупки!');
                    }
                });
                generatorList.appendChild(button);
            });
        }

        function loadInventory() {
            const inventoryList = document.getElementById('inventoryList');
            inventoryList.innerHTML = '';

            if (currentUser.generators.length > 0) {
                currentUser.generators.forEach((gen) => {
                    const item = document.createElement('div');
                    item.textContent = `${gen.name} (Фармит ${gen.income} коин в минуту)`;
                    inventoryList.appendChild(item);
                });
            } else {
                inventoryList.textContent = 'У вас нет генераторов.';
            }
        }

        document.getElementById('logoutButton').addEventListener('click', function() {
            localStorage.removeItem('currentUser');
            document.getElementById('authSection').style.display = 'block';
            document.getElementById('gameSection').style.display = 'none';
        });

        document.getElementById('backToGameFromShopButton').addEventListener('click', function() {
            document.getElementById('shopSection').style.display = 'none';
            document.getElementById('gameSection').style.display = 'block';
        });

        document.getElementById('backToGameFromInventoryButton').addEventListener('click', function() {
            document.getElementById('inventorySection').style.display = 'none';
            document.getElementById('gameSection').style.display = 'block';
        });

        setInterval(generateCoins, 60000); // Генерация 1 коин каждую минуту
    </script>
</body>
</html>
