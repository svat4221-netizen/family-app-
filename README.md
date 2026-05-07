<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Family Courier Cloud</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
        import { getDatabase, ref, set, onValue, push, update } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-database.js";

        // !!! ВСТАВЬ СВОИ КЛЮЧИ ОТСЮДА !!!
        const firebaseConfig = {
            apiKey: "ТВОЙ_API_KEY",
            authDomain: "ТВОЙ_PROJECT.firebaseapp.com",
            databaseURL: "https://ТВОЙ_PROJECT-default-rtdb.firebaseio.com",
            projectId: "ТВОЙ_PROJECT_ID",
            storageBucket: "ТВОЙ_PROJECT.appspot.com",
            messagingSenderId: "ID",
            appId: "APP_ID"
        };
        // !!! ДО СЮДА !!!

        const app = initializeApp(firebaseConfig);
        const db = getDatabase(app);

        // Дальше идет магия связи данных...
        window.db = db; window.dbRef = ref; window.dbSet = set; 
        window.dbOnValue = onValue; window.dbPush = push; window.dbUpdate = update;
    </script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700;900&display=swap');
        body { font-family: 'Inter', sans-serif; }
        .hidden { display: none !important; }
        .modal { position: fixed; inset: 0; background: rgba(0,0,0,0.8); z-index: 50; display: flex; align-items: center; justify-content: center; padding: 20px; }
    </style>
</head>
<body class="bg-gray-50 min-h-screen">

    <div id="authScreen" class="modal">
        <div class="bg-white w-full max-w-sm rounded-[40px] p-10 text-center shadow-2xl">
            <h1 class="text-3xl font-black italic mb-10 tracking-tighter">FAMILY COURIER</h1>
            <button onclick="login('courier')" class="w-full bg-yellow-400 py-5 rounded-2xl font-black uppercase mb-4">Вход: Курьер</button>
            <button onclick="login('dispatcher')" class="w-full bg-slate-900 text-white py-5 rounded-2xl font-black uppercase">Вход: Диспетчер</button>
        </div>
    </div>

    <div id="app" class="hidden">
        <header id="header" class="p-6 rounded-b-[40px] shadow-lg sticky top-0 z-10 transition-colors">
            <div class="flex justify-between items-center text-white">
                <h1 class="text-xl font-black italic">COURIER APP</h1>
                <button onclick="location.reload()" class="text-[10px] underline uppercase">Выйти</button>
            </div>
            <div class="mt-6 bg-white rounded-3xl p-6 text-gray-900 flex justify-between shadow-md">
                <div>
                    <p class="text-[10px] font-bold text-gray-400">БАЛАНС</p>
                    <p class="text-3xl font-black"><span id="tokenCount">0</span> ₮</p>
                </div>
                <div class="text-right">
                    <p class="text-[10px] font-bold text-gray-400">РЕЙТИНГ</p>
                    <p class="text-xl font-black italic">★ <span id="ratingDisplay">0.00</span></p>
                </div>
            </div>
        </header>

        <main class="p-6">
            <div class="flex justify-between items-center mb-6">
                <h2 class="font-black italic uppercase">Заказы</h2>
                <button id="addBtn" onclick="document.getElementById('createModal').classList.remove('hidden')" class="bg-green-600 text-white w-10 h-10 rounded-full font-bold hidden">+</button>
            </div>
            <div id="taskList" class="space-y-4"></div>
        </main>
    </div>

    <div id="createModal" class="modal hidden">
        <div class="bg-white p-8 rounded-[30px] w-full max-w-sm">
            <input id="taskName" type="text" placeholder="Что сделать?" class="w-full bg-gray-100 p-4 rounded-xl mb-4 font-bold outline-none">
            <input id="taskPrice" type="number" placeholder="Цена ₮" class="w-full bg-gray-100 p-4 rounded-xl mb-6 font-bold outline-none">
            <button onclick="createTask()" class="w-full bg-green-600 text-white py-4 rounded-xl font-black">ОПУБЛИКОВАТЬ</button>
        </div>
    </div>

    <script>
        let role = '';
        let stats = { tokens: 0, rating: 5.0 };

        function login(selectedRole) {
            role = selectedRole;
            document.getElementById('authScreen').classList.add('hidden');
            document.getElementById('app').classList.remove('hidden');
            
            const header = document.getElementById('header');
            if(role === 'dispatcher') {
                header.classList.add('bg-slate-900');
                document.getElementById('addBtn').classList.remove('hidden');
            } else {
                header.classList.add('bg-yellow-400');
            }

            // ПОДКЛЮЧАЕМ ЖИВЫЕ ДАННЫЕ
            window.dbOnValue(window.dbRef(window.db, 'state'), (snapshot) => {
                const data = snapshot.val();
                if(data) {
                    document.getElementById('tokenCount').innerText = data.tokens;
                    document.getElementById('ratingDisplay').innerText = data.rating.toFixed(2);
                }
            });

            window.dbOnValue(window.dbRef(window.db, 'tasks'), (snapshot) => {
                renderTasks(snapshot.val());
            });
        }

        function createTask() {
            const name = document.getElementById('taskName').value;
            const price = parseInt(document.getElementById('taskPrice').value);
            if(name && price) {
                const newTaskRef = window.dbPush(window.dbRef(window.db, 'tasks'));
                window.dbSet(newTaskRef, { title: name, price: price, type: 'Standard' });
                document.getElementById('createModal').classList.add('hidden');
            }
        }

        function renderTasks(tasksData) {
            const list = document.getElementById('taskList');
            list.innerHTML = '';
            for(let id in tasksData) {
                const task = tasksData[id];
                const card = document.createElement('div');
                card.className = "bg-white p-6 rounded-[30px] shadow-sm";
                card.innerHTML = `
                    <div class="flex justify-between mb-4">
                        <span class="text-xs font-bold uppercase opacity-40">${task.type}</span>
                        <span class="font-black">${task.price} ₮</span>
                    </div>
                    <h3 class="font-bold text-lg mb-4">${task.title}</h3>
                    <button onclick="deleteTask('${id}')" class="w-full bg-gray-100 py-3 rounded-xl text-[10px] font-black uppercase">Удалить / Выполнить</button>
                `;
                list.appendChild(card);
            }
        }

        function deleteTask(id) {
            window.dbSet(window.dbRef(window.db, 'tasks/' + id), null);
        }
    </script>
</body>
</html>
