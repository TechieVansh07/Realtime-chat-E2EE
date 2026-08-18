index.html
{
    <!DOCTYPE html>
<html lang="en" data-theme="sunset">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
    <title>Realtime Chat</title>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;500;600;700&display=swap" rel="stylesheet">
    <script defer src="http://127.0.0.1:8000/socket.io/socket.io.js"></script>
    <script defer src="js/client.js"></script>
    <link rel="stylesheet" href="css/style.css">
    <script>
    // apply saved theme before paint (no flash)
    try{
        var t = localStorage.getItem('chat-theme');
        if (t) document.documentElement.setAttribute('data-theme', t);
    }catch(e){}
    </script>
</head>
<body>
    <nav>
        <img class="logo" src="chat.png" alt="">
        <div class="brand">
            <h1>Realtime Chat</h1>
            <span class="status"><i></i> connected</span>
        </div>

        <!-- Theme selector: purely visual, swaps CSS variables only -->
        <div class="theme-picker">
            <button class="theme-toggle" id="themeToggle" type="button" aria-haspopup="true" aria-expanded="false">
                <span class="dot"></span><span class="label">Theme</span>
            </button>
            <div class="theme-menu" id="themeMenu" role="menu" hidden></div>
        </div>
    </nav>

    <!-- chat area wraps the original .container so system popups can float over it -->
    <div class="chat-area">
        <div class="container">
            <div class="message left">NOTE : Chatroom Disclaimer: Please be respectful, protect your privacy, and avoid sharing sensitive content</div>
        </div>
        <!-- floating, non-persistent system notifications -->
        <div class="system-layer" id="systemLayer" aria-live="polite" aria-atomic="false"></div>
    </div>

    <div class="send">
        <form action="#" id="send-container">
            <input type="text" name="messageInp" id="messageInp" placeholder="Type a message…" autocomplete="off">
            <button class="btn" type="submit">Send</button>
        </form>
    </div>

    <!-- Timestamps + system notifications: purely visual add-ons. They watch for
         nodes that client.js already appends. No socket logic touched. -->
    <script>
    (function () {
        var container = document.querySelector('.container');
        var layer = document.getElementById('systemLayer');
        if (!container) return;

        /* ---------- floating system popup ---------- */
        var SYSTEM_RE = /(joined|left|entered|disconnected)\b[^]{0,24}(chat)?\s*$/i;

        function isSystem(text) {
            var t = (text || '').trim();
            if (!t || t.length > 90) return false;
            return /\b(joined|left|entered|disconnected)\b/i.test(t) && !/:/.test(t.split(/\b(joined|left)\b/i)[0]);
        }

        function icon(kind) {
            return kind === 'leave'
                ? '<svg viewBox="0 0 24 24" aria-hidden="true"><path d="M15 4H6a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h9"/><path d="M10 12h10"/><path d="m17 8 4 4-4 4"/></svg>'
                : '<svg viewBox="0 0 24 24" aria-hidden="true"><path d="M9 4h9a2 2 0 0 1 2 2v12a2 2 0 0 1-2 2H9"/><path d="M14 12H4"/><path d="m8 8-4 4 4 4"/></svg>';
        }

        function popup(text, kind, ms) {
            if (!layer) return;
            var el = document.createElement('div');
            el.className = 'system-pop' + (kind ? ' is-' + kind : '');
            el.setAttribute('role', 'status');
            el.innerHTML = '<span class="system-pop__icon">' + icon(kind) + '</span>' +
                           '<span class="system-pop__text"></span>';
            el.querySelector('.system-pop__text').textContent = text;
            layer.appendChild(el);

            // keep the stack tidy
            while (layer.children.length > 3) layer.removeChild(layer.firstChild);

            setTimeout(function () {
                el.classList.add('is-out');
                el.addEventListener('animationend', function () { el.remove(); }, { once: true });
                setTimeout(function () { el.remove(); }, 800);
            }, ms || 3200);
        }

        /* ---------- timestamps ---------- */
        function label(date) {
            var time = date.toLocaleTimeString([], { hour: 'numeric', minute: '2-digit' });
            var diff = (Date.now() - date.getTime()) / 1000;
            if (diff < 60) return 'just now';
            return time;
        }

        function stamp(el) {
            if (!el || el.nodeType !== 1) return;
            if (!el.classList.contains('message')) return;
            if (el.querySelector('.msg-time')) return;
            var now = new Date();
            var span = document.createElement('span');
            span.className = 'msg-time';
            span.dataset.ts = now.getTime();
            span.textContent = label(now);
            el.appendChild(span);
        }

        function handle(el) {
            if (!el || el.nodeType !== 1) return;
            if (!el.classList.contains('message')) return;
            var text = (el.textContent || '').trim();
            if (isSystem(text)) {
                // never keep join/leave notices in the chat history
                el.remove();
                popup(text, /\bleft|disconnected\b/i.test(text) ? 'leave' : 'join');
                return;
            }
            stamp(el);
        }

        Array.prototype.forEach.call(Array.prototype.slice.call(container.children), handle);

        new MutationObserver(function (records) {
            records.forEach(function (r) {
                Array.prototype.forEach.call(r.addedNodes, handle);
            });
            container.scrollTop = container.scrollHeight;
        }).observe(container, { childList: true });

        setInterval(function () {
            container.querySelectorAll('.msg-time').forEach(function (s) {
                s.textContent = label(new Date(Number(s.dataset.ts)));
            });
        }, 30000);

        /* friendly greeting when you join */
        setTimeout(function () {
            popup('Welcome to Realtime Chat 👋', 'welcome', 3600);
        }, 600);
    })();
    </script>

    <!-- Theme selector logic: only sets data-theme on <html>. -->
    <script>
    (function () {
        var THEMES = [
            { id:'sunset',      colors:['#ffc043','#ff7a59','#e8449b'] },
            { id:'aurora',      colors:['#a6ff7a','#38e8b0','#2ec5ff'] },
            { id:'ultraviolet', colors:['#d67bff','#8b5cf6','#5b8bff'] },
            { id:'ocean',       colors:['#7ad3ff','#39a8ff','#1de4d6'] },
            { id:'ember',       colors:['#ffd166','#ff9d00','#ff5f3d'] },
            { id:'forest',      colors:['#b7e04b','#63d471','#2fbf9b'] },
            { id:'rose',        colors:['#ffb86b','#ff6f91','#ff9ec7'] },
            { id:'midnight',    colors:['#4fd1ff','#6c8cff','#9aa8ff'] },
            { id:'cyberpunk',   colors:['#ffe600','#ff2fb9','#00f0ff'] },
            { id:'sand',        colors:['#f4dfa8','#e6b566','#d98b5f'] },
            { id:'mint',        colors:['#bff5e6','#57e0c8','#7fe8a0'] },
            { id:'mono',        colors:['#efeff3','#c9c9cf','#8f8f98'] }
        ];

        var root = document.documentElement;
        var toggle = document.getElementById('themeToggle');
        var menu = document.getElementById('themeMenu');
        if (!toggle || !menu) return;

        function current() { return root.getAttribute('data-theme') || 'sunset'; }

        function apply(id) {
            root.setAttribute('data-theme', id);
            try { localStorage.setItem('chat-theme', id); } catch (e) {}
            menu.querySelectorAll('.theme-option').forEach(function (b) {
                b.setAttribute('aria-checked', String(b.dataset.theme === id));
            });
        }

        THEMES.forEach(function (t) {
            var b = document.createElement('button');
            b.type = 'button';
            b.className = 'theme-option';
            b.dataset.theme = t.id;
            b.setAttribute('role', 'menuitemradio');
            b.setAttribute('aria-checked', String(t.id === current()));
            b.innerHTML = '<span class="swatch" style="background:linear-gradient(135deg,' +
                t.colors.join(',') + ')"></span>' + t.id;
            b.addEventListener('click', function () { apply(t.id); close(); });
            menu.appendChild(b);
        });

        function open()  { menu.hidden = false; toggle.setAttribute('aria-expanded', 'true'); }
        function close() { menu.hidden = true;  toggle.setAttribute('aria-expanded', 'false'); }

        toggle.addEventListener('click', function (e) {
            e.stopPropagation();
            menu.hidden ? open() : close();
        });
        document.addEventListener('click', function (e) {
            if (!menu.hidden && !menu.contains(e.target)) close();
        });
        document.addEventListener('keydown', function (e) {
            if (e.key === 'Escape') close();
        });
    })();
    </script>
</body>
</html>

}


