<!DOCTYPE html>
<html lang="en" data-theme="light">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gemini ProPOS - V13 Traceability Suite</title>
    <link href="https://fonts.googleapis.com/css2?family=Libre+Barcode+39+Text&family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    
    <style>
        /* ============================
           VARIABLES & RESET
           ============================ */
        :root {
            --primary: #2563eb; --primary-hover: #1d4ed8;
            --bg: #f8fafc; --panel: #ffffff; --border: #e2e8f0;
            --text: #0f172a; --text-muted: #64748b;
            --danger: #ef4444; --success: #10b981; --warning: #f59e0b;
            --shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
            --radius: 12px;
        }

        [data-theme="dark"] {
            --primary: #3b82f6; --primary-hover: #60a5fa;
            --bg: #0f172a; --panel: #1e293b; --border: #334155;
            --text: #f8fafc; --text-muted: #94a3b8;
        }

        * { box-sizing: border-box; margin: 0; padding: 0; font-family: 'Inter', sans-serif; outline: none; }
        body { background-color: var(--bg); color: var(--text); height: 100vh; display: flex; overflow: hidden; }
        button { cursor: pointer; transition: 0.2s; }
        
        /* ============================
           LAYOUT
           ============================ */
        .app-container { display: flex; width: 100%; height: 100%; }
        .sidebar { width: 80px; background: var(--panel); border-right: 1px solid var(--border); display: flex; flex-direction: column; align-items: center; padding: 20px 0; z-index: 10; }
        .nav-item { width: 48px; height: 48px; border-radius: var(--radius); display: flex; align-items: center; justify-content: center; font-size: 1.5rem; margin-bottom: 15px; color: var(--text-muted); position: relative; }
        .nav-item:hover { background: var(--bg); color: var(--primary); }
        .nav-item.active { background: var(--primary); color: white; box-shadow: 0 4px 10px rgba(37, 99, 235, 0.3); }
        .badge-dot { position: absolute; top: 5px; right: 5px; width: 10px; height: 10px; background: var(--danger); border-radius: 50%; border: 2px solid var(--panel); display: none; }

        .main-content { flex: 1; display: flex; overflow: hidden; position: relative; }
        .view { display: none; flex: 1; height: 100%; flex-direction: column; padding: 20px; overflow-y: auto; }
        .view.active { display: flex; }
        .view.pos-view { padding: 0; flex-direction: row; } 

        /* ============================
           POS INTERFACE
           ============================ */
        .pos-grid-area { flex: 1; padding: 20px; display: flex; flex-direction: column; overflow: hidden; }
        .search-bar-container { display:flex; justify-content:space-between; align-items:center; margin-bottom:15px; }
        .search-input { background: var(--panel); border: 1px solid var(--border); padding: 10px 15px; border-radius: 8px; color: var(--text); width: 100%; transition: border 0.2s; }
        .search-input:focus { border-color: var(--primary); }

        .category-scroll { display: flex; gap: 10px; overflow-x: auto; padding-bottom: 10px; margin-bottom: 10px; scrollbar-width: none; }
        .cat-pill { padding: 8px 16px; background: var(--panel); border: 1px solid var(--border); border-radius: 20px; font-size: 0.85rem; font-weight: 600; color: var(--text-muted); white-space: nowrap; }
        .cat-pill.active { background: var(--primary); color: white; border-color: var(--primary); }

        .product-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(140px, 1fr)); gap: 15px; overflow-y: auto; padding-right: 5px; }
        .card { background: var(--panel); border: 1px solid var(--border); border-radius: var(--radius); padding: 15px; display: flex; flex-direction: column; align-items: center; cursor: pointer; position: relative; transition: transform 0.1s; }
        .card:active { transform: scale(0.98); }
        .card.disabled { opacity: 0.5; pointer-events: none; filter: grayscale(1); }
        .stock-tag { position: absolute; top: 8px; right: 8px; font-size: 0.65rem; padding: 2px 6px; border-radius: 4px; font-weight: 700; }
        .stock-ok { background: #dcfce7; color: #166534; } 
        .stock-low { background: #fef3c7; color: #92400e; }

        .pos-cart-area { width: 380px; background: var(--panel); border-left: 1px solid var(--border); display: flex; flex-direction: column; }
        .cart-header { padding: 20px; border-bottom: 1px solid var(--border); }
        .cart-list { flex: 1; overflow-y: auto; padding: 20px; }
        .cart-item { display: flex; justify-content: space-between; align-items: center; margin-bottom: 15px; border-bottom: 1px dashed var(--border); padding-bottom: 10px; }
        .qty-box { display: flex; align-items: center; gap: 8px; background: var(--bg); padding: 4px; border-radius: 6px; }
        .qty-btn { width: 24px; height: 24px; border: none; background: var(--panel); border-radius: 4px; color: var(--text); font-weight: bold; }
        
        #edit-mode-bar { display:none; padding: 15px; background: #fffbeb; border-bottom: 1px solid #fcd34d; color: #92400e; font-size: 0.85rem; animation: slideDown 0.3s; }
        @keyframes slideDown { from { transform: translateY(-100%); } to { transform: translateY(0); } }

        .cart-footer { padding: 20px; background: var(--bg); border-top: 1px solid var(--border); }
        .total-display { font-size: 1.5rem; font-weight: 800; color: var(--text); display: flex; justify-content: space-between; margin-bottom: 15px; }
        .btn-lg { padding: 15px; border: none; border-radius: 8px; font-weight: 700; font-size: 1rem; color: white; display: flex; align-items: center; justify-content: center; gap: 8px; width: 100%; }
        .btn-pay { background: var(--success); }
        .btn-pay:disabled { background: var(--text-muted); cursor: not-allowed; }

        /* ============================
           RECEIPTS & TIMELINE
           ============================ */
        .receipt-paper { 
            background: #fff; padding: 20px; width: 100%; max-width: 340px; margin: 0 auto;
            font-family: 'Courier New', Courier, monospace; font-size: 13px; color: #000;
            box-shadow: 0 0 10px rgba(0,0,0,0.1); border: 1px solid #eee;
        }
        .receipt-header, .receipt-footer { text-align: center; margin-bottom: 10px; }
        .receipt-logo { font-weight: 900; font-size: 1.2rem; letter-spacing: 2px; margin-bottom: 5px; }
        .dashed-line { border-top: 1px dashed #000; margin: 10px 0; }
        .receipt-row { display: flex; justify-content: space-between; margin-bottom: 4px; }
        .receipt-row.bold { font-weight: 700; }
        .barcode { font-family: 'Libre Barcode 39 Text', cursive; font-size: 2.5rem; text-align: center; margin-top: 15px; line-height: 1; }

        .timeline-wrapper { position: relative; margin-top: 30px; padding-left: 20px; }
        .timeline-node { position: relative; padding-left: 20px; margin-bottom: 20px; }
        .timeline-left { position: absolute; left: -14px; top: 0; display:flex; flex-direction:column; align-items:center; height:100%; }
        .timeline-icon { width: 32px; height: 32px; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 14px; z-index: 2; border: 3px solid var(--bg); }
        .timeline-line { flex:1; width:2px; background:var(--border); margin-top:5px; }
        .timeline-content { background: var(--panel); border: 1px solid var(--border); border-radius: 8px; padding: 15px; box-shadow: var(--shadow); transition:0.2s; }
        .timeline-content:hover { border-color: var(--primary); }
        .badge-status { font-size: 0.7rem; padding: 2px 8px; border-radius: 4px; font-weight: 700; text-transform: uppercase; }

        /* ============================
           MODULES & UTILS
           ============================ */
        .dash-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 20px; margin-bottom: 30px; }
        .stat-card { background: var(--panel); border: 1px solid var(--border); padding: 20px; border-radius: var(--radius); box-shadow: var(--shadow); }
        
        .history-layout { display: flex; height: 100%; gap: 20px; }
        .history-list-col { flex: 1; background: var(--panel); border: 1px solid var(--border); border-radius: var(--radius); overflow-y: auto; }
        .history-preview-col { flex: 1; background: var(--panel); border: 1px solid var(--border); border-radius: var(--radius); padding: 20px; overflow-y: auto; display: flex; justify-content: center; }

        .modal-overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.5); backdrop-filter: blur(4px); display: none; justify-content: center; align-items: center; z-index: 100; }
        .modal { background: var(--panel); padding: 30px; border-radius: 16px; width: 450px; box-shadow: 0 20px 50px rgba(0,0,0,0.3); display: flex; flex-direction: column; max-height: 90vh; overflow-y: auto; }
        
        /* Detailed Transaction Modal Specifics */
        .detail-modal { width: 800px; max-width: 95vw; display: grid; grid-template-columns: 1fr 1fr; gap: 20px; }
        .detail-info-pane { padding: 10px; display: flex; flex-direction: column; }
        .info-row { display: flex; justify-content: space-between; padding: 10px 0; border-bottom: 1px solid var(--border); font-size: 0.9rem; }
        
        #toast-box { position: fixed; top: 20px; left: 50%; transform: translateX(-50%); z-index: 200; }
        .toast { background: var(--panel); padding: 10px 20px; border-radius: 8px; box-shadow: 0 5px 15px rgba(0,0,0,0.2); margin-bottom: 10px; display: flex; align-items: center; gap: 10px; border-left: 4px solid var(--primary); }
        
        @keyframes fadeIn { from { opacity:0; transform:translateY(10px); } to { opacity:1; transform:translateY(0); } }
    </style>
