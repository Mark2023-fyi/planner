<!DOCTYPE html>
<html lang="en" data-theme="light">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gemini ProPOS - V39 Performance Edition</title>
    <link href="https://fonts.googleapis.com/css2?family=Libre+Barcode+39+Text&family=Inter:wght@400;500;600;700;800&display=swap" rel="stylesheet">
    
    <style>
        /* ============================
           1. VARIABLES & THEME
           ============================ */
        :root {
            --primary: #4f46e5; --primary-hover: #4338ca;
            --bg: #f3f4f6; --panel: #ffffff; --border: #e5e7eb;
            --text: #1f2937; --text-muted: #6b7280;
            --danger: #ef4444; --danger-bg: #fee2e2;
            --success: #10b981; --success-bg: #d1fae5;
            --warning: #f59e0b; --warning-bg: #fef3c7;
            --shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
            --radius: 16px;
            --font-mono: 'Courier New', monospace;
        }

        [data-theme="dark"] {
            --primary: #6366f1; --primary-hover: #818cf8;
            --bg: #0f172a; --panel: #1e293b; --border: #334155;
            --text: #f8fafc; --text-muted: #94a3b8;
            --danger: #f87171; --danger-bg: #450a0a;
            --success: #34d399; --success-bg: #064e3b;
            --warning: #fbbf24; --warning-bg: #451a03;
            --shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.5);
        }

        * { box-sizing: border-box; margin: 0; padding: 0; font-family: 'Inter', sans-serif; outline: none; -webkit-tap-highlight-color: transparent; }
        body { background-color: var(--bg); color: var(--text); height: 100vh; display: flex; overflow: hidden; transition: background 0.3s; }
        button { cursor: pointer; transition: all 0.2s ease; font-weight: 600; }
        button:active { transform: scale(0.98); }
        
        ::-webkit-scrollbar { width: 6px; height: 6px; }
        ::-webkit-scrollbar-track { background: transparent; }
        ::-webkit-scrollbar-thumb { background: var(--border); border-radius: 10px; }

        /* ============================
           2. LAYOUT & SIDEBAR
           ============================ */
        .app-container { display: flex; width: 100%; height: 100%; }
        
        .sidebar { width: 88px; background: var(--panel); border-right: 1px solid var(--border); display: flex; flex-direction: column; align-items: center; padding: 24px 0; z-index: 20; box-shadow: var(--shadow); }
        .nav-item { width: 52px; height: 52px; border-radius: var(--radius); display: flex; align-items: center; justify-content: center; font-size: 1.6rem; margin-bottom: 16px; color: var(--text-muted); position: relative; transition: 0.3s; }
        .nav-item:hover { background: var(--bg); color: var(--primary); transform: translateX(2px); }
        .nav-item.active { background: var(--primary); color: white; box-shadow: 0 4px 12px rgba(79, 70, 229, 0.4); }
        .badge-dot { position: absolute; top: 8px; right: 8px; width: 10px; height: 10px; background: var(--danger); border-radius: 50%; border: 2px solid var(--panel); display: none; }

        #live-clock-container { margin-bottom: 20px; text-align: center; padding-bottom: 15px; border-bottom: 1px solid var(--border); width: 100%; }
        #clock-time { font-size: 0.9rem; font-weight: 800; color: var(--text); font-variant-numeric: tabular-nums; }
        #clock-date { font-size: 0.65rem; color: var(--text-muted); margin-top: 2px; text-transform: uppercase; letter-spacing: 0.5px; }

        .main-content { flex: 1; display: flex; overflow: hidden; position: relative; }
        .view { display: none; flex: 1; height: 100%; flex-direction: column; padding: 24px; overflow-y: auto; }
        .view.active { display: flex; animation: fadeIn 0.3s ease-out; }
        .view.pos-view { padding: 0; flex-direction: row; } 

        /* ============================
           3. POS & GRID
           ============================ */
        .pos-grid-area { flex: 1; padding: 24px; display: flex; flex-direction: column; overflow: hidden; }
        .search-bar-container { display:flex; justify-content:space-between; align-items:center; margin-bottom:20px; gap: 20px; }
        .title-area h1 { font-weight: 800; font-size: 1.5rem; letter-spacing: -0.5px; background: linear-gradient(45deg, var(--primary), #8b5cf6); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
        .search-input { background: var(--panel); border: 1px solid var(--border); padding: 12px 16px; border-radius: 10px; color: var(--text); width: 100%; transition: 0.2s; font-size: 0.95rem; }
        .search-input:focus { border-color: var(--primary); box-shadow: 0 0 0 3px rgba(79, 70, 229, 0.1); }

        .category-scroll { display: flex; gap: 10px; overflow-x: auto; padding-bottom: 5px; margin-bottom: 15px; scrollbar-width: none; }
        .cat-pill { padding: 8px 20px; background: var(--panel); border: 1px solid var(--border); border-radius: 50px; font-size: 0.9rem; font-weight: 600; color: var(--text-muted); white-space: nowrap; box-shadow: 0 2px 4px rgba(0,0,0,0.02); }
        .cat-pill.active { background: var(--primary); color: white; border-color: var(--primary); }

        .product-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(150px, 1fr)); gap: 16px; overflow-y: auto; padding-right: 5px; padding-bottom: 20px; }
        .card { background: var(--panel); border: 1px solid var(--border); border-radius: var(--radius); padding: 16px; display: flex; flex-direction: column; align-items: center; cursor: pointer; position: relative; transition: all 0.2s; box-shadow: 0 2px 4px rgba(0,0,0,0.02); }
        .card:hover { transform: translateY(-3px); box-shadow: var(--shadow); border-color: var(--primary); }
        .card:active { transform: scale(0.96); }
        .card.disabled { opacity: 0.5; pointer-events: none; filter: grayscale(1); }
        
        .stock-tag { position: absolute; top: 10px; right: 10px; font-size: 0.7rem; padding: 3px 8px; border-radius: 6px; font-weight: 700; }
        .stock-ok { background: var(--success-bg); color: var(--success); } 
        .stock-low { background: var(--warning-bg); color: var(--warning); }
        .stock-out { background: var(--danger-bg); color: var(--danger); }

        /* ============================
           4. CART
           ============================ */
        .pos-cart-area { width: 400px; background: var(--panel); border-left: 1px solid var(--border); display: flex; flex-direction: column; z-index: 15; box-shadow: -4px 0 20px rgba(0,0,0,0.02); }
        .cart-header { padding: 20px; border-bottom: 1px solid var(--border); }
        .customer-chip { display:flex; justify-content:space-between; align-items:center; background:var(--bg); padding:8px 12px; border-radius:8px; margin-top:10px; font-size:0.9rem; cursor:pointer; border:1px solid var(--border); transition:0.2s;}
        .customer-chip:hover { border-color:var(--primary); background:white; }
        
        .tier-badge { font-size: 0.8rem; padding: 2px 6px; border-radius: 4px; font-weight: 700; margin-right: 5px; }
        .tier-gold { background: #fef3c7; color: #d97706; }
        .tier-silver { background: #f1f5f9; color: #64748b; }
        .tier-bronze { background: #ffedd5; color: #7c2d12; }

        .cart-list { flex: 1; overflow-y: auto; padding: 20px; }
        .cart-item { display: flex; justify-content: space-between; align-items: center; margin-bottom: 12px; padding: 12px; border-radius: 12px; background: var(--bg); transition: 0.2s; border: 1px solid transparent; }
        .cart-item:hover { border-color: var(--border); background: #fff; box-shadow: 0 2px 5px rgba(0,0,0,0.05); }
        .cart-item.zero-qty { opacity: 0.7; background: var(--danger-bg); border: 1px dashed var(--danger); }
        .removed-badge { font-size: 0.65rem; font-weight: 800; color: var(--danger); text-transform: uppercase; letter-spacing: 1px; margin-bottom: 2px; }
        .diff-badge { font-size: 0.7rem; font-weight: 800; text-transform: uppercase; padding: 2px 6px; border-radius: 4px; margin-top: 4px; display: inline-block; }
        .diff-plus { background: var(--success-bg); color: var(--success); }
        .diff-minus { background: var(--danger-bg); color: var(--danger); }
        .promo-badge { font-size: 0.65rem; background: #ef4444; color: white; padding: 2px 5px; border-radius: 4px; font-weight: 700; display: inline-block; margin-left: 5px; vertical-align: middle;}

        .qty-box { display: flex; align-items: center; gap: 10px; background: var(--panel); padding: 4px; border-radius: 8px; border: 1px solid var(--border); }
        .qty-btn { width: 28px; height: 28px; border: none; background: transparent; border-radius: 6px; color: var(--text); font-weight: 800; }
        .qty-btn:hover { background: var(--bg); color: var(--primary); }

        .cart-footer { padding: 24px; background: var(--panel); border-top: 1px solid var(--border); }
        .total-row { display: flex; justify-content: space-between; margin-bottom: 5px; font-size: 0.9rem; color: var(--text-muted); }
        .total-display { font-size: 1.8rem; font-weight: 800; color: var(--text); display: flex; justify-content: space-between; margin-top: 10px; margin-bottom: 20px; align-items: center; }
        .btn-lg { padding: 16px; border: none; border-radius: 12px; font-weight: 700; font-size: 1rem; color: white; display: flex; align-items: center; justify-content: center; gap: 10px; width: 100%; box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
        .btn-pay { background: var(--success); }
        .btn-pay:disabled { background: var(--text-muted); cursor: not-allowed; box-shadow: none; opacity: 0.7; }
        #edit-mode-bar { display:none; padding: 15px; background: var(--warning-bg); border-bottom: 1px solid var(--warning); color: #92400e; font-size: 0.9rem; }

        /* ============================
           5. MODULES
           ============================ */
        .receipt-paper { background: #fff; padding: 25px; width: 100%; max-width: 360px; margin: 0 auto; font-family: var(--font-mono); font-size: 13px; color: #000; box-shadow: 0 10px 30px rgba(0,0,0,0.1); border: 1px solid #e5e7eb; position: relative; }
        .receipt-paper::after { content: ""; position: absolute; bottom: -10px; left: 0; width: 100%; height: 10px; background: radial-gradient(circle, transparent 50%, #fff 50%) 0 -5px; background-size: 20px 20px; }
        .receipt-header { text-align: center; margin-bottom: 15px; }
        .receipt-logo { font-weight: 900; font-size: 1.4rem; letter-spacing: 2px; margin-bottom: 5px; text-transform: uppercase; }
        .receipt-row { display: flex; justify-content: space-between; margin-bottom: 6px; }
        .receipt-row.bold { font-weight: 700; font-size: 1.05em; }
        .dashed-line { border-top: 1px dashed #333; margin: 15px 0; }
        .barcode { font-family: 'Libre Barcode 39 Text', cursive; font-size: 2.8rem; text-align: center; margin-top: 20px; line-height: 1; }
        .footer-note { margin-top: 15px; text-align: center; font-weight: bold; font-size: 0.85rem; border: 1px solid #000; padding: 5px; }

        .timeline-node { position: relative; padding-left: 30px; margin-bottom: 25px; }
        .timeline-left { position: absolute; left: -10px; top: 0; display:flex; flex-direction:column; align-items:center; height:100%; }
        .timeline-icon { width: 36px; height: 36px; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 16px; z-index: 2; border: 4px solid var(--bg); box-shadow: 0 2px 5px rgba(0,0,0,0.1); }
        .timeline-line { flex:1; width: 3px; background: var(--border); margin-top: 5px; border-radius: 2px; }
        .timeline-content { background: var(--panel); border: 1px solid var(--border); border-radius: 12px; padding: 18px; box-shadow: var(--shadow); transition: 0.2s; }
        .badge-status { font-size: 0.7rem; padding: 2px 8px; border-radius: 4px; font-weight: 700; text-transform: uppercase; }

        .trace-table { width: 100%; border-collapse: collapse; font-size: 0.9rem; margin-top: 20px; background: var(--panel); border-radius: 12px; overflow: hidden; border: 1px solid var(--border); box-shadow: var(--shadow); }
        .trace-table th { background: var(--bg); text-align: left; padding: 14px; font-weight: 700; color: var(--text-muted); text-transform: uppercase; font-size: 0.75rem; letter-spacing: 1px; }
        .trace-table td { padding: 14px; border-bottom: 1px solid var(--border); }
        .movement-plus { color: var(--success); font-weight: 800; background: var(--success-bg); padding: 2px 8px; border-radius: 4px; font-size: 0.8rem; }
        .movement-minus { color: var(--danger); font-weight: 800; background: var(--danger-bg); padding: 2px 8px; border-radius: 4px; font-size: 0.8rem; }
        .inv-promo { color: #ef4444; font-weight: bold; }
        .inv-old-price { text-decoration: line-through; color: #9ca3af; font-size: 0.8em; margin-right: 5px; }

        .dash-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 24px; margin-bottom: 30px; }
        .stat-card { background: var(--panel); border: 1px solid var(--border); padding: 24px; border-radius: var(--radius); box-shadow: var(--shadow); }
        
        .history-layout { display: flex; height: 100%; gap: 24px; overflow:hidden; }
        .history-list-col { flex: 1; background: var(--panel); border: 1px solid var(--border); border-radius: var(--radius); overflow-y: auto; box-shadow: var(--shadow); }
        .history-preview-col { flex: 1; background: var(--bg); border: 1px dashed var(--border); border-radius: var(--radius); padding: 30px; overflow-y: auto; display: flex; justify-content: center; align-items: flex-start; }
        .history-item { padding: 18px; border-bottom: 1px solid var(--border); cursor: pointer; transition:0.2s; }
        .history-item:hover { background: var(--bg); }
        .history-item.active { background: #eef2ff; border-left: 4px solid var(--primary); }

        .modal-overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.4); backdrop-filter: blur(5px); display: none; justify-content: center; align-items: center; z-index: 100; animation: fadeIn 0.2s; }
        .modal { background: var(--panel); padding: 32px; border-radius: 20px; width: 480px; box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.25); display: flex; flex-direction: column; max-height: 90vh; overflow-y: auto; position: relative; transform: scale(0.95); animation: modalPop 0.3s forwards; }
        .modal-close-x { position: absolute; top: 20px; right: 20px; background: var(--bg); border: 1px solid var(--border); width: 32px; height: 32px; border-radius: 50%; font-size: 1.2rem; cursor: pointer; color: var(--text-muted); display:flex; align-items:center; justify-content:center; transition:0.2s;}
        .modal-close-x:hover { color: var(--text); background: var(--border); }

        .detail-modal { width: 900px; max-width: 95vw; display: grid; grid-template-columns: 1fr 1.3fr; gap: 0; padding: 0; overflow: hidden; }
        .dm-left { background: #f8fafc; padding: 40px; display: flex; justify-content: center; overflow-y: auto; border-right: 1px solid var(--border); }
        .dm-right { padding: 40px; display: flex; flex-direction: column; overflow-y: auto; background: var(--panel); }
        .return-item-row { display: flex; justify-content: space-between; align-items: center; padding: 14px; background: var(--bg); margin-bottom: 10px; border-radius: 10px; border: 1px solid var(--border); }
        
        .ps-group-card { background: var(--panel); border: 1px solid var(--border); border-radius: var(--radius); overflow: hidden; box-shadow: var(--shadow); margin-bottom: 20px;}
        .ps-group-header { background: var(--bg); padding: 12px 15px; border-bottom: 1px solid var(--border); display: flex; justify-content: space-between; align-items: center; }
        .ps-item-list { padding: 15px; }
        .ps-item-row { display: flex; flex-direction: column; padding: 10px; border-bottom: 1px solid var(--border); }
        .ps-item-row:last-child { border-bottom: none; }
        .ps-item-meta { display: flex; justify-content: space-between; margin-bottom: 8px; }
        .ps-actions { display: flex; gap: 10px; }

        .icon-grid { display: grid; grid-template-columns: repeat(5, 1fr); gap: 10px; margin-top: 10px; }
        .icon-option { font-size: 1.5rem; padding: 10px; border: 1px solid var(--border); border-radius: 8px; cursor: pointer; text-align: center; transition: 0.2s; }
        .icon-option:hover { background: var(--bg); }
        .icon-option.selected { background: var(--primary); color: white; border-color: var(--primary); }

        .form-group { margin-bottom: 15px; }
        .form-group label { display: block; margin-bottom: 5px; font-weight: 600; font-size: 0.9rem; }
        .table-wrap { border-radius: 12px; border: 1px solid var(--border); overflow: hidden; }
        
        #toast-box { position: fixed; top: 30px; left: 50%; transform: translateX(-50%); z-index: 200; display: flex; flex-direction: column; gap: 10px; }
        .toast { background: var(--panel); padding: 14px 24px; border-radius: 12px; box-shadow: 0 10px 15px -3px rgba(0,0,0,0.1); display: flex; align-items: center; gap: 12px; border-left: 5px solid var(--primary); animation: slideDown 0.3s; font-weight: 500; min-width: 300px; }

        @keyframes fadeIn { from { opacity:0; } to { opacity:1; } }
        @keyframes modalPop { to { transform: scale(1); } }
        @keyframes slideDown { from { transform: translateY(-20px); opacity:0; } to { transform: translateY(0); opacity:1; } }
        @keyframes pulse { 0% {transform: scale(1);} 50% {transform: scale(1.02);} 100% {transform: scale(1);} }

        @media print {
            .app-container, .modal-close-x, .btn-pay, .btn-lg { display: none !important; }
            .modal-overlay { position: static; background: white; display: block; }
            .modal { box-shadow: none; width: 100%; padding: 0; overflow: visible; }
            .receipt-paper { box-shadow: none; border: none; width: 100%; max-width: 100%; }
            body { background: white; height: auto; overflow: visible; }
        }
        
        .empty-state { text-align:center; color:var(--text-muted); padding: 40px; }
        .empty-state div { font-size: 3rem; margin-bottom: 10px; opacity: 0.5; }
    </style>
</head>
<body>

<div class="app-container">
    <div class="sidebar">
        <div id="live-clock-container">
            <div id="clock-time">00:00:00</div>
            <div id="clock-date">Mon, Jan 1</div>
        </div>

        <div class="nav-item active" onclick="Controller.switchView('pos')" title="POS">🖥️</div>
        <div class="nav-item" onclick="Controller.openCustomerDisplay()" title="📺 Customer Display">📺</div>
        <div class="nav-item" onclick="Controller.switchView('returns')" title="Returns Lookup">↩️</div>
        <div class="nav-item" onclick="Controller.switchView('ps')" title="Problem Solve">
            🛠️ <div class="badge-dot" id="ps-badge"></div>
        </div>
        <div class="nav-item" onclick="Controller.switchView('customers')" title="Customers">👥</div>
        <div class="nav-item" onclick="Controller.switchView('dashboard')" title="Analytics">📊</div>
        <div class="nav-item" onclick="Controller.switchView('history')" title="History">📜</div>
        <div class="nav-item" onclick="Controller.switchView('inventory')" title="Inventory">📦</div>
        <div class="nav-item" onclick="Controller.switchView('register')" title="Cash Drawer">💵</div>
        
        <div style="margin-top:auto;">
            <div class="nav-item" onclick="Controller.switchView('settings')" title="Settings">⚙️</div>
            <div class="nav-item" onclick="Utils.toggleTheme()" title="Theme">🌙</div>
            <div class="nav-item" onclick="Utils.resetData()" title="Reset" style="color:var(--danger);">🗑️</div>
        </div>
    </div>

    <div class="main-content">
        <div id="pos" class="view pos-view active">
            <div class="pos-grid-area">
                <div class="search-bar-container">
                    <div class="title-area">
                        <h1 id="store-title">Gemini ProPOS</h1>
                    </div>
                    <input type="text" class="search-input" style="width:300px;" placeholder="Scan or search products..." onkeyup="View.renderGrid(this.value)">
                </div>
                <div class="category-scroll" id="categoryList"></div>
                <div class="product-grid" id="productGrid"></div>
            </div>
            <div class="pos-cart-area">
                <div class="cart-header">
                    <h3>Current Order</h3>
                    <span style="color:var(--text-muted); font-size:0.9rem;" id="txn-id">Scan/Add to start</span>
                    <div id="txn-timer" style="font-size:0.8rem; color:#6b7280; margin-top:4px; display:none; font-variant-numeric: tabular-nums;">⏱️ 00:00</div>
                    
                    <div class="customer-chip" onclick="Controller.openCustomerModal()">
                        <span id="active-customer-name">👤 Guest Customer</span>
                        <small style="color:var(--primary);">Change</small>
                    </div>
                </div>
                <div id="edit-mode-bar">
                    <div style="display:flex; justify-content:space-between;">
                        <strong>✏️ Modifying <span id="ref-id-display"></span></strong>
                        <button onclick="Controller.cancelEditMode()" style="background:none;border:none;color:#b45309; font-weight:bold;">Cancel</button>
                    </div>
                    <input type="text" id="txn-comment" placeholder="Reason (e.g. Defective)" style="width:100%; margin-top:8px; border:1px solid #fcd34d; padding:6px; border-radius:6px;">
                </div>
                <div class="cart-list" id="cartList"></div>
                <div class="cart-footer">
                    <div style="margin-bottom:10px;">
                        <div class="total-row"><span>Subtotal</span><span id="sub-disp">$0.00</span></div>
                        <div class="total-row" id="discount-row" style="display:none; color:var(--success);"><span>Discount</span><span id="disc-disp">-$0.00</span></div>
                        <div class="total-row"><span>Tax</span><span id="tax-disp">$0.00</span></div>
                    </div>
                    <div class="total-display">
                        <span style="font-size:1rem; color:var(--text-muted);">Total</span>
                        <span id="total-disp">$0.00</span>
                    </div>
                    
                    <div style="display:grid; grid-template-columns: 1fr 1fr 1fr; gap:5px; margin-bottom:10px;">
                        <button class="btn-lg" style="background:var(--bg); color:var(--text); border:1px solid var(--border); padding:10px; font-size:0.8rem;" onclick="Controller.openDiscountModal()">🏷️ Disc</button>
                        <button class="btn-lg" id="btn-loyalty" style="background:var(--bg); color:var(--text); border:1px solid var(--border); padding:10px; font-size:0.8rem;" onclick="Controller.openLoyaltyRedeem()" disabled>👑 Points</button>
                        <button class="btn-lg" style="background:var(--bg); color:var(--text); border:1px solid var(--border); padding:10px; font-size:0.8rem;" onclick="Controller.clearCart()">🗑️ Clear</button>
                    </div>
                    <button class="btn-lg btn-pay" id="payBtn" onclick="Controller.openPayModal()" disabled>PAY $0.00</button>
                </div>
            </div>
        </div>

        <div id="returns" class="view">
            <div style="max-width: 900px; margin: 0 auto; width: 100%; padding-bottom:50px;">
                <h1 style="margin-bottom:20px;">Returns & Traceability</h1>
                <div style="background: var(--panel); padding:40px; border-radius:16px; border:1px solid var(--border); text-align:center; box-shadow: var(--shadow);">
                    <div style="font-size:3.5rem; margin-bottom:15px;">🧾</div>
                    <h3 style="margin-bottom:10px;">Transaction Tracer</h3>
                    <p style="color:var(--text-muted); margin-bottom:25px;">Enter a receipt ID to view the entire history chain or issue returns.</p>
                    <div style="display:flex; gap:10px; justify-content:center;">
                        <input type="text" id="return-search" class="search-input" placeholder="ORD-XXXXXX" style="width:350px;">
                        <button class="btn-lg" style="background:var(--primary); width:auto; padding:0 30px;" onclick="Controller.lookupChain()">Trace Chain</button>
                    </div>
                </div>
                <div id="finalize-container" style="margin-top:20px; display:none; justify-content:center;"><button class="btn-lg" style="background:var(--success); width:100%;" onclick="Controller.generateFinalReceipt()">🔒 Generate Final Audit Receipt (Locks Chain)</button></div>
                <div id="timeline-area" class="timeline-wrapper" style="display:none;"></div>
                <div id="chain-table-area" style="display:none;">
                    <h3 style="margin-top:40px; margin-bottom:15px;">Inventory Movement Log</h3>
                    <table class="trace-table"><thead><tr><th>Date</th><th>Receipt ID</th><th>Item</th><th>Price</th><th>Units</th></tr></thead><tbody id="trace-table-body"></tbody></table>
                </div>
            </div>
        </div>

        <div id="ps" class="view"><h1 style="margin-bottom:20px;">Problem Solve (Quarantine)</h1><div id="psGrid"></div></div>

        <div id="customers" class="view">
            <div style="display:flex; justify-content:space-between; margin-bottom:20px; align-items:center;"><h1>Customer Database</h1><button class="btn-lg" style="width:auto; background:var(--success);" onclick="Controller.openAddCustomerModal()">+ New Customer</button></div>
            <div class="table-wrap"><table class="trace-table"><thead><tr><th>Name</th><th>Phone</th><th>Email</th><th>Tier</th><th>Points</th></tr></thead><tbody id="custTableBody"></tbody></table></div>
        </div>

        <div id="dashboard" class="view">
            <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:20px;"><h1>Analytics Dashboard</h1><button class="btn-lg" style="width:auto; background:var(--primary); font-size:0.9rem;" onclick="Controller.exportCSV()">⬇️ Export Data (CSV)</button></div>
            <div class="dash-grid">
                <div class="stat-card"><div style="color:var(--text-muted); font-size:0.9rem;">Total Revenue</div><div style="font-size:2.5rem; font-weight:800; color:var(--success); margin-top:5px;" id="d-rev">$0.00</div></div>
                <div class="stat-card"><div style="color:var(--text-muted); font-size:0.9rem;">Transactions</div><div style="font-size:2.5rem; font-weight:800; color:var(--primary); margin-top:5px;" id="d-count">0</div></div>
                <div class="stat-card"><div style="color:var(--text-muted); font-size:0.9rem;">Pending Returns</div><div style="font-size:2.5rem; font-weight:800; color:var(--warning); margin-top:5px;" id="d-returns">0</div></div>
            </div>
        </div>

        <div id="history" class="view">
            <div style="margin-bottom:20px; display:flex; justify-content:space-between; align-items:center;"><h1>Transaction History</h1><input type="text" class="search-input" placeholder="Filter by ID..." style="width:300px;" onkeyup="View.renderHistory(this.value)"></div>
            <div class="history-layout"><div class="history-list-col" id="historyList"></div><div class="history-preview-col" id="historyPreview"><div class="empty-state"><div>📜</div>Select a transaction to view details</div></div></div>
        </div>

        <div id="inventory" class="view">
            <div style="display:flex; justify-content:space-between; margin-bottom:20px; align-items:center;"><h1>Inventory</h1><button class="btn-lg" style="background:var(--success); width:auto; padding:0 25px;" onclick="Controller.openAddProductModal()">+ Add Product</button></div>
            <div style="background:var(--panel); border-radius:12px; border:1px solid var(--border); overflow:hidden; box-shadow: var(--shadow);"><table style="width:100%; border-collapse:collapse;"><thead style="background:var(--bg);"><tr><th style="text-align:left; padding:15px; font-weight:600; color:var(--text-muted);">Item Name</th><th>Category</th><th>Price</th><th>Deals</th><th>Stock</th><th>Action</th></tr></thead><tbody id="invTableBody"></tbody></table></div>
        </div>

        <div id="register" class="view"><h1>Cash Drawer Management</h1><div id="shift-ui" style="margin-top:20px;"></div><h2 style="margin-top:40px; margin-bottom:15px;">Shift History</h2><div class="table-wrap" style="max-height:300px; overflow-y:auto;"><table class="trace-table"><thead><tr><th>Date Opened</th><th>Start Cash</th><th>Exp. Close</th><th>Actual Close</th><th>Variance</th></tr></thead><tbody id="shiftHistoryBody"></tbody></table></div></div>

        <div id="settings" class="view">
            <h1>System Configuration</h1>
            <div class="stat-card" style="max-width:600px; margin-top:20px;">
                <div class="form-group"><label>Store Name</label><input id="conf-storename" class="search-input" placeholder="Store Name"></div>
                <div class="form-group"><label>Tax Rate (decimal, e.g. 0.08)</label><input id="conf-tax" class="search-input" type="number" step="0.01"></div>
                <div class="form-group"><label>Currency Symbol</label><input id="conf-curr" class="search-input" placeholder="$"></div>
                <button class="btn-lg" style="background:var(--primary);" onclick="Controller.saveSettings()">Save Settings</button>
            </div>
            <h2 style="margin-top:40px;">2nd Screen Promo Box</h2>
            <div class="stat-card" style="max-width:600px; margin-top:10px;">
                <div class="form-group"><label>Ad Mode</label>
                <select id="ad-mode" class="search-input" onchange="View.toggleAdInputs()"><option value="manual">Manual Message</option><option value="auto">Auto Slideshow (Deals)</option></select></div>
                <div id="ad-manual-inputs">
                    <div class="form-group"><label>Headline</label><input id="ad-title" class="search-input" placeholder="e.g. Deal of the Day!"></div>
                    <div class="form-group"><label>Message</label><input id="ad-msg" class="search-input" placeholder="e.g. 50% off all socks"></div>
                    <div class="form-group"><label>Featured Product ID (Optional)</label><select id="ad-prod-select" class="search-input"><option value="">-- None --</option></select></div>
                </div>
                <div id="ad-auto-msg" style="display:none; color:var(--text-muted); padding:10px;">Slideshow will automatically cycle through active deals found in Inventory.</div>
                <div class="form-group"><label>Slide Duration (seconds)</label><input id="ad-duration" type="number" class="search-input" value="5"></div>
                <button class="btn-lg" style="background:var(--success);" onclick="Controller.saveAdSettings()">Update Display</button>
            </div>
            <h2 style="margin-top:40px;">Database</h2>
            <div style="display:flex; gap:10px; margin-top:10px;"><button class="btn-lg" style="width:auto; background:var(--text);" onclick="Controller.backupData()">⬇️ Download Backup</button><button class="btn-lg" style="width:auto; background:var(--bg); color:var(--text); border:1px solid var(--border);" onclick="document.getElementById('file-input').click()">⬆️ Restore Backup</button><input type="file" id="file-input" style="display:none;" onchange="Controller.restoreData(this)"></div>
        </div>
    </div>
</div>

<div class="modal-overlay" id="customerModal"><div class="modal"><button class="modal-close-x" onclick="Utils.closeModal('customerModal')">×</button><h2>Select Customer</h2><input class="search-input" placeholder="Search by name..." onkeyup="View.renderCustSelect(this.value)" style="margin-bottom:10px;"><div id="cust-select-list" style="height:200px; overflow-y:auto; border:1px solid var(--border); border-radius:8px; margin-bottom:10px;"></div><button class="btn-lg" onclick="Controller.selectCustomer(null)">Guest Checkout</button></div></div>
<div class="modal-overlay" id="addCustomerModal"><div class="modal"><button class="modal-close-x" onclick="Utils.closeModal('addCustomerModal')">×</button><h2>New Customer</h2><input id="nc-name" class="search-input" style="margin-top:10px;" placeholder="Full Name"><input id="nc-phone" class="search-input" style="margin-top:10px;" placeholder="Phone"><input id="nc-email" class="search-input" style="margin-top:10px;" placeholder="Email"><button class="btn-lg btn-pay" style="margin-top:20px;" onclick="Controller.saveCustomer()">Create Customer</button></div></div>
<div class="modal-overlay" id="discountModal"><div class="modal"><button class="modal-close-x" onclick="Utils.closeModal('discountModal')">×</button><h2>Apply Discount</h2><div style="display:flex; gap:10px; margin:20px 0;"><button class="btn-lg" style="flex:1; background:var(--bg); color:var(--text); border:1px solid var(--primary);" onclick="Controller.applyDiscount('percent', 10)">10% Off</button><button class="btn-lg" style="flex:1; background:var(--bg); color:var(--text); border:1px solid var(--primary);" onclick="Controller.applyDiscount('percent', 20)">20% Off</button></div><div class="form-group"><label>Custom Percentage (%)</label><input id="disc-percent" type="number" class="search-input"></div><div class="form-group"><label>Fixed Amount ($)</label><input id="disc-fixed" type="number" class="search-input"></div><button class="btn-lg btn-pay" onclick="Controller.applyCustomDiscount()">Apply</button><button class="btn-lg" style="margin-top:10px; background:var(--danger);" onclick="Controller.applyDiscount(null, 0)">Remove Discount</button></div></div>
<div class="modal-overlay" id="txnDetailModal"><div class="modal detail-modal"><button class="modal-close-x" onclick="Utils.closeModal('txnDetailModal')">×</button><div class="dm-left"><div id="detail-receipt-preview"></div></div><div class="dm-right"><div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:25px;"><h2 style="font-size:1.4rem;">Manage Return</h2><div id="dm-status-badge"></div></div><div style="margin-bottom:20px; background:var(--bg); padding:10px; border-radius:8px;"><div id="dm-original-comment" style="font-size:0.85rem; color:var(--text-muted); margin-bottom:5px; font-style:italic;"></div><input id="dm-new-comment" class="search-input" placeholder="Reason for this return (e.g. Damaged)" style="width:100%; font-size:0.9rem;"></div><div id="dm-interactive-list" style="flex:1; overflow-y:auto; padding-right:5px;"></div><div style="margin-top:20px; border-top:2px solid var(--border); padding-top:20px;"><div style="display:flex; justify-content:space-between; margin-bottom:15px; align-items:center;"><span style="color:var(--text-muted);">Refund Total</span><strong style="font-size:1.5rem; color:var(--danger);" id="dm-refund-total">$0.00</strong></div><div id="dm-action-buttons" style="display:flex; flex-direction:column; gap:10px;"><button class="btn-lg" style="background:var(--danger);" onclick="Controller.processInModalRefund()">Issue Refund ($)</button><button class="btn-lg" style="background:var(--bg); color:var(--primary); border:1px solid var(--border);" onclick="Controller.switchToExchange()">Start Exchange</button></div><div id="dm-closed-msg" style="display:none; text-align:center; padding:15px; background:var(--bg); border-radius:8px; color:var(--text-muted);">Chain is closed or modified.</div></div></div></div></div>
<div class="modal-overlay" id="payModal"><div class="modal"><button class="modal-close-x" onclick="Utils.closeModal('payModal')">×</button><h2 style="text-align:center; margin-bottom:10px;" id="payModalTitle">Payment</h2><div id="pay-summary-txt" style="text-align:center; margin-bottom:20px; color:var(--text-muted);"></div><h1 style="text-align:center; margin-bottom:30px; font-size:2.5rem;" id="payModalTotal">$0.00</h1><button class="btn-lg" style="background:var(--success); height:60px; font-size:1.2rem;" onclick="Controller.finalizeTransaction('Cash')">💵 Confirm Cash Payment</button></div></div>
<div class="modal-overlay" id="receiptModal"><div class="modal" style="width:420px;"><button class="modal-close-x" onclick="Utils.closeModal('receiptModal')">×</button><h3 style="text-align:center; color:var(--success); margin-bottom:20px;">Transaction Complete</h3><div id="receipt-content"></div><div style="display:flex; gap:10px; margin-top:20px;"><button class="btn-lg" style="flex:1; background:var(--text); color:white;" onclick="window.print()">🖨️ Print</button><button class="btn-lg btn-pay" style="flex:1;" onclick="Utils.closeModal('receiptModal'); Controller.startNewTransaction();">New Sale</button></div></div></div>
<div class="modal-overlay" id="addProductModal"><div class="modal"><button class="modal-close-x" onclick="Utils.closeModal('addProductModal')">×</button><h2 style="margin-bottom:20px;">Add Product</h2><div style="display:flex; flex-direction:column; gap:15px;"><input id="n-name" class="search-input" placeholder="Product Name"><input id="n-cat" class="search-input" placeholder="Category (e.g. Office)"><div style="display:grid; grid-template-columns: 1fr 1fr; gap:10px;"><input id="n-price" type="number" class="search-input" placeholder="Price ($)"><input id="n-stock" type="number" class="search-input" placeholder="Initial Stock"></div><div style="display:grid; grid-template-columns: 1fr 1fr; gap:10px;"><div class="form-group"><label>Promo Price ($)</label><input id="n-promo-price" type="number" class="search-input" placeholder="Optional"></div><div class="form-group"><label>Promo End Date</label><input id="n-promo-date" type="date" class="search-input"></div></div><label style="font-weight:600;">Icon</label><div class="icon-grid" id="icon-picker"></div><input type="hidden" id="n-icon" value="📦"><button class="btn-lg btn-pay" style="margin-top:10px;" onclick="Controller.saveProduct()">Save Item</button></div></div></div>
<div class="modal-overlay" id="loyaltyModal"><div class="modal"><button class="modal-close-x" onclick="Utils.closeModal('loyaltyModal')">×</button><h2>Redeem Points</h2><p style="margin-bottom:20px;">Available Balance: <strong id="loyalty-bal-disp">0</strong> Pts</p><input id="loyalty-input" type="number" class="search-input" placeholder="Points to redeem (100pts = $1.00)"><button class="btn-lg btn-pay" style="margin-top:20px; background:var(--warning); color:black;" onclick="Controller.applyLoyalty()">Apply Discount</button></div></div>

<div id="toast-box"></div>

<script>
    let CONFIG = { TAX_RATE: 0.08, CURRENCY: 'USD', STORE_NAME: 'Gemini ProPOS', KEYS: { INV:'v38_inv', SALES:'v38_sales', PS:'v38_ps', THEME:'v38_theme', CUST:'v38_cust', CONF:'v38_conf', SHIFT:'v38_shift', AD:'v38_ad' } };
    
    const State = {
        inventory: [], cart: [], sales: [], psQueue: [], customers: [], settings: {}, cashDrawer: { status: 'closed', history: [] }, adSettings: { mode:'manual', title:'', msg:'', prodId:'', duration:5 },
        currentTxn: { id: "", editRefId: null, originalSnapshot: null, originalTotal: 0, customer: null, discount: null, pointsRedeemed: 0 },
        filter: "All", activeDetailTxn: null, tempReturnCart: [], activeChainFamily: [], activeChainFinalized: false,
        customerWindow: null, slideIndex: 0, slideshowInterval: null, displayMode: 'idle',
        txnStartTime: null, txnTimerInterval: null
    };

    const Utils = {
        fmt: (n) => new Intl.NumberFormat('en-US', { style: 'currency', currency: CONFIG.CURRENCY }).format(n),
        genId: (pre) => `${pre}-${Math.floor(100000 + Math.random() * 900000)}`,
        load: () => {
            State.inventory = JSON.parse(localStorage.getItem(CONFIG.KEYS.INV)) || [{id:1,name:"Premium Pen",price:2.5,cat:"Office",stock:50,icon:"🖊️"}, {id:2,name:"Journal",price:12.00,cat:"Office",stock:20,icon:"📓"}];
            State.sales = JSON.parse(localStorage.getItem(CONFIG.KEYS.SALES)) || [];
            State.psQueue = JSON.parse(localStorage.getItem(CONFIG.KEYS.PS)) || [];
            State.customers = JSON.parse(localStorage.getItem(CONFIG.KEYS.CUST)) || [];
            State.settings = JSON.parse(localStorage.getItem(CONFIG.KEYS.CONF)) || { taxRate: 0.08, currency: 'USD', storeName: 'Gemini ProPOS' };
            State.cashDrawer = JSON.parse(localStorage.getItem(CONFIG.KEYS.SHIFT)) || { status: 'closed', history: [] };
            State.adSettings = JSON.parse(localStorage.getItem(CONFIG.KEYS.AD)) || { mode:'manual', title: 'Special Offer', msg: 'Ask about our deals!', prodId: '', duration: 5 };
            
            CONFIG.TAX_RATE = State.settings.taxRate;
            CONFIG.CURRENCY = State.settings.currency;
            CONFIG.STORE_NAME = State.settings.storeName;
            document.getElementById('store-title').innerText = CONFIG.STORE_NAME;

            if(localStorage.getItem(CONFIG.KEYS.THEME)==='dark') document.documentElement.setAttribute('data-theme','dark');
        },
        save: () => {
            localStorage.setItem(CONFIG.KEYS.INV, JSON.stringify(State.inventory));
            localStorage.setItem(CONFIG.KEYS.SALES, JSON.stringify(State.sales));
            localStorage.setItem(CONFIG.KEYS.PS, JSON.stringify(State.psQueue));
            localStorage.setItem(CONFIG.KEYS.CUST, JSON.stringify(State.customers));
            localStorage.setItem(CONFIG.KEYS.SHIFT, JSON.stringify(State.cashDrawer));
            localStorage.setItem(CONFIG.KEYS.AD, JSON.stringify(State.adSettings));
            View.updateDash(); 
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
        resetData: () => { if(confirm("⚠️ Reset all system data?")) { localStorage.clear(); location.reload(); } },
        closeModal: (id) => document.getElementById(id).style.display='none',
        openModal: (id) => document.getElementById(id).style.display='flex',
        startClock: () => {
            const update = () => {
                const now = new Date();
                const timeStr = now.toLocaleTimeString();
                const dateStr = now.toLocaleDateString(undefined, { weekday:'short', month:'short', day:'numeric' });
                const mainTime = document.getElementById('clock-time');
                if(mainTime) { mainTime.innerText = timeStr; document.getElementById('clock-date').innerText = dateStr; }
                if(State.customerWindow && !State.customerWindow.closed) { const custClock = State.customerWindow.document.getElementById('cust-clock'); if(custClock) custClock.innerText = timeStr; }
            };
            update(); setInterval(update, 1000);
        },
        getTier: (pts) => {
            if(pts >= 2000) return { name: 'Gold', icon: '🥇', class: 'tier-gold' };
            if(pts >= 500) return { name: 'Silver', icon: '🥈', class: 'tier-silver' };
            return { name: 'Bronze', icon: '🥉', class: 'tier-bronze' };
        },
        startSlideshow: () => {
            if(State.slideshowInterval) clearInterval(State.slideshowInterval);
            State.slideshowInterval = setInterval(Controller.rotateSlides, (State.adSettings.duration || 5) * 1000);
        }
    };

    const View = {
        init: () => { 
            View.renderCats(); View.renderGrid(); View.renderInv(); View.renderPS(); View.updateDash(); View.renderCustomers(); View.renderShift();
            document.getElementById('conf-storename').value = State.settings.storeName;
            document.getElementById('conf-tax').value = State.settings.taxRate;
            document.getElementById('conf-curr').value = State.settings.currency;
            document.getElementById('ad-mode').value = State.adSettings.mode || 'manual';
            document.getElementById('ad-title').value = State.adSettings.title;
            document.getElementById('ad-msg').value = State.adSettings.msg;
            document.getElementById('ad-duration').value = State.adSettings.duration || 5;
            View.toggleAdInputs(); View.renderAdSelect(); View.renderIconPicker();
        },
        toggleAdInputs: () => {
            const mode = document.getElementById('ad-mode').value;
            document.getElementById('ad-manual-inputs').style.display = mode === 'manual' ? 'block' : 'none';
            document.getElementById('ad-auto-msg').style.display = mode === 'auto' ? 'block' : 'none';
        },
        renderIconPicker: () => {
            const icons = ['📦','👕','👖','👟','💍','🕶️','📱','💻','📷','🎮','🧸','⚽','🍔','☕','🥤','🍞','🍎','🥕','💊','🧼','🏠','🔧','🚗','✈️','🎁'];
            const grid = document.getElementById('icon-picker'); grid.innerHTML = "";
            icons.forEach(icon => { grid.innerHTML += `<div class="icon-option" onclick="Controller.selectIcon(this, '${icon}')">${icon}</div>`; });
        },
        renderCats: () => {
            const cats = ["All", ...new Set(State.inventory.map(i=>i.cat))];
            document.getElementById('categoryList').innerHTML = cats.map(c=>`<button class="cat-pill ${c===State.filter?'active':''}" onclick="Controller.setFilter('${c}',this)">${c}</button>`).join('');
        },
        renderGrid: (term="") => {
            const g = document.getElementById('productGrid'); g.innerHTML = "";
            const today = new Date().toISOString().split('T')[0];
            State.inventory.filter(i=>(State.filter==="All"||i.cat===State.filter) && i.name.toLowerCase().includes(term.toLowerCase())).forEach(i=>{
                let stockClass = i.stock > 10 ? 'stock-ok' : (i.stock > 0 ? 'stock-low' : 'stock-out');
                const hasPromo = i.promoPrice && i.promoDate >= today;
                const displayPrice = hasPromo ? `<span style="text-decoration:line-through; font-size:0.8em; color:#999;">${Utils.fmt(i.price)}</span> <span style="color:#ef4444;">${Utils.fmt(i.promoPrice)}</span>` : Utils.fmt(i.price);
                g.innerHTML += `<div class="card ${i.stock<=0?'disabled':''}" onclick="Controller.addToCart(${i.id})"><span class="stock-tag ${stockClass}">${i.stock} Left</span><div style="font-size:2.5rem; margin:10px 0;">${i.icon||'📦'}</div><div style="font-weight:700; font-size:0.9rem; text-align:center;">${i.name}</div><div style="font-size:0.95rem; color:var(--primary); font-weight:800; margin-top:5px;">${displayPrice}</div></div>`;
            });
        },
        renderCart: () => {
            const l = document.getElementById('cartList'); l.innerHTML = ""; let sub = 0;
            const txnDisplay = document.getElementById('txn-id');
            if(State.cart.length===0) { l.innerHTML=`<div class="empty-state"><div>🛒</div>Start scanning items</div>`; if(!State.currentTxn.editRefId) { txnDisplay.innerText = "Scan/Add to start"; txnDisplay.style.color = "var(--text-muted)"; } else { txnDisplay.innerText = State.currentTxn.id; } } 
            else { if(!State.currentTxn.id) State.currentTxn.id = Utils.genId('ORD'); txnDisplay.innerText = State.currentTxn.id; txnDisplay.style.color = "var(--primary)"; }

            State.cart.forEach(i => {
                let badge = "";
                if (State.currentTxn.editRefId) {
                    const origItem = State.currentTxn.originalSnapshot ? State.currentTxn.originalSnapshot.find(x=>x.id===i.id) : null;
                    const diff = i.qty - (origItem ? origItem.qty : 0);
                    if (diff > 0) badge = `<div class="diff-badge diff-plus">+${diff} ADDED</div>`;
                    else if (diff < 0) badge = `<div class="diff-badge diff-minus">${diff} REMOVED</div>`;
                }
                
                const today = new Date().toISOString().split('T')[0];
                const origInvItem = State.inventory.find(x=>x.id===i.id);
                const isPromo = origInvItem && origInvItem.promoPrice && origInvItem.promoDate >= today;
                const priceDisplay = isPromo ? `<span class="promo-badge">DEAL</span> ${Utils.fmt(i.price)}` : Utils.fmt(i.price);

                if (i.qty === 0) {
                    l.innerHTML += `<div class="cart-item zero-qty"><div><div class="removed-badge">REMOVED</div><small style="text-decoration:line-through;">${i.name}</small></div><button onclick="Controller.restoreCartItem(${i.id})" style="background:none; border:none; font-size:0.8rem; text-decoration:underline; color:var(--primary);">Undo</button></div>`;
                } else {
                    sub += i.price * i.qty;
                    l.innerHTML += `<div class="cart-item"><div><b>${i.name}</b><br><small style="color:var(--text-muted);">${priceDisplay}</small><br>${badge}</div><div class="qty-box"><button class="qty-btn" onclick="Controller.modQty(${i.id},-1)">-</button><span style="font-weight:bold; width:20px; text-align:center;">${i.qty}</span><button class="qty-btn" onclick="Controller.modQty(${i.id},1)">+</button></div></div>`;
                }
            });

            let discountAmount = 0;
            if(State.currentTxn.discount) {
                if(State.currentTxn.discount.type === 'percent') discountAmount = sub * (State.currentTxn.discount.value / 100);
                else discountAmount = State.currentTxn.discount.value;
            }
            if (discountAmount > sub) discountAmount = sub;

            const taxableAmount = sub - discountAmount;
            const tax = taxableAmount * CONFIG.TAX_RATE;
            const total = taxableAmount + tax;

            document.getElementById('sub-disp').innerText = Utils.fmt(sub);
            document.getElementById('tax-disp').innerText = Utils.fmt(tax);
            document.getElementById('total-disp').innerText = Utils.fmt(total);
            const discRow = document.getElementById('discount-row');
            if(discountAmount > 0) { discRow.style.display = 'flex'; document.getElementById('disc-disp').innerText = `-${Utils.fmt(discountAmount)} (${State.currentTxn.discount.type === 'loyalty' ? 'Pts' : 'Disc'})`; } else { discRow.style.display = 'none'; }

            View.updatePayBtn(total);
            View.updateCustomerDisplay();
        },
        updatePayBtn: (tot) => {
            const b = document.getElementById('payBtn');
            const lBtn = document.getElementById('btn-loyalty');
            if(State.currentTxn.customer && State.currentTxn.customer.points > 0 && tot > 0) lBtn.disabled = false; else lBtn.disabled = true;

            if(State.currentTxn.editRefId) {
                const diff = tot - State.currentTxn.originalTotal;
                if(Math.abs(diff)<0.01) { b.innerText="NO CHARGE"; b.style.background="var(--text-muted)"; }
                else if(diff>0) { b.innerText=`PAY ${Utils.fmt(diff)}`; b.style.background="var(--success)"; }
                else { b.innerText=`REFUND ${Utils.fmt(Math.abs(diff))}`; b.style.background="var(--danger)"; }
                b.disabled=false;
            } else { b.innerText = tot<=0 ? "PAY $0.00" : `PAY ${Utils.fmt(tot)}`; b.disabled = tot<=0; b.style.background="var(--success)"; }
        },
        
        // --- DUAL SCREEN ---
        updateCustomerDisplay: (overrideStatus = null, payload = null) => {
            if(!State.customerWindow || State.customerWindow.closed) return;
            const doc = State.customerWindow.document;
            const cart = State.cart.filter(i=>i.qty>0);
            
            let sub = 0;
            let grossTotalWithTax = 0; 
            let totalSavings = 0;

            cart.forEach(item => {
                sub += item.price * item.qty;
                let originalPrice = item.originalPrice || item.price; 
                grossTotalWithTax += (originalPrice * item.qty);
            });

            grossTotalWithTax = grossTotalWithTax * (1 + CONFIG.TAX_RATE);
            let discount = 0;
            if(State.currentTxn.discount) { discount = State.currentTxn.discount.type === 'percent' ? sub * (State.currentTxn.discount.value/100) : State.currentTxn.discount.value; if(discount > sub) discount = sub; }
            let tax = (sub - discount) * CONFIG.TAX_RATE;
            let total = (sub - discount) + tax;
            totalSavings = grossTotalWithTax - total;
            if(totalSavings < 0.01) totalSavings = 0;
            
            // NEW: Override state handling
            if(overrideStatus) State.displayMode = overrideStatus; 
            else if (cart.length === 0 && !State.currentTxn.editRefId) State.displayMode = 'idle';
            else State.displayMode = 'active';

            const cust = State.currentTxn.customer;
            let custHtml = "";
            if(cust) {
                const tier = Utils.getTier(cust.points);
                const progress = ((cust.points % 100) / 100) * 100;
                custHtml = `<div style="background:white; padding:15px; border-radius:12px; margin-bottom:20px; box-shadow:0 4px 6px rgba(0,0,0,0.05);"><div style="font-size:1.4rem; font-weight:700; color:#4f46e5;">Hello, ${cust.name.split(' ')[0]}!</div><div style="display:flex; justify-content:space-between; align-items:center; margin-top:5px;"><div style="font-weight:800;">${tier.icon} ${tier.name}</div><div style="font-size:0.9rem; color:#666;">${cust.points} Pts</div></div><div style="width:100%; height:8px; background:#e5e7eb; border-radius:4px; margin-top:5px; overflow:hidden;"><div style="width:${progress}%; height:100%; background:#f59e0b;"></div></div></div>`;
            }

            let adHtml = "";
            if (State.displayMode === 'idle' || State.displayMode === 'payment') {
                if(State.adSettings.mode === 'auto') {
                    const today = new Date().toISOString().split('T')[0];
                    const activeDeals = State.inventory.filter(p => p.promoPrice && p.promoDate >= today);
                    if(activeDeals.length > 0) {
                        const deal = activeDeals[State.slideIndex % activeDeals.length];
                        adHtml = `<div style="margin-top:20px; background:#fff; padding:20px; border-radius:16px; border:2px solid #ef4444; box-shadow:0 10px 20px rgba(239,68,68,0.1); animation:fadeIn 1s;"><div style="font-weight:900; font-size:1.2rem; color:#ef4444; margin-bottom:10px; display:flex; align-items:center; gap:5px;">🔥 LIMITED DEAL</div><div style="display:flex; align-items:center; gap:15px;"><div style="font-size:3rem;">${deal.icon}</div><div><div style="font-weight:bold; font-size:1.2rem; color:#1f2937;">${deal.name}</div><div><span style="text-decoration:line-through; color:#9ca3af;">${Utils.fmt(deal.price)}</span> <span style="color:#ef4444; font-weight:900; font-size:1.4rem;">${Utils.fmt(deal.promoPrice)}</span></div></div></div><div style="font-size:0.8rem; color:#6b7280; margin-top:5px;">Offer ends ${new Date(deal.promoDate).toLocaleDateString()}</div></div>`;
                    } else { adHtml = `<div style="margin-top:20px; text-align:center; color:#9ca3af;">Check back soon for new deals!</div>`; }
                } else if (State.adSettings.title) {
                    let prodHtml = "";
                    if(State.adSettings.prodId) { const p = State.inventory.find(x => x.id == State.adSettings.prodId); if(p) prodHtml = `<div style="margin-top:10px; padding:10px; background:rgba(255,255,255,0.2); border-radius:8px; display:flex; align-items:center; gap:10px;"><div style="font-size:2rem;">${p.icon}</div><div style="text-align:left;"><div style="font-weight:bold;">${p.name}</div><div>${Utils.fmt(p.price)}</div></div></div>`; }
                    adHtml = `<div style="margin-top:20px; background:rgba(0,0,0,0.2); padding:20px; border-radius:16px;"><div style="font-weight:800; font-size:1.2rem; text-transform:uppercase; margin-bottom:5px;">📢 ${State.adSettings.title}</div><div style="opacity:0.9;">${State.adSettings.msg}</div>${prodHtml}</div>`;
                }
                if(adHtml) adHtml = `<div style="margin-top:20px; padding-top:20px; border-top:1px solid rgba(0,0,0,0.1); animation:pulse 3s infinite;"><div class="ad-label">AD</div>${adHtml}</div>`;
            }

            // Savings so far
            let savingsHtml = "";
            if(totalSavings > 0) {
                savingsHtml = `<div style="margin-top:5px; color:#22c55e; font-weight:700; font-size:1.2rem;">Savings so far: ${Utils.fmt(totalSavings)}</div>`;
            }

            const nowStr = new Date().toLocaleTimeString();
            const styles = `<style>@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;800&display=swap'); * { box-sizing: border-box; } body { background: #fff; margin: 0; height: 100vh; display: grid; grid-template-columns: 1.5fr 1fr; font-family: 'Inter', sans-serif; overflow: hidden; } .left-panel { background: #f8fafc; padding: 40px; display: flex; flex-direction: column; border-right: 1px solid #e2e8f0; } .header { margin-bottom: 20px; border-bottom: 2px solid #e2e8f0; padding-bottom: 20px; display: flex; justify-content: space-between; align-items: center; } .store-name { font-weight: 900; font-size: 1.8rem; color: #4f46e5; text-transform: uppercase; letter-spacing: 1px; } .clock { font-size: 1.2rem; font-weight:700; color: #64748b; font-variant-numeric: tabular-nums; } .item-list { flex: 1; overflow-y: auto; padding-right: 10px; } .item-row { display: flex; justify-content: space-between; align-items: center; background: white; margin-bottom: 15px; padding: 20px; border-radius: 16px; box-shadow: 0 4px 6px -1px rgba(0,0,0,0.05); transition: 0.2s; } .item-info { display: flex; align-items: center; gap: 20px; } .item-icon { font-size: 2.5rem; } .item-details { display: flex; flex-direction: column; } .item-qty { background: #e0e7ff; color: #4f46e5; padding: 4px 12px; border-radius: 20px; font-weight: 800; font-size: 0.9rem; display: inline-block; margin-bottom: 4px; width: fit-content;} .item-name { font-size: 1.3rem; font-weight: 700; color: #334155; } .item-price { font-size: 1.4rem; font-weight: 800; color: #0f172a; } .right-panel { background: #fff; padding: 50px; display: flex; flex-direction: column; justify-content: center; position: relative; } .summary-card { background: #4f46e5; color: white; padding: 40px; border-radius: 24px; box-shadow: 0 20px 40px -10px rgba(79, 70, 229, 0.5); } .summary-row { display: flex; justify-content: space-between; font-size: 1.4rem; margin-bottom: 15px; opacity: 0.9; } .total-section { margin-top: 30px; padding-top: 20px; border-top: 2px solid rgba(255,255,255,0.2); } .total-label { font-size: 1.2rem; text-transform: uppercase; letter-spacing: 2px; opacity: 0.8; } .total-value { font-size: 4.5rem; font-weight: 900; line-height: 1; margin-top: 10px; } .state-overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: white; display: flex; flex-direction: column; align-items: center; justify-content: center; z-index: 100; text-align: center; } .state-icon { font-size: 8rem; margin-bottom: 20px; animation: bounce 2s infinite; } .state-text { font-size: 3rem; font-weight: 900; color: #333; } @keyframes bounce { 0%, 20%, 50%, 80%, 100% {transform: translateY(0);} 40% {transform: translateY(-20px);} 60% {transform: translateY(-10px);} } @keyframes pulse { 0% {transform: scale(1);} 50% {transform: scale(1.02);} 100% {transform: scale(1);} } .return-mode-banner { background: #fff7ed; color: #c2410c; padding: 15px; text-align: center; font-weight: 800; border-radius: 12px; margin-bottom: 20px; font-size: 1.2rem; border: 2px solid #fdba74; } .diff-tag { font-size: 0.8rem; font-weight: 800; padding: 2px 6px; border-radius: 4px; margin-left: 10px; } .dt-plus { background: #d1fae5; color: #065f46; } .dt-minus { background: #fee2e2; color: #991b1b; } @keyframes fadeIn { from { opacity:0; transform:translateY(10px); } to { opacity:1; transform:translateY(0); } } .ad-label { color: #e5e7eb; font-size: 0.7rem; font-weight: 700; border: 1px solid #a5b4fc; padding: 2px 6px; border-radius: 4px; display: inline-block; margin-bottom: 5px; opacity: 0.6; } .old-price-cross { text-decoration: line-through; color: rgba(255,255,255,0.6); font-size: 2rem; font-weight: 700; text-align: right; }</style>`;

            if(State.displayMode === 'payment') { 
                doc.body.innerHTML = styles + `<div class="state-overlay"><div class="state-icon">💳</div><div class="state-text">Processing ${payload.type}...</div><div style="font-size:4rem; font-weight:800; color:${payload.amount < 0 ? 'red' : 'green'}; margin-top:20px;">${Utils.fmt(Math.abs(payload.amount))}</div><p style="font-size:1.5rem; color:#666; margin-top:20px;">Please wait.</p>${adHtml}</div>`; 
                return; 
            }
            if(State.displayMode === 'success') { 
                doc.body.innerHTML = styles + `<div class="state-overlay"><div class="state-icon">✅</div><div class="state-text">Thank You!</div>
                ${payload.saved > 0 ? `<div style="font-size:2.5rem; color:#22c55e; font-weight:800; margin-top:10px;">You Saved ${Utils.fmt(payload.saved)}!</div>` : ''}
                ${payload.points > 0 ? `<div style="font-size:1.5rem; color:#eab308; margin-top:10px;">⭐ +${payload.points} Points Earned</div>` : ''}
                <div style="margin-top:20px; font-size:1.5rem; color:#333; font-weight:bold;">Order ${State.sales[0].id}</div>
                </div>`; 
                return; 
            }
            if(State.displayMode === 'idle') { 
                doc.body.innerHTML = styles + `<div class="state-overlay" style="background:#f8fafc;"><div class="state-icon">👋</div><div class="state-text" style="color:#4f46e5;">${CONFIG.STORE_NAME}</div><p style="font-size:2rem; color:#64748b; margin-top:10px;">Next Customer Please</p>${adHtml}</div>`; 
                return; 
            }

            // --- ACTIVE CART ---
            let rows = cart.map(i => {
                let badge = "";
                if (State.currentTxn.editRefId) {
                    const origItem = State.currentTxn.originalSnapshot ? State.currentTxn.originalSnapshot.find(x=>x.id===i.id) : null;
                    const diff = i.qty - (origItem ? origItem.qty : 0);
                    if (diff > 0) badge = `<span class="diff-tag dt-plus">+${diff} Added</span>`;
                    else if (diff < 0) badge = `<span class="diff-tag dt-minus">${diff} Removed</span>`;
                }
                const today = new Date().toISOString().split('T')[0];
                const origInvItem = State.inventory.find(x=>x.id===i.id);
                const isPromo = origInvItem && origInvItem.promoPrice && origInvItem.promoDate >= today;
                const promoBadge = isPromo ? `<span style="background:#ef4444; color:white; padding:2px 6px; border-radius:4px; font-size:0.7rem; font-weight:bold; margin-left:5px;">🔥 LIMITED DEAL</span>` : '';
                const priceDisplay = isPromo ? `<span style="text-decoration:line-through; color:#9ca3af; font-size:0.8em;">${Utils.fmt(origInvItem.price)}</span> <span style="color:#ef4444;">${Utils.fmt(i.price * i.qty)}</span>` : Utils.fmt(i.price * i.qty);

                return `<div class="item-row"><div class="item-info"><div class="item-icon">${i.icon}</div><div class="item-details"><div class="item-qty">x${i.qty}</div><span class="item-name">${i.name}${badge}${promoBadge}</span></div></div><div class="item-price">${priceDisplay}</div></div>`;
            }).join('');
            
            let returnBanner = State.currentTxn.editRefId ? `<div class="return-mode-banner">⚠️ RETURN / EXCHANGE MODE</div>` : '';

            doc.body.innerHTML = styles + `<div class="left-panel"><div class="header"><div class="store-name">${CONFIG.STORE_NAME}</div><div class="clock" id="cust-clock">${nowStr}</div><div style="font-size:0.8rem; color:#999; font-weight:700;">${State.currentTxn.id}</div></div>${custHtml}${returnBanner}<div class="item-list">${rows}</div></div><div class="right-panel"><div class="summary-card"><div class="summary-row"><span>Subtotal</span><span>${Utils.fmt(sub)}</span></div>${discount > 0 ? `<div class="summary-row" style="color:#a7f3d0;"><span>Savings</span><span>-${Utils.fmt(discount)}</span></div>` : ''}<div class="summary-row"><span>Tax</span><span>${Utils.fmt(tax)}</span></div><div class="total-section"><div class="total-label">${State.currentTxn.editRefId ? 'Exchange Balance' : 'Total to Pay'}</div>${totalSavings > 0 ? `<div class="old-price-cross">${Utils.fmt(grossTotalWithTax)}</div>` : ''}<div class="total-value">${Utils.fmt(total)}</div>${savingsHtml}</div></div></div>`;
        },
        
        renderAdSelect: () => {
            const sel = document.getElementById('ad-prod-select'); sel.innerHTML = '<option value="">-- None --</option>';
            State.inventory.forEach(p => sel.innerHTML += `<option value="${p.id}">${p.name} (${Utils.fmt(p.price)})</option>`); sel.value = State.adSettings.prodId;
        },
        saveAdSettings: () => { State.adSettings = { mode: document.getElementById('ad-mode').value, title: document.getElementById('ad-title').value, msg: document.getElementById('ad-msg').value, prodId: document.getElementById('ad-prod-select').value, duration: parseInt(document.getElementById('ad-duration').value) }; Utils.save(); Utils.toast("Display Updated"); View.updateCustomerDisplay(); Controller.startSlideshow(); },

        // ... (Rest of standard renderers)
        renderCustomers: () => { const tbody = document.getElementById('custTableBody'); tbody.innerHTML = ""; State.customers.forEach(c => { const t = Utils.getTier(c.points); tbody.innerHTML += `<tr><td>${c.name}</td><td>${c.phone}</td><td>${c.email}</td><td><span class="tier-badge ${t.class}">${t.icon} ${t.name}</span></td><td>${c.points||0}</td></tr>`; }); },
        renderCustSelect: (filter="") => { const l = document.getElementById('cust-select-list'); l.innerHTML = ""; State.customers.filter(c=>c.name.toLowerCase().includes(filter.toLowerCase())).forEach(c => { l.innerHTML += `<div onclick="Controller.selectCustomer(${c.id})" style="padding:10px; border-bottom:1px solid var(--border); cursor:pointer; hover:background:var(--bg);"><b>${c.name}</b> <small>(${c.phone})</small></div>`; }); },
        renderShift: () => { const ui = document.getElementById('shift-ui'); if(State.cashDrawer.status === 'closed') { ui.innerHTML = `<div class="stat-card" style="text-align:center; max-width:400px; margin:auto;"><h2>🏁 Start Shift</h2><p style="margin-bottom:20px;">Enter starting cash in drawer.</p><input id="shift-start-amt" type="number" class="search-input" placeholder="$0.00" style="margin-bottom:15px;"><button class="btn-lg" style="background:var(--primary);" onclick="Controller.startShift()">Open Register</button></div>`; } else { const s = State.cashDrawer.currentShift; const cashSales = State.sales.filter(x => x.method==='Cash' && new Date(x.date) > new Date(s.startTime)).reduce((a,b)=>a+b.netDifference, 0); const expected = s.startCash + cashSales; ui.innerHTML = `<div class="stat-card" style="max-width:500px; margin:auto;"><h2>🟢 Shift Open</h2><div style="margin:20px 0; font-size:0.9rem;"><div class="receipt-row"><span>Started:</span><span>${new Date(s.startTime).toLocaleString()}</span></div><div class="receipt-row"><span>Start Cash:</span><span>${Utils.fmt(s.startCash)}</span></div><div class="receipt-row"><span>Cash Sales (Est):</span><span>${Utils.fmt(cashSales)}</span></div><div class="receipt-row bold" style="margin-top:10px; border-top:1px solid var(--border); padding-top:5px;"><span>Expected Total:</span><span>${Utils.fmt(expected)}</span></div></div><h3>End Shift</h3><p>Enter actual cash count:</p><input id="shift-end-amt" type="number" class="search-input" placeholder="$0.00" style="margin-bottom:15px;"><button class="btn-lg" style="background:var(--danger);" onclick="Controller.endShift(${expected})">Close Shift & Print Report</button></div>`; } const hBody = document.getElementById('shiftHistoryBody'); hBody.innerHTML = ""; (State.cashDrawer.history || []).forEach(h => { hBody.innerHTML += `<tr><td>${new Date(h.startTime).toLocaleString()}</td><td>${Utils.fmt(h.startCash)}</td><td>${h.actualEnd ? Utils.fmt(h.actualEnd) : '-'}</td><td>${h.variance ? Utils.fmt(h.variance) : '-'}</td></tr>`; }); },
        renderHistory: (f="") => { const l=document.getElementById('historyList'); l.innerHTML=""; const filtered = State.sales.filter(s=>s.id.toLowerCase().includes(f.toLowerCase())); if(filtered.length === 0) l.innerHTML = `<div class="empty-state"><div>🔍</div>No history found</div>`; filtered.forEach(s=>{ const isMod = s.status==='Modified'||s.status==='Finalized'; l.innerHTML += `<div onclick="View.previewHistory('${s.id}', this)" class="history-item" style="opacity:${isMod?0.6:1}"><div style="display:flex; justify-content:space-between; font-weight:bold; margin-bottom:5px;"><span>${s.refId?'↩️':'🛒'} ${s.id}</span><span style="color:var(--primary);">${Utils.fmt(s.total)}</span></div><div style="display:flex; justify-content:space-between; font-size:0.75rem; color:var(--text-muted);"><span>${new Date(s.date).toLocaleDateString()}</span><span>${s.status}</span></div></div>`; }); },
        previewHistory: (id, el) => { document.querySelectorAll('.history-item').forEach(e=>e.classList.remove('active')); el.classList.add('active'); const s = State.sales.find(x=>x.id===id); document.getElementById('historyPreview').innerHTML = View.genReceipt(s); },
        openDetailModal: (sale) => { State.activeDetailTxn = sale; State.tempReturnCart = JSON.parse(JSON.stringify(sale.items)); document.getElementById('detail-receipt-preview').innerHTML = View.genReceipt(sale); const isMod = sale.status === 'Modified' || sale.status === 'Finalized'; document.getElementById('dm-status-badge').innerHTML = `<span class="badge-status" style="background:${isMod?'var(--warning-bg)':'var(--success-bg)'}; color:${isMod?'var(--warning)':'var(--success)'}; font-size:1rem; padding:5px 12px;">${sale.status}</span>`; const actsEl = document.getElementById('dm-action-buttons'), msgEl = document.getElementById('dm-closed-msg'); const commDisplay = document.getElementById('dm-original-comment'); commDisplay.innerText = sale.comment ? `Original Comment: "${sale.comment}"` : ''; document.getElementById('dm-new-comment').value = ""; if(isMod || State.activeChainFinalized) { actsEl.style.display = 'none'; msgEl.style.display = 'block'; document.getElementById('dm-closed-msg').innerHTML = State.activeChainFinalized ? "⚠️ **CHAIN IS FINALIZED**. No further changes allowed." : "This transaction is modified. See newest transaction."; } else { actsEl.style.display = 'flex'; msgEl.style.display = 'none'; View.renderModalList(); } Utils.openModal('txnDetailModal'); },
        renderModalList: () => { const listEl = document.getElementById('dm-interactive-list'); listEl.innerHTML = ""; let currentSubtotal = 0; State.tempReturnCart.forEach(item => { currentSubtotal += item.price * item.qty; const originalItem = State.activeDetailTxn.items.find(x=>x.id===item.id); const maxQty = originalItem ? originalItem.qty : 0; if (maxQty === 0) return; listEl.innerHTML += `<div class="return-item-row"><div><div style="font-weight:bold;">${item.name}</div><div style="font-size:0.85rem; color:var(--text-muted);">${Utils.fmt(item.price)} ea.</div></div><div class="return-ctrl"><span style="font-size:0.8rem; color:var(--text-muted);">Keep:</span><button onclick="Controller.modalModQty(${item.id}, -1)">-</button><span style="width:20px; text-align:center; font-weight:bold;">${item.qty}</span><button onclick="Controller.modalModQty(${item.id}, 1)" ${item.qty>=maxQty?'disabled style="opacity:0.5"':''}>+</button><span style="font-size:0.8rem; color:var(--text-muted);">of ${maxQty}</span></div></div>`; }); const refundSubtotal = State.activeDetailTxn.subtotal - currentSubtotal; const refundTotal = refundSubtotal * (1 + CONFIG.TAX_RATE); const refEl = document.getElementById('dm-refund-total'); refEl.innerText = Utils.fmt(refundTotal); refEl.style.color = refundTotal < 0.01 ? "#ccc" : "var(--danger)"; },
        genReceipt: (sale) => { if (sale.status === 'Finalized') return View.genFinalReceipt(sale); const isExchange = sale.refId !== null; let itemsHtml = ''; if(!isExchange) { itemsHtml = sale.items.map(i=>`<div class="receipt-row"><span>${i.qty} x ${i.name}</span><span>${Utils.fmt(i.price*i.qty)}</span></div>`).join(''); } else { itemsHtml += `<div class="receipt-row bold" style="margin-top:10px; border-bottom:1px solid #000; padding-bottom:5px;">EXCHANGE DETAILS:</div>`; sale.changes.forEach(c => { const isRet = c.qtyDiff < 0; const label = isRet ? "RETURNED" : "ADDED"; const color = isRet ? "var(--danger)" : "black"; itemsHtml += `<div class="receipt-row" style="color:${color}; margin-top:4px;"><span>${label}: ${Math.abs(c.qtyDiff)} x ${c.name}</span><span>${Utils.fmt(Math.abs(c.totalDiff))}</span></div>`; }); } let custHtml = sale.customer ? `<div style="text-align:center; font-weight:bold; margin-bottom:5px;">Customer: ${sale.customer.name}</div>` : ''; let label = sale.netDifference < 0 ? "REFUND ISSUED" : (sale.netDifference===0 ? "EVEN EXCHANGE" : "TOTAL CHARGED"); let footerNote = sale.comment ? `<div class="footer-note">This Order has an internal comment.</div>` : ''; return `<div class="receipt-paper"><div class="receipt-header"><div class="receipt-logo">${CONFIG.STORE_NAME}</div><div style="font-size:0.8rem;">Store #8842 • Traceable</div><div style="font-size:0.8rem;">${new Date(sale.date).toLocaleString()}</div><div>Trx: <b>${sale.id}</b></div>${custHtml}</div><div class="dashed-line"></div>${itemsHtml}<div class="dashed-line"></div><div class="receipt-row"><span>New Subtotal</span><span>${Utils.fmt(sale.subtotal)}</span></div><div class="receipt-row"><span>Tax (${(CONFIG.TAX_RATE*100).toFixed(0)}%)</span><span>${Utils.fmt(sale.tax)}</span></div><div class="receipt-row bold" style="font-size:1.1em; margin-bottom:10px;"><span>NEW CART TOTAL</span><span>${Utils.fmt(sale.total)}</span></div>${isExchange ? `<div style="background:#f3f4f6; padding:8px; margin-bottom:5px; border-radius:4px;"><div class="receipt-row" style="color:var(--text-muted);"><span>Previous Balance</span><span>-${Utils.fmt(sale.originalTotal)}</span></div><div class="receipt-row bold" style="border-top:1px solid #ccc; margin-top:5px; padding-top:5px;"><span>${label}</span><span>${Utils.fmt(Math.abs(sale.netDifference))}</span></div></div>` : `<div class="receipt-row bold"><span>TOTAL DUE</span><span>${Utils.fmt(sale.total)}</span></div>`}<div class="receipt-row"><span>Payment (${sale.method})</span><span>${Utils.fmt(Math.abs(sale.netDifference))}</span></div>${footerNote}<div class="receipt-footer" style="margin-top:25px;"><div class="barcode">*${sale.id}*</div></div></div>`; },
        genFinalReceipt: (finalTxn) => { const $ = Utils.fmt; let netRevenue = finalTxn.netRevenue || 0, finalSubtotal = netRevenue / (1 + CONFIG.TAX_RATE), finalTax = netRevenue - finalSubtotal; return `<div class="receipt-paper" style="border: 2px solid var(--primary); background: #eff6ff;"><div class="receipt-header" style="color:var(--primary);"><div class="receipt-logo">AUDIT FINALIZATION</div><div style="font-size:0.8rem;">Chain Locked</div><div>Trx: <b>${finalTxn.id}</b></div><div style="font-size:11px;">Root: ${finalTxn.refId}</div></div><div class="dashed-line" style="border-top: 2px dashed var(--primary);"></div><div style="text-align:center; font-weight:bold; margin-bottom:10px;">NET ITEMS KEPT:</div>${finalTxn.finalItems.map(i => `<div class="receipt-row"><span>${i.qty} x ${i.name}</span><span>${$(i.price * i.qty)}</span></div>`).join('')}<div class="dashed-line"></div><div class="receipt-row bold" style="font-size:1.2em; color:var(--primary);"><span>NET REVENUE</span><span>${$(netRevenue)}</span></div><div class="receipt-footer" style="margin-top:20px;"><div class="barcode">*${finalTxn.id}*</div><div style="font-size:10px; margin-top:5px; color:var(--danger); font-weight:bold;">CHAIN IS IMMUTABLE. NO FURTHER ACTION POSSIBLE.</div></div></div>`; },
        renderTimeline: (family) => { const c = document.getElementById('timeline-area'); c.innerHTML = ""; c.style.display='block'; State.activeChainFamily = family; State.activeChainFinalized = family.some(t => t.status === "Finalized"); let displayFamily = [...family], finalTxn = null; if (State.activeChainFinalized) { finalTxn = displayFamily.find(t => t.status === "Finalized"); displayFamily = displayFamily.filter(t => t.status !== "Finalized"); } const finalizeContainer = document.getElementById('finalize-container'); const isFinished = family.length > 0 && !State.activeChainFinalized; finalizeContainer.style.display = isFinished ? 'flex' : 'none'; if (finalTxn) { const s = finalTxn; c.innerHTML += `<div class="timeline-node" style="opacity:1; background: var(--bg); padding:15px; border-radius:16px; margin-bottom:25px; border: 2px solid var(--primary);"><div class="timeline-left" style="height:32px;"><div class="timeline-icon" style="background:#fff; border-color:var(--primary);">✅</div></div><div class="timeline-content" style="border:none; box-shadow:none; background:transparent; padding:0;"><div style="display:flex; justify-content:space-between; align-items:center;"><div style="display:flex; align-items:center; gap:10px;"><span style="font-weight:900; font-size:1.2rem; color:var(--primary);">FINALIZED AUDIT</span><button onclick="Controller.openDetailModal('${s.id}')" title="View Document" style="border:none; background:none; font-size:1.3rem; padding:0;">👁️</button></div><span class="badge-status" style="background:var(--primary); color:white;">${s.status}</span></div><div style="font-size:0.9rem; color:var(--text-muted); margin-top:5px;">Net Chain Revenue: ${Utils.fmt(s.netRevenue)}</div></div></div>`; } displayFamily.forEach((s, i) => { const isRoot=i===0, isRef=s.netDifference<0, isMod=s.status==="Modified"; let icon="🛒", bg="#fff", bc="var(--border)"; if(isRef) { icon="💸"; bc="var(--danger)"; } else if(s.netDifference>0 && !isRoot){ icon="💳"; bc="var(--success)"; } c.innerHTML += `<div class="timeline-node"><div class="timeline-left"><div class="timeline-icon" style="background:${bg}; border-color:${bc};">${icon}</div>${i!==displayFamily.length-1?'<div class="timeline-line"></div>':''}</div><div class="timeline-content"><div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:10px;"><div style="display:flex; align-items:center; gap:10px;"><span style="font-weight:700; font-size:1.1rem;">${s.id}</span><button onclick="Controller.openDetailModal('${s.id}')" title="View Document" style="border:none; background:none; font-size:1.2rem; padding:0;">👁️</button></div><span class="badge-status" style="background:${isMod?'var(--warning-bg)':'var(--bg)'}; color:${isMod?'var(--warning)':'var(--text-muted)'};">${s.status}</span></div><div style="background:var(--bg); padding:12px; border-radius:8px; font-size:0.9rem;">${isRoot?`Original Purchase of <b>${s.items.reduce((a,b)=>a+b.qty,0)} items</b>.`:`Activity: ${s.changes.map(x=>`<span style="color:${x.qtyDiff<0?'var(--danger)':'var(--success)'}">${x.qtyDiff>0?'+':''}${x.qtyDiff} ${x.name}</span>`).join(', ')}`}</div><div style="display:flex; justify-content:space-between; margin-top:15px; font-weight:600;"><div>Total: ${Utils.fmt(s.total)}</div><div style="color:${isRef?'var(--danger)':'var(--success)'}">${isRef?'Refunded':'Paid'}: ${Utils.fmt(Math.abs(s.netDifference))}</div></div></div></div>`; }); View.renderTraceTable(family); },
        renderTraceTable: (family) => { const tbody = document.getElementById('trace-table-body'); document.getElementById('chain-table-area').style.display = 'block'; tbody.innerHTML = ""; const chronological = [...family].sort((a,b) => new Date(a.date) - new Date(b.date)); chronological.forEach(txn => { if (txn.status === 'Finalized') return; const dateStr = new Date(txn.date).toLocaleString(); const isReturn = txn.changes && txn.changes.length > 0; if (!isReturn) { txn.items.forEach(item => { tbody.innerHTML += `<tr><td>${dateStr}</td><td>${txn.id}</td><td>${item.name}</td><td>${Utils.fmt(item.price)}</td><td><span class="movement-plus">+${item.qty}</span></td></tr>`; }); } else { txn.changes.forEach(change => { const sign = change.qtyDiff > 0 ? '+' : ''; const cls = change.qtyDiff > 0 ? 'movement-plus' : 'movement-minus'; tbody.innerHTML += `<tr><td>${dateStr}</td><td>${txn.id}</td><td>${change.name}</td><td>${Utils.fmt(change.price)}</td><td><span class="${cls}">${sign}${change.qtyDiff}</span></td></tr>`; }); } }); },
        renderPS: () => { const c = document.getElementById('psGrid'); document.getElementById('ps-badge').style.display=State.psQueue.length?'block':'none'; if (State.psQueue.length === 0) { c.innerHTML = `<div class="empty-state"><div>📦</div>Quarantine is empty</div>`; return; } const groups = {}; State.psQueue.forEach(item => { if(!groups[item.origTxn]) groups[item.origTxn] = []; groups[item.origTxn].push(item); }); c.innerHTML = ""; Object.keys(groups).forEach(txnId => { const items = groups[txnId]; const dateStr = items[0].origTxnDate ? new Date(items[0].origTxnDate).toLocaleDateString() : "Unknown"; let itemsHtml = items.map(item => `<div class="ps-item-row"><div class="ps-item-meta"><strong>${item.name} (x${item.qty})</strong><span style="font-size:0.8rem; color:var(--text-muted);">Reason: ${item.reason}</span></div><div class="ps-actions"><button class="btn-lg" style="flex:1; font-size:0.8rem; padding:8px; background:var(--success);" onclick="Controller.resolvePs('${item.id}', true)">Restock</button><button class="btn-lg" style="flex:1; font-size:0.8rem; padding:8px; background:var(--danger);" onclick="Controller.resolvePs('${item.id}', false)">Discard</button></div></div>`).join(''); c.innerHTML += `<div class="ps-group-card"><div class="ps-group-header"><div><strong style="color:var(--primary);">${txnId}</strong> <span style="font-size:0.8rem; color:var(--text-muted);">(${dateStr})</span></div><button onclick="Controller.openDetailModal('${txnId}')" style="border:1px solid var(--border); background:var(--panel); padding:6px 10px; border-radius:6px; font-size:0.8rem; cursor:pointer;">📄 Receipt</button></div><div class="ps-item-list">${itemsHtml}</div></div>`; }); },
        renderInv: () => { const tbody = document.getElementById('invTableBody'); tbody.innerHTML = ""; const today = new Date().toISOString().split('T')[0]; State.inventory.forEach(i => { const deal = i.promoPrice && i.promoDate >= today ? `<div class="inv-promo">${Utils.fmt(i.promoPrice)} until ${i.promoDate}</div>` : '-'; const priceDisplay = i.promoPrice && i.promoDate >= today ? `<span class="inv-old-price">${Utils.fmt(i.price)}</span>` : Utils.fmt(i.price); tbody.innerHTML += `<tr style="border-bottom:1px solid var(--border);"><td style="padding:15px; font-weight:600;">${i.icon} ${i.name}</td><td style="color:var(--text-muted);">${i.cat}</td><td>${priceDisplay}</td><td>${deal}</td><td>${i.stock}</td><td><button onclick="Controller.delProd(${i.id})" style="color:var(--danger); border:none; background:none; font-size:1.2rem;">×</button></td></tr>`; }); },
        updateDash: () => { const v = State.sales.filter(s=>s.status!=='Modified'&&s.status!=='Finalized'); document.getElementById('d-rev').innerText = Utils.fmt(v.reduce((a,b)=>a+b.total,0)); document.getElementById('d-count').innerText = v.length; document.getElementById('d-returns').innerText = State.psQueue.length; }
    };

    const Controller = {
        init: () => { Utils.load(); Controller.newTxn(); View.init(); Utils.startClock(); Controller.startSlideshow(); },
        switchView: (id) => { document.querySelectorAll('.view').forEach(e=>e.classList.remove('active')); document.getElementById(id).classList.add('active'); document.querySelectorAll('.nav-item').forEach(e=>e.classList.remove('active')); if(event) event.currentTarget.classList.add('active'); if (id !== 'returns') { document.getElementById('timeline-area').style.display='none'; document.getElementById('chain-table-area').style.display='none'; } if (id === 'history') View.renderHistory(); if (id === 'register') View.renderShift(); },
        newTxn: () => { 
            State.cart=[]; 
            // Add txnStartTime & txnTimerInterval reset
            if (State.txnTimerInterval) clearInterval(State.txnTimerInterval);
            State.txnStartTime = null;
            document.getElementById('txn-timer').style.display = 'none';
            
            State.currentTxn={id:Utils.genId('ORD'), editRefId:null, originalSnapshot:null, originalTotal:0, customer:null, discount:null, pointsRedeemed: 0}; 
            document.getElementById('edit-mode-bar').style.display='none'; 
            document.getElementById('active-customer-name').innerText = "👤 Guest Customer"; 
            View.renderCart(); View.updateCustomerDisplay(); 
        },
        
        // NEW TIMER CONTROLLER METHODS
        startTxnTimer: () => {
            if(State.txnTimerInterval) clearInterval(State.txnTimerInterval);
            State.txnStartTime = Date.now();
            // Immediate update
            Controller.updateTxnTimer();
            State.txnTimerInterval = setInterval(Controller.updateTxnTimer, 1000);
        },
        updateTxnTimer: () => {
            if(!State.txnStartTime) return;
            const now = Date.now();
            const diff = Math.floor((now - State.txnStartTime) / 1000);
            const m = Math.floor(diff / 60).toString().padStart(2,'0');
            const s = (diff % 60).toString().padStart(2,'0');
            const el = document.getElementById('txn-timer');
            if(el) {
                el.style.display = 'block';
                el.innerText = `⏱️ ${m}:${s}`;
            }
        },
        stopTxnTimer: () => {
             if(State.txnTimerInterval) clearInterval(State.txnTimerInterval);
             State.txnStartTime = null;
             const el = document.getElementById('txn-timer');
             if(el) el.style.display = 'none';
        },

        selectIcon: (el, icon) => { document.querySelectorAll('.icon-option').forEach(e => e.classList.remove('selected')); el.classList.add('selected'); document.getElementById('n-icon').value = icon; },
        setFilter: (c,b) => { State.filter=c; document.querySelectorAll('.cat-pill').forEach(e=>e.classList.remove('active')); b.classList.add('active'); View.renderGrid(); },
        addToCart: (id) => { 
            const i=State.inventory.find(x=>x.id===id), c=State.cart.find(x=>x.id===id); 
            if(c&&c.qty>=i.stock)return Utils.toast("Stock Limit Reached","error"); 
            
            // Start timer on first add
            if(State.cart.length === 0 && !State.txnStartTime) Controller.startTxnTimer();

            const today = new Date().toISOString().split('T')[0];
            const hasPromo = i.promoPrice && i.promoDate >= today;
            const priceToUse = hasPromo ? i.promoPrice : i.price;
            if(c) c.qty++; 
            else State.cart.push({...i, price: priceToUse, originalPrice: i.price, qty:1}); 
            View.renderCart(); 
        },
        modQty: (id,d) => { const c=State.cart.find(x=>x.id===id); if(!c) return; if (c.qty + d < 0) return; const i=State.inventory.find(x=>x.id===id); if(d>0 && c.qty>=i.stock) return Utils.toast("Stock Limit Reached","error"); c.qty+=d; View.renderCart(); },
        restoreCartItem: (id) => { const c=State.cart.find(x=>x.id===id); if(c) { c.qty=1; View.renderCart(); } },
        clearCart: () => { 
            State.cart=[]; 
            Controller.stopTxnTimer(); // Reset timer
            View.renderCart(); 
        },
        openPayModal: () => { 
            if(State.cashDrawer.status === 'closed') return Utils.toast("⚠️ Register Closed. Open Shift first.", "error");
            const t=document.getElementById('total-disp').innerText; document.getElementById('payModalTotal').innerText=t; document.getElementById('payModalTotal').style.color=t.includes("REFUND")?"var(--danger)":"var(--success)"; document.getElementById('pay-summary-txt').innerText=State.currentTxn.editRefId?"Returns will be sent to Quarantine.":""; Utils.openModal('payModal');
            const rawTotal = parseFloat(t.replace(/[^0-9.-]+/g,""));
            View.updateCustomerDisplay('payment', { type: State.currentTxn.editRefId ? (rawTotal < 0 ? 'Refund' : 'Exchange') : 'Payment', amount: rawTotal });
        },
        finalizeTransaction: (method) => {
            const finalItems = State.cart.filter(i => i.qty > 0);
            let curSub=finalItems.reduce((a,b)=>a+(b.price*b.qty),0);
            let promoSavings = finalItems.reduce((acc, item) => { let orig = item.originalPrice || item.price; if(orig > item.price) return acc + ((orig - item.price) * item.qty); return acc; }, 0);
            let discAmt = 0;
            if(State.currentTxn.discount) { if(State.currentTxn.discount.type==='percent') discAmt = curSub * (State.currentTxn.discount.value/100); else discAmt = State.currentTxn.discount.value; }
            if(discAmt > curSub) discAmt = curSub;
            const totalSavings = promoSavings + discAmt;
            const taxable = curSub - discAmt; const curTax = taxable * CONFIG.TAX_RATE; const curTot = taxable + curTax;
            let origTot=0, origItems=[]; if(State.currentTxn.editRefId) { origItems=State.currentTxn.originalSnapshot||[]; origTot=origItems.reduce((a,b)=>a+(b.price*b.qty),0)*(1+CONFIG.TAX_RATE); }
            const netDiff = curTot - origTot;
            if(!State.currentTxn.id) State.currentTxn.id = Utils.genId('ORD');
            let changes = []; const allIds = new Set([...finalItems.map(i=>i.id), ...origItems.map(i=>i.id)]); const mainComment = document.getElementById('txn-comment').value;
            allIds.forEach(id => { const nQ=(finalItems.find(x=>x.id===id)||{}).qty||0, oQ=(origItems.find(x=>x.id===id)||{}).qty||0, diff=nQ-oQ; const ref=State.inventory.find(x=>x.id===id)||origItems.find(x=>x.id===id); if(diff!==0 && ref) { changes.push({name:ref.name, qtyDiff:diff, price:ref.price, totalDiff:diff*ref.price}); if(diff>0) { const inv=State.inventory.find(x=>x.id===id); if(inv) inv.stock-=diff; } else { State.psQueue.push({id:Date.now() + Math.random(), itemId:id, name:ref.name, qty:Math.abs(diff), reason: mainComment || "Return", origTxn: State.currentTxn.id, origTxnDate: new Date().toISOString()}); } } });
            if(State.currentTxn.editRefId) { const old=State.sales.find(s=>s.id===State.currentTxn.editRefId); if(old)old.status="Modified"; }
            if(State.currentTxn.discount && State.currentTxn.discount.type === 'loyalty') { const c = State.customers.find(x=>x.id===State.currentTxn.customer.id); if(c) c.points -= State.currentTxn.pointsRedeemed; }
            const rec = { id:State.currentTxn.id, date:new Date().toISOString(), refId:State.currentTxn.editRefId, subtotal:curSub, tax:curTax, total:curTot, originalTotal:origTot, netDifference:netDiff, method:method, items:JSON.parse(JSON.stringify(finalItems)), changes:changes, status:"Completed", customer: State.currentTxn.customer, discount: State.currentTxn.discount, pointsRedeemed: State.currentTxn.pointsRedeemed, comment: mainComment };
            State.sales.unshift(rec); 
            let earnedPts = 0; if(rec.customer && rec.netDifference > 0) { earnedPts = Math.floor(rec.netDifference); const c = State.customers.find(x=>x.id===rec.customer.id); if(c) c.points = (c.points || 0) + earnedPts; }
            Utils.save();
            View.updateCustomerDisplay('success', { saved: totalSavings, points: earnedPts });
            
            // Stop Timer
            Controller.stopTxnTimer();
            
            document.getElementById('receipt-content').innerHTML=View.genReceipt(rec); Utils.closeModal('payModal'); Utils.openModal('receiptModal'); View.renderGrid(); View.renderPS();
        },
        lookupChain: (searchId) => { if(!searchId) searchId = document.getElementById('return-search').value.trim(); if(!searchId) return Utils.toast("Enter ID","error"); document.getElementById('return-search').value = searchId; let cur = State.sales.find(s=>s.id===searchId); if(!cur) return Utils.toast("Not Found","error"); while(cur.refId && cur.status !== "Finalized") { const p=State.sales.find(s=>s.id===cur.refId); if(p)cur=p; else break; } let fam=[cur], add=true; while(add){add=false; const k=fam.map(f=>f.id); const ch=State.sales.filter(s=>k.includes(s.refId)&&!k.includes(s.id)); if(ch.length){fam=[...fam,...ch];add=true;}} fam.sort((a,b)=>new Date(a.date)-new Date(b.date)); View.renderTimeline(fam); },
        generateFinalReceipt: () => { if (State.activeChainFinalized || State.activeChainFamily.length === 0) return Utils.toast("Chain is already finalized or not traced.", "error"); let netRevenue = 0; const originalTxn = State.activeChainFamily.find(t => !t.refId); State.activeChainFamily.forEach(txn => { if(txn.status !== 'Finalized') { netRevenue += txn.netDifference; } }); const lastActiveTxn = State.activeChainFamily.filter(t => t.status !== 'Finalized').pop(); const finalItems = lastActiveTxn.items.reduce((acc, item) => { if (item.qty > 0) { acc.push({ id: item.id, name: item.name, price: item.price, qty: item.qty }); } return acc; }, []); const finalRecord = { id: Utils.genId('FIN'), date: new Date().toISOString(), refId: originalTxn.id, status: "Finalized", total: netRevenue, netDifference: 0, originalTotal: 0, subtotal: 0, tax: 0, method: 'N/A', netRevenue: netRevenue, finalItems: finalItems }; State.sales.unshift(finalRecord); Utils.save(); Controller.lookupChain(finalRecord.id); Utils.toast("Chain finalized. No more returns allowed.", "success"); },
        openDetailModal: (id) => { const s=State.sales.find(x=>x.id===id); if (!s) return; if (s.status === 'Finalized') { document.getElementById('receipt-content').innerHTML = View.genReceipt(s); const payBtn = document.querySelector('#receiptModal .btn-pay'); const h3 = document.querySelector('#receiptModal h3'); payBtn.style.display = 'none'; h3.innerText = 'Final Audit Document'; h3.style.color = 'var(--primary)'; Utils.openModal('receiptModal'); setTimeout(() => { payBtn.style.display = 'flex'; h3.innerText = 'Success!'; h3.style.color = 'var(--success)'; }, 100); return; } if(s) View.openDetailModal(s); },
        modalModQty: (id, delta) => { const item = State.tempReturnCart.find(x=>x.id===id); if(!item) return; const originalItem = State.activeDetailTxn.items.find(x=>x.id===id); const max = originalItem ? originalItem.qty : 0; if (item.qty + delta < 0) return; if (item.qty + delta > max) return; item.qty += delta; View.renderModalList(); },
        processInModalRefund: () => { const s = State.activeDetailTxn; const origItems = s.items; const newItems = State.tempReturnCart; let hasChange = false; origItems.forEach(o => { const n = newItems.find(x=>x.id===o.id); if(n && n.qty !== o.qty) hasChange = true; }); if(!hasChange) return Utils.toast("No items selected for return.", "warning"); const newSub = newItems.reduce((a,b)=>a+(b.price*b.qty),0); const netDiff = (newSub - s.subtotal) * (1 + CONFIG.TAX_RATE); if (netDiff > -0.01) return Utils.toast("Cannot issue refund without items marked for return.", "error"); const retId = Utils.genId('RET'); const newComment = document.getElementById('dm-new-comment').value; let changes = []; origItems.forEach(o => { const n = newItems.find(x=>x.id===o.id); const diff = n.qty - o.qty; if(diff !== 0) { changes.push({name:o.name, qtyDiff:diff, price:o.price, totalDiff:diff*o.price}); State.psQueue.push({ id: Date.now() + Math.random(), itemId:o.id, name:o.name, qty:Math.abs(diff), reason: newComment || "Quick Refund", origTxn: retId, origTxnDate: new Date().toISOString() }); } }); s.status = "Modified"; const rec = { id: retId, date: new Date().toISOString(), refId: s.id, subtotal: newSub, tax: newSub*CONFIG.TAX_RATE, total: newSub + (newSub*CONFIG.TAX_RATE), originalTotal: s.total, netDifference: netDiff, method: "Refund", items: JSON.parse(JSON.stringify(newItems)), changes: changes, status: "Completed", comment: newComment }; State.sales.unshift(rec); Utils.save(); Utils.closeModal('txnDetailModal'); Utils.toast("Refund Issued Successfully"); Controller.lookupChain(s.id); View.renderGrid(); View.renderPS(); },
        switchToExchange: () => { const s = State.activeDetailTxn; State.cart = JSON.parse(JSON.stringify(State.tempReturnCart)); State.currentTxn.originalSnapshot = JSON.parse(JSON.stringify(s.items)); State.currentTxn.editRefId = s.id; State.currentTxn.originalTotal = s.total; State.currentTxn.id = Utils.genId("EXC"); document.getElementById('txn-id').innerText = State.currentTxn.id; document.getElementById('edit-mode-bar').style.display = 'block'; document.getElementById('ref-id-display').innerText = s.id; document.getElementById('txn-comment').value = ""; Utils.closeModal('txnDetailModal'); Controller.switchView('pos'); View.renderCart(); Utils.toast("Loaded for Exchange. Add new items from grid."); },
        cancelEditMode: () => Controller.newTxn(),
        openAddProductModal: () => Utils.openModal('addProductModal'),
        saveProduct: () => { const n=document.getElementById('n-name').value, p=parseFloat(document.getElementById('n-price').value), s=parseInt(document.getElementById('n-stock').value), icon=document.getElementById('n-icon').value, pp=parseFloat(document.getElementById('n-promo-price').value), pd=document.getElementById('n-promo-date').value; if(!n||isNaN(p)) return Utils.toast("Invalid","error"); State.inventory.push({id:Date.now(), name:n, cat:document.getElementById('n-cat').value||"Gen", price:p, stock:s||0, icon:icon||'📦', promoPrice: pp || null, promoDate: pd || null}); Utils.save(); View.renderGrid(); Utils.closeModal('addProductModal'); },
        delProd: (id) => { if(confirm("Delete?")){State.inventory=State.inventory.filter(x=>x.id!==id); Utils.save(); View.renderGrid(); View.renderInv();} },
        resolvePs: (itemId, res) => { const idx = State.psQueue.findIndex(x => x.id == itemId); if(idx === -1) return; const q = State.psQueue[idx]; if(res){ const inv = State.inventory.find(x=>x.id===q.itemId); if(inv) inv.stock+=q.qty; } State.psQueue.splice(idx,1); Utils.save(); View.renderPS(); View.renderGrid(); Utils.toast(res?"Restocked":"Discarded"); },
        exportCSV: () => { const csv = "ID,Date,Type,Total,Details\n" + State.sales.map(s => `${s.id},${s.date},${s.status},${s.total},${s.items.length} items`).join("\n"); const blob = new Blob([csv], { type: 'text/csv' }); const url = window.URL.createObjectURL(blob); const a = document.createElement('a'); a.href = url; a.download = 'sales_history.csv'; a.click(); },
        openCustomerModal: () => { View.renderCustSelect(); Utils.openModal('customerModal'); },
        openAddCustomerModal: () => Utils.openModal('addCustomerModal'),
        selectCustomer: (id) => { if(id) { const c = State.customers.find(x=>x.id===id); State.currentTxn.customer = c; document.getElementById('active-customer-name').innerText = `👤 ${c.name}`; } else { State.currentTxn.customer = null; document.getElementById('active-customer-name').innerText = `👤 Guest Customer`; } Utils.closeModal('customerModal'); View.renderCart(); },
        saveCustomer: () => { const n = document.getElementById('nc-name').value; if(!n) return Utils.toast("Name required", "error"); State.customers.push({id:Date.now(), name:n, phone:document.getElementById('nc-phone').value, email:document.getElementById('nc-email').value, points:0}); Utils.save(); View.renderCustSelect(); View.renderCustomers(); Utils.closeModal('addCustomerModal'); Utils.toast("Customer Created"); },
        openLoyaltyRedeem: () => { const pts = State.currentTxn.customer.points || 0; document.getElementById('loyalty-bal-disp').innerText = pts; Utils.openModal('loyaltyModal'); },
        applyLoyalty: () => { const pts = parseInt(document.getElementById('loyalty-input').value); const max = State.currentTxn.customer.points || 0; if(isNaN(pts) || pts <= 0) return Utils.toast("Invalid Amount", "error"); if(pts > max) return Utils.toast("Insufficient Points", "error"); const discountVal = pts * 0.01; State.currentTxn.discount = { type: 'loyalty', value: discountVal }; State.currentTxn.pointsRedeemed = pts; Utils.closeModal('loyaltyModal'); View.renderCart(); Utils.toast(`Redeemed ${pts} Points (-$${discountVal.toFixed(2)})`); },
        openDiscountModal: () => Utils.openModal('discountModal'),
        applyDiscount: (type, val) => { if(!type) { State.currentTxn.discount = null; State.currentTxn.pointsRedeemed = 0; } else State.currentTxn.discount = { type: type, value: val }; Utils.closeModal('discountModal'); View.renderCart(); },
        applyCustomDiscount: () => { const p = parseFloat(document.getElementById('disc-percent').value); const f = parseFloat(document.getElementById('disc-fixed').value); if(p > 0) Controller.applyDiscount('percent', p); else if(f > 0) Controller.applyDiscount('fixed', f); },
        openCustomerDisplay: () => { if(State.customerWindow && !State.customerWindow.closed) { State.customerWindow.focus(); return; } State.customerWindow = window.open('', 'CustomerDisplay', 'width=1024,height=768'); View.updateCustomerDisplay(); },
        startShift: () => { const start = parseFloat(document.getElementById('shift-start-amt').value); if(isNaN(start)) return Utils.toast("Invalid Start Amount", "error"); State.cashDrawer = { status: 'open', currentShift: { id: Date.now(), startTime: new Date().toISOString(), startCash: start, sales: 0 } }; Utils.save(); View.renderShift(); },
        endShift: (expected) => { if(State.psQueue.length > 0) return Utils.toast("⚠️ Cannot close shift. Resolve Quarantine items first.", "error"); const end = parseFloat(document.getElementById('shift-end-amt').value); if(isNaN(end)) return Utils.toast("Invalid End Amount", "error"); const s = State.cashDrawer.currentShift; const variance = end - expected; const record = { startTime: s.startTime, startCash: s.startCash, actualEnd: end, variance: variance }; State.cashDrawer = { status: 'closed', history: [record, ...(State.cashDrawer.history||[])] }; alert(`Shift Closed.\nExpected: ${Utils.fmt(expected)}\nActual: ${Utils.fmt(end)}\nVariance: ${Utils.fmt(variance)}`); Utils.save(); View.renderShift(); },
        saveSettings: () => { State.settings = { storeName: document.getElementById('conf-storename').value, taxRate: parseFloat(document.getElementById('conf-tax').value), currency: document.getElementById('conf-curr').value }; Utils.save(); alert("Settings Saved. Refresh to apply fully."); },
        saveAdSettings: () => { State.adSettings = { mode: document.getElementById('ad-mode').value, title: document.getElementById('ad-title').value, msg: document.getElementById('ad-msg').value, prodId: document.getElementById('ad-prod-select').value, duration: parseInt(document.getElementById('ad-duration').value) }; Utils.save(); Utils.toast("Display Updated"); View.updateCustomerDisplay(); Controller.startSlideshow(); },
        rotateSlides: () => { if(State.adSettings.mode === 'auto') { State.slideIndex++; View.updateCustomerDisplay(); } },
        startSlideshow: () => { if(State.slideshowInterval) clearInterval(State.slideshowInterval); State.slideshowInterval = setInterval(Controller.rotateSlides, (State.adSettings.duration || 5) * 1000); },
        backupData: () => { const data = JSON.stringify(localStorage); const blob = new Blob([data], {type: 'application/json'}); const url = URL.createObjectURL(blob); const a = document.createElement('a'); a.href=url; a.download='pos_backup.json'; a.click(); },
        restoreData: (input) => { const file = input.files[0]; const reader = new FileReader(); reader.onload = (e) => { const data = JSON.parse(e.target.result); Object.keys(data).forEach(k => localStorage.setItem(k, data[k])); alert("Data Restored. Reloading..."); location.reload(); }; reader.readAsText(file); },
        startNewTransaction: () => { Controller.newTxn(); State.currentTxn.customer = null; State.currentTxn.discount = null; State.currentTxn.pointsRedeemed = 0; View.renderCart(); View.updateCustomerDisplay(); }
    };

    window.onload = Controller.init;
</script>
</body>
</html>