index.js
{
    const io = require('socket.io')(8000, {
    cors: {
        origin: "http://127.0.0.1:5500",
        methods: ["GET", "POST"]
    }
});

const users = {};

io.on('connection', socket => {

    console.log("A user connected");

    socket.on('new-user-joined', name => {
        console.log("New user joined:", name);

        users[socket.id] = name;

        socket.broadcast.emit('user-joined', name);
    });

    socket.on('send', message => {
        socket.broadcast.emit('receive', {
            message: message,
            name: users[socket.id]
        });
    });
    socket.on('disconnect', () => {
    socket.broadcast.emit('left', users[socket.id]);
    delete users[socket.id];
});

});
}

client.js
{
    console.log("CLIENT.JS LOADED");
const socket = io('http://127.0.0.1:8000');

const form = document.getElementById('send-container');
const messageInput= document.getElementById('messageInp');
const messageContainer = document.querySelector('.container');
var audio = new Audio("ding.mp3");


const append =(message,position) =>{
    const messageElement=document.createElement('div');
    messageElement.innerText = message;
    messageElement.classList.add('message');
    messageElement.classList.add(position);
    messageContainer.append(messageElement);
    if(position =='left'){
        audio.play();
    }
}

form.addEventListener('submit',(e)=>{
    e.preventDefault();
    const message = messageInp.value;
    append(`You : ${message}`,'right');
    socket.emit('send',message);
    messageInput.value = ''
})
const name =prompt("Enter your name to join");
socket.emit('new-user-joined',name);

socket.on('user-joined',name=>{
    append(`${name} joined the chat`,'right')

})

socket.on('receive',data=>{
    append(`${data.name} : ${data.message}`,'left');

})

socket.on('left', name => {
    append(`${name} left the chat`, 'left');
});
}