</head>
<body>

<div class="app-container">
    
    <div class="sidebar">
        <div class="nav-item active" onclick="Controller.switchView('pos')" title="POS">🖥️</div>
        <div class="nav-item" onclick="Controller.switchView('returns')" title="Returns Lookup">↩️</div>
        <div class="nav-item" onclick="Controller.switchView('ps')" title="Problem Solve">
            🛠️ <div class="badge-dot" id="ps-badge"></div>
        </div>
        <div class="nav-item" onclick="Controller.switchView('dashboard')" title="Analytics">📊</div>
        <div class="nav-item" onclick="Controller.switchView('history')" title="Audit Trail">📜</div>
        <div class="nav-item" onclick="Controller.switchView('inventory')" title="Inventory">📦</div>
        
        <div style="margin-top:auto;">
            <div class="nav-item" onclick="Utils.toggleTheme()" title="Theme">🌙</div>
            <div class="nav-item" onclick="Utils.resetData()" title="Reset" style="color:var(--danger);">🗑️</div>
        </div>
    </div>

    <div class="main-content">
        <div id="pos" class="view pos-view active">
            <div class="pos-grid-area">
                <div class="search-bar-container">
                    <h1 style="font-weight:800; font-size:1.4rem;">ProPOS V13</h1>
                    <input type="text" class="search-input" style="width:250px;" placeholder="Search products..." onkeyup="View.renderGrid(this.value)">
                </div>
                <div class="category-scroll" id="categoryList"></div>
                <div class="product-grid" id="productGrid"></div>
            </div>
            
            <div class="pos-cart-area">
                <div class="cart-header">
                    <h3>Current Order</h3>
                    <span style="color:var(--text-muted); font-size:0.9rem;" id="txn-id">Loading...</span>
                </div>
                <div id="edit-mode-bar">
                    <div style="display:flex; justify-content:space-between;">
                        <strong>✏️ Modifying <span id="ref-id-display"></span></strong>
                        <button onclick="Controller.cancelEditMode()" style="background:none;border:none;color:red;">Cancel</button>
                    </div>
                    <input type="text" id="txn-comment" placeholder="Reason (e.g. Defective, Wrong size)" style="width:100%; margin-top:5px; border:1px solid #ccc; padding:4px;">
                </div>
                <div class="cart-list" id="cartList"></div>
                <div class="cart-footer">
                    <div class="total-display">
                        <span>Total</span><span id="total-disp">$0.00</span>
                    </div>
                    <button class="btn-lg btn-pay" id="payBtn" onclick="Controller.openPayModal()" disabled>PAY $0.00</button>
                    <button class="btn-lg" style="background:var(--danger); margin-top:10px;" onclick="Controller.clearCart()">Clear Cart</button>
                </div>
            </div>
        </div>

        <div id="returns" class="view">
            <div style="max-width: 800px; margin: 0 auto; width: 100%;">
                <h1 style="margin-bottom:20px;">Returns & Traceability</h1>
                <div style="background: var(--panel); padding:30px; border-radius:12px; border:1px solid var(--border); text-align:center;">
                    <div style="font-size:3rem; margin-bottom:10px;">🧾</div>
                    <h3>Transaction Tree</h3>
                    <p style="color:var(--text-muted); margin-bottom:20px;">Scan any receipt ID to trace its full lifecycle.</p>
                    <div style="display:flex; gap:10px; justify-content:center;">
                        <input type="text" id="return-search" class="search-input" placeholder="ORD-12345" style="width:300px;">
                        <button class="btn-lg" style="background:var(--primary); width:auto; padding:0 20px;" onclick="Controller.lookupChain()">Trace</button>
                    </div>
                </div>
                <div id="timeline-area" class="timeline-wrapper" style="display:none;"></div>
            </div>
        </div>

        <div id="ps" class="view">
            <h1 style="margin-bottom:10px;">Problem Solve (Quarantine)</h1>
            <div id="psGrid" style="display: grid; grid-template-columns: repeat(auto-fill, minmax(250px, 1fr)); gap: 20px;"></div>
        </div>

        <div id="dashboard" class="view">
            <h1 style="margin-bottom:20px;">Analytics</h1>
            <div class="dash-grid">
                <div class="stat-card">
                    <div style="color:var(--text-muted); font-size:0.9rem;">Total Revenue</div>
                    <div style="font-size:2rem; font-weight:800; color:var(--success);" id="d-rev">$0.00</div>
                </div>
                <div class="stat-card">
                    <div style="color:var(--text-muted); font-size:0.9rem;">Transactions</div>
                    <div style="font-size:2rem; font-weight:800; color:var(--primary);" id="d-count">0</div>
                </div>
                <div class="stat-card">
                    <div style="color:var(--text-muted); font-size:0.9rem;">Pending Returns</div>
                    <div style="font-size:2rem; font-weight:800; color:var(--warning);" id="d-returns">0</div>
                </div>
            </div>
        </div>

        <div id="history" class="view">
            <div style="margin-bottom:20px;">
                <h1>Audit Trail</h1>
                <input type="text" class="search-input" placeholder="Filter ID..." onkeyup="View.renderHistoryList(this.value)" style="width:300px; margin-top:10px;">
            </div>
            <div class="history-layout">
                <div class="history-list-col" id="historyList"></div>
                <div class="history-preview-col" id="historyPreview">
                    <div style="align-self:center; color:var(--text-muted);">Select a transaction</div>
                </div>
            </div>
        </div>

        <div id="inventory" class="view">
            <div style="display:flex; justify-content:space-between; margin-bottom:20px;">
                <h1>Inventory Management</h1>
                <button class="btn-lg" style="background:var(--success); width:auto; padding:0 20px;" onclick="Controller.openAddProductModal()">+ New Product</button>
            </div>
            <div style="background:var(--panel); border-radius:12px; border:1px solid var(--border); overflow:hidden;">
                <table style="width:100%; border-collapse:collapse;">
                    <thead style="background:var(--bg);"><tr><th style="text-align:left; padding:15px;">Item</th><th>Price</th><th>Stock</th><th>Action</th></tr></thead>
                    <tbody id="invTableBody"></tbody>
                </table>
            </div>
        </div>
    </div>
