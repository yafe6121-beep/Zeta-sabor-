 <!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Zeta Sabor - Ultimate POS Fixed</title>
    <style>
        :root { --blue: #00d4ff; --green: #2ecc71; --red: #ff4d4d; --dark: #0d1117; --card: #161b22; --text: #f0f6fc; }
        
        body { font-family: 'Segoe UI', sans-serif; background: var(--dark); color: var(--text); margin: 0; padding: 15px; user-select: none; }
        .container { max-width: 500px; margin: auto; animation: slideUp 0.5s ease; }
        @keyframes slideUp { from { opacity: 0; transform: translateY(20px); } to { opacity: 1; transform: translateY(0); } }

        h1 { text-align: center; color: var(--blue); font-size: 1.8rem; margin-bottom: 20px; text-shadow: 0 0 10px rgba(0,212,255,0.3); }

        .section { background: var(--card); padding: 18px; border-radius: 20px; margin-bottom: 15px; border: 1px solid #30363d; box-shadow: 0 10px 30px rgba(0,0,0,0.3); }
        h3 { margin: 0 0 15px 0; color: var(--blue); font-size: 1rem; display: flex; align-items: center; gap: 10px; }

        input, select { width: 100%; padding: 14px; margin: 8px 0; border-radius: 12px; border: 1px solid #30363d; background: #010409; color: white; font-size: 16px; box-sizing: border-box; transition: 0.3s; }
        
        .btn { padding: 14px; border: none; border-radius: 12px; font-weight: bold; cursor: pointer; transition: 0.2s; display: flex; align-items: center; justify-content: center; gap: 8px; font-size: 14px; }
        .btn:active { transform: scale(0.95); }
        .btn-blue { background: var(--blue); color: #000; width: 100%; }
        .btn-green { background: var(--green); color: #000; width: 100%; }
        .btn-add { background: #30363d; color: var(--blue); font-size: 12px; margin-top: 5px; }
        .btn-del { background: var(--red); color: white; width: 30px; height: 30px; border-radius: 8px; font-size: 12px; padding: 0; }

        .item-row { display: flex; justify-content: space-between; align-items: center; background: #0d1117; padding: 10px; border-radius: 10px; margin-bottom: 8px; border: 1px solid #21262d; }

        .cart-box { background: rgba(0,212,255,0.05); border: 1px dashed var(--blue); border-radius: 12px; padding: 10px; margin: 10px 0; }
        
        #custom-alert { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.8); display: none; align-items: center; justify-content: center; z-index: 1000; backdrop-filter: blur(5px); }
        .alert-card { background: var(--card); padding: 25px; border-radius: 25px; border: 1px solid var(--blue); text-align: center; width: 80%; max-width: 300px; animation: pop 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275); }

        .history-card { background: #1c2128; padding: 15px; border-radius: 15px; margin-top: 10px; border-left: 5px solid var(--blue); position: relative; transition: 0.3s; }
        .badge { font-size: 10px; padding: 3px 8px; border-radius: 10px; text-transform: uppercase; cursor: pointer; border: 1px solid transparent; }
        .btn-change-status { background: #30363d; color: #ccc; border: none; border-radius: 8px; padding: 6px 10px; font-size: 11px; margin-top: 10px; width: auto; cursor: pointer; }
    </style>
</head>
<body>

<div id="custom-alert">
    <div class="alert-card">
        <h2 id="alert-title" style="margin-top:0; color:var(--blue);">¡Atención!</h2>
        <p id="alert-msg"></p>
        <button class="btn btn-blue" onclick="closeAlert()">ENTENDIDO</button>
    </div>
</div>

<div class="container">
    <h1>ZETA SABOR ⚡ POS ELITE</h1>

    <div class="section">
        <h3>📦 Gestión de Stock</h3>
        <div style="display:grid; grid-template-columns: 2fr 1fr; gap:10px;">
            <input type="text" id="f-name" placeholder="Nuevo Sabor">
            <input type="number" id="f-qty" placeholder="Cant.">
        </div>
        <button class="btn btn-blue" onclick="addInv()">GUARDAR EN INVENTARIO</button>
        <div id="inv-list" style="margin-top:15px;"></div>
    </div>

    <div class="section">
        <h3>💰 Venta por Carrito</h3>
        <input type="text" id="c-name" placeholder="Nombre del Cliente">
        
        <div style="background:#0d1117; padding:10px; border-radius:12px; margin-bottom:10px;">
            <select id="sel-flavor"></select>
            <input type="number" id="s-qty" placeholder="¿Cuántos lleva?">
            <button class="btn btn-add" onclick="addToCart()">+ Añadir al pedido</button>
        </div>

        <div id="cart-display" class="cart-box"></div>

        <div style="display:grid; grid-template-columns: 1fr 1fr; gap:10px;">
            <input type="number" id="u-price" value="1.5" step="0.1" placeholder="Precio $">
            <input type="number" id="tasa-bs" value="45" placeholder="Tasa BS">
        </div>
        
        <select id="p-status">
            <option value="pendiente">⏳ Pendiente</option>
            <option value="friendo">🍳 Friendo</option>
            <option value="pagado">✅ Pagado</option>
        </select>
        
        <input type="text" id="p-method" placeholder="Método (Efectivo, BS, $...)">
        <button class="btn btn-green" onclick="confirmSale()">REGISTRAR VENTA TOTAL ✅</button>
    </div>

    <div class="section">
        <h3>📋 Historial de Ventas</h3>
        <div id="hist-list"></div>
        <button onclick="clearHistory()" style="background:none; border:none; color:#444; font-size:10px; cursor:pointer; margin-top:10px;">Limpiar Ventas Completadas</button>
    </div>
</div>

<script>
    let inventory = JSON.parse(localStorage.getItem('z_inv')) || {};
    let sales = JSON.parse(localStorage.getItem('z_hist')) || [];
    let currentCart = [];

    function showAlert(title, msg) {
        document.getElementById('alert-title').innerText = title;
        document.getElementById('alert-msg').innerText = msg;
        document.getElementById('custom-alert').style.display = 'flex';
    }
    function closeAlert() { document.getElementById('custom-alert').style.display = 'none'; }

    function addInv() {
        const n = document.getElementById('f-name').value.trim();
        const q = parseInt(document.getElementById('f-qty').value);
        if(n && !isNaN(q)) {
            inventory[n] = q;
            save();
            document.getElementById('f-name').value = '';
            document.getElementById('f-qty').value = '';
        }
    }

    function removeInv(name) {
        delete inventory[name];
        save();
    }

    function addToCart() {
        const flavor = document.getElementById('sel-flavor').value;
        const qty = parseInt(document.getElementById('s-qty').value);
        if(!flavor) return showAlert("Error", "No hay sabores en el inventario.");
        if(qty > 0 && inventory[flavor] >= qty) {
            currentCart.push({ flavor, qty });
            inventory[flavor] -= qty;
            render();
            document.getElementById('s-qty').value = '';
        } else { showAlert("Sin Stock", `Solo quedan ${inventory[flavor] || 0} de ${flavor}`); }
    }

    function changeSaleStatus(id) {
        const sale = sales.find(s => s.id === id);
        const states = ['pendiente', 'friendo', 'pagado'];
        let currentIndex = states.indexOf(sale.status);
        let nextIndex = (currentIndex + 1) % states.length;
        sale.status = states[nextIndex];
        save();
    }

    function confirmSale() {
        if(currentCart.length === 0) return showAlert("Vacío", "El carrito no tiene nada.");
        const client = document.getElementById('c-name').value || "Cliente";
        const price = parseFloat(document.getElementById('u-price').value);
        const tasa = parseFloat(document.getElementById('tasa-bs').value);
        const status = document.getElementById('p-status').value;
        const method = document.getElementById('p-method').value || "Efectivo";
        
        let totalItems = 0;
        let saboresText = currentCart.map(i => {
            totalItems += i.qty;
            return `${i.qty} ${i.flavor}`;
        }).join(", ");

        const totalUSD = (totalItems * price).toFixed(2);
        const totalBS = (totalUSD * tasa).toFixed(2);
        const now = new Date();
        const time = `${now.getHours()}:${now.getMinutes().toString().padStart(2,'0')}`;

        sales.push({ id: Date.now(), client, sabores: saboresText, totalUSD, totalBS, status, method, time });
        currentCart = [];
        save();
        showAlert("¡Listo!", "Venta registrada con éxito.");
    }

    function save() {
        localStorage.setItem('z_inv', JSON.stringify(inventory));
        localStorage.setItem('z_hist', JSON.stringify(sales));
        render();
    }

    function clearHistory() { if(confirm("¿Borrar historial?")) { sales = []; save(); } }

    function render() {
        const invList = document.getElementById('inv-list');
        const sel = document.getElementById('sel-flavor');
        invList.innerHTML = ''; sel.innerHTML = '';
        for(let f in inventory) {
            invList.innerHTML += `
                <div class="item-row">
                    <span><b>${f}</b>: ${inventory[f]}</span>
                    <button class="btn btn-del" onclick="removeInv('${f}')">×</button>
                </div>`;
            sel.innerHTML += `<option value="${f}">${f}</option>`;
        }

        const cartDiv = document.getElementById('cart-display');
        cartDiv.innerHTML = currentCart.length === 0 ? '<div style="color:#444; font-size:12px;">Carrito vacío...</div>' : 
            currentCart.map(i => `<div style="font-size:13px; border-bottom:1px solid #222; padding:3px;">• ${i.qty} x ${i.flavor}</div>`).join('');

        const histDiv = document.getElementById('hist-list');
        histDiv.innerHTML = sales.slice().reverse().map(s => `
            <div class="history-card" style="border-color: ${s.status === 'pagado' ? '#2ecc71' : s.status === 'friendo' ? '#00d4ff' : '#e67e22'}">
                <span class="badge" style="background:${s.status === 'pagado' ? '#2ecc71' : s.status === 'friendo' ? '#00d4ff' : '#e67e22'}; color:#000">${s.status}</span>
                <span style="float:right; font-size:10px; color:#555">${s.time}</span>
                <div style="margin-top:10px">
                    <b>${s.client}</b>: ${s.sabores} <br>
                    <span style="color:var(--green); font-weight:bold;">$${s.totalUSD} | ${s.totalBS} BS</span> <br>
                    <small style="color:#888;">Pago: ${s.method}</small>
                </div>
                <button class="btn-change-status" onclick="changeSaleStatus(${s.id})">Cambiar Estado 🔄</button>
            </div>
        `).join('');
    }

    render();
</script>

</body>
</html>
