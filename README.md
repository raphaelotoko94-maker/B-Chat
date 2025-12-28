
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <title>RealTime Chat</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div id="auth-container" class="container">
        <h2>Bienvenue sur ChatApp</h2>
        <input type="email" id="email" placeholder="Email">
        <input type="password" id="password" placeholder="Mot de passe">
        <button onclick="signUp()">S'inscrire</button>
        <button onclick="login()">Se connecter</button>
    </div>

    <div id="chat-container" class="container" style="display:none;">
        <header>
            <span id="user-display"></span>
            <button onclick="logout()">Déconnexion</button>
        </header>
        <div id="messages"></div>
        <div class="input-group">
            <input type="text" id="msg-input" placeholder="Votre message...">
            <button onclick="sendMessage()">Envoyer</button>
        </div>
    </div>

    <script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-auth-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-database-compat.js"></script>
    <script src="script.js"></script>
</body>
</html>
body { font-family: 'Arial', sans-serif; background: #ece5dd; display: flex; justify-content: center; align-items: center; height: 100vh; margin: 0; }
.container { background: white; padding: 20px; border-radius: 10px; box-shadow: 0 4px 10px rgba(0,0,0,0.1); width: 350px; }
#messages { height: 300px; overflow-y: auto; border: 1px solid #ddd; margin-bottom: 10px; padding: 10px; display: flex; flex-direction: column; }
.msg { padding: 8px; margin: 5px; border-radius: 8px; max-width: 80%; }
.sent { background: #dcf8c6; align-self: flex-end; }
.received { background: #fff; align-self: flex-start; border: 1px solid #eee; }
input { width: 100%; padding: 10px; margin: 5px 0; border: 1px solid #ccc; border-radius: 5px; box-sizing: border-box; }
button { width: 100%; padding: 10px; background: #075e54; color: white; border: none; border-radius: 5px; cursor: pointer; margin-top: 5px; }
// CONFIGURATION FIREBASE (À remplacer par vos clés Firebase)
const firebaseConfig = {
    apiKey: "VOTRE_API_KEY",
    authDomain: "VOTRE_PROJET.firebaseapp.com",
    databaseURL: "https://VOTRE_PROJET.firebaseio.com",
    projectId: "VOTRE_PROJET",
    storageBucket: "VOTRE_PROJET.appspot.com",
    messagingSenderId: "VOTRE_ID",
    appId: "VOTRE_APP_ID"
};

firebase.initializeApp(firebaseConfig);
const auth = firebase.auth();
const db = firebase.database();

// INSCRIPTION
function signUp() {
    const email = document.getElementById('email').value;
    const pass = document.getElementById('password').value;
    auth.createUserWithEmailAndPassword(email, pass).catch(e => alert(e.message));
}

// CONNEXION
function login() {
    const email = document.getElementById('email').value;
    const pass = document.getElementById('password').value;
    auth.signInWithEmailAndPassword(email, pass).catch(e => alert(e.message));
}

// GESTION DE L'ÉTAT (Connecté ou non)
auth.onAuthStateChanged(user => {
    if (user) {
        document.getElementById('auth-container').style.display = 'none';
        document.getElementById('chat-container').style.display = 'block';
        document.getElementById('user-display').innerText = user.email;
        loadMessages();
    } else {
        document.getElementById('auth-container').style.display = 'block';
        document.getElementById('chat-container').style.display = 'none';
    }
});

// ENVOYER UN MESSAGE
function sendMessage() {
    const text = document.getElementById('msg-input').value;
    if (!text) return;
    db.ref('messages').push({
        email: auth.currentUser.email,
        text: text,
        timestamp: Date.now()
    });
    document.getElementById('msg-input').value = '';
}

// CHARGER LES MESSAGES EN TEMPS RÉEL
function loadMessages() {
    db.ref('messages').on('value', snapshot => {
        const msgDiv = document.getElementById('messages');
        msgDiv.innerHTML = '';
        snapshot.forEach(child => {
            const data = child.val();
            const type = data.email === auth.currentUser.email ? 'sent' : 'received';
            msgDiv.innerHTML += `<div class="msg ${type}"><strong>${data.email}</strong><br>${data.text}</div>`;
        });
        msgDiv.scrollTop = msgDiv.scrollHeight;
    });
}