</div>

<div class="modal-overlay" id="txnDetailModal">
    <div class="modal detail-modal">
        <div style="background:#eee; padding:20px; border-radius:8px; overflow-y:auto; display:flex; justify-content:center;">
            <div id="detail-receipt-preview"></div>
        </div>
        
        <div class="detail-info-pane">
            <h2 style="margin-bottom:20px;">Transaction Details</h2>
            
            <div class="info-row"><span>Transaction ID</span><strong id="dt-id">...</strong></div>
            <div class="info-row"><span>Date</span><span id="dt-date">...</span></div>
            <div class="info-row"><span>Status</span><span id="dt-status">...</span></div>
            <div class="info-row"><span>Payment Method</span><span id="dt-method">...</span></div>
            
            <h4 style="margin-top:20px; color:var(--text-muted);">Financials</h4>
            <div class="info-row"><span>Subtotal</span><span id="dt-sub">...</span></div>
            <div class="info-row"><span>Tax</span><span id="dt-tax">...</span></div>
            <div class="info-row"><span>Total</span><strong id="dt-total">...</strong></div>

            <div style="margin-top:auto;">
                <div id="dt-warning" style="background:#fffbeb; color:#b45309; padding:10px; border-radius:8px; font-size:0.9rem; margin-bottom:10px; display:none;">
                    ⚠️ This transaction has already been modified.
                </div>
                <button class="btn-lg" id="dt-action-btn" style="background:var(--primary);">Process Return / Exchange</button>
                <button style="width:100%; margin-top:10px; padding:10px; background:none; border:none; color:var(--text-muted);" onclick="Utils.closeModal('txnDetailModal')">Close Window</button>
            </div>
        </div>
    </div>
