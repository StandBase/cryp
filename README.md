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
        #authSection, #gameSection, #betSection {
            margin-top: 20px;
            background: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
        }
        #gameSection, #betSection {
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
        .cup {
            width: 100px;
            height: 100px;
            background-color: #007bff;
            border-radius: 50%;
            margin: 10px;
            position: relative;
            transition: transform 0.6s;
        }
        .cup:hover {
            background-color: #0056b3;
        }
        #notification {
            background-color: #ffeb3b;
            color: black;
            padding: 10px;
            margin-top: 10px;
            display: none;
            border-radius: 5px;
        }
        .hidden {
            display: none;
        }
        .loader {
            border: 6px solid #f3f3f3;
            border-top: 6px solid #3498db;
            border-radius: 50%;
            width: 30px;
            height: 30px;
            animation: spin 1s linear infinite;
            display: none;
            margin: 20px auto;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
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
        <div id="registerLoader" class="loader"></div>

        <h2>Вход</h2>
        <input type="text" id="loginEmail" placeholder="Логин/Почта" required>
        <input type="password" id="loginPassword" placeholder="Пароль" required>
        <button class="gradient-button" id="loginButton">Войти</button>
        <div id="loginLoader" class="loader"></div>

        <p id="authMessage" style="color: red;"></p>
    </div>

    <div id="gameSection">
        <h1>Ваш баланс: <span id="balance">0</span></h1>
        <h2>Клики за раз: <span id="clickValue">1</span></h2>
        <button class="click-button" id="increaseButton">Клик</button>
        <br><br>
        <button class="gradient-button" id="upgradeButton">Купить апгрейд (<span id="upgradeCost">5</span> монет, +2 клика)</button>
        <br><br>
        <button class="gradient-button" id="bonusButton">Получить ежедневный бонус</button>
        <button class="gradient-button" id="playCupsButton">Играть в 3 стакана</button>
        <p id="bonusMessage"></p>
        <button class="gradient-button" id="logoutButton">Выйти</button>
    </div>

    <div id="betSection">
        <h2>Игра 3 стакана</h2>
        <input type="number" id="betAmount" placeholder="Ставка (100-1000000000000)" min="100" max="1000000000000">
        <br><br>
        <h3>Выберите стакан:</h3>
        <div class="cupContainer">
            <div class="cup" data-cup="0"></div>
            <div class="cup" data-cup="1"></div>
            <div class="cup" data-cup="2"></div>
        </div>
        <p id="betMessage"></p>
        <button class="gradient-button" id="backToGameButton">Назад в игру</button>
    </div>

    <div id="notification"></div>

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
            const currentTime = new Date().getTime();
            
            if (currentTime - currentUser.lastResetTime >= 7 * 24 * 60 * 60 * 1000) {
                currentUser.balance = 0;
                currentUser.lastResetTime = currentTime;
                saveGameData();
            }

            document.getElementById('balance').textContent = currentUser.balance;
            document.getElementById('clickValue').textContent = currentUser.clickValue;
            document.getElementById('upgradeCost').textContent = currentUser.upgradeCost;
        }

        function saveGameData() {
            localStorage.setItem(currentUser.email, JSON.stringify(currentUser));
        }

        function showNotification(message) {
            let notification = document.getElementById('notification');
            notification.textContent = message;
            notification.style.display = 'block';
            setTimeout(() => {
                notification.style.display = 'none';
            }, 3000);
        }

        document.getElementById('registerButton').addEventListener('click', function() {
            let login = document.getElementById('regLogin').value;
            let email = document.getElementById('regEmail').value;
            let password = document.getElementById('regPassword').value;

            if (login && email && password) {
                document.getElementById('registerLoader').style.display = 'block';
                setTimeout(() => {
                    if (localStorage.getItem(email)) {
                        document.getElementById('authMessage').textContent = 'Пользователь с такой почтой уже существует!';
                    } else {
                        let user = {
                            login: login,
                            email: email,
                            password: password,
                            balance: 0,
                            clickValue: 1,
                            upgradeCost: 5,
                            lastBonusTime: 0,
                            lastResetTime: new Date().getTime()
                        };
                        localStorage.setItem(email, JSON.stringify(user));
                        document.getElementById('authMessage').textContent = 'Регистрация успешна! Войдите в систему.';
                        // Очищаем поля
                        document.getElementById('regLogin').value = '';
                        document.getElementById('regEmail').value = '';
                        document.getElementById('regPassword').value = '';
                    }
                    document.getElementById('registerLoader').style.display = 'none';
                }, 1000);
            } else {
                showNotification('Заполните все поля для регистрации!');
            }
        });

        document.getElementById('loginButton').addEventListener('click', function() {
            let email = document.getElementById('loginEmail').value;
            let password = document.getElementById('loginPassword').value;

            if (email && password) {
                document.getElementById('loginLoader').style.display = 'block';
                setTimeout(() => {
                    let user = JSON.parse(localStorage.getItem(email));
                    if (user && user.password === password) {
                        localStorage.setItem('currentUser', email);
                        showGameSection();
                    } else {
                        document.getElementById('authMessage').textContent = 'Неверная почта или пароль!';
                    }
                    document.getElementById('loginLoader').style.display = 'none';
                }, 1000);
            } else {
                showNotification('Заполните все поля для входа!');
            }
        });

        document.getElementById('playCupsButton').addEventListener('click', function() {
            document.getElementById('gameSection').style.display = 'none';
            document.getElementById('betSection').style.display = 'block';

            let cups = document.querySelectorAll('.cup');
            cups.forEach(cup => {
                cup.style.transform = 'translateY(0)';
            });

            setTimeout(() => {
                cups.forEach(cup => {
                    cup.style.transform = 'translateY(-100px)';
                });
            }, 1000);

            setTimeout(() => {
                cups.forEach(cup => {
                    cup.style.transform = 'translateY(0)';
                });
            }, 2000);
        });

        document.getElementById('backToGameButton').addEventListener('click', function() {
            document.getElementById('betSection').style.display = 'none';
            document.getElementById('gameSection').style.display = 'block';
        });

        document.getElementById('logoutButton').addEventListener('click', function() {
            localStorage.removeItem('currentUser');
            location.reload();
        });
    </script>
</body>
</html>
