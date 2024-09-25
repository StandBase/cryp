<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Баланс с регистрацией и входом</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f0f0f0;
            color: #333;
            text-align: center;
            margin-top: 50px;
        }
        button, input {
            padding: 8px 16px;
            font-size: 14px;
            margin-top: 10px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            transition: background-color 0.3s, transform 0.2s;
        }
        .gradient-button {
            background: linear-gradient(90deg, #ff7e5f, #feb47b);
            color: white;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);
        }
        .gradient-button:hover {
            transform: scale(1.05);
        }
        .click-button {
            background: linear-gradient(90deg, #4caf50, #81c784);
        }
        input {
            border: 1px solid #ccc;
            border-radius: 5px;
            width: 200px;
        }
        #bonusMessage {
            color: red;
            margin-top: 20px;
        }
        #authSection, #gameSection, #betSection, #shopSection {
            margin-top: 20px;
            background: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
        }
        #gameSection, #betSection, #shopSection {
            display: none;
        }
        .cupButton {
            background-color: #007bff;
            color: white;
            margin: 5px;
            padding: 6px 12px;
            width: 100px;
        }
        .cupButton:hover {
            background-color: #0056b3;
        }
        .cupContainer {
            display: flex;
            justify-content: center;
            margin-top: 10px;
        }
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
        <h1>Ваш баланс: <span id="balance">0</span></h1>
        <h1>💰 Баланс: <span id="moneyBalance">0</span></h1>
        <h1>💎 Баланс коины: <span id="coinBalance">0</span></h1>
        <h2>Клики за раз: <span id="clickValue">1</span></h2>
        <button class="click-button" id="increaseButton">Клик</button>
        <br><br>
        <button class="gradient-button" id="upgradeButton">Купить апгрейд (<span id="upgradeCost">5</span> монет, +2 клика)</button>
        <br><br>
        <button class="gradient-button" id="bonusButton">Получить ежедневный бонус</button>
        <button class="gradient-button" id="playCupsButton">Играть в 3 стакана</button>
        <button class="gradient-button" id="shopButton">Перейти в магазин</button>
        <p id="bonusMessage"></p>
        <button class="gradient-button" id="logoutButton">Выйти</button>
    </div>

    <div id="betSection">
        <h2>Игра 3 стакана</h2>
        <input type="number" id="betAmount" placeholder="Ставка (100-1000000000000)" min="100" max="1000000000000>
        <br><br>
        <h3>Выберите стакан:</h3>
        <div class="cupContainer">
            <button class="cupButton" data-cup="0">Стакан 1</button>
            <button class="cupButton" data-cup="1">Стакан 2</button>
            <button class="cupButton" data-cup="2">Стакан 3</button>
        </div>
        <p id="betMessage"></p>
        <button class="gradient-button" id="backToGameButton">Назад в игру</button>
    </div>

    <div id="shopSection">
        <h2>Магазин</h2>
        <h3>Генераторы:</h3>
        <div id="generatorList"></div>
        <button class="gradient-button" id="backToGameFromShopButton">Назад в игру</button>
    </div>

    <script>
        let currentUser = localStorage.getItem('currentUser');

        if (currentUser) {
            showGameSection();
        }

        function showGameSection() {
            document.getElementById('authSection').style.display = 'none';
            document.getElementById('gameSection').style.display = 'block';
            loadGameData();
        }

        function loadGameData() {
            currentUser = JSON.parse(localStorage.getItem(localStorage.getItem('currentUser')));
            document.getElementById('balance').textContent = currentUser.balance;
            document.getElementById('moneyBalance').textContent = currentUser.moneyBalance;
            document.getElementById('coinBalance').textContent = currentUser.coinBalance;
            document.getElementById('clickValue').textContent = currentUser.clickValue;
            document.getElementById('upgradeCost').textContent = currentUser.upgradeCost;
        }

        function saveGameData() {
            localStorage.setItem(currentUser.email, JSON.stringify(currentUser));
        }

        function generateMoney() {
            currentUser.moneyBalance += 1; // Генерация 1 💰 каждую минуту
            document.getElementById('moneyBalance').textContent = currentUser.moneyBalance;
            saveGameData();
        }

        function generateCoins() {
            currentUser.coinBalance += 1; // Генерация 1 💎 каждую минуту
            document.getElementById('coinBalance').textContent = currentUser.coinBalance;
            saveGameData();
        }

        document.getElementById('registerButton').addEventListener('click', function() {
            let login = document.getElementById('regLogin').value;
            let email = document.getElementById('regEmail').value;
            let password = document.getElementById('regPassword').value;

            if (login && email && password) {
                if (localStorage.getItem(email)) {
                    document.getElementById('authMessage').textContent = 'Пользователь с такой почтой уже существует!';
                } else {
                    let user = {
                        login: login,
                        email: email,
                        password: password,
                        balance: 0,
                        moneyBalance: 0,
                        coinBalance: 0,
                        clickValue: 1,
                        upgradeCost: 5,
                        lastBonusTime: 0,
                        generators: []
                    };
                    localStorage.setItem(email, JSON.stringify(user));
                    document.getElementById('authMessage').textContent = 'Регистрация успешна! Войдите в систему.';
                }
            } else {
                document.getElementById('authMessage').textContent = 'Заполните все поля!';
            }
        });

        document.getElementById('loginButton').addEventListener('click', function() {
            let loginEmail = document.getElementById('loginEmail').value;
            let password = document.getElementById('loginPassword').value;

            if (loginEmail && password) {
                let user = JSON.parse(localStorage.getItem(loginEmail)) || null;
                if (!user) {
                    for (let key in localStorage) {
                        if (localStorage.hasOwnProperty(key)) {
                            let u = JSON.parse(localStorage.getItem(key));
                            if (u && (u.login === loginEmail || u.email === loginEmail)) {
                                user = u;
                                break;
                            }
                        }
                    }
                }

                if (user && user.password === password) {
                    localStorage.setItem('currentUser', user.email);
                    currentUser = user;
                    showGameSection();
                } else {
                    document.getElementById('authMessage').textContent = 'Неверный логин или пароль!';
                }
            } else {
                document.getElementById('authMessage').textContent = 'Заполните все поля!';
            }
        });

        document.getElementById('increaseButton').addEventListener('click', function() {
            currentUser.balance += currentUser.clickValue;
            document.getElementById('balance').textContent = currentUser.balance;
            saveGameData();
        });

        document.getElementById('upgradeButton').addEventListener('click', function() {
            if (currentUser.balance >= currentUser.upgradeCost) {
                currentUser.balance -= currentUser.upgradeCost;
                currentUser.clickValue += 2;
                currentUser.upgradeCost = Math.ceil(currentUser.upgradeCost * 1.5);
                document.getElementById('balance').textContent = currentUser.balance;
                document.getElementById('clickValue').textContent = currentUser.clickValue;
                document.getElementById('upgradeCost').textContent = currentUser.upgradeCost;
                saveGameData();
            } else {
                alert('Недостаточно средств для апгрейда!');
            }
        });

        document.getElementById('bonusButton').addEventListener('click', function() {
            let currentTime = new Date().getTime();
            if (currentTime - currentUser.lastBonusTime >= 24 * 60 * 60 * 1000) {
                let bonus = Math.floor(Math.random() * 50000) + 1;
                currentUser.balance += bonus;
                currentUser.lastBonusTime = currentTime;
                document.getElementById('balance').textContent = currentUser.balance;
                document.getElementById('bonusMessage').textContent = `Вы получили бонус: ${bonus} монет!`;
                saveGameData();
            } else {
                let timeLeft = 24 * 60 * 60 * 1000 - (currentTime - currentUser.lastBonusTime);
                let hoursLeft = Math.floor(timeLeft / (60 * 60 * 1000));
                document.getElementById('bonusMessage').textContent = `Бонус доступен через ${hoursLeft} часов.`;
            }
        });

        document.getElementById('logoutButton').addEventListener('click', function() {
            localStorage.removeItem('currentUser');
            document.getElementById('authSection').style.display = 'block';
            document.getElementById('gameSection').style.display = 'none';
        });

        document.getElementById('playCupsButton').addEventListener('click', function() {
            document.getElementById('gameSection').style.display = 'none';
            document.getElementById('betSection').style.display = 'block';
        });

        const cupsButtons = document.querySelectorAll('.cupButton');
        cupsButtons.forEach(button => {
            button.addEventListener('click', function() {
                let userChoice = parseInt(this.getAttribute('data-cup'));
                let betAmount = parseInt(document.getElementById('betAmount').value);

                if (isNaN(betAmount) || betAmount < 100 || betAmount > 1000000000000) {
                    document.getElementById('betMessage').textContent = 'Ставка должна быть от 100 до 1,000,000,000,000!';
                    return;
                }

                if (currentUser.balance < betAmount) {
                    document.getElementById('betMessage').textContent = 'Недостаточно средств для ставки!';
                    return;
                }

                let winningChance = Math.random() < 0.05; // 5% шанс на выигрыш
                let winningCup = winningChance ? userChoice : (Math.floor(Math.random() * 3));

                if (userChoice === winningCup) {
                    currentUser.balance += betAmount * 2;
                    document.getElementById('betMessage').textContent = `Вы выиграли! Ваш новый баланс: ${currentUser.balance}`;
                } else {
                    currentUser.balance -= betAmount;
                    document.getElementById('betMessage').textContent = `Вы проиграли! Ваш новый баланс: ${currentUser.balance}`;
                }

                saveGameData();
                document.getElementById('balance').textContent = currentUser.balance;
            });
        });

        document.getElementById('backToGameButton').addEventListener('click', function() {
            document.getElementById('betSection').style.display = 'none';
            document.getElementById('gameSection').style.display = 'block';
        });

        document.getElementById('shopButton').addEventListener('click', function() {
            document.getElementById('gameSection').style.display = 'none';
            document.getElementById('shopSection').style.display = 'block';
            loadGenerators();
        });

        function loadGenerators() {
            const generatorList = document.getElementById('generatorList');
            generatorList.innerHTML = '';
            const generators = [
                { name: "Генератор 1", cost: 100, income: 1 },
                { name: "Генератор 2", cost: 500, income: 5 },
                { name: "Генератор 3", cost: 1000, income: 10 },
            ];
            generators.forEach(generator => {
                const div = document.createElement('div');
                div.innerHTML = `${generator.name} - Стоимость: ${generator.cost} коина, Доход: ${generator.income} в минуту. 
                                <button class="gradient-button" onclick="buyGenerator(${generator.cost}, ${generator.income})">Купить</button>`;
                generatorList.appendChild(div);
            });
        }

        function buyGenerator(cost, income) {
            if (currentUser.coinBalance >= cost) {
                currentUser.coinBalance -= cost;
                currentUser.moneyBalance += income; // или можете добавить income к обоим балансам
                document.getElementById('coinBalance').textContent = currentUser.coinBalance;
                document.getElementById('moneyBalance').textContent = currentUser.moneyBalance;
                saveGameData();
            } else {
                alert('Недостаточно коина для покупки!');
            }
        }

        document.getElementById('backToGameFromShopButton').addEventListener('click', function() {
            document.getElementById('shopSection').style.display = 'none';
            document.getElementById('gameSection').style.display = 'block';
        });

        setInterval(generateMoney, 60000); // Генерация 1 💰 каждую минуту
        setInterval(generateCoins, 60000); // Генерация 1 💎 каждую минуту
    </script>
</body>
</html>