</div>

<div class="modal-overlay" id="payModal">
    <div class="modal">
        <h2 style="text-align:center;" id="payModalTitle">Confirm Payment</h2>
        <div id="pay-summary-txt" style="text-align:center; margin:10px 0; color:var(--text-muted);"></div>
        <h1 style="text-align:center; margin-bottom:20px;" id="payModalTotal">$0.00</h1>
        <div style="display:grid; grid-template-columns:1fr 1fr; gap:10px;">
            <button class="btn-lg" style="background:var(--bg); color:black; border:1px solid #ccc;" onclick="Controller.finalizeTransaction('Card')">💳 Card</button>
            <button class="btn-lg" style="background:var(--bg); color:black; border:1px solid #ccc;" onclick="Controller.finalizeTransaction('Cash')">💵 Cash</button>
        </div>
        <button style="margin-top:20px; background:none; border:none; color:var(--text-muted);" onclick="Utils.closeModal('payModal')">Cancel</button>
    </div>
</div>

<div class="modal-overlay" id="receiptModal">
    <div class="modal" style="width:400px;">
        <h3 style="text-align:center; color:var(--success); margin-bottom:10px;">Success!</h3>
        <div id="receipt-content"></div>
        <button class="btn-lg btn-pay" style="margin-top:20px;" onclick="Utils.closeModal('receiptModal'); Controller.startNewTransaction();">Start New Sale</button>
    </div>
</div>

<div class="modal-overlay" id="addProductModal">
    <div class="modal">
        <h2>Add Product</h2>
        <input id="n-name" class="search-input" style="margin-top:10px;" placeholder="Product Name">
        <input id="n-cat" class="search-input" style="margin-top:10px;" placeholder="Category">
        <input id="n-price" type="number" class="search-input" style="margin-top:10px;" placeholder="Price ($)">
        <input id="n-stock" type="number" class="search-input" style="margin-top:10px;" placeholder="Stock">
        <button class="btn-lg btn-pay" style="margin-top:20px;" onclick="Controller.saveProduct()">Save Item</button>
        <button style="margin-top:10px; background:none; border:none;" onclick="Utils.closeModal('addProductModal')">Cancel</button>
    </div>
</div>

<div id="toast-box"></div>

