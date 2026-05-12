<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Family Courier Cloud</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/appwrite@14.0.0"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700;900&display=swap');
        body { font-family: 'Inter', sans-serif; background-color: #f9fafb; }
        .hidden { display: none !important; }
        .modal { position: fixed; inset: 0; background: rgba(0,0,0,0.85); z-index: 50; display: flex; align-items: center; justify-content: center; padding: 20px; }
        .btn-shadow { box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1); }
    </style>
</head>
<body class="min-h-screen">

    <div id="authScreen" class="modal">
        <div class="bg-white w-full max-w-sm rounded-[40px] p-10 text-center shadow-2xl">
            <div class="mb-6 text-4xl">🚚</div>
            <h1 class="text-3xl font-black italic mb-2 tracking-tighter uppercase">Family Courier</h1>
            <p class="text-gray-400 text-[10px] font-bold uppercase mb-10 tracking-widest">Облачная система заданий</p>
            
            <button onclick="login('courier')" class="w-full bg-yellow-400 py-5 rounded-2xl font-black uppercase mb-4 btn-shadow active:scale-95 transition-all">Я Курьер (Сын)</button>
            <button onclick="login('dispatcher')" class="w-full bg-slate-900 text-white py-5 rounded-2xl font-black uppercase btn-shadow active:scale-95 transition-all">Диспетчер (Мама)</button>
        </div>
    </div>

    <div id="app" class="hidden">
        <header id="header" class="p-6 rounded-b-[45px] shadow-2xl sticky top-0 z-10 transition-colors duration-500">
            <div class="flex justify-between items-center text-white">
                <h1 id="roleTitle" class="text-xl font-black italic tracking-tighter">COURIER SERVICE</h1>
                <button onclick="location.reload()" class="bg-white/20 px-4 py-1 rounded-full text-[9px] font-bold uppercase backdrop-blur-md">Выйти</button>
            </div>
            
            <div class="mt-8 bg-white rounded-[32px] p-6 text-gray-900 flex justify-between items-center shadow-xl">
                <div>
                    <p class="text-[9px] font-black text-gray-400 uppercase tracking-widest mb-1">Ваш баланс</p>
                    <p class="text-4xl font-black tracking-tighter">1250 <span class="text-yellow-500 font-normal">₮</span></p>
                </div>
                <div class="h-12 w-[1px] bg-gray-100"></div>
                <div class="text-right">
                    <p class="text-[9px] font-black text-gray-400 uppercase tracking-widest mb-1">Рейтинг</p>
                    <p class="text-xl font-black italic">★ 5.0</p>
                </div>
            </div>
        </header>

        <main class="p-6">
            <div class="flex justify-between items-center mb-8">
                <h2 class="font-black italic uppercase text-gray-400 tracking-widest text-xs">Доступные заказы</h2>
                <button id="addBtn" onclick="document.getElementById('createModal').classList.remove('hidden')" class="bg-green-500 text-white w-14 h-14 rounded-2xl font-bold shadow-lg hidden flex items-center justify-center text-2xl">+</button>
            </div>

            <div id="taskList" class="space-y-4 pb-20">
                <div class="animate-pulse bg-gray-200 h-32 rounded-[35px]"></div>
            </div>
        </main>
    </div>

    <div id="createModal" class="modal hidden">
        <div class="bg-white p-8 rounded-[40px] w-full max-w-sm shadow-2xl">
            <h3 class="font-black mb-8 uppercase text-center text-xl tracking-tighter">Новый заказ для курьера</h3>
            <input id="taskName" type="text" placeholder="Что нужно сделать?" class="w-full bg-gray-50 p-5 rounded-2xl mb-4 font-bold outline-none border-2 border-transparent focus:border-green-500 transition-all text-sm">
            <input id="taskPrice" type="number" placeholder="Награда в ₮" class="w-full bg-gray-50 p-5 rounded-2xl mb-8 font-bold outline-none border-2 border-transparent focus:border-green-500 transition-all text-sm">
            
            <div class="flex gap-3">
                <button onclick="document.getElementById('createModal').classList.add('hidden')" class="flex-1 bg-gray-100 py-5 rounded-2xl font-black uppercase text-[10px]">Отмена</button>
                <button onclick="createTask()" class="flex-[2] bg-green-500 text-white py-5 rounded-2xl font-black uppercase text-[10px] shadow-lg shadow-green-200">Отправить в облако</button>
            </div>
        </div>
    </div>

    <script>
        const { Client, Databases, ID } = Appwrite;

        // ДАННЫЕ ВЗЯТЫ С ТВОИХ СКРИНШОТОВ
        const client = new Client()
            .setEndpoint('https://cloud.appwrite.io/v1')
            .setProject('6640ae230018df13b28f'); // Твой ID проекта из скриншота

        const databases = Databases(client);
        const DB_ID = 'CourierDB'; // Твое название базы
        const COLL_ID = 'Tasks';    // Твое название таблицы

        let currentRole = '';

        function login(role) {
            currentRole = role;
            document.getElementById('authScreen').classList.add('hidden');
            document.getElementById('app').classList.remove('hidden');
            
            const header = document.getElementById('header');
            const roleTitle = document.getElementById('roleTitle');
            
            if(role === 'dispatcher') {
                header.classList.add('bg-slate-900');
                roleTitle.innerText = 'DISPATCHER MODE';
                document.getElementById('addBtn').classList.remove('flex');
                document.getElementById('addBtn').classList.remove('hidden');
            } else {
                header.classList.add('bg-yellow-400');
                roleTitle.innerText = 'COURIER MODE';
            }
            
            loadTasks();
            setInterval(loadTasks, 3000); // Обновление каждые 3 сек
        }

        async function loadTasks() {
            try {
                const response = await databases.listDocuments(DB_ID, COLL_ID);
                renderTasks(response.documents);
            } catch (err) {
                console.log("Ждем настройки таблиц...");
            }
        }

        function renderTasks(tasks) {
            const list = document.getElementById('taskList');
            list.innerHTML = '';
            
            if (tasks.length === 0) {
                list.innerHTML = '<div class="text-center py-20"><p class="font-black uppercase text-gray-300 text-xs tracking-widest">Заказов пока нет</p></div>';
                return;
            }

            tasks.reverse().forEach(task => {
                const card = document.createElement('div');
                card.className = "bg-white p-7 rounded-[35px] shadow-sm border-2 border-gray-50 flex flex-col gap-4";
                card.innerHTML = `
                    <div class="flex justify-between items-start">
                        <div class="bg-gray-100 px-3 py-1 rounded-full text-[8px] font-black uppercase text-gray-400">Срочно</div>
                        <div class="text-2xl font-black tracking-tighter text-green-600">${task.награда} ₮</div>
                    </div>
                    <h3 class="font-black text-xl text-gray-800 leading-tight">${task.заголовок}</h3>
                    <button onclick="completeTask('${task.$id}')" class="w-full ${currentRole === 'dispatcher' ? 'bg-red-50 text-red-500' : 'bg-slate-900 text-white'} py-4 rounded-2xl text-[10px] font-black uppercase tracking-widest transition-all active:scale-95 shadow-md">
                        ${currentRole === 'dispatcher' ? 'Удалить из списка' : 'Выполнить заказ'}
                    </button>
                `;
                list.appendChild(card);
            });
        }

        async function createTask() {
            const name = document.getElementById('taskName').value;
            const price = document.getElementById('taskPrice').value;
            if(!name || !price) return;

            try {
                await databases.createDocument(DB_ID, COLL_ID, ID.unique(), {
                    "заголовок": name,
                    "награда": parseInt(price),
                    "статус": "активно"
                });
                document.getElementById('createModal').classList.add('hidden');
                document.getElementById('taskName').value = '';
                document.getElementById('taskPrice').value = '';
                loadTasks();
            } catch (err) {
                alert("Ошибка! Проверь, создал ли ты базу 'CourierDB' и таблицу 'Tasks' с колонками 'заголовок' и 'награда'");
            }
        }

        async function completeTask(id) {
            try {
                await databases.deleteDocument(DB_ID, COLL_ID, id);
                loadTasks();
            } catch (err) {
                console.error(err);
            }
        }
    </script>
</body>
</html>

