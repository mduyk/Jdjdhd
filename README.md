<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>DELTA SHOP - AUTO BANK</title>
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-database.js"></script>
    <style>
        :root { --main: #00f2ff; --bg: #0b0b12; --card: #151521; }
        body { background: var(--bg); color: #fff; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; display: flex; justify-content: center; align-items: center; min-height: 100vh; margin: 0; }
        .box { background: var(--card); padding: 30px; border-radius: 15px; border: 1px solid #2a2a3a; width: 330px; text-align: center; box-shadow: 0 10px 30px rgba(0,0,0,0.5); }
        h2 { margin-bottom: 20px; font-weight: 800; letter-spacing: 2px; }
        input { width: 100%; padding: 12px; margin: 10px 0; border-radius: 8px; border: 1px solid #222; background: #000; color: #fff; box-sizing: border-box; outline: none; transition: 0.3s; }
        input:focus { border-color: var(--main); }
        .btn-group { display: flex; gap: 10px; margin-top: 10px; }
        button { flex: 1; padding: 12px; border-radius: 8px; border: none; font-weight: bold; cursor: pointer; transition: 0.2s; text-transform: uppercase; }
        .btn-in { background: var(--main); color: #000; }
        .btn-up { background: transparent; color: var(--main); border: 1px solid var(--main); }
        button:hover { opacity: 0.8; transform: translateY(-2px); }
        .hidden { display: none; }
        .balance-box { background: #000; padding: 15px; border-radius: 10px; margin: 15px 0; border-left: 4px solid var(--main); }
        #qr-code { width: 100%; max-width: 200px; border-radius: 8px; background: #fff; padding: 5px; margin-top: 10px; }
        .memo-text { color: var(--main); font-weight: bold; font-size: 18px; background: #1a1a2e; padding: 5px 10px; border-radius: 5px; }
    </style>
</head>
<body>

    <div class="box">
        <div id="auth-ui">
            <h2 style="color: var(--main);">DELTA SHOP</h2>
            <input type="text" id="u" placeholder="Tên tài khoản (viết liền)...">
            <input type="password" id="p" placeholder="Mật khẩu...">
            <div class="btn-group">
                <button class="btn-in" onclick="login()">VÀO SHOP</button>
                <button class="btn-up" onclick="reg()">ĐĂNG KÝ</button>
            </div>
        </div>

        <div id="shop-ui" class="hidden">
            <h3 id="user-display" style="color: var(--main); margin-bottom: 5px;"></h3>
            <div class="balance-box">
                <div style="font-size: 11px; color: #888; letter-spacing: 1px;">SỐ DƯ TÀI KHOẢN</div>
                <div style="font-size: 28px; color: #ffcc00; font-weight: bold;" id="balance">0đ</div>
            </div>
            
            <div style="padding: 10px; border: 1px dashed #444; border-radius: 10px;">
                <p style="font-size: 12px; margin: 0 0 5px 0; color: #aaa;">Nội dung chuyển khoản chuẩn:</p>
                <span id="memo" class="memo-text"></span>
                <br>
                <img id="qr-code" src="" alt="QR Nạp tiền">
            </div>

            <button onclick="logout()" style="background: #ff4d4d; color: #fff; margin-top: 20px; width: 100%;">ĐĂNG XUẤT</button>
        </div>
    </div>

<script>
    // 1. CẤU HÌNH FIREBASE (Dùng link Firebase của ông)
    const firebaseConfig = { databaseURL: "https://shopctv-61f96-default-rtdb.asia-southeast1.firebasedatabase.app/" };
    firebase.initializeApp(firebaseConfig);
    const db = firebase.database();

    // Kiểm tra xem đã đăng nhập chưa
    const savedUser = localStorage.getItem('user');
    if (savedUser) loadShop(savedUser);

    // HÀM ĐĂNG KÝ
    function reg() {
        let user = document.getElementById('u').value.trim().toLowerCase();
        let pass = document.getElementById('p').value.trim();
        if(!user || !pass) return alert("Vui lòng nhập đủ thông tin!");
        if(user.includes(" ")) return alert("Tên tài khoản không được có khoảng trắng!");

        db.ref('users/' + user).once('value', s => {
            if(s.exists()) {
                alert("Tên tài khoản này đã tồn tại!");
            } else {
                db.ref('users/' + user).set({ password: pass, balance: 0 })
                .then(
