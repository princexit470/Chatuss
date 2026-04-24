
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>XIT</title>
    <style>
        :root {
            --xit-black: #000000; --xit-dark-gray: #1a1a1a; --wa-light-green: #25D366;
            --wa-bg: #ffffff; --wa-chat-bg: #e5ddd5; --wa-sent: #dcf8c6;
            --wa-recv: #ffffff; --wa-gray: #667781; --modal-bg: rgba(0,0,0,0.8);
        }
        * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; font-family: 'Segoe UI', sans-serif; }
        body, html { margin: 0; padding: 0; height: 100%; width: 100%; background: var(--wa-bg); overflow: hidden; }

        /* Modals & Fast Animations */
        .modal-overlay { position: fixed; top:0; left:0; width:100%; height:100%; background: var(--modal-bg); z-index: 9000; display:none; align-items:center; justify-content:center; padding: 20px; transition: opacity 0.2s; }
        .modal-card { background: white; width: 100%; max-width: 320px; border-radius: 15px; padding: 20px; box-shadow: 0 10px 25px rgba(0,0,0,0.5); transform: scale(0.8); opacity: 0; transition: all 0.2s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
        .modal-overlay.active { display: flex; opacity: 1; }
        .modal-overlay.active .modal-card { transform: scale(1); opacity: 1; }
        .modal-card input { width: 100%; padding: 12px; margin: 10px 0; border: 1px solid #ddd; border-radius: 8px; outline: none; transition: 0.2s; }
        .modal-card input:focus { border-color: black; box-shadow: 0 0 5px rgba(0,0,0,0.2); }

        /* Buttons */
        button { cursor: pointer; transition: transform 0.1s, background 0.2s; }
        button:active { transform: scale(0.95); }

        /* Black Header & 3 Tabs */
        header { background: var(--xit-black); color: white; z-index: 100; position: relative; box-shadow: 0 2px 10px rgba(0,0,0,0.2); }
        .top-bar { display: flex; align-items: center; padding: 15px; gap: 15px; font-weight: bold; font-size: 20px; }
        .tabs { display: flex; background: var(--xit-black); }
        .tab { flex: 1; padding: 12px 0; text-align: center; font-size: 14px; font-weight: bold; opacity: 0.6; border-bottom: 3px solid transparent; color: white; cursor: pointer; transition: 0.2s; }
        .tab.active { opacity: 1; border-bottom: 3px solid white; }

        /* Swipe Container (Updated for 3 views) */
        .view-container { display: flex; width: 300%; height: calc(100vh - 105px); transition: transform 0.2s ease-out; will-change: transform; }
        .view { width: 33.333%; height: 100%; overflow-y: auto; padding-bottom: 80px; display: flex; flex-direction: column; } /* Flex for dynamic sorting */

        /* Sidebar Black Theme */
        #sidebar { position: fixed; left: -285px; top: 0; width: 280px; height: 100%; background: white; z-index: 2000; transition: 0.2s ease-in-out; box-shadow: 2px 0 15px rgba(0,0,0,0.3); }
        #sidebar.open { left: 0; }
        .sidebar-header { background: var(--xit-black); color: white; padding: 50px 20px 20px; }
        .menu-item { padding: 15px 20px; border-bottom: 1px solid #eee; cursor: pointer; color: #333; font-weight: 500; transition: 0.2s; }
        .menu-item:hover { background: #f5f5f5; padding-left: 25px; }
        #overlay { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.6); z-index: 1900; animation: fadeIn 0.2s; }

        /* Chat UI */
        #chat-window { position: fixed; top: 0; left: 100%; width: 100%; height: 100%; background: var(--wa-chat-bg) url('https://user-images.githubusercontent.com/15075759/28719144-86dc0f70-73b1-11e7-911d-60d70fcded21.png'); z-index: 1500; display: flex; flex-direction: column; transition: 0.2s ease-in-out; }
        #chat-window.open { left: 0; }
        
        /* Private Room Enhancements */
        #chat-window.private { background: #000000 !important; }
        .chat-header { background: var(--xit-black); color: white; padding: 10px 15px; display: flex; align-items: center; gap: 10px; box-shadow: 0 2px 5px rgba(0,0,0,0.2); z-index: 10; border-bottom: 1px solid #333;}
        .msg-flow { flex: 1; overflow-y: auto; padding: 15px; display: flex; flex-direction: column; gap: 10px; z-index: 2; position: relative; }
        
        /* Message Link Styling */
        .bubble { max-width: 80%; padding: 8px 12px; border-radius: 10px; font-size: 14px; position: relative; word-wrap: break-word; animation: popIn 0.2s cubic-bezier(0.175, 0.885, 0.32, 1.275) forwards; opacity: 0; z-index: 3; cursor:pointer;}
        .bubble a { color: #0066cc; text-decoration: underline; font-weight: 500; }
        #chat-window.private .bubble a { color: #4da6ff; }
        
        .sent { align-self: flex-end; background: var(--wa-sent); transform-origin: bottom right; }
        .recv { align-self: flex-start; background: var(--wa-recv); transform-origin: bottom left; }
        #chat-window.private .sent { background: #0f331b; color: white; }
        #chat-window.private .recv { background: #1a1a1a; color: white; }
        .sending-tag { font-size: 10px; color: #667781; display: block; margin-top: 4px; text-align: right; }

        /* Rows */
        .chat-row, .status-row, .req-row { display: flex; align-items: center; padding: 12px 15px; border-bottom: 1px solid #f2f2f2; cursor: pointer; transition: 0.2s; animation: slideUp 0.2s ease-out forwards; }
        .chat-row:hover, .status-row:hover, .req-row:hover { background: #f9f9f9; }
        .chat-row.unread b { color: #25D366; } /* Highlighting unread */
        .chat-row.unread { background: #f0fff4; }
        .avatar { width: 50px; height: 50px; border-radius: 50%; background: #444; display: flex; align-items: center; justify-content: center; color: white; margin-right: 15px; font-weight: bold; text-transform: uppercase; box-shadow: 0 2px 5px rgba(0,0,0,0.2); flex-shrink: 0;}
        
        .fab { position: fixed; bottom: 25px; right: 25px; width: 56px; height: 56px; background: var(--xit-black); border-radius: 50%; display: flex; align-items: center; justify-content: center; color: white; font-size: 24px; border:none; z-index: 90; box-shadow: 0 4px 12px rgba(0,0,0,0.4); transition: 0.2s; }
        .fab:hover { transform: scale(1.1) rotate(90deg); }
        
        #login-screen { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: var(--xit-black); z-index: 9999; display: none; flex-direction: column; align-items: center; justify-content: center; color: white; }
        #status-viewer { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: #000; color: white; z-index: 8000; display: none; flex-direction: column; align-items: center; justify-content: center; animation: fadeIn 0.2s; }

        /* Full Screen Share View */
        #share-screen { position: fixed; top: 100%; left: 0; width: 100%; height: 100%; background: white; z-index: 9500; display: flex; flex-direction: column; transition: top 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
        #share-screen.active { top: 0; }
        .share-header { background: var(--xit-black); color: white; padding: 15px; display: flex; align-items: center; font-size: 18px; font-weight: bold; }
        .share-list { flex: 1; overflow-y: auto; padding: 10px 0; }
        .share-external { padding: 20px; background: #f9f9f9; border-top: 1px solid #ddd; box-shadow: 0 -2px 10px rgba(0,0,0,0.05); }

        /* Private Room Big Watermark */
        .watermark { position: absolute; top: 0; left: 0; width: 100%; height: 100%; display: none; align-items: center; justify-content: center; flex-direction: column; pointer-events: none; z-index: 1; opacity: 0.08; color: white; text-align: center; }
        #chat-window.private .watermark { display: flex; }

        /* Keyframes */
        @keyframes popIn { from { opacity: 0; transform: translateY(10px) scale(0.9); } to { opacity: 1; transform: translateY(0) scale(1); } }
        @keyframes slideUp { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
        @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }

        .btn-primary { background: black; color: white; border: none; padding: 12px; border-radius: 8px; width: 100%; font-weight: bold; margin-bottom: 10px; font-size: 15px; }
        .btn-secondary { background: #eee; color: black; border: none; padding: 12px; border-radius: 8px; width: 100%; font-weight: bold; font-size: 15px; }
        
        /* Sub Headers */
        .req-section-title { padding: 15px 20px 5px; font-weight: bold; color: #555; font-size: 13px; text-transform: uppercase; background: #fafafa; border-bottom: 1px solid #eee; }
    </style>
</head>
<body>

    <div id="modal-container" class="modal-overlay">
        <div class="modal-card">
            <h3 id="modal-title" style="color:black; margin-top:0;"></h3>
            <div id="modal-body"></div>
            <div class="modal-btns" id="modal-btns-wrapper" style="display:flex; justify-content:flex-end; gap:10px; margin-top:15px;">
                <button onclick="closeModal()" style="border:none; background:#eee; padding:8px 15px; border-radius:5px; font-weight:bold;">Cancel</button>
                <button id="modal-confirm-btn" style="border:none; background:black; color:white; padding:8px 15px; border-radius:5px; font-weight:bold;">Confirm</button>
            </div>
        </div>
    </div>

    <div id="login-screen">
        <h1 style="letter-spacing: 5px; margin-bottom: 40px; font-size: 45px; animation: popIn 0.5s;">XIT</h1>
        <div style="width: 80%; max-width: 320px; display: flex; flex-direction: column; gap: 15px; animation: slideUp 0.5s;">
            <input type="text" id="user-input" placeholder="Username" style="padding:15px; border-radius:8px; border:none; outline:none; font-size:16px;">
            <input type="password" id="pass-input" placeholder="Password" style="padding:15px; border-radius:8px; border:none; outline:none; font-size:16px;">
            <button onclick="performLogin()" style="padding:15px; background:white; color:black; border:none; border-radius:8px; font-weight:bold; cursor:pointer; font-size:16px; margin-top:10px;">GET STARTED</button>
        </div>
    </div>

    <div id="status-viewer" onclick="closeStatus()">
        <div style="position:absolute; top:30px; left:20px; display:flex; align-items:center; gap:12px;">
            <div id="st-avatar" class="avatar" style="width:40px; height:40px; background:white; color:black;"></div>
            <div><b id="st-user"></b><br><small id="st-time" style="opacity: 0.8;"></small></div>
        </div>
        <div id="st-text" style="padding:40px; text-align:center; font-size:32px; word-wrap: break-word; font-weight:bold;"></div>
        <div style="position:absolute; bottom:30px; width:100%; text-align:center;">
            <button id="delete-st-btn" onclick="deleteMyStatus(event)" style="display:none; background:red; color:white; border:none; padding:10px 20px; border-radius:20px; font-weight:bold;">🗑️ Delete Status</button>
        </div>
    </div>

    <div id="overlay" onclick="toggleSidebar(false)"></div>
    <div id="sidebar">
        <div class="sidebar-header">
            <div id="my-avatar" class="avatar" style="width:70px; height:70px; font-size:28px; background:rgba(255,255,255,0.2); margin-bottom:15px;"></div>
            <b id="side-user-info" style="font-size: 20px;"></b>
        </div>
        <div class="menu-item" onclick="showPrivateRoomMenu()">🔒 Private Rooms</div>
        <div class="menu-item" onclick="showChangePass()">🔑 Change Password</div>
        <div class="menu-item" style="color: #dc3545;" onclick="logout()">🚪 Logout</div>
    </div>

    <header>
        <div class="top-bar"><span onclick="toggleSidebar(true)" style="cursor:pointer; display:flex; align-items:center; gap:10px;">☰ XIT</span></div>
        <div class="tabs">
            <div class="tab active" id="tab0" onclick="moveTab(0)">CHATS</div>
            <div class="tab" id="tab1" onclick="moveTab(1)">STATUS</div>
            <div class="tab" id="tab2" onclick="moveTab(2)">REQUESTS</div>
        </div>
    </header>

    <div class="view-container" id="main-container">
        <div class="view" id="chat-list"></div>
        <div class="view" id="status-list"></div>
        <div class="view" id="request-list">
            <div class="req-section-title">Received Requests</div>
            <div id="received-reqs"></div>
            <div class="req-section-title" style="margin-top:10px;">Sent Requests</div>
            <div id="sent-reqs"></div>
        </div>
    </div>

    <button class="fab" onclick="handleFab()">+</button>

    <div id="chat-window">
        <div class="watermark">
            <span style="font-size:48px; font-weight:900; letter-spacing:2px; line-height:1.2;">PRIVATE<br>ROOM</span><br>
            <span style="font-size:18px; font-weight:bold; letter-spacing:1px; background:#111; padding:5px 15px; border-radius:20px; border:1px solid #333;">End-to-End Encrypted</span>
        </div>
        
        <div class="chat-header">
            <span onclick="closeChat()" style="font-size:26px; cursor:pointer; padding-right:10px;">←</span>
            <b id="active-name" style="flex:1; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; font-size:18px;"></b>
            
            <button id="pr-share-btn" onclick="openShareScreen()" title="Share Link" style="display:none; background:white; color:black; border:none; border-radius:50%; width:35px; height:35px; align-items:center; justify-content:center; font-size:16px; box-shadow:0 2px 5px rgba(0,0,0,0.2);">🔗</button>
            <button id="pr-exit-btn" onclick="exitPrivateRoom()" title="Exit Room" style="display:none; background:#ff4d4d; color:white; border:none; border-radius:50%; width:35px; height:35px; align-items:center; justify-content:center; font-size:16px; box-shadow:0 2px 5px rgba(0,0,0,0.2);">🚪</button>
        </div>
        <div class="msg-flow" id="msg-flow"></div>
        <div style="padding:10px; display:flex; gap:8px; background:#f0f2f5; z-index:10; border-top:1px solid #ddd;" id="chat-input-bar">
            <label style="font-size:24px; cursor:pointer; display:flex; align-items:center; padding-left:5px;"><input type="file" id="media-input" hidden onchange="sendMedia()">📎</label>
            <input type="text" id="msg-input" placeholder="Type a message..." onkeydown="if(event.key==='Enter') sendMsg()" style="flex:1; border-radius:20px; border:none; padding:12px 18px; outline:none; box-shadow:inset 0 1px 3px rgba(0,0,0,0.1); font-size:15px;">
            <button onclick="sendMsg()" style="background:black; border:none; color:white; border-radius:50%; width:45px; height:45px; display:flex; align-items:center; justify-content:center; box-shadow:0 2px 5px rgba(0,0,0,0.2); font-size:18px;">➔</button>
        </div>
    </div>

    <div id="share-screen">
        <div class="share-header">
            <span onclick="closeShareScreen()" style="font-size:26px; cursor:pointer; margin-right:15px;">←</span>
            <span>Share Room Link</span>
        </div>
        <div class="share-list" id="share-internal-list"></div>
        <div class="share-external">
            <p style="margin-top:0; font-size:14px; color:#555; text-align:center; margin-bottom:15px;">Share with friends via other apps</p>
            <button onclick="copyRoomLink()" class="btn-primary" style="background:#000;">📋 Copy Direct Link</button>
            <button onclick="shareViaWhatsApp()" class="btn-primary" style="background:#25D366; margin-bottom:10px;">💬 Share on WhatsApp</button>
            <button onclick="shareViaDeviceNative()" class="btn-secondary" style="background:#ddd;">📲 More Apps...</button>
        </div>
    </div>

    <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-database-compat.js"></script>

    <script>
        const firebaseConfig = {
            apiKey: "AIzaSyB6I0uj92aNft_TLwtYXhLNBi136LTYr8g",
            authDomain: "prince-a6721.firebaseapp.com",
            databaseURL: "https://prince-a6721-default-rtdb.firebaseio.com",
            projectId: "prince-a6721",
            storageBucket: "prince-a6721.firebasestorage.app",
            messagingSenderId: "320131851435",
            appId: "1:320131851435:web:fe98ec2fa7641aa41583ee"
        };
        firebase.initializeApp(firebaseConfig);
        const db = firebase.database();

        let myUser = localStorage.getItem('xit_username');
        let activeRoom = null;
        let activeRoomType = 'normal'; 
        let currentTab = 0;
        let currentActiveStatusKey = null; // for deletion
        
        window.myPeers = {};
        window.myPrivateRooms = {};
        window.lastRead = JSON.parse(localStorage.getItem('xit_reads') || '{}');

        // URL parsing for links in messages
        function urlify(text) {
            const urlRegex = /(https?:\/\/[^\s]+)/g;
            return text.replace(urlRegex, function(url) {
                return `<a href="${url}" target="_blank" onclick="event.stopPropagation()">${url}</a>`;
            });
        }

        // --- Swipe Logic (3 Tabs) ---
        let touchStartX = 0;
        const container = document.getElementById('main-container');
        container.addEventListener('touchstart', e => { touchStartX = e.touches[0].clientX; }, {passive: true});
        container.addEventListener('touchend', e => {
            let diff = touchStartX - e.changedTouches[0].clientX;
            if (Math.abs(diff) > 60) {
                if (diff > 0 && currentTab < 2) moveTab(currentTab + 1);
                else if (diff < 0 && currentTab > 0) moveTab(currentTab - 1);
            }
        }, {passive: true});

        function moveTab(i) {
            currentTab = i;
            container.style.transform = `translateX(-${i * 33.333}%)`;
            document.getElementById('tab0').classList.toggle('active', i === 0);
            document.getElementById('tab1').classList.toggle('active', i === 1);
            document.getElementById('tab2').classList.toggle('active', i === 2);
        }

        // --- Routing ---
        history.replaceState({page: 'home'}, "");
        window.onpopstate = () => {
            if (document.getElementById('share-screen').classList.contains('active')) closeShareScreen(false);
            else if (document.getElementById('status-viewer').style.display === 'flex') closeStatus(false);
            else if (document.getElementById('chat-window').classList.contains('open')) closeChat(false);
            else if (document.getElementById('sidebar').classList.contains('open')) toggleSidebar(false);
        };

        function showModal(title, body, confirmFn, showButtons = true) {
            document.getElementById('modal-title').innerText = title;
            document.getElementById('modal-body').innerHTML = body;
            const btnWrapper = document.getElementById('modal-btns-wrapper');
            if(showButtons) {
                btnWrapper.style.display = 'flex';
                document.getElementById('modal-confirm-btn').onclick = confirmFn;
            } else btnWrapper.style.display = 'none';
            document.getElementById('modal-container').classList.add('active');
        }
        function closeModal() { document.getElementById('modal-container').classList.remove('active'); }

        async function performLogin() {
            const name = document.getElementById('user-input').value.trim().toLowerCase();
            const pass = document.getElementById('pass-input').value.trim();
            if(!name || !pass) return;
            const snap = await db.ref('registered_users/'+name).once('value');
            if(snap.exists()){
                if(snap.val().password === pass) login(name);
                else alert("Wrong Password");
            } else {
                await db.ref('registered_users/'+name).set({password:pass, ts:Date.now()});
                login(name);
            }
        }
        function login(n){ localStorage.setItem('xit_username', n); location.reload(); }
        function logout(){ localStorage.clear(); location.reload(); }

        // --- Start App ---
        async function startApp() {
            // Private Room Deep Link check
            const params = new URLSearchParams(window.location.search);
            const linkRoom = params.get('room');
            if(linkRoom) {
                showModal("Join Secure Room", `
                    <p style="color:#555; margin-bottom:15px; font-size:15px;">Enter password to unlock room <b>${linkRoom}</b>.</p>
                    <input type="password" id="link-p" placeholder="Room Password" style="font-size:16px;">
                `, async () => {
                    const p = document.getElementById('link-p').value;
                    const snap = await db.ref('private_rooms/'+linkRoom).once('value');
                    if(!snap.exists() || snap.val().password !== p) { alert("Incorrect Password or Room doesn't exist!"); return; }
                    await db.ref('private_rooms/'+linkRoom+'/users/'+myUser).set(true);
                    window.history.replaceState({}, document.title, window.location.pathname);
                    closeModal();
                    openChat(linkRoom, "🔒 " + linkRoom, 'private');
                });
            }

            // Listeners
            db.ref('peers/'+myUser).on('value', snap => { window.myPeers = snap.val() || {}; renderChatList(); });
            db.ref('private_rooms').on('value', snap => { window.myPrivateRooms = snap.val() || {}; renderChatList(); });
            
            // Listen to all requests for this user
            db.ref('requests/'+myUser).on('value', snap => renderRequests(snap.val() || {}));

            db.ref('statuses').on('value', snap => {
                const list = document.getElementById('status-list'); list.innerHTML = "";
                let arr = [];
                snap.forEach(s => {
                    const d = s.val(); d.key = s.key;
                    if(Date.now() - d.ts > 86400000) { db.ref('statuses/'+s.key).remove(); return; }
                    let isContact = (d.user === myUser) || Object.values(window.myPeers).some(p => p.peer === d.user);
                    if(isContact) arr.push(d);
                });
                
                // Sort status by newest first
                arr.sort((a,b) => b.ts - a.ts);
                
                arr.forEach(d => {
                    let time = new Date(d.ts).toLocaleTimeString([], {hour:'2-digit', minute:'2-digit'});
                    list.innerHTML += `<div class="status-row" onclick="viewStatus('${d.user}','${d.text}','${time}', '${d.key}')">
                        <div class="avatar" style="border:2px solid black">${d.user[0]}</div>
                        <div style="flex:1;"><b>${d.user}</b><br><small style="color:#777;">${time}</small></div>
                    </div>`;
                });
            });
        }

        // Listen for message updates to dynamic sorting and unread
        function updateRoomMeta(room, isPrivate) {
            db.ref((isPrivate?'private_messages/':'messages/')+room).limitToLast(1).on('value', snap => {
                if(snap.exists()) {
                    let lastMsg = Object.values(snap.val())[0];
                    let el = document.getElementById('chat-row-'+room);
                    if(el) {
                        el.style.order = -lastMsg.ts; // Move to top using flex order
                        if(lastMsg.ts > (window.lastRead[room]||0) && lastMsg.from !== myUser) el.classList.add('unread');
                        else el.classList.remove('unread');
                    }
                }
            });
        }

        function renderChatList() {
            const list = document.getElementById('chat-list'); 
            list.innerHTML = "";
            
            Object.keys(window.myPrivateRooms).forEach(roomName => {
                if(window.myPrivateRooms[roomName].users && window.myPrivateRooms[roomName].users[myUser]) {
                    const row = document.createElement('div'); row.className = "chat-row"; row.id = "chat-row-"+roomName;
                    row.innerHTML = `<div class="avatar" style="background:#000;">🔒</div><div style="flex:1;"><b>${roomName}</b><br><small style="color:#25D366; font-weight:bold;">Private Room</small></div>`;
                    row.onclick = () => openChat(roomName, "🔒 " + roomName, 'private');
                    list.appendChild(row);
                    updateRoomMeta(roomName, true);
                }
            });

            Object.values(window.myPeers).forEach(p => {
                const row = document.createElement('div'); row.className = "chat-row"; row.id = "chat-row-"+p.room;
                row.innerHTML = `<div class="avatar">${p.peer[0]}</div><div style="flex:1;"><b>${p.peer}</b></div>`;
                row.onclick = () => openChat(p.room, p.peer, 'normal');
                row.oncontextmenu = (e) => { e.preventDefault(); confirmDeleteChat(p.room, p.peer); }; // Long press delete
                list.appendChild(row);
                updateRoomMeta(p.room, false);
            });
        }

        function renderRequests(reqData) {
            const recvDiv = document.getElementById('received-reqs'); recvDiv.innerHTML = "";
            const sentDiv = document.getElementById('sent-reqs'); sentDiv.innerHTML = "";
            
            Object.keys(reqData).forEach(uid => {
                const req = reqData[uid];
                const row = document.createElement('div'); row.className = "req-row"; row.style.order = -req.ts;
                row.oncontextmenu = (e) => { e.preventDefault(); deleteRequest(uid); }; // Long press delete

                if(req.type === 'received') {
                    row.innerHTML = `<div class="avatar">${uid[0]}</div><div style="flex:1;"><b>${uid}</b><br><small>${req.status}</small></div>`;
                    if(req.status === 'pending') {
                        row.innerHTML += `<button style="background:#25D366; color:white; border:none; padding:5px 10px; border-radius:5px; margin-right:5px; font-weight:bold;" onclick="acceptReq('${uid}')">Accept</button>
                                          <button style="background:#ff4d4d; color:white; border:none; padding:5px 10px; border-radius:5px; font-weight:bold;" onclick="rejectReq('${uid}')">Reject</button>`;
                    }
                    recvDiv.appendChild(row);
                } else if(req.type === 'sent') {
                    row.innerHTML = `<div class="avatar">${uid[0]}</div><div style="flex:1;"><b>${uid}</b><br><small>Status: ${req.status}</small></div>
                                     <button style="background:#ddd; color:black; border:none; padding:5px 10px; border-radius:5px;" onclick="deleteRequest('${uid}')">Cancel</button>`;
                    sentDiv.appendChild(row);
                }
            });
        }

        // --- Requests Logic ---
        function sendFriendRequest(targetUser) {
            if(targetUser === myUser) return alert("You cannot send request to yourself!");
            db.ref('registered_users/'+targetUser).once('value', snap => {
                if(!snap.exists()) return alert("User not found!");
                db.ref('requests/'+myUser+'/'+targetUser).set({type: 'sent', status: 'pending', ts: Date.now()});
                db.ref('requests/'+targetUser+'/'+myUser).set({type: 'received', status: 'pending', ts: Date.now()});
                alert("Request Sent!"); closeModal();
            });
        }

        async function acceptReq(uid) {
            await db.ref('requests/'+myUser+'/'+uid).update({status: 'accepted'});
            await db.ref('requests/'+uid+'/'+myUser).update({status: 'accepted'});
            const room = [myUser, uid].sort().join('_');
            await db.ref('peers/'+myUser+'/'+uid).set({peer:uid, room:room});
            await db.ref('peers/'+uid+'/'+myUser).set({peer:myUser, room:room});
        }
        function rejectReq(uid) {
            db.ref('requests/'+myUser+'/'+uid).update({status: 'rejected'});
            db.ref('requests/'+uid+'/'+myUser).update({status: 'rejected'});
        }
        function deleteRequest(uid) {
            if(confirm("Delete this request record?")) {
                db.ref('requests/'+myUser+'/'+uid).remove();
            }
        }
        function confirmDeleteChat(room, peerName) {
            if(confirm(`Delete chat and remove ${peerName} from your list?`)) {
                db.ref('peers/'+myUser+'/'+peerName).remove();
                // Optionally remove messages: db.ref('messages/'+room).remove();
            }
        }

        // --- Chat Logic ---
        function openChat(room, name, type='normal') {
            if(activeRoom) { db.ref('messages/'+activeRoom).off(); db.ref('private_messages/'+activeRoom).off(); }

            activeRoom = room; activeRoomType = type;
            window.lastRead[room] = Date.now(); localStorage.setItem('xit_reads', JSON.stringify(window.lastRead));
            let rowEl = document.getElementById('chat-row-'+room); if(rowEl) rowEl.classList.remove('unread');

            document.getElementById('active-name').innerText = name;
            
            // UI Toggles
            let chatWin = document.getElementById('chat-window');
            chatWin.classList.toggle('private', type === 'private');
            document.getElementById('pr-share-btn').style.display = (type === 'private') ? 'flex' : 'none';
            document.getElementById('pr-exit-btn').style.display = (type === 'private') ? 'flex' : 'none';
            
            // Bar background handling for private
            document.getElementById('chat-input-bar').style.background = (type === 'private') ? '#1a1a1a' : '#f0f2f5';

            chatWin.classList.add('open'); history.pushState({page: 'chat'}, "");
            
            const refPath = type === 'private' ? 'private_messages/'+room : 'messages/'+room;
            db.ref(refPath).on('value', snap => {
                const flow = document.getElementById('msg-flow'); flow.innerHTML = "";
                snap.forEach(ms => {
                    const m = ms.val(); const div = document.createElement('div');
                    div.className = `bubble ${m.from === myUser ? 'sent' : 'recv'}`;
                    
                    // Delete prompt on click
                    div.onclick = () => { if(m.from === myUser && confirm("Delete your message?")) db.ref(refPath+'/'+ms.key).remove(); };
                    
                    let content = '';
                    if(type === 'private' && m.from !== myUser) content = `<small style="display:block; opacity:0.6; font-weight:bold; margin-bottom:5px; font-size:11px;">~ ${m.from}</small>`;
                    
                    if(m.type === 'image') content += `<img src="${m.data}" style="width:100%; border-radius:5px; margin-top:5px; pointer-events:none;">`;
                    else content += urlify(m.data); // URL Parse!
                    
                    div.innerHTML = content; flow.appendChild(div);
                });
                flow.scrollTop = flow.scrollHeight;
                window.lastRead[room] = Date.now(); localStorage.setItem('xit_reads', JSON.stringify(window.lastRead));
            });
        }

        function sendMsg() {
            const inp = document.getElementById('msg-input');
            const val = inp.value.trim(); if(!val) return;
            inp.value = "";
            db.ref((activeRoomType==='private'?'private_messages/':'messages/')+activeRoom).push().set({from:myUser, data:val, ts:Date.now()});
        }

        function sendMedia() {
            const fileInput = document.getElementById('media-input');
            const file = fileInput.files[0]; if(!file) return;
            const r = new FileReader();
            r.onload = (e) => {
                const img = new Image();
                img.onload = () => {
                    const canvas = document.createElement('canvas'); const MAX_WIDTH = 800;
                    const scaleSize = MAX_WIDTH / img.width;
                    canvas.width = MAX_WIDTH; canvas.height = img.height * scaleSize;
                    const ctx = canvas.getContext('2d'); ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
                    const compressedDataUrl = canvas.toDataURL('image/jpeg', 0.6);
                    db.ref((activeRoomType==='private'?'private_messages/':'messages/')+activeRoom).push().set({from:myUser, data:compressedDataUrl, type:'image', ts:Date.now()});
                }
                img.src = e.target.result;
            };
            r.readAsDataURL(file); fileInput.value = '';
        }

        function closeChat(doBack=true) {
            document.getElementById('chat-window').classList.remove('open');
            if(activeRoom) { db.ref('messages/'+activeRoom).off(); db.ref('private_messages/'+activeRoom).off(); }
            activeRoom = null; if(doBack) history.back();
        }

        // --- Status & Delete Status ---
        function viewStatus(u, t, tm, sKey) {
            document.getElementById('st-avatar').innerText = u[0];
            document.getElementById('st-user').innerText = "@" + u;
            document.getElementById('st-time').innerText = tm;
            document.getElementById('st-text').innerText = urlify(t);
            currentActiveStatusKey = sKey;
            
            document.getElementById('delete-st-btn').style.display = (u === myUser) ? 'inline-block' : 'none';
            document.getElementById('status-viewer').style.display = 'flex';
            history.pushState({page: 'status'}, "");
        }
        function deleteMyStatus(e) {
            e.stopPropagation();
            if(confirm("Delete this status?") && currentActiveStatusKey) {
                db.ref('statuses/'+currentActiveStatusKey).remove();
                closeStatus();
            }
        }
        function closeStatus(doBack=true) { 
            document.getElementById('status-viewer').style.display = 'none'; currentActiveStatusKey = null;
            if(doBack) history.back();
        }

        // --- Fab Logic ---
        function handleFab() {
            if(currentTab === 0 || currentTab === 2) {
                showModal("🔍 Send Request", `<input id="t-user" placeholder="Search exact username...">`, () => {
                    const t = document.getElementById('t-user').value.toLowerCase().trim();
                    if(t) sendFriendRequest(t);
                });
            } else if(currentTab === 1) {
                showModal("New Status", `<input id="s-txt" placeholder="What's on your mind?">`, async () => {
                    const s = document.getElementById('s-txt').value;
                    if(s) { await db.ref('statuses').push({user:myUser, text:s, ts:Date.now()}); closeModal(); }
                });
            }
        }

        // --- Private Room Menus ---
        function showPrivateRoomMenu() {
            toggleSidebar(false);
            showModal("🔒 Private Rooms", `
                <button class="btn-primary" onclick="openCreateRoomUI()">➕ Create New Room</button>
                <button class="btn-secondary" onclick="openJoinRoomUI()">📥 Join Existing Room</button>
            `, null, false);
        }
        function openCreateRoomUI() {
            showModal("Create Private Room", `<input type="text" id="pr-name" placeholder="Enter Room Name"><input type="password" id="pr-pass" placeholder="Create Password">`, async () => {
                const n = document.getElementById('pr-name').value.trim(), p = document.getElementById('pr-pass').value.trim();
                if(!n || !p) return;
                if((await db.ref('private_rooms/'+n).once('value')).exists()) return alert("Room Name taken!");
                await db.ref('private_rooms/'+n).set({password: p, users: {[myUser]: true}});
                closeModal(); openChat(n, "🔒 " + n, 'private');
            });
        }
        function openJoinRoomUI() {
            showModal("Join Private Room", `<input type="text" id="pr-name" placeholder="Room Name"><input type="password" id="pr-pass" placeholder="Password">`, async () => {
                const n = document.getElementById('pr-name').value.trim(), p = document.getElementById('pr-pass').value.trim();
                if(!n || !p) return;
                const snap = await db.ref('private_rooms/'+n).once('value');
                if(!snap.exists() || snap.val().password !== p) return alert("Invalid details!");
                await db.ref('private_rooms/'+n+'/users/'+myUser).set(true);
                closeModal(); openChat(n, "🔒 " + n, 'private');
            });
        }
        function exitPrivateRoom() {
            const roomToExit = activeRoom;
            showModal("Exit Room", `
                <p style="margin-bottom:20px; color:#333;">Exit <b>${roomToExit}</b>?</p>
                <label style="display:flex; align-items:center; gap:10px; cursor:pointer; background:#f9f9f9; padding:15px; border-radius:8px; border:1px solid #ddd;">
                    <input type="checkbox" id="clear-msgs-cb" style="width:20px; height:20px;" checked> <b>Clear my messages</b>
                </label>
            `, async () => {
                const clear = document.getElementById('clear-msgs-cb').checked;
                closeModal(); closeChat(); 
                if(clear) {
                    const snap = await db.ref('private_messages/'+roomToExit).once('value');
                    const updates = {}; snap.forEach(m => { if(m.val().from === myUser) updates[m.key] = null; });
                    if(Object.keys(updates).length > 0) await db.ref('private_messages/'+roomToExit).update(updates);
                }
                await db.ref('private_rooms/'+roomToExit+'/users/'+myUser).remove();
                const usersSnap = await db.ref('private_rooms/'+roomToExit+'/users').once('value');
                if(!usersSnap.exists() || Object.keys(usersSnap.val() || {}).length === 0) {
                    db.ref('private_rooms/'+roomToExit).remove(); db.ref('private_messages/'+roomToExit).remove();
                }
            });
        }

        // --- Sharing ---
        function getRoomLink() { return window.location.origin + window.location.pathname + "?room=" + encodeURIComponent(activeRoom); }
        function openShareScreen() {
            const list = document.getElementById('share-internal-list'); list.innerHTML = "";
            const url = getRoomLink();
            if(Object.keys(window.myPeers).length === 0) list.innerHTML = "<p style='text-align:center; padding:20px; color:#777;'>Add users to share directly.</p>";
            else {
                Object.values(window.myPeers).forEach(p => {
                    const row = document.createElement('div'); row.className = "chat-row";
                    row.innerHTML = `<div class="avatar" style="width:40px; height:40px; font-size:14px;">${p.peer[0]}</div><b style="flex:1;">${p.peer}</b><button style="background:black; color:white; border:none; padding:8px 15px; border-radius:20px; font-weight:bold; font-size:12px;">Send</button>`;
                    row.onclick = () => { db.ref('messages/'+p.room).push().set({from:myUser, data:`Join my secure room: ${url}`, ts:Date.now()}); alert(`Sent to ${p.peer}!`); };
                    list.appendChild(row);
                });
            }
            document.getElementById('share-screen').classList.add('active'); history.pushState({page: 'share'}, "");
        }
        function closeShareScreen(doBack=true) { document.getElementById('share-screen').classList.remove('active'); if(doBack) history.back(); }
        function copyRoomLink() { navigator.clipboard.writeText(getRoomLink()).then(() => alert("Link Copied!")); }
        function shareViaWhatsApp() { window.open('whatsapp://send?text=' + encodeURIComponent('Join private room: ' + getRoomLink())); }
        function shareViaDeviceNative() {
            if (navigator.share) navigator.share({ title: 'XIT Private Room', url: getRoomLink() }).catch(console.error);
            else alert("Native sharing not supported.");
        }

        function toggleSidebar(o) { document.getElementById('sidebar').classList.toggle('open', o); document.getElementById('overlay').style.display = o ? 'block' : 'none'; }
        function showChangePass() {
            showModal("Change Password", `<input type="password" id="old-p" placeholder="Old Password"><input type="password" id="new-p" placeholder="New Password">`, async () => {
                const snap = await db.ref('registered_users/'+myUser).once('value');
                if(snap.val().password === document.getElementById('old-p').value) {
                    await db.ref('registered_users/'+myUser+'/password').set(document.getElementById('new-p').value);
                    alert("Success!"); closeModal();
                } else alert("Wrong Password");
            });
        }

        if(!myUser) document.getElementById('login-screen').style.display='flex';
        else { document.getElementById('side-user-info').innerText = "@"+myUser; document.getElementById('my-avatar').innerText = myUser[0]; startApp(); }
    </script>
</body>
</html>