<script>
    const CONFIG = { TAX_RATE: 0.08, CURRENCY: 'USD', KEYS: { INV:'v13_inv', SALES:'v13_sales', PS:'v13_ps', THEME:'v13_theme' } };
    
    const State = {
        inventory: [], cart: [], sales: [], psQueue: [],
        currentTxn: { id: "", editRefId: null, originalSnapshot: null, originalTotal: 0 },
        filter: "All"
    };

    const Utils = {
        fmt: (n) => new Intl.NumberFormat('en-US', { style: 'currency', currency: CONFIG.CURRENCY }).format(n),
        genId: (pre) => `${pre}-${Math.floor(100000 + Math.random() * 900000)}`,
        load: () => {
            State.inventory = JSON.parse(localStorage.getItem(CONFIG.KEYS.INV)) || [{id:1,name:"Pen",price:1.5,cat:"Office",stock:100,icon:"🖊️"}];
            State.sales = JSON.parse(localStorage.getItem(CONFIG.KEYS.SALES)) || [];
            State.psQueue = JSON.parse(localStorage.getItem(CONFIG.KEYS.PS)) || [];
            if(localStorage.getItem(CONFIG.KEYS.THEME)==='dark') document.documentElement.setAttribute('data-theme','dark');
        },
        save: () => {
            localStorage.setItem(CONFIG.KEYS.INV, JSON.stringify(State.inventory));
            localStorage.setItem(CONFIG.KEYS.SALES, JSON.stringify(State.sales));
            localStorage.setItem(CONFIG.KEYS.PS, JSON.stringify(State.psQueue));
            View.updateDash(); View.renderHistory();
        },
        toast: (m,t='success') => {
            const d = document.createElement('div'); d.className='toast';
            d.style.borderLeftColor = `var(--${t==='error'?'danger':t==='warning'?'warning':'success'})`;
            d.innerHTML = `<span>${m}</span>`; document.getElementById('toast-box').appendChild(d);
            setTimeout(()=>d.remove(),3000);
        },
        toggleTheme: () => {
            const t = document.documentElement.getAttribute('data-theme')==='dark'?'light':'dark';
            document.documentElement.setAttribute('data-theme',t); localStorage.setItem(CONFIG.KEYS.THEME,t);
        },
        resetData: () => { if(confirm("Reset all data?")) { localStorage.clear(); location.reload(); } },
        closeModal: (id) => document.getElementById(id).style.display='none',
        openModal: (id) => document.getElementById(id).style.display='flex'
    };

    const View = {
        init: () => { View.renderCats(); View.renderGrid(); View.renderInv(); View.renderPS(); View.updateDash(); View.renderHistory(); },
        
        renderCats: () => {
            const cats = ["All", ...new Set(State.inventory.map(i=>i.cat))];
            document.getElementById('categoryList').innerHTML = cats.map(c=>`<button class="cat-pill ${c===State.filter?'active':''}" onclick="Controller.setFilter('${c}',this)">${c}</button>`).join('');
        },
        renderGrid: (term="") => {
            const g = document.getElementById('productGrid'); g.innerHTML = "";
            State.inventory.filter(i=>(State.filter==="All"||i.cat===State.filter) && i.name.toLowerCase().includes(term.toLowerCase())).forEach(i=>{
                g.innerHTML += `<div class="card ${i.stock<=0?'disabled':''}" onclick="Controller.addToCart(${i.id})">
                    <span class="stock-tag ${i.stock>0?'stock-ok':'stock-low'}">${i.stock} Left</span>
                    <div style="font-size:2rem; margin:10px 0;">${i.icon||'📦'}</div>
                    <div style="font-weight:600; font-size:0.85rem;">${i.name}</div>
                    <div style="font-size:0.9rem; color:var(--primary); font-weight:bold;">${Utils.fmt(i.price)}</div>
                </div>`;
            });
        },
        renderCart: () => {
            const l = document.getElementById('cartList'); l.innerHTML = ""; let sub = 0;
            if(State.cart.length===0) l.innerHTML=`<div style="text-align:center; margin-top:20px; color:var(--text-muted);">Cart Empty</div>`;
            State.cart.forEach(i => {
                sub += i.price * i.qty;
                l.innerHTML += `<div class="cart-item"><div><b>${i.name}</b><br><small>${Utils.fmt(i.price)}</small></div>
                <div class="qty-box"><button class="qty-btn" onclick="Controller.modQty(${i.id},-1)">-</button><span>${i.qty}</span><button class="qty-btn" onclick="Controller.modQty(${i.id},1)">+</button></div></div>`;
            });
            const tot = sub * (1+CONFIG.TAX_RATE);
            document.getElementById('total-disp').innerText = Utils.fmt(tot);
            View.updatePayBtn(tot);
        },
        updatePayBtn: (tot) => {
            const b = document.getElementById('payBtn');
            if(State.currentTxn.editRefId) {
                const diff = tot - State.currentTxn.originalTotal;
                if(Math.abs(diff)<0.01) { b.innerText="NO CHARGE"; b.style.background="var(--text-muted)"; }
                else if(diff>0) { b.innerText=`PAY ${Utils.fmt(diff)}`; b.style.background="var(--success)"; }
                else { b.innerText=`REFUND ${Utils.fmt(Math.abs(diff))}`; b.style.background="var(--danger)"; }
                b.disabled=false;
            } else {
                b.innerText = tot<=0 ? "PAY $0.00" : `PAY ${Utils.fmt(tot)}`;
                b.disabled = tot<=0; b.style.background="var(--success)";
            }
        },
        
        // --- NEW: PREVIEW & DETAILS ---
        previewTxn: (sale) => {
            // Populate Right Pane (Data)
            document.getElementById('dt-id').innerText = sale.id;
            document.getElementById('dt-date').innerText = new Date(sale.date).toLocaleString();
            document.getElementById('dt-status').innerText = sale.status;
            document.getElementById('dt-status').style.color = sale.status==='Modified' ? 'var(--warning)' : 'var(--success)';
            document.getElementById('dt-method').innerText = sale.method;
            
            document.getElementById('dt-sub').innerText = Utils.fmt(sale.subtotal);
            document.getElementById('dt-tax').innerText = Utils.fmt(sale.tax);
            document.getElementById('dt-total').innerText = Utils.fmt(sale.total);

            // Populate Left Pane (Receipt)
            document.getElementById('detail-receipt-preview').innerHTML = View.genReceipt(sale);

            // Handle Action Button
            const btn = document.getElementById('dt-action-btn');
            const warn = document.getElementById('dt-warning');
            if(sale.status === 'Modified') {
                btn.style.display = 'none';
                warn.style.display = 'block';
            } else {
                btn.style.display = 'block';
                warn.style.display = 'none';
                btn.onclick = () => {
                    Utils.closeModal('txnDetailModal');
                    Controller.loadReturnToPOS(sale.id);
                };
            }
            Utils.openModal('txnDetailModal');
        },

        genReceipt: (sale) => {
            const isReturn = sale.netDifference < 0;
            const isExchange = sale.refId !== null;
            let itemsHtml = '';
            if(!isExchange) {
                itemsHtml = sale.items.map(i=>`<div class="receipt-row"><span>${i.qty} x ${i.name}</span><span>${Utils.fmt(i.price*i.qty)}</span></div>`).join('');
            } else {
                itemsHtml += `<div class="receipt-row bold" style="margin-top:5px; border-bottom:1px solid #ccc;">ACTIVITY LOG:</div>`;
                sale.changes.forEach(c => {
                    const isRet = c.qtyDiff < 0;
                    itemsHtml += `<div class="receipt-row" style="color:${isRet?'red':'black'}">
                        <span>${isRet?'-':'+'} ${Math.abs(c.qtyDiff)} ${c.name} @ ${Utils.fmt(c.price)}</span>
                        <span>${isRet?'-':'+'}${Utils.fmt(Math.abs(c.totalDiff))}</span></div>`;
                });
            }
            let label = sale.netDifference < 0 ? "REFUND ISSUED" : (sale.netDifference===0 ? "EVEN EXCHANGE" : "TOTAL CHARGED");
            return `<div class="receipt-paper">
                <div class="receipt-header"><div class="receipt-logo">GEMINI POS</div><div>Store #8842 • Traceable</div><div>${new Date(sale.date).toLocaleString()}</div><div>Trx: <b>${sale.id}</b></div>${sale.refId?`<div style="font-size:11px;">Ref: ${sale.refId}</div>`:''}</div>
                <div class="dashed-line"></div>${itemsHtml}<div class="dashed-line"></div>
                <div class="receipt-row"><span>New Subtotal</span><span>${Utils.fmt(sale.subtotal)}</span></div>
                <div class="receipt-row"><span>Tax (${(CONFIG.TAX_RATE*100).toFixed(0)}%)</span><span>${Utils.fmt(sale.tax)}</span></div>
                <div class="receipt-row bold" style="font-size:1.1em; margin-bottom:10px;"><span>NEW CART TOTAL</span><span>${Utils.fmt(sale.total)}</span></div>
                ${isExchange ? `<div style="background:#f0f0f0; padding:5px; margin-bottom:5px;"><div class="receipt-row" style="color:#666;"><span>Previous Balance</span><span>-${Utils.fmt(sale.originalTotal)}</span></div><div class="receipt-row bold" style="border-top:1px solid #ccc;"><span>${label}</span><span>${Utils.fmt(Math.abs(sale.netDifference))}</span></div></div>` : `<div class="receipt-row bold"><span>TOTAL DUE</span><span>${Utils.fmt(sale.total)}</span></div>`}
                <div class="receipt-row"><span>Payment (${sale.method})</span><span>${Utils.fmt(Math.abs(sale.netDifference))}</span></div>
                <div class="receipt-footer" style="margin-top:15px;"><div class="barcode">*${sale.id}*</div></div></div>`;
        },
        
        // Updated Timeline to include Eye Icon
        renderTimeline: (family) => {
            const c = document.getElementById('timeline-area'); c.innerHTML = ""; c.style.display='block';
            family.forEach((s, i) => {
                const isRoot=i===0, isRef=s.netDifference<0, isMod=s.status==="Modified";
                let icon="🛒", bg="#fff", bc="var(--border)";
                if(isRef){icon="💸";bc="var(--danger)";} if(s.netDifference>0&&!isRoot){icon="💳";bc="var(--success)";}
                
                c.innerHTML += `<div class="timeline-node">
                    <div class="timeline-left"><div class="timeline-icon" style="background:${bg}; border-color:${bc};">${icon}</div>${i!==family.length-1?'<div class="timeline-line"></div>':''}</div>
                    <div class="timeline-content">
                        <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:10px;">
                            <div style="display:flex; align-items:center; gap:10px;">
                                <span style="font-weight:700; font-size:1.1rem;">${s.id}</span>
                                <button onclick="Controller.previewTxn('${s.id}')" title="View Receipt" style="border:none; background:none; font-size:1.2rem; padding:0;">👁️</button>
                            </div>
                            <span class="badge-status" style="background:${isMod?'#fef3c7':'#e2e8f0'}; color:${isMod?'#b45309':'#475569'};">${s.status}</span>
                        </div>
                        <div style="background:var(--bg); padding:10px; border-radius:6px; font-size:0.9rem;">${isRoot?`Original Purchase of <b>${s.items.reduce((a,b)=>a+b.qty,0)} items</b>.`:`Activity: ${s.changes.map(x=>`<span style="color:${x.qtyDiff<0?'var(--danger)':'var(--success)'}">${x.qtyDiff>0?'+':''}${x.qtyDiff} ${x.name}</span>`).join(', ')}`}</div>
                        <div style="display:flex; justify-content:space-between; margin-top:15px; font-weight:600;">
                            <div>Cart: ${Utils.fmt(s.total)}</div>
                            <div style="color:${isRef?'var(--danger)':'var(--success)'}">${isRef?'Refunded':'Paid'}: ${Utils.fmt(Math.abs(s.netDifference))}</div>
                        </div>
                        ${!isMod ? `<button style="width:100%; margin-top:10px; border:1px solid var(--primary); background:none; color:var(--primary); padding:8px; border-radius:6px;" onclick="Controller.previewTxn('${s.id}')">Modify / Return This</button>` : ''}
                    </div>
                </div>`;
            });
        },

        renderPS: () => {
            const g=document.getElementById('psGrid'); document.getElementById('ps-badge').style.display=State.psQueue.length?'block':'none';
            g.innerHTML = State.psQueue.length ? "" : `<div style="grid-column:1/-1; text-align:center; color:#ccc;">Quarantine Empty</div>`;
            State.psQueue.forEach((item, i) => {
                g.innerHTML += `<div class="stat-card" style="border-left:4px solid var(--warning);"><h4>${item.name} (x${item.qty})</h4><div style="font-size:0.8rem; color:var(--text-muted);">From: ${item.origTxn}</div><div style="font-size:0.8rem; margin-bottom:10px;">Reason: ${item.reason}</div><div style="display:flex; gap:5px;"><button class="btn-lg" style="flex:1; padding:5px; background:var(--success); font-size:0.8rem;" onclick="Controller.resolvePs(${i}, true)">Restock</button><button class="btn-lg" style="flex:1; padding:5px; background:var(--danger); font-size:0.8rem;" onclick="Controller.resolvePs(${i}, false)">Discard</button></div></div>`;
            });
        },
        renderHistory: (f="") => {
            const l=document.getElementById('historyList'); l.innerHTML="";
            State.sales.filter(s=>s.id.toLowerCase().includes(f.toLowerCase())).forEach(s=>{
                l.innerHTML += `<div onclick="View.previewTxn('${s.id}')" style="padding:15px; border-bottom:1px solid var(--border); cursor:pointer; opacity:${s.status==='Modified'?0.5:1}"><div style="display:flex; justify-content:space-between; font-weight:bold;"><span>${s.refId?'↩️':'🛒'} ${s.id}</span><span>${Utils.fmt(s.total)}</span></div><div style="font-size:0.8rem; color:var(--text-muted);">${new Date(s.date).toLocaleDateString()}</div></div>`;
            });
        },
        renderInv: () => document.getElementById('invTableBody').innerHTML=State.inventory.map(i=>`<tr style="border-bottom:1px solid var(--border);"><td style="padding:10px;">${i.name}</td><td>${Utils.fmt(i.price)}</td><td>${i.stock}</td><td><button onclick="Controller.delProd(${i.id})" style="color:red;border:none;background:none;">×</button></td></tr>`).join(''),
        updateDash: () => {
            const v = State.sales.filter(s=>s.status!=='Modified');
            document.getElementById('d-rev').innerText = Utils.fmt(v.reduce((a,b)=>a+b.total,0));
            document.getElementById('d-count').innerText = v.length;
            document.getElementById('d-returns').innerText = State.psQueue.length;
        }
    };

    const Controller = {
        init: () => { Utils.load(); Controller.newTxn(); View.init(); },
        switchView: (id) => { document.querySelectorAll('.view').forEach(e=>e.classList.remove('active')); document.getElementById(id).classList.add('active'); document.querySelectorAll('.nav-item').forEach(e=>e.classList.remove('active')); if(event) event.currentTarget.classList.add('active'); },
        newTxn: () => { State.cart=[]; State.currentTxn={id:Utils.genId('ORD'), editRefId:null, originalSnapshot:null, originalTotal:0}; document.getElementById('txn-id').innerText=State.currentTxn.id; document.getElementById('edit-mode-bar').style.display='none'; View.renderCart(); },
        setFilter: (c,b) => { State.filter=c; document.querySelectorAll('.cat-pill').forEach(e=>e.classList.remove('active')); b.classList.add('active'); View.renderGrid(); },
        addToCart: (id) => { const i=State.inventory.find(x=>x.id===id), c=State.cart.find(x=>x.id===id); if(c&&c.qty>=i.stock)return Utils.toast("No Stock","error"); if(c)c.qty++; else State.cart.push({...i,qty:1}); View.renderCart(); },
        modQty: (id,d) => { const c=State.cart.find(x=>x.id===id), i=State.inventory.find(x=>x.id===id); if(d>0&&c.qty>=i.stock)return Utils.toast("Max Stock","error"); c.qty+=d; if(c.qty<=0)State.cart=State.cart.filter(x=>x.id!==id); View.renderCart(); },
        clearCart: () => { State.cart=[]; View.renderCart(); },
        openPayModal: () => { 
            const t=document.getElementById('payBtn').innerText; 
            document.getElementById('payModalTotal').innerText=t; 
            document.getElementById('payModalTotal').style.color=t.includes("REFUND")?"var(--danger)":"var(--success)";
            document.getElementById('pay-summary-txt').innerText=State.currentTxn.editRefId?"Returns sent to Quarantine.":"";
            Utils.openModal('payModal');
        },
        finalizeTransaction: (method) => {
            const curSub=State.cart.reduce((a,b)=>a+(b.price*b.qty),0), curTax=curSub*CONFIG.TAX_RATE, curTot=curSub+curTax;
            let origTot=0, origItems=[];
            if(State.currentTxn.editRefId) {
                origItems=State.currentTxn.originalSnapshot||[];
                origTot=origItems.reduce((a,b)=>a+(b.price*b.qty),0)*(1+CONFIG.TAX_RATE);
            }
            const netDiff = curTot - origTot;
            let changes = [];
            const allIds = new Set([...State.cart.map(i=>i.id), ...origItems.map(i=>i.id)]);
            
            allIds.forEach(id => {
                const nQ=(State.cart.find(x=>x.id===id)||{}).qty||0, oQ=(origItems.find(x=>x.id===id)||{}).qty||0, diff=nQ-oQ;
                const ref=State.inventory.find(x=>x.id===id)||origItems.find(x=>x.id===id);
                if(diff!==0 && ref) {
                    changes.push({name:ref.name, qtyDiff:diff, price:ref.price, totalDiff:diff*ref.price});
                    if(diff>0) { const inv=State.inventory.find(x=>x.id===id); if(inv) inv.stock-=diff; }
                    else { State.psQueue.push({id:Date.now(), itemId:id, name:ref.name, qty:Math.abs(diff), reason:document.getElementById('txn-comment').value||"Return", origTxn:State.currentTxn.editRefId||State.currentTxn.id}); }
                }
            });

            if(State.currentTxn.editRefId) { const old=State.sales.find(s=>s.id===State.currentTxn.editRefId); if(old)old.status="Modified"; }
            
            const rec = { id:State.currentTxn.id, date:new Date().toISOString(), refId:State.currentTxn.editRefId, subtotal:curSub, tax:curTax, total:curTot, originalTotal:origTot, netDifference:netDiff, method:method, items:JSON.parse(JSON.stringify(State.cart)), changes:changes, status:"Completed" };
            State.sales.unshift(rec); Utils.save();
            document.getElementById('receipt-content').innerHTML=View.genReceipt(rec); Utils.closeModal('payModal'); Utils.openModal('receiptModal'); View.renderGrid(); View.renderPS();
        },

        lookupChain: () => {
            const sid = document.getElementById('return-search').value.trim();
            if(!sid) return Utils.toast("Enter ID","error");
            let cur = State.sales.find(s=>s.id===sid);
            if(!cur) return Utils.toast("Not Found","error");
            while(cur.refId) { const p=State.sales.find(s=>s.id===cur.refId); if(p)cur=p; else break; }
            let fam=[cur], add=true;
            while(add){add=false; const k=fam.map(f=>f.id); const ch=State.sales.filter(s=>k.includes(s.refId)&&!k.includes(s.id)); if(ch.length){fam=[...fam,...ch];add=true;}}
            fam.sort((a,b)=>new Date(a.date)-new Date(b.date));
            View.renderTimeline(fam);
        },
        previewTxn: (id) => { const s=State.sales.find(x=>x.id===id); if(s) View.previewTxn(s); },
        loadReturnToPOS: (id) => {
            const s=State.sales.find(x=>x.id===id); if(!s)return;
            State.cart=JSON.parse(JSON.stringify(s.items)); State.currentTxn.originalSnapshot=JSON.parse(JSON.stringify(s.items));
            State.currentTxn.editRefId=s.id; State.currentTxn.originalTotal=s.total; State.currentTxn.id=Utils.genId("RET");
            document.getElementById('txn-id').innerText=State.currentTxn.id; document.getElementById('edit-mode-bar').style.display='block'; document.getElementById('ref-id-display').innerText=s.id; document.getElementById('txn-comment').value="";
            Controller.switchView('pos'); View.renderCart(); Utils.toast("Loaded for Return/Exchange");
        },
        cancelEditMode: () => Controller.newTxn(),
        openAddProductModal: () => Utils.openModal('addProductModal'),
        saveProduct: () => { 
            const n=document.getElementById('n-name').value, p=parseFloat(document.getElementById('n-price').value), s=parseInt(document.getElementById('n-stock').value);
            if(!n||isNaN(p)) return Utils.toast("Invalid","error");
            State.inventory.push({id:Date.now(), name:n, cat:document.getElementById('n-cat').value||"Gen", price:p, stock:s||0, icon:'📦'});
            Utils.save(); View.renderGrid(); Utils.closeModal('addProductModal');
        },
        delProd: (id) => { if(confirm("Delete?")){State.inventory=State.inventory.filter(x=>x.id!==id); Utils.save(); View.renderGrid(); View.renderInv();} },
        resolvePs: (i,res) => {
            const q=State.psQueue[i]; if(res){const inv=State.inventory.find(x=>x.id===q.itemId); if(inv)inv.stock+=q.qty;}
            State.psQueue.splice(i,1); Utils.save(); View.renderPS(); View.renderGrid(); Utils.toast(res?"Restocked":"Discarded");
        }
    };

    window.onload = Controller.init;
</script>
</body>
</html>
