
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
        .modal-overlay { position: fixed; top:0; left:0; width:100%; height:100%; background: var(--modal-bg); z-index: 9000; display:none; align-items:flex-end; justify-content:center; transition: opacity 0.3s; cursor: pointer; }
        .modal-card { background: white; width: 100%; max-width: 600px; border-radius: 20px 20px 0 0; padding: 25px 20px; box-shadow: 0 -10px 25px rgba(0,0,0,0.2); transform: translateY(100%); transition: transform 0.3s cubic-bezier(0.175, 0.885, 0.32, 1); cursor: default; max-height: 85vh; overflow-y: auto; margin: 0 auto; }
        .modal-overlay.active { display: flex; opacity: 1; }
        .modal-overlay.active .modal-card { transform: translateY(0); }
        .modal-card input, .modal-card select { width: 100%; padding: 12px; margin: 10px 0; border: 1px solid #ddd; border-radius: 8px; outline: none; transition: 0.2s; font-size:15px; }
        .modal-card input:focus, .modal-card select:focus { border-color: black; box-shadow: 0 0 5px rgba(0,0,0,0.2); }

        /* Buttons */
        button { cursor: pointer; transition: transform 0.1s, background 0.2s; }
        button:active { transform: scale(0.95); }
        .btn-primary { background: black; color: white; border: none; padding: 12px; border-radius: 8px; width: 100%; font-weight: bold; margin-bottom: 10px; font-size: 15px; }
        .btn-secondary { background: #eee; color: black; border: none; padding: 12px; border-radius: 8px; width: 100%; font-weight: bold; font-size: 15px; margin-bottom: 10px;}
        .btn-danger { background: #ff4d4d; color: white; border: none; padding: 12px; border-radius: 8px; width: 100%; font-weight: bold; font-size: 15px; margin-bottom: 10px;}

        /* Header & Tabs */
        header { background: var(--xit-black); color: white; z-index: 100; position: relative; box-shadow: 0 2px 10px rgba(0,0,0,0.2); }
        .top-bar { display: flex; align-items: center; padding: 15px; gap: 15px; font-weight: bold; font-size: 20px; }
        .tabs { display: flex; background: var(--xit-black); }
        .tab { flex: 1; padding: 12px 0; text-align: center; font-size: 14px; font-weight: bold; opacity: 0.6; border-bottom: 3px solid transparent; color: white; cursor: pointer; transition: 0.2s; }
        .tab.active { opacity: 1; border-bottom: 3px solid white; }

        /* Swipe Container */
        .view-container { display: flex; width: 300%; height: calc(100vh - 105px); transition: transform 0.2s ease-out; will-change: transform; }
        .view { width: 33.333%; height: 100%; overflow-y: auto; padding-bottom: 80px; display: flex; flex-direction: column; }

        /* Sidebar Theme */
        #sidebar { position: fixed; left: -285px; top: 0; width: 280px; height: 100%; background: white; z-index: 2000; transition: 0.2s ease-in-out; box-shadow: 2px 0 15px rgba(0,0,0,0.3); }
        #sidebar.open { left: 0; }
        .sidebar-header { background: var(--xit-black); color: white; padding: 50px 20px 20px; }
        .menu-item { padding: 15px 20px; border-bottom: 1px solid #eee; cursor: pointer; color: #333; font-weight: 500; transition: 0.2s; }
        .menu-item:hover { background: #f5f5f5; padding-left: 25px; }
        #overlay { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.6); z-index: 1900; animation: fadeIn 0.2s; cursor:pointer;}

        /* Chat UI & Swipe Wrappers */
        #chat-window { position: fixed; top: 0; left: 100%; width: 100%; height: 100%; background: var(--wa-chat-bg) url('https://user-images.githubusercontent.com/15075759/28719144-86dc0f70-73b1-11e7-911d-60d70fcded21.png'); z-index: 1500; display: flex; flex-direction: column; transition: 0.2s ease-in-out; }
        #chat-window.open { left: 0; }
        #chat-window.private { background: #000000 !important; }
        .chat-header { background: var(--xit-black); color: white; padding: 10px 15px; display: flex; align-items: center; gap: 10px; box-shadow: 0 2px 5px rgba(0,0,0,0.2); z-index: 10; border-bottom: 1px solid #333;}
        
        .msg-flow { flex: 1; overflow-y: auto; padding: 15px; display: flex; flex-direction: column; gap: 10px; z-index: 2; position: relative; overflow-x: hidden;}
        
        .msg-wrapper { display:flex; align-items:center; width:100%; position:relative; }
        .sent-wrap { justify-content:flex-end; }
        .recv-wrap { justify-content:flex-start; }

        .bubble { max-width: 80%; padding: 8px 12px 4px 12px; border-radius: 10px; font-size: 14px; position: relative; word-wrap: break-word; animation: popIn 0.2s cubic-bezier(0.175, 0.885, 0.32, 1.275) forwards; opacity: 0; z-index: 3; transition: transform 0.1s linear; display: flex; flex-direction: column; min-width: 80px; }
        .bubble img, .bubble video { width: 100%; border-radius: 5px; margin-top: 5px; margin-bottom: 5px; cursor: pointer; pointer-events: auto; }
        .bubble a { color: #0066cc; text-decoration: underline; font-weight: 500; }
        .bubble audio { width: 100%; min-width: 200px; margin-top:5px; outline: none; }
        #chat-window.private .bubble a { color: #4da6ff; }
        
        /* Time & Date Styling inside Bubble */
        .msg-time { font-size: 10px; margin-top: 5px; text-align: right; opacity: 0.6; align-self: flex-end; white-space: nowrap; }
        
        .sent { background: var(--wa-sent); transform-origin: bottom right; }
        .recv { background: var(--wa-recv); transform-origin: bottom left; }
        #chat-window.private .sent { background: #0f331b; color: white; }
        #chat-window.private .recv { background: #1a1a1a; color: white; }

        .reply-block { background: rgba(0,0,0,0.05); padding: 5px 8px; border-left: 4px solid #25D366; border-radius: 5px; margin-bottom: 5px; font-size: 12px; cursor: pointer; opacity: 0.8; }
        #chat-window.private .reply-block { background: rgba(255,255,255,0.1); }
        .swipe-ind { position:absolute; left:10px; display:flex; align-items:center; justify-content:center; opacity:0; pointer-events:none; z-index:1; }

        /* Reply Input Preview Bar */
        #reply-preview { display:none; background:#eee; padding:10px 15px; border-left:4px solid #25D366; position:relative; z-index: 11; box-shadow: 0 -2px 10px rgba(0,0,0,0.05);}
        #chat-window.private #reply-preview { background:#111; color:white; border-color: #25D366;}
        #reply-preview .close-btn { position:absolute; right:15px; top:15px; cursor:pointer; font-weight:bold; color:red; font-size:18px;}
        
        /* List Rows */
        .chat-row, .status-row, .req-row, .history-row { display: flex; align-items: center; padding: 12px 15px; border-bottom: 1px solid #f2f2f2; cursor: pointer; transition: 0.2s; animation: slideUp 0.2s ease-out forwards; }
        .chat-row:hover, .status-row:hover, .req-row:hover, .history-row:hover { background: #f9f9f9; }
        .chat-row.unread b { color: #25D366; } 
        .chat-row.unread { background: #f0fff4; }
        .avatar { width: 50px; height: 50px; border-radius: 50%; background: #444; display: flex; align-items: center; justify-content: center; color: white; margin-right: 15px; font-weight: bold; text-transform: uppercase; box-shadow: 0 2px 5px rgba(0,0,0,0.2); flex-shrink: 0;}
        
        .fab { position: fixed; bottom: 25px; right: 25px; width: 56px; height: 56px; background: var(--xit-black); border-radius: 50%; display: flex; align-items: center; justify-content: center; color: white; font-size: 24px; border:none; z-index: 90; box-shadow: 0 4px 12px rgba(0,0,0,0.4); transition: 0.2s; }
        .fab:hover { transform: scale(1.1) rotate(90deg); }
        
        #login-screen { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: var(--xit-black); z-index: 9999; display: none; flex-direction: column; align-items: center; justify-content: center; color: white; }
        #status-viewer { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: #000; color: white; z-index: 8000; display: none; flex-direction: column; align-items: center; justify-content: center; animation: fadeIn 0.2s; }

        /* Full Screen Media Viewer */
        #image-viewer { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.95); z-index: 9999; display: none; flex-direction: column; align-items: center; justify-content: center; animation: fadeIn 0.2s; }
        #image-viewer img, #image-viewer video { max-width: 95%; max-height: 75vh; border-radius: 10px; box-shadow: 0 0 25px rgba(255,255,255,0.1); margin-bottom: 20px;}
        
        .fwd-item { display:flex; align-items:center; gap:10px; padding:12px; border-bottom:1px solid #eee; cursor:pointer; }
        .fwd-item input { width:22px; height:22px; margin:0;}

        /* Private Room Big Watermark */
        .watermark { position: absolute; top: 0; left: 0; width: 100%; height: 100%; display: none; align-items: center; justify-content: center; flex-direction: column; pointer-events: none; z-index: 1; opacity: 0.08; color: white; text-align: center; }
        #chat-window.private .watermark { display: flex; }

        /* Real-time Calling Screen */
        #call-screen { position:fixed; top:0; left:0; width:100%; height:100%; background:#111; z-index:9999; display:none; flex-direction:column; color:white; animation: fadeIn 0.2s;}
        #remote-video { width:100%; height:100%; object-fit:cover; background:#000; }
        #local-video { position:absolute; bottom:140px; right:20px; width:100px; height:140px; object-fit:cover; border-radius:15px; border:2px solid white; box-shadow:0 5px 15px rgba(0,0,0,0.5); background:#333; z-index:2; }
        
        /* Audio Only Call UI */
        .audio-avatar-wrapper { display:none; flex-direction:column; align-items:center; justify-content:center; height:100%; width:100%; background:#1a1a1a; position:absolute; top:0; left:0; z-index:1; }
        .audio-avatar { width: 150px; height: 150px; border-radius: 50%; background: #444; display: flex; align-items: center; justify-content: center; font-size: 60px; color: white; box-shadow: 0 0 30px rgba(37, 211, 102, 0.5); text-transform: uppercase; }

        .call-controls { position:absolute; bottom:30px; left:0; width:100%; display:flex; justify-content:center; gap:12px; z-index:10; padding: 0 10px;}
        .call-btn { width:55px; height:55px; border-radius:50%; border:none; display:flex; align-items:center; justify-content:center; font-size:22px; color:white; cursor:pointer; box-shadow:0 4px 10px rgba(0,0,0,0.3); }
        .call-btn.end { background:#ff4d4d; width: 65px; height: 65px; font-size: 26px; }
        .call-btn.mute { background:rgba(255,255,255,0.2); backdrop-filter:blur(5px); }
        .call-status-text { position:absolute; top:60px; width:100%; text-align:center; font-size:22px; z-index:10; text-shadow: 0 2px 4px rgba(0,0,0,0.8); font-weight:bold;}
        .call-timer-text { position:absolute; top:100px; width:100%; text-align:center; font-size:16px; z-index:10; color:#ddd; }

        /* Keyframes */
        @keyframes popIn { from { opacity: 0; transform: translateY(10px) scale(0.9); } to { opacity: 1; transform: translateY(0) scale(1); } }
        @keyframes slideUp { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
        @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
        
        .req-section-title { padding: 15px 20px 5px; font-weight: bold; color: #555; font-size: 13px; text-transform: uppercase; background: #fafafa; border-bottom: 1px solid #eee; }
    </style>
</head>
<body>

    <div id="modal-container" class="modal-overlay" onclick="closeModal()">
        <div class="modal-card" id="modal-card" onclick="event.stopPropagation()">
            <h3 id="modal-title" style="color:black; margin-top:0; border-bottom: 1px solid #eee; padding-bottom: 10px;"></h3>
            <div id="modal-body"></div>
            <div class="modal-btns" id="modal-btns-wrapper" style="display:flex; justify-content:flex-end; gap:10px; margin-top:15px;">
                <button onclick="closeModal()" style="border:none; background:#eee; padding:10px 20px; border-radius:8px; font-weight:bold; font-size:15px;">Cancel</button>
                <button id="modal-confirm-btn" style="border:none; background:black; color:white; padding:10px 20px; border-radius:8px; font-weight:bold; font-size:15px;">Confirm</button>
            </div>
        </div>
    </div>

    <div id="image-viewer">
        <span onclick="document.getElementById('image-viewer').style.display='none'; document.getElementById('iv-video').pause();" style="position:absolute; top:20px; right:30px; color:white; font-size:40px; cursor:pointer; font-weight:bold; z-index:10000;">&times;</span>
        <img id="iv-img" src="" style="display:none;">
        <video id="iv-video" src="" controls style="display:none;"></video>
        <a id="iv-download" class="btn-primary" style="width:auto; padding:15px 30px; text-decoration:none;" href="" target="_blank" download="xit_media">📥 Download Media</a>
    </div>

    <div id="call-screen">
        <div class="call-status-text" id="call-status-text">Calling...</div>
        <div class="call-timer-text" id="call-timer-text"></div>
        
        <video id="remote-video" autoplay playsinline></video>
        <video id="local-video" autoplay playsinline muted></video>
        
        <div class="audio-avatar-wrapper" id="audio-avatar-wrap">
            <div class="audio-avatar" id="audio-avatar-letter">A</div>
        </div>

        <div class="call-controls">
            <button class="call-btn mute" id="toggle-mic-btn" onclick="toggleMute()" title="Mute/Unmute">🎤</button>
            <button class="call-btn mute" id="toggle-vid-btn" onclick="toggleVideo()" title="Video On/Off">📹</button>
            <button class="call-btn end" onclick="endCall()" title="End Call">⛔</button>
            <button class="call-btn mute" id="flip-cam-btn" onclick="flipCamera()" title="Flip Camera">🔄</button>
            <button class="call-btn mute" id="toggle-speaker-btn" onclick="toggleSpeaker()" title="Speaker On/Off">🔊</button>
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
        <div class="menu-item" onclick="showCallHistory()">📞 Call History</div>
        <div class="menu-item" onclick="showChatBgSettings()">🖼️ Chat Background</div>
        <div class="menu-item" onclick="showSwipeSettings()">⚙️ Swipe/Line Settings</div>
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
            
            <button id="btn-audio-call" onclick="startCall('audio')" title="Audio Call" style="background:transparent; color:white; border:none; font-size:20px; padding:5px; display:none;">📞</button>
            <button id="btn-video-call" onclick="startCall('video')" title="Video Call" style="background:transparent; color:white; border:none; font-size:20px; padding:5px; display:none;">📹</button>
            
            <button id="pr-share-btn" onclick="openShareLinkModal()" title="Share Link" style="display:none; background:white; color:black; border:none; border-radius:50%; width:35px; height:35px; align-items:center; justify-content:center; font-size:16px; margin-left:5px;">🔗</button>
            <button id="pr-exit-btn" onclick="exitPrivateRoom()" title="Exit Room" style="display:none; background:#ff4d4d; color:white; border:none; border-radius:50%; width:35px; height:35px; align-items:center; justify-content:center; font-size:16px; margin-left:5px;">🚪</button>
        </div>
        
        <svg id="reply-svg-layer" style="position:absolute; top:0; left:0; width:100%; height:100%; pointer-events:none; z-index:1; display:none;"></svg>

        <div class="msg-flow" id="msg-flow"></div>
        
        <div style="display:flex; flex-direction:column; z-index:10;" id="chat-input-bar">
            <div id="upload-indicator" style="display:none; padding:8px 15px; background:#fff3cd; color:#155724; font-size:13px; font-weight:bold; text-align:center;">Preparing file for free transfer... ⏳</div>
            
            <div id="reply-preview">
                <div class="close-btn" onclick="cancelReply()">×</div>
                <small style="font-weight:bold; color:#25D366" id="reply-to-name"></small><br>
                <div id="reply-to-text" style="font-size:13px; opacity:0.8; white-space:nowrap; overflow:hidden; text-overflow:ellipsis;"></div>
            </div>
            
            <div style="padding:10px; display:flex; gap:8px; background:#f0f2f5; border-top:1px solid #ddd;" id="chat-input-row">
                <label style="font-size:24px; cursor:pointer; display:flex; align-items:center; padding-left:5px;">
                    <input type="file" id="media-input" hidden onchange="sendMedia()">📎
                </label>
                <input type="text" id="msg-input" placeholder="Type a message..." onkeydown="if(event.key==='Enter') sendMsg()" style="flex:1; border-radius:20px; border:none; padding:12px 18px; outline:none; box-shadow:inset 0 1px 3px rgba(0,0,0,0.1); font-size:15px;">
                <button onclick="sendMsg()" style="background:black; border:none; color:white; border-radius:50%; width:45px; height:45px; display:flex; align-items:center; justify-content:center; box-shadow:0 2px 5px rgba(0,0,0,0.2); font-size:18px;">➔</button>
            </div>
        </div>
    </div>

    <div id="share-screen" style="position: fixed; top: 100%; left: 0; width: 100%; height: 100%; background: white; z-index: 9500; display: flex; flex-direction: column; transition: top 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);">
        <div class="share-header" style="background: var(--xit-black); color: white; padding: 15px; display: flex; align-items: center; font-size: 18px; font-weight: bold;">
            <span onclick="closeShareScreen()" style="font-size:26px; cursor:pointer; margin-right:15px;">←</span>
            <span>Share Room Link</span>
        </div>
        <div class="share-list" id="share-internal-list" style="flex: 1; overflow-y: auto; padding: 10px 0;"></div>
        <div class="share-external" style="padding: 20px; background: #f9f9f9; border-top: 1px solid #ddd;">
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
        let activeRoom = null; let activeRoomType = 'normal'; let activePeerName = null;
        let currentTab = 0; let currentActiveStatusKey = null; 
        
        let swipeSettings = JSON.parse(localStorage.getItem('xit_swipe_set')) || { style: 'normal', color: '#ff4d4d', width: 4, autoColor: true };
        let replyingToContext = null; let actionMsgContext = null; 
        
        window.myPeers = {}; window.myPrivateRooms = {};
        window.lastRead = JSON.parse(localStorage.getItem('xit_reads') || '{}');

        // Background Logic
        function applySavedBg() {
            let bg = JSON.parse(localStorage.getItem('xit_chat_bg'));
            let cw = document.getElementById('chat-window');
            if(!bg || bg.type === 'default') cw.style.background = '';
            else if(bg.type === 'color') cw.style.background = bg.value;
            else if(bg.type === 'image') cw.style.background = `url(${bg.value}) center/cover no-repeat`;
        }
        applySavedBg();

        function showChatBgSettings() {
            toggleSidebar(false);
            let html = `
                <div style="margin-bottom:10px;">
                    <label style="font-weight:bold;">Select Background Type:</label>
                    <select id="bg-type" onchange="document.getElementById('bg-col-wrap').style.display=this.value==='color'?'block':'none'; document.getElementById('bg-img-wrap').style.display=this.value==='image'?'block':'none';">
                        <option value="default">Default Pattern</option>
                        <option value="color">Solid Color</option>
                        <option value="image">Custom Image</option>
                    </select>
                </div>
                <div id="bg-col-wrap" style="display:none; margin-bottom:10px;">
                    <label>Choose Color:</label><input type="color" id="bg-color" value="#ffffff" style="height:40px; padding:0;">
                </div>
                <div id="bg-img-wrap" style="display:none; margin-bottom:10px;">
                    <label>Select Image (From Gallery):</label><input type="file" id="bg-file" accept="image/*">
                </div>
            `;
            showModal("🖼️ Chat Background", html, () => {
                let type = document.getElementById('bg-type').value;
                if(type === 'default') {
                    localStorage.setItem('xit_chat_bg', JSON.stringify({type:'default'}));
                    applySavedBg(); closeModal(); alert("Background Reset to Default!");
                } else if(type === 'color') {
                    localStorage.setItem('xit_chat_bg', JSON.stringify({type:'color', value: document.getElementById('bg-color').value}));
                    applySavedBg(); closeModal(); alert("Solid Color Applied!");
                } else if(type === 'image') {
                    let file = document.getElementById('bg-file').files[0];
                    if(!file) return alert("Please select an image file first.");
                    let r = new FileReader();
                    r.onload = function(e) {
                        let img = new Image();
                        img.onload = () => {
                            let canvas = document.createElement('canvas');
                            let maxW = 1080; let scale = Math.min(maxW/img.width, 1);
                            canvas.width = img.width * scale; canvas.height = img.height * scale;
                            let ctx = canvas.getContext('2d'); ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
                            let data = canvas.toDataURL('image/jpeg', 0.5);
                            try {
                                localStorage.setItem('xit_chat_bg', JSON.stringify({type:'image', value: data}));
                                applySavedBg(); closeModal(); alert("Background Image Applied!");
                            } catch(err) { alert("Image size is too large."); }
                        };
                        img.src = e.target.result;
                    };
                    r.readAsDataURL(file);
                }
            });
        }

        function getRandomColor() {
            const letters = '0123456789ABCDEF'; let color = '#';
            for (let i = 0; i < 6; i++) color += letters[Math.floor(Math.random() * 16)];
            return color;
        }

        function urlify(text) {
            const urlRegex = /(https?:\/\/[^\s]+)/g;
            return text.replace(urlRegex, function(url) {
                return `<a href="${url}" target="_blank" onclick="event.stopPropagation()">${url}</a>`;
            });
        }

        function makeLongPressable(el, onLongPress, onClick) {
            let pressTimer = null; let isDragging = false; let startX, startY;
            el.addEventListener('touchstart', e => { 
                startX = e.touches[0].clientX; startY = e.touches[0].clientY; isDragging = false;
                pressTimer = setTimeout(() => { onLongPress(); pressTimer=null; }, 500); 
            }, {passive:true});
            el.addEventListener('touchmove', e => { 
                if(Math.abs(e.touches[0].clientX - startX) > 10 || Math.abs(e.touches[0].clientY - startY) > 10) { isDragging = true; if(pressTimer) clearTimeout(pressTimer); }
            }, {passive:true});
            el.addEventListener('touchend', e => { if(pressTimer) { clearTimeout(pressTimer); if(!isDragging && onClick) onClick(e); } });
            el.addEventListener('mousedown', e => { pressTimer = setTimeout(() => { onLongPress(); pressTimer=null; }, 500); });
            el.addEventListener('mousemove', () => { if(pressTimer) clearTimeout(pressTimer); });
            el.addEventListener('mouseup', e => { if(pressTimer) { clearTimeout(pressTimer); if(onClick && e.button===0) onClick(e); } });
            el.addEventListener('contextmenu', e => { e.preventDefault(); onLongPress(); });
        }

        let touchStartX = 0; const container = document.getElementById('main-container');
        container.addEventListener('touchstart', e => { touchStartX = e.touches[0].clientX; }, {passive: true});
        container.addEventListener('touchend', e => {
            let diff = touchStartX - e.changedTouches[0].clientX;
            if (Math.abs(diff) > 60) {
                if (diff > 0 && currentTab < 2) moveTab(currentTab + 1);
                else if (diff < 0 && currentTab > 0) moveTab(currentTab - 1);
            }
        }, {passive: true});

        function moveTab(i) {
            currentTab = i; container.style.transform = `translateX(-${i * 33.333}%)`;
            document.getElementById('tab0').classList.toggle('active', i === 0);
            document.getElementById('tab1').classList.toggle('active', i === 1);
            document.getElementById('tab2').classList.toggle('active', i === 2);
        }

        history.replaceState({page: 'home'}, "");
        window.onpopstate = () => {
            if (document.getElementById('modal-container').classList.contains('active')) closeModal();
            else if (document.getElementById('image-viewer').style.display === 'flex') { document.getElementById('image-viewer').style.display = 'none'; document.getElementById('iv-video').pause(); }
            else if (document.getElementById('share-screen').classList.contains('active')) closeShareScreen(false);
            else if (document.getElementById('status-viewer').style.display === 'flex') closeStatus(false);
            else if (document.getElementById('call-screen').style.display === 'flex') endCall(); 
            else if (document.getElementById('chat-window').classList.contains('open')) closeChat(false);
            else if (document.getElementById('sidebar').classList.contains('open')) toggleSidebar(false);
        };

        function showModal(title, body, confirmFn, showButtons = true) {
            document.getElementById('modal-title').innerText = title;
            document.getElementById('modal-body').innerHTML = body;
            const btnWrapper = document.getElementById('modal-btns-wrapper');
            if(showButtons) { btnWrapper.style.display = 'flex'; document.getElementById('modal-confirm-btn').onclick = confirmFn; } 
            else btnWrapper.style.display = 'none';
            document.getElementById('modal-container').classList.add('active');
        }
        function closeModal() { document.getElementById('modal-container').classList.remove('active'); }

        async function performLogin() {
            const name = document.getElementById('user-input').value.trim().toLowerCase();
            const pass = document.getElementById('pass-input').value.trim();
            if(!name || !pass) return;
            const snap = await db.ref('registered_users/'+name).once('value');
            if(snap.exists()){
                if(snap.val().password === pass) login(name); else alert("Wrong Password");
            } else {
                await db.ref('registered_users/'+name).set({password:pass, ts:Date.now()}); login(name);
            }
        }
        function login(n){ localStorage.setItem('xit_username', n); location.reload(); }
        function logout(){ localStorage.clear(); location.reload(); }

        async function startApp() {
            listenForCalls();

            const params = new URLSearchParams(window.location.search); const linkRoom = params.get('room');
            if(linkRoom) {
                showModal("Join Secure Room", `<p style="color:#555; margin-bottom:15px;">Enter password to unlock room <b>${linkRoom}</b>.</p><input type="password" id="link-p" placeholder="Room Password">`, async () => {
                    const p = document.getElementById('link-p').value;
                    const snap = await db.ref('private_rooms/'+linkRoom).once('value');
                    if(!snap.exists() || snap.val().password !== p) return alert("Incorrect Password or Room doesn't exist!");
                    await db.ref('private_rooms/'+linkRoom+'/users/'+myUser).set(true);
                    window.history.replaceState({}, document.title, window.location.pathname);
                    closeModal(); openChat(linkRoom, "🔒 " + linkRoom, 'private');
                });
            }

            db.ref('peers/'+myUser).on('value', snap => { window.myPeers = snap.val() || {}; renderChatList(); });
            db.ref('private_rooms').on('value', snap => { window.myPrivateRooms = snap.val() || {}; renderChatList(); });
            db.ref('requests/'+myUser).on('value', snap => renderRequests(snap.val() || {}));
            db.ref('statuses').on('value', snap => {
                const list = document.getElementById('status-list'); list.innerHTML = ""; let arr = [];
                snap.forEach(s => {
                    const d = s.val(); d.key = s.key;
                    if(Date.now() - d.ts > 86400000) { db.ref('statuses/'+s.key).remove(); return; }
                    if((d.user === myUser) || Object.values(window.myPeers).some(p => p.peer === d.user)) arr.push(d);
                });
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

        function updateRoomMeta(room, isPrivate) {
            db.ref((isPrivate?'private_messages/':'messages/')+room).limitToLast(1).on('value', snap => {
                if(snap.exists()) {
                    let lastMsg = Object.values(snap.val())[0]; let el = document.getElementById('chat-row-'+room);
                    if(el) {
                        el.style.order = -lastMsg.ts;
                        if(lastMsg.ts > (window.lastRead[room]||0) && lastMsg.from !== myUser) el.classList.add('unread');
                        else el.classList.remove('unread');
                    }
                }
            });
        }

        function renderChatList() {
            const list = document.getElementById('chat-list'); list.innerHTML = "";
            Object.keys(window.myPrivateRooms).forEach(roomName => {
                if(window.myPrivateRooms[roomName].users && window.myPrivateRooms[roomName].users[myUser]) {
                    const row = document.createElement('div'); row.className = "chat-row"; row.id = "chat-row-"+roomName;
                    row.innerHTML = `<div class="avatar" style="background:#000;">🔒</div><div style="flex:1;"><b>${roomName}</b><br><small style="color:#25D366; font-weight:bold;">Private Room</small></div>`;
                    makeLongPressable(row, () => showChatActionMenu(roomName, true), () => openChat(roomName, "🔒 " + roomName, 'private'));
                    list.appendChild(row); updateRoomMeta(roomName, true);
                }
            });
            Object.values(window.myPeers).forEach(p => {
                const row = document.createElement('div'); row.className = "chat-row"; row.id = "chat-row-"+p.room;
                row.innerHTML = `<div class="avatar">${p.peer[0]}</div><div style="flex:1;"><b>${p.peer}</b></div>`;
                makeLongPressable(row, () => showChatActionMenu(p.peer, false), () => openChat(p.room, p.peer, 'normal'));
                list.appendChild(row); updateRoomMeta(p.room, false);
            });
        }

        function showChatActionMenu(target, isPrivate) {
            if(isPrivate) showModal("Room Options", `<button class="btn-danger" onclick="exitPrivateRoomFromName('${target}')">🚪 Exit Private Room</button>`, null, false);
            else showModal("Chat Options", `<button class="btn-danger" onclick="deletePeerChat('${target}')">🗑️ Delete Contact & Chat</button>`, null, false);
        }
        function deletePeerChat(peerName) { db.ref('peers/'+myUser+'/'+peerName).remove(); closeModal(); alert("Deleted!"); }

        function renderRequests(reqData) {
            const recvDiv = document.getElementById('received-reqs'); recvDiv.innerHTML = "";
            const sentDiv = document.getElementById('sent-reqs'); sentDiv.innerHTML = "";
            Object.keys(reqData).forEach(uid => {
                const req = reqData[uid];
                const row = document.createElement('div'); row.className = "req-row"; row.style.order = -req.ts;
                makeLongPressable(row, () => { showModal("Request Option", `<button class="btn-danger" onclick="deleteRequest('${uid}')">🗑️ Delete Request History</button>`, null, false); }, null);

                if(req.type === 'received') {
                    row.innerHTML = `<div class="avatar">${uid[0]}</div><div style="flex:1;"><b>${uid}</b><br><small>${req.status}</small></div>`;
                    if(req.status === 'pending') {
                        row.innerHTML += `<button style="background:#25D366; color:white; border:none; padding:5px 10px; border-radius:5px; margin-right:5px; font-weight:bold;" onclick="acceptReq('${uid}'); event.stopPropagation();">Accept</button>
                                          <button style="background:#ff4d4d; color:white; border:none; padding:5px 10px; border-radius:5px; font-weight:bold;" onclick="rejectReq('${uid}'); event.stopPropagation();">Reject</button>`;
                    }
                    recvDiv.appendChild(row);
                } else if(req.type === 'sent') {
                    row.innerHTML = `<div class="avatar">${uid[0]}</div><div style="flex:1;"><b>${uid}</b><br><small>Status: ${req.status}</small></div>
                                     <button style="background:#ddd; color:black; border:none; padding:5px 10px; border-radius:5px;" onclick="deleteRequest('${uid}'); event.stopPropagation();">Cancel</button>`;
                    sentDiv.appendChild(row);
                }
            });
        }
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
        function rejectReq(uid) { db.ref('requests/'+myUser+'/'+uid).update({status: 'rejected'}); db.ref('requests/'+uid+'/'+myUser).update({status: 'rejected'}); }
        function deleteRequest(uid) { db.ref('requests/'+myUser+'/'+uid).remove(); closeModal(); }

        // --- SVG Connectors Logic ---
        function getPathD(x1, y1, x2, y2, style) {
            if(style === 'line') return `M ${x1} ${y1} L ${x2} ${y2}`;
            else if(style === 'square') { let midY = (y1 + y2) / 2; return `M ${x1} ${y1} V ${midY} H ${x2} V ${y2}`; } 
            else { let midY = (y1 + y2) / 2; return `M ${x1} ${y1} C ${x1} ${midY}, ${x2} ${midY}, ${x2} ${y2}`; }
        }

        function drawAllReplyConnectors() {
            const svgLayer = document.getElementById('reply-svg-layer');
            const chatRect = document.getElementById('chat-window').getBoundingClientRect();
            let svgHTML = '';

            if(swipeSettings.style !== 'normal') {
                const allMsgs = document.querySelectorAll('.msg-wrapper[data-reply-key]');
                allMsgs.forEach(msgEl => {
                    const targetKey = msgEl.getAttribute('data-reply-key');
                    const lineColor = msgEl.getAttribute('data-reply-color') || swipeSettings.color;
                    const targetEl = document.getElementById('msg-wrap-' + targetKey);
                    
                    if(targetEl) {
                        const startBubble = targetEl.querySelector('.bubble');
                        const endBubble = msgEl.querySelector('.bubble');
                        if(startBubble && endBubble) {
                            const sRect = startBubble.getBoundingClientRect(); const eRect = endBubble.getBoundingClientRect();
                            let startX = sRect.left + 25 - chatRect.left; let startY = sRect.bottom - chatRect.top;
                            let endX = eRect.left + 25 - chatRect.left; let endY = eRect.top - chatRect.top;
                            if(startY > endY) startY = endY;
                            let pathD = getPathD(startX, startY, endX, endY, swipeSettings.style);
                            svgHTML += `<path d="${pathD}" fill="none" stroke="${lineColor}" stroke-width="${swipeSettings.width}" stroke-linecap="round" opacity="0.8"/>`;
                        }
                    }
                });

                const previewEl = document.getElementById('reply-preview');
                if(replyingToContext && previewEl.style.display !== 'none') {
                    let msgEl = document.getElementById('msg-wrap-' + replyingToContext.key);
                    if(msgEl) {
                        let prevRect = previewEl.getBoundingClientRect();
                        let bubble = msgEl.querySelector('.bubble');
                        if(bubble) {
                            let bRect = bubble.getBoundingClientRect();
                            let startX = bRect.left + 25 - chatRect.left; let startY = bRect.bottom - chatRect.top;
                            let endX = prevRect.left + 30 - chatRect.left; let endY = prevRect.top - chatRect.top;
                            if(startY > endY) startY = endY;
                            let liveColor = swipeSettings.autoColor ? '#25D366' : swipeSettings.color;
                            let pathD = getPathD(startX, startY, endX, endY, swipeSettings.style);
                            svgHTML += `<path d="${pathD}" fill="none" stroke="${liveColor}" stroke-width="${swipeSettings.width}" stroke-linecap="round"/>`;
                        }
                    }
                }
            }
            svgLayer.innerHTML = svgHTML; svgLayer.style.display = svgHTML ? 'block' : 'none';
        }

        let isDrawing = false;
        document.getElementById('msg-flow').addEventListener('scroll', () => {
            if(!isDrawing) { window.requestAnimationFrame(() => { drawAllReplyConnectors(); isDrawing = false; }); isDrawing = true; }
        });
        window.addEventListener('resize', drawAllReplyConnectors);

        // --- Chat Viewer Logic ---
        function openChat(room, name, type='normal') {
            if(activeRoom) { db.ref('messages/'+activeRoom).off(); db.ref('private_messages/'+activeRoom).off(); cancelReply(); }

            activeRoom = room; activeRoomType = type; activePeerName = (type==='normal') ? name : null;
            window.lastRead[room] = Date.now(); localStorage.setItem('xit_reads', JSON.stringify(window.lastRead));
            let rowEl = document.getElementById('chat-row-'+room); if(rowEl) rowEl.classList.remove('unread');

            document.getElementById('active-name').innerText = name;
            let chatWin = document.getElementById('chat-window');
            chatWin.classList.toggle('private', type === 'private');
            
            document.getElementById('btn-audio-call').style.display = (type === 'normal') ? 'inline-block' : 'none';
            document.getElementById('btn-video-call').style.display = (type === 'normal') ? 'inline-block' : 'none';
            
            document.getElementById('pr-share-btn').style.display = (type === 'private') ? 'flex' : 'none';
            document.getElementById('pr-exit-btn').style.display = (type === 'private') ? 'flex' : 'none';
            document.getElementById('chat-input-row').style.background = (type === 'private') ? '#1a1a1a' : '#f0f2f5';

            chatWin.classList.add('open'); history.pushState({page: 'chat'}, "");
            
            const refPath = type === 'private' ? 'private_messages/'+room : 'messages/'+room;
            db.ref(refPath).on('value', snap => {
                const flow = document.getElementById('msg-flow'); flow.innerHTML = "";
                snap.forEach(ms => {
                    const m = ms.val(); m.key = ms.key;
                    
                    const wrap = document.createElement('div');
                    wrap.className = `msg-wrapper ${m.from === myUser ? 'sent-wrap' : 'recv-wrap'}`;
                    wrap.id = `msg-wrap-${m.key}`;
                    
                    const ind = document.createElement('div'); ind.className = 'swipe-ind';
                    if(swipeSettings.style === 'normal') ind.innerHTML = `<span style="font-size:18px; color:${swipeSettings.color}">↩️</span>`;

                    if(m.replyTo && m.replyTo.key) {
                        wrap.setAttribute('data-reply-key', m.replyTo.key);
                        wrap.setAttribute('data-reply-color', m.replyTo.color || swipeSettings.color);
                    }
                    
                    const bubble = document.createElement('div');
                    bubble.className = `bubble ${m.from === myUser ? 'sent' : 'recv'}`;
                    
                    let content = '';
                    if(type === 'private' && m.from !== myUser) content += `<small style="display:block; opacity:0.6; font-weight:bold; margin-bottom:5px; font-size:11px;">~ ${m.from}</small>`;
                    
                    if(m.replyTo && swipeSettings.style === 'normal') {
                        content += `<div class="reply-block"><b>${m.replyTo.from}</b><br><div style="font-size:11px; opacity:0.8; white-space:nowrap; overflow:hidden; text-overflow:ellipsis;">${m.replyTo.type==='image'?'📷 Image':m.replyTo.type==='video'?'📹 Video':m.replyTo.type==='audio'?'🎵 Audio':m.replyTo.type==='document'?'📄 Document':m.replyTo.data}</div></div>`;
                    }

                    // FILE RENDERING UI
                    if(m.type === 'image') content += `<img src="${m.data}" onclick="openMediaViewer('${m.data}', 'image')">`;
                    else if(m.type === 'video') content += `<video src="${m.data}" controls style="width:100%; border-radius:5px; margin-top:5px;"></video><br><small style="cursor:pointer; color:blue; font-weight:bold;" onclick="openMediaViewer('${m.data}', 'video')">⛶ Full Screen</small>`;
                    else if(m.type === 'audio') content += `<audio src="${m.data}" controls></audio>`;
                    else if(m.type === 'document') content += `<div style="background:rgba(0,0,0,0.1); padding:15px; border-radius:5px; margin-top:5px; display:flex; align-items:center; gap:10px;"><span style="font-size:24px;">📄</span><a href="${m.data}" target="_blank" download style="color:inherit; font-weight:bold; flex:1;">Download Document</a></div>`;
                    else content += urlify(m.data);
                    
                    let msgDateObj = new Date(m.ts);
                    let formattedTime = msgDateObj.toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'});
                    let formattedDate = msgDateObj.toLocaleDateString([], {day:'numeric', month:'short'});
                    content += `<div class="msg-time">${formattedTime} • ${formattedDate}</div>`;

                    bubble.innerHTML = content; wrap.appendChild(ind); wrap.appendChild(bubble); flow.appendChild(wrap);

                    // Touch Swipe Gestures
                    let startX = 0, isSwiping = false, pressTimer = null;
                    bubble.addEventListener('touchstart', e => { 
                        startX = e.touches[0].clientX; isSwiping=false; 
                        pressTimer = setTimeout(() => { showMsgActionMenu(m); pressTimer=null; }, 500);
                    }, {passive:true});
                    bubble.addEventListener('touchmove', e => {
                        let dx = e.touches[0].clientX - startX;
                        if(Math.abs(dx) > 5 && pressTimer) clearTimeout(pressTimer);
                        if(dx > 10 && dx < 100) { 
                            isSwiping = true; bubble.style.transform = `translateX(${dx}px)`;
                            if(swipeSettings.style === 'normal') ind.style.opacity = dx/80;
                        }
                    }, {passive:true});
                    bubble.addEventListener('touchend', e => {
                        if(pressTimer) clearTimeout(pressTimer);
                        let dx = e.changedTouches[0].clientX - startX;
                        bubble.style.transform = 'translateX(0px)';
                        ind.style.opacity = 0;
                        if(dx > 50 && isSwiping) setReplyContext(m);
                    });
                    bubble.addEventListener('mousedown', e => { pressTimer = setTimeout(() => { showMsgActionMenu(m); pressTimer=null; }, 500); });
                    bubble.addEventListener('mousemove', () => { if(pressTimer) clearTimeout(pressTimer); });
                    bubble.addEventListener('mouseup', e => { if(pressTimer) clearTimeout(pressTimer); });
                    bubble.addEventListener('contextmenu', e => { e.preventDefault(); showMsgActionMenu(m); });
                });
                flow.scrollTop = flow.scrollHeight; 
                setTimeout(drawAllReplyConnectors, 50);
                window.lastRead[room] = Date.now(); localStorage.setItem('xit_reads', JSON.stringify(window.lastRead));
            });
        }

        function showMsgActionMenu(mObj) {
            actionMsgContext = mObj;
            let html = `<button class="btn-secondary" onclick="setReplyContext(actionMsgContext); closeModal();">↩️ Reply</button><button class="btn-secondary" onclick="openForwardModal();">➡️ Forward</button>`;
            if(mObj.from === myUser) html += `<button class="btn-danger" onclick="deleteTargetMsg('${mObj.key}'); closeModal();">🗑️ Delete</button>`;
            showModal("Message Options", html, null, false);
        }
        function deleteTargetMsg(msgKey) { db.ref((activeRoomType==='private'?'private_messages/':'messages/')+activeRoom+'/'+msgKey).remove(); }

        function openMediaViewer(dataUrl, type) {
            document.getElementById('iv-img').style.display = 'none';
            document.getElementById('iv-video').style.display = 'none';
            document.getElementById('iv-video').pause();
            
            if(type === 'image') {
                document.getElementById('iv-img').src = dataUrl;
                document.getElementById('iv-img').style.display = 'block';
            } else if(type === 'video') {
                document.getElementById('iv-video').src = dataUrl;
                document.getElementById('iv-video').style.display = 'block';
                document.getElementById('iv-video').play();
            }
            document.getElementById('iv-download').href = dataUrl;
            document.getElementById('image-viewer').style.display = 'flex';
        }

        function setReplyContext(mObj) {
            replyingToContext = mObj;
            document.getElementById('reply-preview').style.display = 'block';
            document.getElementById('reply-to-name').innerText = mObj.from === myUser ? "You" : mObj.from;
            document.getElementById('reply-to-text').innerText = mObj.type === 'image' ? '📷 Image' : mObj.type === 'video' ? '📹 Video' : mObj.type === 'audio' ? '🎵 Audio' : mObj.type === 'document' ? '📄 Document' : mObj.data;
            setTimeout(drawAllReplyConnectors, 50); 
        }
        function cancelReply() { replyingToContext = null; document.getElementById('reply-preview').style.display = 'none'; drawAllReplyConnectors(); }

        function sendMsg() {
            const inp = document.getElementById('msg-input');
            const val = inp.value.trim(); if(!val) return;
            inp.value = "";
            let payload = { from: myUser, data: val, ts: Date.now() };
            if(replyingToContext) {
                let lineColor = swipeSettings.autoColor ? getRandomColor() : swipeSettings.color;
                payload.replyTo = { key: replyingToContext.key, color: lineColor, from: replyingToContext.from, data: replyingToContext.data, type: replyingToContext.type || 'text' };
                cancelReply();
            }
            db.ref((activeRoomType==='private'?'private_messages/':'messages/')+activeRoom).push().set(payload);
        }

        // ===============================================
        // NEW FREE FILE TRANSFER (No Firebase Storage)
        // Uses Base64 String directly into Realtime DB
        // ===============================================
        function sendMedia() {
            const fileInput = document.getElementById('media-input');
            const file = fileInput.files[0]; if(!file) return;

            // Database limits base64 size, 4MB is safe for free tiers.
            if(file.size > 4 * 1024 * 1024) {
                alert("File is too large! Please select a file under 4MB for free transfer.");
                fileInput.value = ''; return;
            }

            document.getElementById('upload-indicator').style.display = 'block';
            
            let fType = 'document';
            if(file.type.startsWith('image/')) fType = 'image';
            else if(file.type.startsWith('video/')) fType = 'video';
            else if(file.type.startsWith('audio/')) fType = 'audio';

            const r = new FileReader();
            r.onload = (e) => {
                let dataURL = e.target.result;

                // Compress images to save space
                if(fType === 'image') {
                    const img = new Image();
                    img.onload = () => {
                        const canvas = document.createElement('canvas'); const MAX_WIDTH = 800;
                        const scaleSize = Math.min(MAX_WIDTH / img.width, 1);
                        canvas.width = img.width * scaleSize; canvas.height = img.height * scaleSize;
                        const ctx = canvas.getContext('2d'); ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
                        const compressedDataUrl = canvas.toDataURL('image/jpeg', 0.6);
                        pushToDB(compressedDataUrl, fType);
                    };
                    img.src = dataURL;
                } else {
                    // Send raw base64 for video/audio/doc
                    pushToDB(dataURL, fType);
                }
            };
            r.readAsDataURL(file);

            function pushToDB(fileData, type) {
                let payload = { from: myUser, data: fileData, type: type, ts: Date.now() };
                if(replyingToContext) {
                    let lineColor = swipeSettings.autoColor ? getRandomColor() : swipeSettings.color;
                    payload.replyTo = { key: replyingToContext.key, color: lineColor, from: replyingToContext.from, data: replyingToContext.data, type: replyingToContext.type || 'text' };
                    cancelReply();
                }
                db.ref((activeRoomType==='private'?'private_messages/':'messages/')+activeRoom).push().set(payload)
                  .then(() => { 
                      document.getElementById('upload-indicator').style.display = 'none'; 
                      fileInput.value = ''; 
                  })
                  .catch(err => { 
                      alert("Error sending file: " + err.message); 
                      document.getElementById('upload-indicator').style.display = 'none'; 
                  });
            }
        }

        function closeChat(doBack=true) {
            document.getElementById('chat-window').classList.remove('open');
            if(activeRoom) { db.ref('messages/'+activeRoom).off(); db.ref('private_messages/'+activeRoom).off(); }
            activeRoom = null; activePeerName = null; cancelReply(); if(doBack) history.back();
        }

        function openForwardModal() {
            closeModal(); 
            let html = `<div style="max-height:60vh; overflow-y:auto; text-align:left; border:1px solid #ddd; border-radius:8px; margin-bottom:10px;">`;
            Object.values(window.myPeers).forEach(p => { html += `<label class="fwd-item"><input type="checkbox" value="normal|${p.room}"> <b>${p.peer}</b></label>`; });
            Object.keys(window.myPrivateRooms).forEach(pr => {
                if(window.myPrivateRooms[pr].users && window.myPrivateRooms[pr].users[myUser]) {
                    html += `<label class="fwd-item"><input type="checkbox" value="private|${pr}"> <b>🔒 ${pr}</b></label>`;
                }
            });
            html += `</div><button class="btn-primary" onclick="submitForward()">➡️ Send Forward</button>`;
            showModal("Forward Message To:", html, null, false);
        }

        function submitForward() {
            let checkboxes = document.querySelectorAll('.fwd-item input:checked');
            if(checkboxes.length === 0) return alert("Select at least one chat!");
            checkboxes.forEach(cb => {
                let [type, targetRoom] = cb.value.split('|');
                let refPath = type === 'private' ? 'private_messages/'+targetRoom : 'messages/'+targetRoom;
                let payload = { from: myUser, data: actionMsgContext.data, type: actionMsgContext.type || 'text', ts: Date.now() };
                db.ref(refPath).push().set(payload);
            });
            closeModal(); alert("Message Forwarded Successfully!");
        }

        function showSwipeSettings() {
            toggleSidebar(false);
            let html = `
                <label style="font-weight:bold;">Line Connection Style:</label>
                <select id="ss-style">
                    <option value="normal" ${swipeSettings.style==='normal'?'selected':''}>Normal (WhatsApp Style)</option>
                    <option value="line" ${swipeSettings.style==='line'?'selected':''}>Straight Line</option>
                    <option value="square" ${swipeSettings.style==='square'?'selected':''}>Square Line (Right Angle)</option>
                    <option value="circle" ${swipeSettings.style==='circle'?'selected':''}>Circle / Snake Line</option>
                </select>
                <label style="font-weight:bold; display:flex; align-items:center; gap:10px; margin-top:15px; background:#f9f9f9; padding:10px; border-radius:8px;">
                    <input type="checkbox" id="ss-auto-color" style="width:20px; height:20px; margin:0;" ${swipeSettings.autoColor?'checked':''}> Auto Random Colors
                </label>
                <div style="margin-top:10px;">
                    <label style="font-weight:bold;">Default Color (If Auto is Off):</label>
                    <input type="color" id="ss-color" value="${swipeSettings.color}" style="height:50px; padding:5px;">
                </div>
                <label style="font-weight:bold;">Width / Thickness (px):</label>
                <input type="number" id="ss-width" value="${swipeSettings.width}" min="1" max="10">
            `;
            showModal("⚙️ Swipe Reply Settings", html, () => {
                swipeSettings = {
                    style: document.getElementById('ss-style').value, autoColor: document.getElementById('ss-auto-color').checked,
                    color: document.getElementById('ss-color').value, width: document.getElementById('ss-width').value
                };
                localStorage.setItem('xit_swipe_set', JSON.stringify(swipeSettings));
                closeModal(); alert("Settings Saved!"); drawAllReplyConnectors();
            });
        }

        function handleFab() {
            if(currentTab === 0 || currentTab === 2) {
                showModal("🔍 Send Request", `<input id="t-user" placeholder="Search exact username...">`, () => {
                    const t = document.getElementById('t-user').value.toLowerCase().trim(); if(t) sendFriendRequest(t);
                });
            } else if(currentTab === 1) {
                showModal("New Status", `<input id="s-txt" placeholder="What's on your mind?">`, async () => {
                    const s = document.getElementById('s-txt').value;
                    if(s) { await db.ref('statuses').push({user:myUser, text:s, ts:Date.now()}); closeModal(); }
                });
            }
        }

        function showPrivateRoomMenu() { toggleSidebar(false); showModal("🔒 Private Rooms", `<button class="btn-primary" onclick="openCreateRoomUI()">➕ Create New Room</button><button class="btn-secondary" onclick="openJoinRoomUI()">📥 Join Existing Room</button>`, null, false); }
        function openCreateRoomUI() { showModal("Create Private Room", `<input type="text" id="pr-name" placeholder="Enter Room Name"><input type="password" id="pr-pass" placeholder="Create Password">`, async () => { const n = document.getElementById('pr-name').value.trim(), p = document.getElementById('pr-pass').value.trim(); if(!n || !p) return; if((await db.ref('private_rooms/'+n).once('value')).exists()) return alert("Room Name taken!"); await db.ref('private_rooms/'+n).set({password: p, users: {[myUser]: true}}); closeModal(); openChat(n, "🔒 " + n, 'private'); }); }
        function openJoinRoomUI() { showModal("Join Private Room", `<input type="text" id="pr-name" placeholder="Room Name"><input type="password" id="pr-pass" placeholder="Password">`, async () => { const n = document.getElementById('pr-name').value.trim(), p = document.getElementById('pr-pass').value.trim(); if(!n || !p) return; const snap = await db.ref('private_rooms/'+n).once('value'); if(!snap.exists() || snap.val().password !== p) return alert("Invalid details!"); await db.ref('private_rooms/'+n+'/users/'+myUser).set(true); closeModal(); openChat(n, "🔒 " + n, 'private'); }); }
        function exitPrivateRoomFromName(rName) { activeRoom = rName; exitPrivateRoom(); }
        function exitPrivateRoom() { const roomToExit = activeRoom; showModal("Exit Room", `<p style="margin-bottom:20px; color:#333;">Exit <b>${roomToExit}</b>?</p><label style="display:flex; align-items:center; gap:10px; cursor:pointer; background:#f9f9f9; padding:15px; border-radius:8px; border:1px solid #ddd;"><input type="checkbox" id="clear-msgs-cb" style="width:20px; height:20px;" checked> <b>Clear my messages</b></label>`, async () => { const clear = document.getElementById('clear-msgs-cb').checked; closeModal(); closeChat(); if(clear) { const snap = await db.ref('private_messages/'+roomToExit).once('value'); const updates = {}; snap.forEach(m => { if(m.val().from === myUser) updates[m.key] = null; }); if(Object.keys(updates).length > 0) await db.ref('private_messages/'+roomToExit).update(updates); } await db.ref('private_rooms/'+roomToExit+'/users/'+myUser).remove(); const usersSnap = await db.ref('private_rooms/'+roomToExit+'/users').once('value'); if(!usersSnap.exists() || Object.keys(usersSnap.val() || {}).length === 0) { db.ref('private_rooms/'+roomToExit).remove(); db.ref('private_messages/'+roomToExit).remove(); } }); }
        function openShareLinkModal() { let url = window.location.origin + window.location.pathname + "?room=" + encodeURIComponent(activeRoom); showModal("Share Room", `<p style="margin-bottom:15px;">Share this link to invite others!</p><button class="btn-primary" onclick="navigator.clipboard.writeText('${url}').then(()=>alert('Copied!'))">📋 Copy Link</button><button class="btn-primary" style="background:#25D366;" onclick="window.open('whatsapp://send?text=' + encodeURIComponent('Join secure room: ${url}'))">💬 WhatsApp</button>`, null, false); }
        function toggleSidebar(o) { document.getElementById('sidebar').classList.toggle('open', o); document.getElementById('overlay').style.display = o ? 'block' : 'none'; }
        function showChangePass() { showModal("Change Password", `<input type="password" id="old-p" placeholder="Old Password"><input type="password" id="new-p" placeholder="New Password">`, async () => { const snap = await db.ref('registered_users/'+myUser).once('value'); if(snap.val().password === document.getElementById('old-p').value) { await db.ref('registered_users/'+myUser+'/password').set(document.getElementById('new-p').value); alert("Success!"); closeModal(); } else alert("Wrong Password"); }); }
        
        function viewStatus(u, t, tm, sKey) { document.getElementById('st-avatar').innerText = u[0]; document.getElementById('st-user').innerText = "@" + u; document.getElementById('st-time').innerText = tm; document.getElementById('st-text').innerText = urlify(t); currentActiveStatusKey = sKey; document.getElementById('delete-st-btn').style.display = (u === myUser) ? 'inline-block' : 'none'; document.getElementById('status-viewer').style.display = 'flex'; history.pushState({page: 'status'}, ""); }
        function deleteMyStatus(e) { e.stopPropagation(); if(confirm("Delete this status?") && currentActiveStatusKey) { db.ref('statuses/'+currentActiveStatusKey).remove(); closeStatus(); } }
        function closeStatus(doBack=true) { document.getElementById('status-viewer').style.display = 'none'; currentActiveStatusKey = null; if(doBack) history.back(); }

        // ==========================================
        // REAL-TIME CALLING LOGIC (WebRTC + Firebase)
        // ==========================================
        
        const iceServers = { iceServers: [
            { urls: 'stun:stun1.l.google.com:19302' },
            { urls: 'stun:stun2.l.google.com:19302' },
            { urls: 'stun:stun3.l.google.com:19302' },
            { urls: 'stun:stun4.l.google.com:19302' }
        ]};
        let pc = null; let localStream = null; let remoteStream = null; let currentCallRef = null;
        let isVideoCall = false; let incomingCallObj = null; let callHistoryLogged = false;
        
        let callStartTime = 0; let callTimerInterval = null;
        let isFrontCamera = true; let isSpeakerOn = true; 

        function updateCallTimer() {
            if(callStartTime === 0) return;
            let diff = Math.floor((Date.now() - callStartTime) / 1000);
            let m = String(Math.floor(diff/60)).padStart(2, '0');
            let s = String(diff % 60).padStart(2, '0');
            document.getElementById('call-timer-text').innerText = `${m}:${s}`;
        }

        function startCallTimerUI() {
            callStartTime = Date.now();
            document.getElementById('call-status-text').innerText = ''; 
            document.getElementById('call-timer-text').innerText = '00:00';
            callTimerInterval = setInterval(updateCallTimer, 1000);
        }

        function stopCallTimerUI() {
            if(callTimerInterval) clearInterval(callTimerInterval);
            document.getElementById('call-timer-text').innerText = '';
            let durationStr = "00:00";
            if(callStartTime !== 0) {
                let diff = Math.floor((Date.now() - callStartTime) / 1000);
                let m = String(Math.floor(diff/60)).padStart(2, '0');
                let s = String(diff % 60).padStart(2, '0');
                durationStr = `${m}:${s}`;
            }
            callStartTime = 0; return durationStr;
        }

        async function startCall(type) {
            if(!activePeerName) return alert("Can only call direct contacts!");
            isVideoCall = (type === 'video'); callHistoryLogged = false;
            
            try {
                localStream = await navigator.mediaDevices.getUserMedia({ video: isVideoCall ? { facingMode: 'user' } : false, audio: true });
                document.getElementById('local-video').srcObject = localStream; isFrontCamera = true;
                
                let callScrn = document.getElementById('call-screen');
                callScrn.style.display = 'flex';
                document.getElementById('call-status-text').innerText = `Calling ${activePeerName}...`;
                document.getElementById('flip-cam-btn').style.display = isVideoCall ? 'flex' : 'none';

                if(isVideoCall) {
                    callScrn.classList.remove('audio-only');
                    document.getElementById('local-video').style.display = 'block'; document.getElementById('remote-video').style.display = 'block';
                    document.getElementById('audio-avatar-wrap').style.display = 'none';
                } else {
                    callScrn.classList.add('audio-only');
                    document.getElementById('local-video').style.display = 'none'; document.getElementById('remote-video').style.display = 'none';
                    document.getElementById('audio-avatar-wrap').style.display = 'flex'; document.getElementById('audio-avatar-letter').innerText = activePeerName[0].toUpperCase();
                }

                pc = new RTCPeerConnection(iceServers);
                localStream.getTracks().forEach(track => pc.addTrack(track, localStream));

                remoteStream = new MediaStream(); document.getElementById('remote-video').srcObject = remoteStream;
                pc.ontrack = event => { remoteStream.addTrack(event.track); document.getElementById('remote-video').play().catch(e=>console.log(e)); };
                pc.onconnectionstatechange = () => { if (pc.connectionState === 'connected') startCallTimerUI(); };

                const callId = db.ref(`incoming_calls/${activePeerName}`).push().key;
                currentCallRef = db.ref(`calls_signaling/${callId}`);

                let callerQueue = [];
                pc.onicecandidate = event => { if(event.candidate) currentCallRef.child('callerCandidates').push(event.candidate.toJSON()); };

                const offer = await pc.createOffer(); await pc.setLocalDescription(offer);
                await db.ref(`incoming_calls/${activePeerName}`).set({ caller: myUser, type: type, callId: callId, offer: offer, ts: Date.now() });

                currentCallRef.child('answer').on('value', snap => {
                    if(snap.exists() && !pc.currentRemoteDescription) {
                        pc.setRemoteDescription(new RTCSessionDescription(snap.val())).then(() => { callerQueue.forEach(c => pc.addIceCandidate(c)); callerQueue = []; });
                    }
                });

                currentCallRef.child('calleeCandidates').on('child_added', snap => {
                    if(snap.exists()) {
                        let cand = new RTCIceCandidate(snap.val());
                        if(pc.remoteDescription) pc.addIceCandidate(cand); else callerQueue.push(cand);
                    }
                });
                
                currentCallRef.child('status').on('value', snap => {
                    if(snap.val() === 'ended') {
                        let finalDur = stopCallTimerUI(); let finalStat = finalDur === "00:00" ? "missed" : "outgoing";
                        if(!callHistoryLogged) { db.ref(`call_history/${myUser}`).push({ peer: activePeerName, type: type, status: finalStat, duration: finalDur, ts: Date.now() }); callHistoryLogged = true; }
                        endCallLocal();
                    }
                });

            } catch(e) { console.error(e); alert("Camera/Mic Permission Denied!"); endCallLocal(); }
        }

        function listenForCalls() {
            db.ref(`incoming_calls/${myUser}`).on('value', snap => {
                if(snap.exists() && !pc) { 
                    incomingCallObj = snap.val();
                    showModal("Incoming Call 📞", `<h2 style="text-align:center;">${incomingCallObj.caller}</h2><p style="text-align:center;">Incoming ${incomingCallObj.type} call...</p>`, () => answerCall(), true);
                    
                    document.getElementById('modal-confirm-btn').innerText = "Accept";
                    document.getElementById('modal-confirm-btn').style.background = "#25D366";
                    
                    let cancelBtn = document.querySelector('.modal-btns button:first-child');
                    cancelBtn.innerText = "Reject"; cancelBtn.style.background = "#ff4d4d"; cancelBtn.style.color = "white";
                    cancelBtn.onclick = () => { rejectCall(); closeModal(); };
                }
            });
        }

        async function answerCall() {
            closeModal(); let data = incomingCallObj; callHistoryLogged = false;
            currentCallRef = db.ref(`calls_signaling/${data.callId}`); isVideoCall = (data.type === 'video');

            try {
                localStream = await navigator.mediaDevices.getUserMedia({ video: isVideoCall ? { facingMode: 'user' } : false, audio: true });
                document.getElementById('local-video').srcObject = localStream; isFrontCamera = true;
                
                let callScrn = document.getElementById('call-screen'); callScrn.style.display = 'flex';
                document.getElementById('flip-cam-btn').style.display = isVideoCall ? 'flex' : 'none';
                
                if(isVideoCall) {
                    callScrn.classList.remove('audio-only');
                    document.getElementById('local-video').style.display = 'block'; document.getElementById('remote-video').style.display = 'block';
                    document.getElementById('audio-avatar-wrap').style.display = 'none';
                } else {
                    callScrn.classList.add('audio-only');
                    document.getElementById('local-video').style.display = 'none'; document.getElementById('remote-video').style.display = 'none';
                    document.getElementById('audio-avatar-wrap').style.display = 'flex'; document.getElementById('audio-avatar-letter').innerText = data.caller[0].toUpperCase();
                }

                pc = new RTCPeerConnection(iceServers);
                localStream.getTracks().forEach(track => pc.addTrack(track, localStream));

                remoteStream = new MediaStream(); document.getElementById('remote-video').srcObject = remoteStream;

                pc.ontrack = event => { remoteStream.addTrack(event.track); document.getElementById('remote-video').play().catch(e=>console.log(e)); };
                pc.onconnectionstatechange = () => { if (pc.connectionState === 'connected') startCallTimerUI(); };

                let calleeQueue = [];
                pc.onicecandidate = event => { if(event.candidate) currentCallRef.child('calleeCandidates').push(event.candidate.toJSON()); };

                await pc.setRemoteDescription(new RTCSessionDescription(data.offer));
                const answer = await pc.createAnswer(); await pc.setLocalDescription(answer);
                await currentCallRef.child('answer').set(answer);

                currentCallRef.child('callerCandidates').on('child_added', snap => {
                    if(snap.exists()) {
                        let cand = new RTCIceCandidate(snap.val());
                        if(pc.remoteDescription) pc.addIceCandidate(cand); else calleeQueue.push(cand);
                    }
                });

                currentCallRef.child('status').on('value', snap => {
                    if(snap.val() === 'ended') {
                        let finalDur = stopCallTimerUI();
                        if(!callHistoryLogged) { db.ref(`call_history/${myUser}`).push({ peer: data.caller, type: data.type, status: 'received', duration: finalDur, ts: Date.now() }); callHistoryLogged = true; }
                        endCallLocal();
                    }
                });
                
                db.ref(`incoming_calls/${myUser}`).remove();
            } catch(e) { console.error(e); alert("Camera/Mic Permission Denied!"); rejectCall(); }
        }

        function rejectCall() {
            if(incomingCallObj) {
                db.ref(`calls_signaling/${incomingCallObj.callId}/status`).set('ended'); db.ref(`incoming_calls/${myUser}`).remove();
                if(!callHistoryLogged) { db.ref(`call_history/${myUser}`).push({ peer: incomingCallObj.caller, type: incomingCallObj.type, status: 'missed', duration: '00:00', ts: Date.now() }); callHistoryLogged=true; }
                incomingCallObj = null;
            }
        }

        function endCall() { if(currentCallRef) currentCallRef.child('status').set('ended'); else endCallLocal(); }

        function endCallLocal() {
            stopCallTimerUI();
            if(pc) { pc.close(); pc = null; }
            if(localStream) { localStream.getTracks().forEach(track => track.stop()); localStream = null; }
            if(remoteStream) { remoteStream.getTracks().forEach(track => track.stop()); remoteStream = null; }
            
            document.getElementById('call-screen').style.display = 'none';
            document.getElementById('remote-video').srcObject = null; document.getElementById('local-video').srcObject = null;
            if(currentCallRef) { currentCallRef.off(); currentCallRef = null; }
            db.ref(`incoming_calls/${myUser}`).remove();
        }

        function toggleMute() {
            if(localStream) {
                let audioTrack = localStream.getAudioTracks()[0];
                if(audioTrack) {
                    audioTrack.enabled = !audioTrack.enabled;
                    let btn = document.getElementById('toggle-mic-btn');
                    btn.innerText = audioTrack.enabled ? '🎤' : '🔇'; btn.style.background = audioTrack.enabled ? 'rgba(255,255,255,0.2)' : '#ff4d4d';
                }
            }
        }

        function toggleVideo() {
            if(localStream && isVideoCall) {
                let videoTrack = localStream.getVideoTracks()[0];
                if(videoTrack) {
                    videoTrack.enabled = !videoTrack.enabled;
                    let btn = document.getElementById('toggle-vid-btn');
                    btn.innerText = videoTrack.enabled ? '📹' : '🚫'; btn.style.background = videoTrack.enabled ? 'rgba(255,255,255,0.2)' : '#ff4d4d';
                }
            }
        }

        async function flipCamera() {
            if (!localStream || !isVideoCall) return;
            isFrontCamera = !isFrontCamera; const videoTrack = localStream.getVideoTracks()[0];
            try {
                const newStream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: isFrontCamera ? 'user' : 'environment' } });
                const newVideoTrack = newStream.getVideoTracks()[0];
                localStream.removeTrack(videoTrack); localStream.addTrack(newVideoTrack);
                document.getElementById('local-video').srcObject = localStream;
                if (pc) { const sender = pc.getSenders().find(s => s.track.kind === 'video'); if (sender) sender.replaceTrack(newVideoTrack); }
                videoTrack.stop();
            } catch (e) { console.error("Camera flip failed:", e); isFrontCamera = !isFrontCamera; }
        }

        function toggleSpeaker() {
            const remoteVideo = document.getElementById('remote-video'); isSpeakerOn = !isSpeakerOn;
            const btn = document.getElementById('toggle-speaker-btn');
            btn.innerText = isSpeakerOn ? '🔊' : '🔈'; btn.style.background = isSpeakerOn ? 'rgba(255,255,255,0.2)' : '#ff4d4d';
            remoteVideo.volume = isSpeakerOn ? 1.0 : 0.2; 
        }

        // --- Call History UI ---
        function showCallHistory() {
            toggleSidebar(false);
            db.ref(`call_history/${myUser}`).once('value', snap => {
                let html = `<div style="text-align:left;">`; let data = [];
                snap.forEach(s => data.push(s.val())); data.sort((a,b) => b.ts - a.ts); 
                
                if(data.length === 0) html += `<p style="text-align:center; color:#777;">No calls yet.</p>`;
                data.forEach(c => {
                    let d = new Date(c.ts);
                    let timeStr = d.toLocaleDateString() + ' ' + d.toLocaleTimeString([], {hour:'2-digit', minute:'2-digit'});
                    let icon = c.type === 'video' ? '📹' : '📞';
                    let col = c.status === 'missed' ? '#ff4d4d' : c.status === 'outgoing' ? '#0066cc' : '#25D366';
                    let arrow = c.status === 'missed' ? '↙️ Missed' : c.status === 'outgoing' ? '↗️ Outgoing' : '↙️ Received';
                    let durStr = c.duration && c.duration !== '00:00' ? `(${c.duration})` : '';
                    
                    html += `<div class="history-row" style="margin-bottom:5px; background:#f9f9f9; border-radius:8px; display:flex; align-items:center; padding:10px;">
                                <div style="font-size:24px; margin-right:15px;">${icon}</div>
                                <div style="flex:1;"><b style="color:${col};">${c.peer}</b><br><small style="color:#555;">${arrow} ${durStr}</small></div>
                                <div style="font-size:11px; color:#999; text-align:right;">${timeStr}</div>
                             </div>`;
                });
                html += `</div>`; showModal("📞 Call History", html, null, false);
            });
        }

        if(!myUser) document.getElementById('login-screen').style.display='flex';
        else { document.getElementById('side-user-info').innerText = "@"+myUser; document.getElementById('my-avatar').innerText = myUser[0]; startApp(); }
    </script>
</body>
</html>