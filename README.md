# Gestion-Financière
<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Gestionnaire de Dépenses</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Poppins:wght@600;700;800&family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.0/chart.umd.min.js"></script>
<style>
  :root{
    --bg: #F4F6F5;
    --surface: #FFFFFF;
    --surface-alt: #FBFCFC;
    --ink: #1B2B26;
    --ink-soft: #5C6B66;
    --line: #E4E9E7;
    --primary: #1E6F5C;
    --primary-dark: #144C40;
    --primary-light: #E6F2EE;
    --warn: #F2994A;
    --warn-light: #FDF1E4;
    --danger: #D64545;
    --danger-light: #FBEAEA;
    --shadow: 0 6px 16px rgba(27,43,38,0.08);

    --cat-alimentation: #F2994A;
    --cat-transport: #2F80ED;
    --cat-loisirs: #9B51E0;
    --cat-sante: #EB5757;
    --cat-logement: #16A085;
    --cat-autres: #828B8A;
  }

  html[data-theme="dark"]{
    --bg: #0F1E1A;
    --surface: #172A24;
    --surface-alt: #1D3830;
    --ink: #EAF3F0;
    --ink-soft: #92A9A2;
    --line: #26433A;
    --primary: #3FBE9C;
    --primary-dark: #2C8D73;
    --primary-light: rgba(63,190,156,0.15);
    --warn: #F2994A;
    --warn-light: rgba(242,153,74,0.15);
    --danger: #FF7A7A;
    --danger-light: rgba(255,122,122,0.15);
    --shadow: 0 6px 18px rgba(0,0,0,0.35);
  }

  *{ box-sizing: border-box; }
  html,body{ height:100%; }
  body{
    margin:0;
    background: var(--bg);
    color: var(--ink);
    font-family: 'Inter', system-ui, sans-serif;
    -webkit-font-smoothing: antialiased;
    transition: background .2s ease, color .2s ease;
  }

  h1,h2,h3{ font-family:'Poppins', system-ui, sans-serif; margin:0; }

  .app{
    max-width: 1240px;
    margin: 0 auto;
    padding: 24px 20px 70px;
  }

  /* ===== TOASTS ===== */
  .toast-container{
    position: fixed; top: 18px; right: 18px; z-index: 999;
    display:flex; flex-direction:column; gap:10px; max-width: 320px;
  }
  .toast{
    background: var(--surface);
    border: 1px solid var(--line);
    border-left: 4px solid var(--primary);
    border-radius: 10px;
    padding: 12px 14px;
    box-shadow: var(--shadow);
    font-size: 13.5px;
    color: var(--ink);
    animation: toastIn .25s ease;
  }
  .toast.warning{ border-left-color: var(--warn); }
  .toast.danger{ border-left-color: var(--danger); }
  .toast.fadeout{ animation: toastOut .3s ease forwards; }
  @keyframes toastIn{ from{ opacity:0; transform: translateX(30px);} to{opacity:1; transform:translateX(0);} }
  @keyframes toastOut{ from{ opacity:1;} to{ opacity:0; transform: translateX(30px);} }

  /* ===== UNDO SNACKBAR ===== */
  .undo-bar{
    position: fixed; bottom: 20px; left: 50%; transform: translateX(-50%) translateY(120%);
    background: var(--ink); color: var(--bg);
    padding: 12px 16px; border-radius: 12px;
    display:flex; align-items:center; gap:14px;
    font-size: 13.5px; box-shadow: 0 10px 30px rgba(0,0,0,0.3);
    z-index: 998; transition: transform .25s ease;
  }
  .undo-bar.show{ transform: translateX(-50%) translateY(0); }
  .undo-bar button{
    background: var(--primary); color:#fff; border:none; border-radius:8px;
    padding: 6px 12px; font-weight:600; cursor:pointer; font-family:'Inter',sans-serif;
  }

  /* ===== TOPBAR ===== */
  .topbar{
    display:flex; align-items:center; justify-content:space-between;
    gap:16px; margin-bottom: 18px; flex-wrap: wrap;
  }
  .brand{ display:flex; align-items:center; gap:12px; }
  .brand .coin{
    width:44px; height:44px; border-radius: 12px;
    background: linear-gradient(135deg, var(--primary), var(--primary-dark));
    display:flex; align-items:center; justify-content:center; font-size:22px;
    box-shadow: 0 6px 14px rgba(20,76,64,0.25);
  }
  .brand h1{ font-size: 22px; color: var(--primary-dark); letter-spacing: -0.02em; }
  html[data-theme="dark"] .brand h1{ color: var(--primary); }
  .brand p{ margin:2px 0 0; font-size: 13px; color: var(--ink-soft); }

  .topbar-actions{ display:flex; align-items:center; gap:10px; flex-wrap:wrap; }

  .icon-toggle{
    width:40px; height:40px; border-radius:10px; border:1.5px solid var(--line);
    background: var(--surface); cursor:pointer; font-size:17px;
    display:flex; align-items:center; justify-content:center; transition: all .15s ease;
  }
  .icon-toggle:hover{ border-color: var(--primary); }

  select.compact{
    padding: 9px 10px; border-radius: 10px; border:1.5px solid var(--line);
    background: var(--surface); color: var(--ink); font-family:'Inter',sans-serif; font-size:13.5px;
  }

  .balance-card{
    background: linear-gradient(135deg, var(--primary), var(--primary-dark));
    color:#fff; border-radius: 16px; padding: 14px 22px; min-width: 200px;
    box-shadow: 0 10px 24px rgba(20,76,64,0.22);
  }
  .balance-card .label{ font-size:11.5px; opacity:0.85; text-transform:uppercase; letter-spacing:.08em; }
  .balance-card .amount{ font-family:'Poppins',sans-serif; font-size:26px; font-weight:700; margin-top:4px; }

  /* ===== PANEL BASE ===== */
  .panel{
    background: var(--surface); border: 1px solid var(--line);
    border-radius: 16px; padding: 20px; margin-bottom: 20px;
  }
  .panel h2{ font-size:16px; color: var(--primary-dark); margin-bottom:14px; }
  html[data-theme="dark"] .panel h2{ color: var(--primary); }
  .panel h2 small{ font-weight:500; color: var(--ink-soft); font-size:12px; }

  /* ===== BUDGET PANEL ===== */
  .budget-header{ display:flex; align-items:center; justify-content:space-between; gap:14px; flex-wrap:wrap; margin-bottom: 6px;}
  .budget-input-row{ display:flex; gap:8px; }
  .budget-input-row input{
    padding: 9px 12px; border-radius:10px; border:1.5px solid var(--line);
    background: var(--surface-alt); color: var(--ink); font-family:'Inter',sans-serif; font-size:13.5px; width:170px;
  }
  .btn-sm{ padding: 9px 14px; font-size:13.5px; }
  .budget-progress-wrap{ margin-top: 14px; }
  .budget-stats-row{ display:flex; justify-content:space-between; font-size:13px; color: var(--ink-soft); margin-bottom:8px; flex-wrap:wrap; gap:6px;}
  .budget-stats-row strong{ color: var(--ink); }
  .progress-track{ height: 12px; background: var(--surface-alt); border-radius: 999px; overflow:hidden; border:1px solid var(--line); }
  .progress-fill{ height:100%; border-radius:999px; background: var(--primary); transition: width .4s ease, background .3s ease; }

  /* ===== SUMMARY GRID ===== */
  .summary-grid{ display:grid; grid-template-columns: repeat(6, 1fr); gap: 12px; margin-bottom: 20px; }
  .summary-card{ background: var(--surface); border: 1px solid var(--line); border-radius: 14px; padding: 13px 14px; }
  .summary-card .k{ font-size:11px; color:var(--ink-soft); font-weight:600; text-transform:uppercase; letter-spacing:.03em; }
  .summary-card .v{ font-family:'Poppins',sans-serif; font-size:17px; font-weight:700; margin-top:6px; color: var(--ink); }
  .summary-card.top .v{ color: var(--primary-dark); }
  html[data-theme="dark"] .summary-card.top .v{ color: var(--primary); }

  /* ===== LAYOUT ===== */
  .layout{ display:grid; grid-template-columns: 330px 1fr; gap: 20px; align-items: start; }

  /* ===== FORM ===== */
  .field{ margin-bottom: 14px; }
  .field label{ display:block; font-size:12px; font-weight:600; color: var(--ink-soft); margin-bottom: 6px; text-transform:uppercase; letter-spacing:.03em; }
  .field input, .field select{
    width:100%; padding: 10px 12px; border: 1.5px solid var(--line); border-radius: 10px;
    font-size: 14px; font-family:'Inter',sans-serif; color: var(--ink); background: var(--surface-alt);
    transition: border-color .15s ease, box-shadow .15s ease;
  }
  .field input:focus, .field select:focus{ outline:none; border-color: var(--primary); box-shadow: 0 0 0 3px var(--primary-light); }
  .row2{ display:grid; grid-template-columns: 1fr 1fr; gap: 12px; }

  .btn{ appearance:none; border:none; cursor:pointer; border-radius: 10px; padding: 11px 16px; font-family:'Inter',sans-serif; font-weight:600; font-size: 14px; transition: transform .1s ease, background .15s ease; }
  .btn:active{ transform: scale(0.98); }
  .btn-primary{ background: var(--primary); color:#fff; width:100%; }
  .btn-primary:hover{ background: var(--primary-dark); }
  .btn-ghost{ background: transparent; color: var(--ink-soft); border:1.5px solid var(--line); width:100%; margin-top:8px; }
  .btn-ghost:hover{ border-color: var(--danger); color: var(--danger); }
  .btn-outline{ background: var(--surface-alt); color: var(--ink); border:1.5px solid var(--line); width:100%; margin-bottom:8px; display:flex; align-items:center; justify-content:center; gap:8px; }
  .btn-outline:hover{ border-color: var(--primary); color: var(--primary-dark); }
  .btn-danger-outline{ background: var(--danger-light); color: var(--danger); border:1.5px solid var(--danger-light); width:100%; }
  .btn-danger-outline:hover{ border-color: var(--danger); }

  .cat-legend{ display:flex; flex-wrap:wrap; gap:8px; margin-top: 10px; }
  .cat-chip{ display:flex; align-items:center; gap:6px; font-size: 11.5px; color: var(--ink-soft); background: var(--surface-alt); border:1px solid var(--line); padding: 4px 9px; border-radius: 999px; }
  .dot{ width:9px; height:9px; border-radius:50%; flex-shrink:0; }

  /* ===== SETTINGS / ICON PICKER ===== */
  .settings-toggle{ cursor:pointer; display:flex; align-items:center; justify-content:space-between; }
  .settings-toggle .chev{ transition: transform .2s ease; color: var(--ink-soft); }
  .settings-toggle.open .chev{ transform: rotate(180deg); }
  .icon-picker-row{ display:flex; align-items:center; gap:10px; padding: 8px 0; border-bottom:1px solid var(--line); }
  .icon-picker-row:last-child{ border-bottom:none; }
  .icon-picker-row .cat-name{ font-size:13px; font-weight:600; width:100px; flex-shrink:0; }
  .icon-picker-options{ display:flex; gap:6px; flex-wrap:wrap; }
  .icon-opt{ width:30px; height:30px; border-radius:8px; border:1.5px solid var(--line); background: var(--surface-alt); cursor:pointer; font-size:15px; display:flex; align-items:center; justify-content:center; }
  .icon-opt.selected{ border-color: var(--primary); background: var(--primary-light); }

  /* ===== RIGHT COLUMN ===== */
  .charts-grid{ display:grid; grid-template-columns: 1fr 1fr; gap:20px; }
  .chart-wrap{ display:flex; gap: 18px; align-items:center; flex-wrap:wrap; }
  .chart-canvas-box{ width: 170px; height:170px; flex-shrink:0; position:relative; }
  .chart-legend{ flex:1; min-width:150px; display:flex; flex-direction:column; gap:9px; }
  .legend-row{ display:flex; align-items:center; justify-content:space-between; font-size:12.5px; }
  .legend-left{ display:flex; align-items:center; gap:7px; color: var(--ink); font-weight:500; }
  .legend-amount{ font-weight:700; color: var(--ink); }
  .legend-pct{ color: var(--ink-soft); font-size:11.5px; margin-left:5px; }
  .bar-chart-box{ height: 190px; position:relative; }

  /* ===== CALENDAR ===== */
  .calendar-header{ display:flex; align-items:center; justify-content:space-between; margin-bottom:12px; }
  .calendar-header .cal-nav{ display:flex; align-items:center; gap:10px; }
  .calendar-header button{ background: var(--surface-alt); border:1.5px solid var(--line); border-radius:8px; width:30px; height:30px; cursor:pointer; color: var(--ink); }
  .calendar-header .cal-title{ font-weight:700; font-family:'Poppins',sans-serif; font-size:14px; min-width:130px; text-align:center; }
  .calendar-grid{ display:grid; grid-template-columns: repeat(7, 1fr); gap:6px; }
  .cal-dow{ text-align:center; font-size:10.5px; color: var(--ink-soft); font-weight:700; text-transform:uppercase; padding-bottom:4px; }
  .cal-day{
    aspect-ratio: 1; border-radius: 10px; border:1.5px solid var(--line); background: var(--surface-alt);
    display:flex; flex-direction:column; align-items:center; justify-content:center; cursor:pointer;
    font-size:12px; position:relative; transition: transform .1s ease;
  }
  .cal-day:hover{ transform: scale(1.05); border-color: var(--primary); }
  .cal-day.empty{ visibility:hidden; cursor:default; }
  .cal-day.has-spend{ color:#fff; }
  .cal-day .cal-num{ font-weight:700; }
  .cal-day .cal-amt{ font-size:9px; margin-top:2px; }
  .cal-day.today{ box-shadow: 0 0 0 2px var(--primary) inset; }
  .cal-day.selected{ box-shadow: 0 0 0 2px var(--danger) inset; }

  /* ===== ADVANCED STATS ===== */
  .adv-stats-grid{ display:grid; grid-template-columns: repeat(4, 1fr); gap: 12px; }
  .adv-stat{ background: var(--surface-alt); border:1px solid var(--line); border-radius: 12px; padding: 12px; }
  .adv-stat .k{ font-size:11px; color:var(--ink-soft); font-weight:600; text-transform:uppercase; }
  .adv-stat .v{ font-family:'Poppins',sans-serif; font-size:15px; font-weight:700; margin-top:5px; }
  .adv-stat .sub{ font-size:11px; color:var(--ink-soft); margin-top:2px; }

  /* ===== TOOLBAR ===== */
  .toolbar{ display:flex; gap: 10px; margin-bottom: 12px; flex-wrap: wrap; align-items:center; }
  .toolbar input[type="search"], .toolbar input[type="date"]{
    padding: 9px 13px; border-radius: 10px; border: 1.5px solid var(--line); font-size: 13.5px;
    font-family:'Inter',sans-serif; background: var(--surface-alt); color: var(--ink);
  }
  .toolbar input[type="search"]{ flex: 1 1 200px; }
  .toolbar select{ padding: 9px 13px; border-radius: 10px; border: 1.5px solid var(--line); font-size: 13.5px; background: var(--surface-alt); color: var(--ink); font-family:'Inter',sans-serif; }
  .toolbar input:focus, .toolbar select:focus{ outline:none; border-color: var(--primary); box-shadow: 0 0 0 3px var(--primary-light); }
  .sort-toggle{ display:flex; align-items:center; gap:4px; }
  .sort-dir-btn{ width:36px; height:36px; border-radius:10px; border:1.5px solid var(--line); background: var(--surface-alt); cursor:pointer; color: var(--ink); }
  .period-row{ display:flex; gap:8px; flex-wrap:wrap; margin-bottom:10px; }
  .period-chip{ padding: 7px 13px; border-radius: 999px; border:1.5px solid var(--line); background: var(--surface-alt); font-size:12.5px; cursor:pointer; color: var(--ink-soft); font-weight:600; }
  .period-chip.active{ background: var(--primary); border-color: var(--primary); color:#fff; }
  .custom-range{ display:none; gap:8px; align-items:center; margin-bottom:10px; }
  .custom-range.show{ display:flex; }

  /* ===== EXPENSE LIST ===== */
  .list{ display:flex; flex-direction:column; gap:10px; }
  .expense-item{
    display:flex; align-items:center; gap:14px; background: var(--surface); border: 1px solid var(--line);
    border-radius: 12px; padding: 12px 14px; transition: box-shadow .15s ease, transform .15s ease, opacity .25s ease, max-height .25s ease;
    animation: itemIn .3s ease;
  }
  .expense-item.removing{ opacity:0; transform: scale(0.92) translateX(20px); }
  @keyframes itemIn{ from{ opacity:0; transform: translateY(-8px) scale(0.98);} to{ opacity:1; transform:translateY(0) scale(1);} }
  .expense-item:hover{ box-shadow: var(--shadow); transform: translateY(-1px); }
  .cat-icon{ width: 38px; height:38px; border-radius: 10px; display:flex; align-items:center; justify-content:center; font-size: 18px; flex-shrink:0; color:#fff; }
  .expense-details{ flex:1; min-width:0; }
  .expense-desc{ font-weight:600; font-size: 14px; color: var(--ink); white-space:nowrap; overflow:hidden; text-overflow:ellipsis; }
  .expense-meta{ font-size: 12px; color: var(--ink-soft); margin-top:2px; }
  .expense-amount{ font-family:'Poppins',sans-serif; font-weight:700; font-size:15px; color: var(--ink); white-space:nowrap; }
  .expense-actions{ display:flex; gap:6px; }
  .icon-btn{ width: 30px; height:30px; border-radius:8px; border:1.5px solid var(--line); background: var(--surface-alt); cursor:pointer; display:flex; align-items:center; justify-content:center; font-size:14px; transition: all .15s ease; }
  .icon-btn:hover{ border-color: var(--primary); background: var(--primary-light); }
  .icon-btn.del:hover{ border-color: var(--danger); background: var(--danger-light); }
  .empty-state{ text-align:center; padding: 46px 20px; color: var(--ink-soft); }
  .empty-state .emoji{ font-size: 34px; margin-bottom: 10px; }

  /* ===== PRINT REPORT (hidden on screen) ===== */
  .print-report{ display:none; }
  @media print{
    body *{ visibility: hidden; }
    .print-report, .print-report *{ visibility: visible; }
    .print-report{ display:block; position:absolute; top:0; left:0; width:100%; padding: 20px; }
    .print-report h1{ font-size: 22px; color:#144C40; margin-bottom:4px; }
    .print-report .meta{ font-size:12px; color:#555; margin-bottom:18px; }
    .print-report table{ width:100%; border-collapse: collapse; font-size:12px; }
    .print-report th, .print-report td{ border-bottom: 1px solid #ddd; padding: 6px 8px; text-align:left; }
    .print-report th{ background:#F4F6F5; }
    .print-report .totals{ margin-top:16px; font-size:13px; }
    .print-report .totals strong{ font-size:16px; }
  }

  @media (max-width: 980px){
    .layout{ grid-template-columns: 1fr; }
    .summary-grid{ grid-template-columns: repeat(3,1fr); }
    .charts-grid{ grid-template-columns: 1fr; }
    .adv-stats-grid{ grid-template-columns: repeat(2,1fr); }
  }
  @media (max-width: 560px){
    .summary-grid{ grid-template-columns: repeat(2,1fr); }
    .balance-card{ min-width: 0; width:100%; }
    .row2{ grid-template-columns: 1fr; }
    .adv-stats-grid{ grid-template-columns: 1fr 1fr; }
    .toolbar{ flex-direction:column; align-items:stretch; }
    .toolbar select, .toolbar input{ width:100%; }
    .expense-desc{ white-space:normal; }
    .chart-wrap{ flex-direction:column; }
  }
</style>
</head>
<body>

<div class="toast-container" id="toastContainer"></div>
<div class="undo-bar" id="undoBar">
  <span id="undoMsg">Dépense supprimée</span>
  <button id="undoBtn">Annuler</button>
</div>

<div class="app">

  <div class="topbar">
    <div class="brand">
      <div class="coin">💶</div>
      <div>
        <h1>Gestionnaire de Dépenses</h1>
        <p>Suivez, filtrez et visualisez vos dépenses</p>
      </div>
    </div>
    <div class="topbar-actions">
      <select id="currencySelect" class="compact" title="Devise">
        <option value="EUR">€ Euro</option>
        <option value="USD">$ Dollar</option>
        <option value="GBP">£ Livre</option>
      </select>
      <button class="icon-toggle" id="themeToggle" title="Changer de thème">🌙</button>
      <div class="balance-card">
        <div class="label">Total des dépenses</div>
        <div class="amount" id="totalBalance">0,00 €</div>
      </div>
    </div>
  </div>

  <!-- BUDGET -->
  <div class="panel">
    <div class="budget-header">
      <h2>🎯 Budget mensuel</h2>
      <div class="budget-input-row">
        <input type="number" id="budgetInput" placeholder="Ex : 800" min="0" step="1">
        <button class="btn btn-primary btn-sm" id="setBudgetBtn" type="button">Définir</button>
      </div>
    </div>
    <div class="budget-progress-wrap" id="budgetProgressWrap" style="display:none;">
      <div class="budget-stats-row">
        <span>Dépensé ce mois : <strong id="budgetSpent"></strong></span>
        <span>Restant : <strong id="budgetRemaining"></strong></span>
      </div>
      <div class="progress-track"><div class="progress-fill" id="progressFill"></div></div>
    </div>
  </div>

  <!-- SUMMARY -->
  <div class="summary-grid">
    <div class="summary-card"><div class="k">Nombre</div><div class="v" id="statCount">0</div></div>
    <div class="summary-card"><div class="k">Aujourd'hui</div><div class="v" id="statDay">0,00 €</div></div>
    <div class="summary-card"><div class="k">Cette semaine</div><div class="v" id="statWeek">0,00 €</div></div>
    <div class="summary-card"><div class="k">Ce mois-ci</div><div class="v" id="statMonth">0,00 €</div></div>
    <div class="summary-card"><div class="k">Moyenne</div><div class="v" id="statAvg">0,00 €</div></div>
    <div class="summary-card top"><div class="k">Top catégorie</div><div class="v" id="statTop">—</div></div>
  </div>

  <div class="layout">

    <!-- LEFT COLUMN -->
    <div class="left-col">
      <div class="panel">
        <h2 id="formTitle">➕ Ajouter une dépense</h2>
        <form id="expenseForm">
          <input type="hidden" id="editId">
          <div class="field">
            <label for="desc">Description</label>
            <input type="text" id="desc" placeholder="Ex : Courses de la semaine" required>
          </div>
          <div class="row2">
            <div class="field">
              <label for="amount">Montant</label>
              <input type="number" id="amount" step="0.01" min="0" placeholder="0,00" required>
            </div>
            <div class="field">
              <label for="date">Date</label>
              <input type="date" id="date" required>
            </div>
          </div>
          <div class="field">
            <label for="category">Catégorie</label>
            <select id="category" required>
              <option value="Alimentation">Alimentation</option>
              <option value="Transport">Transport</option>
              <option value="Loisirs">Loisirs</option>
              <option value="Santé">Santé</option>
              <option value="Logement">Logement</option>
              <option value="Autres">Autres</option>
            </select>
          </div>
          <button type="submit" class="btn btn-primary" id="submitBtn">Ajouter la dépense</button>
          <button type="button" class="btn btn-ghost" id="cancelEdit" style="display:none;">Annuler la modification</button>
        </form>
        <div class="cat-legend" id="catLegend"></div>
      </div>

      <div class="panel">
        <h2>⚡ Actions rapides</h2>
        <button class="btn btn-outline" id="exportCsvBtn" type="button">⬇️ Export CSV</button>
        <button class="btn btn-outline" id="pdfReportBtn" type="button">🧾 Rapport PDF</button>
        <button class="btn btn-danger-outline" id="deleteAllBtn" type="button">🗑️ Tout supprimer</button>
      </div>

      <div class="panel">
        <div class="settings-toggle" id="settingsToggle">
          <h2>🎨 Personnaliser les icônes</h2>
          <span class="chev">▾</span>
        </div>
        <div id="iconPickerBox" style="display:none; margin-top:8px;"></div>
      </div>
    </div>

    <!-- RIGHT COLUMN -->
    <div class="right-col">

      <div class="charts-grid">
        <div class="panel">
          <h2>📊 Par catégorie</h2>
          <div class="chart-wrap">
            <div class="chart-canvas-box"><canvas id="categoryChart"></canvas></div>
            <div class="chart-legend" id="chartLegend"></div>
          </div>
        </div>
        <div class="panel">
          <h2>📈 7 derniers jours</h2>
          <div class="bar-chart-box"><canvas id="trendChart"></canvas></div>
        </div>
      </div>

      <div class="panel">
        <h2>🗓️ Calendrier interactif</h2>
        <div class="calendar-header">
          <button type="button" id="calPrev">‹</button>
          <div class="cal-title" id="calTitle"></div>
          <button type="button" id="calNext">›</button>
        </div>
        <div class="calendar-grid" id="calendarGrid"></div>
      </div>

      <div class="panel">
        <h2>🔎 Statistiques avancées</h2>
        <div class="adv-stats-grid" id="advStats"></div>
      </div>

      <div class="panel">
        <h2>📋 Mes dépenses</h2>

        <div class="period-row" id="periodRow">
          <span class="period-chip active" data-period="all">Tout</span>
          <span class="period-chip" data-period="today">Aujourd'hui</span>
          <span class="period-chip" data-period="week">Cette semaine</span>
          <span class="period-chip" data-period="month">Ce mois</span>
          <span class="period-chip" data-period="custom">Période personnalisée</span>
        </div>
        <div class="custom-range" id="customRange">
          <input type="date" id="rangeStart">
          <span style="color:var(--ink-soft); font-size:12px;">à</span>
          <input type="date" id="rangeEnd">
        </div>

        <div class="toolbar">
          <input type="search" id="searchInput" placeholder="🔍 Rechercher une description...">
          <input type="date" id="dateSearch" title="Rechercher une date précise">
          <select id="filterCategory">
            <option value="all">Toutes les catégories</option>
            <option value="Alimentation">Alimentation</option>
            <option value="Transport">Transport</option>
            <option value="Loisirs">Loisirs</option>
            <option value="Santé">Santé</option>
            <option value="Logement">Logement</option>
            <option value="Autres">Autres</option>
          </select>
          <select id="sortBy">
            <option value="date">Trier par date</option>
            <option value="amount">Trier par montant</option>
            <option value="category">Trier par catégorie</option>
          </select>
          <div class="sort-toggle">
            <button type="button" class="sort-dir-btn" id="sortDirBtn" title="Ordre">↓</button>
          </div>
        </div>

        <div class="list" id="expenseList"></div>
      </div>

    </div>
  </div>
</div>

<div class="print-report" id="printReport"></div>

<script>
  // ===================== CONFIG =====================
  const DEFAULT_ICONS = {
    'Alimentation': { color: '#F2994A', icon: '🍎', alt: ['🍎','🍔','🍕','🥗','🍜'] },
    'Transport':    { color: '#2F80ED', icon: '🚗', alt: ['🚗','🚌','🚆','🚲','✈️'] },
    'Loisirs':      { color: '#9B51E0', icon: '🎬', alt: ['🎬','🎮','🎨','🎵','⚽'] },
    'Santé':        { color: '#EB5757', icon: '💊', alt: ['💊','🏥','🩺','💉','❤️'] },
    'Logement':     { color: '#16A085', icon: '🏠', alt: ['🏠','🛋️','🔑','🧾','💡'] },
    'Autres':       { color: '#828B8A', icon: '📦', alt: ['📦','🔖','💼','🎁','❓'] }
  };
  const CURRENCY_SYMBOLS = { EUR:'€', USD:'$', GBP:'£' };

  const EXCHANGE_RATES = {
  EUR: {
    EUR: 1,
    USD: 1.08,
    GBP: 0.86
  },
  USD: {
    EUR: 0.93,
    USD: 1,
    GBP: 0.80
  },
  GBP: {
    EUR: 1.16,
    USD: 1.25,
    GBP: 1
  }
};

  const DOW = ['Lun','Mar','Mer','Jeu','Ven','Sam','Dim'];

  const KEYS = {
    expenses: 'gestionnaire_depenses_v1',
    icons: 'gestionnaire_icons_v1',
    currency: 'gestionnaire_currency_v1',
    theme: 'gestionnaire_theme_v1',
    budget: 'gestionnaire_budget_v1',
    budgetCurrency: 'gestionnaire_budget_currency_v1'
  };

  // ===================== STATE =====================
  let expenses = loadJSON(KEYS.expenses, []);
  let iconOverrides = loadJSON(KEYS.icons, {});
  let currentCurrency = localStorage.getItem(KEYS.currency) || 'EUR';
  let currentTheme = localStorage.getItem(KEYS.theme) || 'light';
  let monthlyBudget = parseFloat(localStorage.getItem(KEYS.budget)) || 0;
  let monthlyBudgetCurrency = localStorage.getItem(KEYS.budgetCurrency) || 'EUR';

  let doughnutChart = null, trendChart = null;
  let lastAction = null; // { type:'single'|'all', data }
  let undoTimer = null;
  let calendarMonth = new Date(); calendarMonth.setDate(1);
  let selectedDate = null;
  let currentPeriod = 'all';

  function CATEGORIES(){
    const merged = {};
    for(const cat in DEFAULT_ICONS){
      merged[cat] = { color: DEFAULT_ICONS[cat].color, icon: (iconOverrides[cat] || DEFAULT_ICONS[cat].icon), alt: DEFAULT_ICONS[cat].alt };
    }
    return merged;
  }

  function loadJSON(key, fallback){
    try{ const raw = localStorage.getItem(key); return raw ? JSON.parse(raw) : fallback; }
    catch(e){ return fallback; }
  }

  // ===================== ELEMENTS =====================
  const form = document.getElementById('expenseForm');
  const descInput = document.getElementById('desc');
  const amountInput = document.getElementById('amount');
  const dateInput = document.getElementById('date');
  const categoryInput = document.getElementById('category');
  const editIdInput = document.getElementById('editId');
  const submitBtn = document.getElementById('submitBtn');
  const cancelEditBtn = document.getElementById('cancelEdit');
  const formTitle = document.getElementById('formTitle');
  const searchInput = document.getElementById('searchInput');
  const dateSearch = document.getElementById('dateSearch');
  const filterCategory = document.getElementById('filterCategory');
  const sortBySel = document.getElementById('sortBy');
  const sortDirBtn = document.getElementById('sortDirBtn');
  let sortDir = 'desc';

  dateInput.value = new Date().toISOString().split('T')[0];
  document.getElementById('currencySelect').value = currentCurrency;
  if(currentTheme === 'dark'){ document.documentElement.setAttribute('data-theme','dark'); document.getElementById('themeToggle').textContent = '☀️'; }
  if(monthlyBudget){ document.getElementById('budgetInput').value = monthlyBudget; }

  // ===================== HELPERS =====================
  function todayISO(){ return new Date().toISOString().split('T')[0]; }

  // Convertit un montant d'une devise source vers une devise cible (indépendant de la devise affichée)
  function toCurrency(amount, fromCurrency, toCurr){
    return amount * EXCHANGE_RATES[fromCurrency || 'EUR'][toCurr];
  }
  // Convertit un montant d'une devise source vers la devise actuellement affichée
  function toCurrent(amount, fromCurrency){
    return toCurrency(amount, fromCurrency, currentCurrency);
  }

  // Formate un montant déjà exprimé dans la devise actuellement affichée (aucune conversion)
  function formatConverted(amount){
    const symbol = CURRENCY_SYMBOLS[currentCurrency];
    if(currentCurrency === 'EUR'){
      return amount.toLocaleString('fr-FR',{
        minimumFractionDigits:2,
        maximumFractionDigits:2
      }) + ' ' + symbol;
    }
    return symbol + amount.toLocaleString('en-US',{
      minimumFractionDigits:2,
      maximumFractionDigits:2
    });
  }

  // Formate un montant brut stocké dans sa devise d'origine, en le convertissant vers la devise affichée
  function formatMoney(amount, expenseCurrency = 'EUR'){
    return formatConverted(toCurrent(amount, expenseCurrency));
  }

  function formatDate(iso){
    const d = new Date(iso + 'T00:00:00');
    return d.toLocaleDateString('fr-FR', { day:'2-digit', month:'short', year:'numeric' });
  }
  function uid(){ return 'e_' + Date.now() + '_' + Math.random().toString(36).slice(2,7); }
  function escapeHtml(str){ const div = document.createElement('div'); div.textContent = str; return div.innerHTML; }

  function startOfWeek(d){
    const date = new Date(d); const day = (date.getDay() + 6) % 7; // Monday = 0
    date.setDate(date.getDate() - day); date.setHours(0,0,0,0); return date;
  }

  // ===================== TOASTS =====================
  function showToast(msg, type){
    const el = document.createElement('div');
    el.className = 'toast' + (type ? ' ' + type : '');
    el.textContent = msg;
    document.getElementById('toastContainer').appendChild(el);
    setTimeout(() => {
      el.classList.add('fadeout');
      setTimeout(() => el.remove(), 300);
    }, 3200);
  }

  // ===================== UNDO SNACKBAR =====================
  function showUndo(msg, action){
    lastAction = action;
    document.getElementById('undoMsg').textContent = msg;
    const bar = document.getElementById('undoBar');
    bar.classList.add('show');
    clearTimeout(undoTimer);
    undoTimer = setTimeout(() => { bar.classList.remove('show'); lastAction = null; }, 6500);
  }
  document.getElementById('undoBtn').addEventListener('click', function(){
    if(!lastAction) return;
    if(lastAction.type === 'single'){
      expenses.splice(lastAction.index, 0, lastAction.data);
    } else if(lastAction.type === 'all'){
      expenses = lastAction.data;
    }
    saveExpenses();
    document.getElementById('undoBar').classList.remove('show');
    lastAction = null;
    render();
    showToast('Action annulée', 'info');
  });

  // ===================== PERSISTENCE =====================
  function saveExpenses(){ localStorage.setItem(KEYS.expenses, JSON.stringify(expenses)); }
  function saveIcons(){ localStorage.setItem(KEYS.icons, JSON.stringify(iconOverrides)); }

  // Filet de sécurité silencieux : chaque ajout/modification/suppression sauvegarde déjà
  // immédiatement, donc pas besoin (et c'était trompeur) d'afficher un toast ici.
  setInterval(saveExpenses, 30000);

  // ===================== THEME / CURRENCY =====================
  document.getElementById('themeToggle').addEventListener('click', function(){
    currentTheme = currentTheme === 'dark' ? 'light' : 'dark';
    localStorage.setItem(KEYS.theme, currentTheme);
    if(currentTheme === 'dark'){ document.documentElement.setAttribute('data-theme','dark'); this.textContent = '☀️'; }
    else { document.documentElement.removeAttribute('data-theme'); this.textContent = '🌙'; }
    render();
  });

  document.getElementById('currencySelect').addEventListener('change', function(){
    currentCurrency = this.value;
    localStorage.setItem(KEYS.currency, currentCurrency);
    render();
  });

  // ===================== BUDGET =====================
  document.getElementById('setBudgetBtn').addEventListener('click', function(){
    const val = parseFloat(document.getElementById('budgetInput').value);
    if(isNaN(val) || val <= 0){ showToast('Entrez un budget valide', 'warning'); return; }
    monthlyBudget = val;
    monthlyBudgetCurrency = currentCurrency;
    localStorage.setItem(KEYS.budget, monthlyBudget);
    localStorage.setItem(KEYS.budgetCurrency, monthlyBudgetCurrency);
    showToast('🎯 Budget mensuel défini (' + CURRENCY_SYMBOLS[currentCurrency] + ')', 'success');
    render();
  });

  // monthSpent doit déjà être exprimé dans la devise actuellement affichée (currentCurrency)
  function renderBudget(monthSpent){
    const wrap = document.getElementById('budgetProgressWrap');
    if(!monthlyBudget){ wrap.style.display = 'none'; return; }
    wrap.style.display = 'block';
    const budgetInCurrent = toCurrency(monthlyBudget, monthlyBudgetCurrency, currentCurrency);
    const remaining = budgetInCurrent - monthSpent;
    document.getElementById('budgetSpent').textContent = formatConverted(monthSpent);
    document.getElementById('budgetRemaining').textContent = formatConverted(remaining);
    const pct = Math.min(100, (monthSpent / budgetInCurrent) * 100);
    const fill = document.getElementById('progressFill');
    fill.style.width = pct + '%';
    if(monthSpent > budgetInCurrent) fill.style.background = 'var(--danger)';
    else if(pct >= 70) fill.style.background = 'var(--warn)';
    else fill.style.background = 'var(--primary)';
  }

  // ===================== FORM SUBMIT =====================
  form.addEventListener('submit', function(e){
    e.preventDefault();
    const data = {
  description: descInput.value.trim(),
  amount: parseFloat(amountInput.value),
  date: dateInput.value,
  category: categoryInput.value,
  currency: currentCurrency
  };
    if(!data.description || isNaN(data.amount) || data.amount < 0) return;

    if(editIdInput.value){
      const idx = expenses.findIndex(x => x.id === editIdInput.value);
      if(idx > -1) expenses[idx] = { ...expenses[idx], ...data };
      exitEditMode();
      showToast('✏️ Dépense modifiée', 'success');
    } else {
      const newExp = { id: uid(), ...data };
      expenses.push(newExp);
      showToast('✅ Dépense ajoutée', 'success');
    }

    saveExpenses();
    checkBudgetWarning();
    render();
    form.reset();
    dateInput.value = todayISO();
  });

  function currentMonthExpenses(){
    const now = new Date();
    return expenses.filter(x => {
      const d = new Date(x.date + 'T00:00:00');
      return d.getMonth() === now.getMonth() && d.getFullYear() === now.getFullYear();
    });
  }

  function checkBudgetWarning(){
    if(!monthlyBudget) return;
    // On compare dans la devise du budget, indépendamment de la devise actuellement affichée
    const monthSpent = currentMonthExpenses().reduce((s,x) => s + toCurrency(x.amount, x.currency, monthlyBudgetCurrency), 0);
    if(monthSpent > monthlyBudget) showToast('⚠️ Budget mensuel dépassé !', 'danger');
    else if(monthSpent > monthlyBudget * 0.9) showToast('⚠️ Vous approchez de votre budget', 'warning');
  }

  function exitEditMode(){
    editIdInput.value = '';
    submitBtn.textContent = 'Ajouter la dépense';
    formTitle.textContent = '➕ Ajouter une dépense';
    cancelEditBtn.style.display = 'none';
  }
  cancelEditBtn.addEventListener('click', function(){
    form.reset(); dateInput.value = todayISO(); exitEditMode();
  });

  function startEdit(id){
    const exp = expenses.find(x => x.id === id);
    if(!exp) return;
    descInput.value = exp.description; amountInput.value = exp.amount;
    dateInput.value = exp.date; categoryInput.value = exp.category;
    editIdInput.value = exp.id;
    submitBtn.textContent = 'Enregistrer les modifications';
    formTitle.textContent = '✏️ Modifier la dépense';
    cancelEditBtn.style.display = 'block';
    window.scrollTo({ top: 0, behavior: 'smooth' });
  }

  function deleteExpense(id){
    if(!confirm('Supprimer cette dépense ?')) return;
    const idx = expenses.findIndex(x => x.id === id);
    if(idx === -1) return;
    const el = document.querySelector('[data-id="' + id + '"]');
    const doRemove = () => {
      const [removed] = expenses.splice(idx, 1);
      saveExpenses();
      if(editIdInput.value === id){ form.reset(); exitEditMode(); }
      showUndo('Dépense supprimée', { type:'single', index: idx, data: removed });
      render();
    };
    if(el){ el.classList.add('removing'); setTimeout(doRemove, 220); }
    else doRemove();
  }

  document.getElementById('deleteAllBtn').addEventListener('click', function(){
    if(expenses.length === 0){ showToast('Aucune dépense à supprimer', 'warning'); return; }
    if(!confirm('Voulez-vous vraiment supprimer TOUTES les dépenses ? Vous pourrez annuler juste après.')) return;
    const snapshot = expenses.slice();
    expenses = [];
    saveExpenses();
    showUndo('Toutes les dépenses ont été supprimées', { type:'all', data: snapshot });
    render();
  });

  window.startEdit = startEdit;
  window.deleteExpense = deleteExpense;

  // ===================== FILTER / SORT CONTROLS =====================
  searchInput.addEventListener('input', render);
  dateSearch.addEventListener('input', render);
  filterCategory.addEventListener('change', render);
  sortBySel.addEventListener('change', render);
  sortDirBtn.addEventListener('click', function(){
    sortDir = sortDir === 'desc' ? 'asc' : 'desc';
    this.textContent = sortDir === 'desc' ? '↓' : '↑';
    render();
  });

  document.querySelectorAll('.period-chip').forEach(chip => {
    chip.addEventListener('click', function(){
      document.querySelectorAll('.period-chip').forEach(c => c.classList.remove('active'));
      this.classList.add('active');
      currentPeriod = this.dataset.period;
      document.getElementById('customRange').classList.toggle('show', currentPeriod === 'custom');
      render();
    });
  });
  document.getElementById('rangeStart').addEventListener('change', render);
  document.getElementById('rangeEnd').addEventListener('change', render);

  function inPeriod(dateStr){
    if(currentPeriod === 'all') return true;
    const d = new Date(dateStr + 'T00:00:00');
    const now = new Date();
    if(currentPeriod === 'today') return dateStr === todayISO();
    if(currentPeriod === 'week'){
      const start = startOfWeek(now);
      const end = new Date(start); end.setDate(end.getDate() + 6);
      return d >= start && d <= end;
    }
    if(currentPeriod === 'month') return d.getMonth() === now.getMonth() && d.getFullYear() === now.getFullYear();
    if(currentPeriod === 'custom'){
      const s = document.getElementById('rangeStart').value;
      const e = document.getElementById('rangeEnd').value;
      if(s && dateStr < s) return false;
      if(e && dateStr > e) return false;
      return true;
    }
    return true;
  }

  function getFilteredExpenses(){
    const term = searchInput.value.trim().toLowerCase();
    const cat = filterCategory.value;
    const exactDate = dateSearch.value;
    let list = expenses
      .filter(x => cat === 'all' || x.category === cat)
      .filter(x => !term || x.description.toLowerCase().includes(term))
      .filter(x => !exactDate || x.date === exactDate)
      .filter(x => exactDate || inPeriod(x.date));

    const dir = sortDir === 'asc' ? 1 : -1;
    const by = sortBySel.value;
    list.sort((a,b) => {
      if(by === 'amount') return (a.amount - b.amount) * dir;
      if(by === 'category') return a.category.localeCompare(b.category) * dir;
      return (new Date(a.date) - new Date(b.date)) * dir;
    });
    return list;
  }

  // ===================== RENDER: LIST =====================
  function renderList(){
    const listEl = document.getElementById('expenseList');
    const CATS = CATEGORIES();
    const filtered = getFilteredExpenses();

    if(filtered.length === 0){
      listEl.innerHTML = '<div class="empty-state"><div class="emoji">🗂️</div><div>Aucune dépense trouvée</div></div>';
      return;
    }

    listEl.innerHTML = filtered.map(exp => {
      const cat = CATS[exp.category] || CATS['Autres'];
      return `
        <div class="expense-item" data-id="${exp.id}">
          <div class="cat-icon" style="background:${cat.color}">${cat.icon}</div>
          <div class="expense-details">
            <div class="expense-desc">${escapeHtml(exp.description)}</div>
            <div class="expense-meta">${exp.category} · ${formatDate(exp.date)}</div>
          </div>
          <div class="expense-amount">${formatMoney(exp.amount, exp.currency || 'EUR')}</div>
          <div class="expense-actions">
            <button class="icon-btn" title="Modifier" onclick="startEdit('${exp.id}')">✏️</button>
            <button class="icon-btn del" title="Supprimer" onclick="deleteExpense('${exp.id}')">🗑️</button>
          </div>
        </div>`;
    }).join('');
  }

  // ===================== RENDER: SUMMARY / STATS =====================
  function groupByCategory(list){
    const CATS = CATEGORIES();
    const result = {};
    for(const cat in CATS) result[cat] = 0;
    list.forEach(x => { result[x.category] = (result[x.category] || 0) + toCurrent(x.amount, x.currency); });
    return result;
  }

  function getConvertedTotal(list){
    return list.reduce((s,x) => s + toCurrent(x.amount, x.currency), 0);
  }

  function renderStats(){
    const total = getConvertedTotal(expenses);
    document.getElementById('totalBalance').textContent = formatConverted(total);
    document.getElementById('statCount').textContent = expenses.length;

    const now = new Date();

    const dayTotal = getConvertedTotal(expenses.filter(x => x.date === todayISO()));
    document.getElementById('statDay').textContent = formatConverted(dayTotal);

    const wStart = startOfWeek(now); const wEnd = new Date(wStart); wEnd.setDate(wEnd.getDate()+6);
    const weekTotal = getConvertedTotal(expenses.filter(x => {
      const d = new Date(x.date+'T00:00:00');
      return d >= wStart && d <= wEnd;
    }));
    document.getElementById('statWeek').textContent = formatConverted(weekTotal);

    const monthTotal = getConvertedTotal(currentMonthExpenses());
    document.getElementById('statMonth').textContent = formatConverted(monthTotal);

    const avg = expenses.length ? total / expenses.length : 0;
    document.getElementById('statAvg').textContent = formatConverted(avg);

    const byCat = groupByCategory(expenses);
    let topCat = '—', topVal = 0;
    for(const [cat, val] of Object.entries(byCat)){ if(val > topVal){ topVal = val; topCat = cat; } }
    const CATS = CATEGORIES();
    document.getElementById('statTop').textContent = topCat === '—' ? '—' : `${CATS[topCat].icon} ${topCat}`;

    renderBudget(monthTotal);
    renderAdvancedStats();
  }

  function renderAdvancedStats(){
    const box = document.getElementById('advStats');
    if(expenses.length === 0){
      box.innerHTML = '<div class="empty-state" style="grid-column:1/-1; padding:20px;">Pas encore assez de données</div>';
      return;
    }
    // On compare/additionne des montants convertis dans la devise affichée, pas des montants bruts
    // (des dépenses en devises différentes ne peuvent pas être comparées ou sommées telles quelles)
    const withConverted = expenses.map(x => ({ ...x, _converted: toCurrent(x.amount, x.currency) }));
    const sorted = withConverted.slice().sort((a,b) => b._converted - a._converted);
    const biggest = sorted[0];
    const smallest = sorted[sorted.length - 1];

    const byDate = {};
    withConverted.forEach(x => { byDate[x.date] = (byDate[x.date] || 0) + x._converted; });
    let busiestDate = null, busiestVal = 0;
    for(const [d,v] of Object.entries(byDate)){ if(v > busiestVal){ busiestVal = v; busiestDate = d; } }

    const distinctDays = Object.keys(byDate).length;
    const total = withConverted.reduce((s,x)=>s+x._converted,0);
    const avgPerDay = distinctDays ? total / distinctDays : 0;

    box.innerHTML = `
      <div class="adv-stat">
        <div class="k">Plus grosse dépense</div>
        <div class="v">${formatConverted(biggest._converted)}</div>
        <div class="sub">${escapeHtml(biggest.description)}</div>
      </div>
      <div class="adv-stat">
        <div class="k">Plus petite dépense</div>
        <div class="v">${formatConverted(smallest._converted)}</div>
        <div class="sub">${escapeHtml(smallest.description)}</div>
      </div>
      <div class="adv-stat">
        <div class="k">Jour le plus dépensier</div>
        <div class="v">${formatConverted(busiestVal)}</div>
        <div class="sub">${busiestDate ? formatDate(busiestDate) : '—'}</div>
      </div>
      <div class="adv-stat">
        <div class="k">Moyenne / jour actif</div>
        <div class="v">${formatConverted(avgPerDay)}</div>
        <div class="sub">${distinctDays} jour(s) avec dépenses</div>
      </div>`;
  }

  // ===================== RENDER: CHARTS =====================
  function renderCategoryChart(){
    const CATS = CATEGORIES();
    const byCat = groupByCategory(expenses);
    const labels = [], values = [], colors = [];
    Object.entries(byCat).forEach(([cat, val]) => { if(val > 0){ labels.push(cat); values.push(val); colors.push(CATS[cat].color); } });

    const ctx = document.getElementById('categoryChart').getContext('2d');
    const total = values.reduce((a,b)=>a+b,0);
    if(doughnutChart) doughnutChart.destroy();

    if(values.length === 0){
      document.getElementById('chartLegend').innerHTML = '<div class="empty-state" style="padding:6px;">Ajoutez une dépense pour voir le graphique</div>';
      doughnutChart = new Chart(ctx, { type:'doughnut', data:{ labels:['Aucune donnée'], datasets:[{ data:[1], backgroundColor:['#E4E9E7'], borderWidth:0 }] }, options:{ plugins:{ legend:{display:false}, tooltip:{enabled:false} }, cutout:'68%' } });
      return;
    }

    doughnutChart = new Chart(ctx, {
      type: 'doughnut',
      data: { labels, datasets: [{ data: values, backgroundColor: colors, borderWidth: 3, borderColor: getComputedStyle(document.documentElement).getPropertyValue('--surface') || '#fff' }] },
      options: { cutout: '68%', plugins: { legend: { display:false }, tooltip: { callbacks: { label: (c) => ` ${c.label}: ${formatConverted(c.raw)}` } } } }
    });

    document.getElementById('chartLegend').innerHTML = labels.map((cat,i) => {
      const pct = ((values[i]/total)*100).toFixed(0);
      return `<div class="legend-row"><div class="legend-left"><span class="dot" style="background:${colors[i]}"></span>${CATS[cat].icon} ${cat}</div><div><span class="legend-amount">${formatConverted(values[i])}</span><span class="legend-pct">(${pct}%)</span></div></div>`;
    }).join('');
  }

  function renderTrendChart(){
    const days = [];
    for(let i = 6; i >= 0; i--){
      const d = new Date(); d.setDate(d.getDate() - i);
      days.push(d.toISOString().split('T')[0]);
    }
    const values = days.map(d => expenses.filter(x => x.date === d).reduce((s,x)=>s+toCurrent(x.amount, x.currency),0));
    const labels = days.map(d => new Date(d+'T00:00:00').toLocaleDateString('fr-FR', { weekday:'short', day:'numeric' }));

    const ctx = document.getElementById('trendChart').getContext('2d');
    if(trendChart) trendChart.destroy();
    const inkColor = getComputedStyle(document.documentElement).getPropertyValue('--ink-soft') || '#5C6B66';
    trendChart = new Chart(ctx, {
      type: 'bar',
      data: { labels, datasets: [{ data: values, backgroundColor: '#1E6F5C', borderRadius: 6, maxBarThickness: 34 }] },
      options: {
        plugins: { legend: { display:false }, tooltip: { callbacks: { label:(c)=>' '+formatConverted(c.raw) } } },
        scales: {
          x: { grid: { display:false }, ticks: { color: inkColor, font:{ size:11 } } },
          y: { grid: { color: 'rgba(128,128,128,0.15)' }, ticks: { color: inkColor, font:{ size:10 }, callback:(v)=>formatConverted(v) } }
        }
      }
    });
  }

  // ===================== RENDER: CALENDAR =====================
  function renderCalendar(){
    const y = calendarMonth.getFullYear(), m = calendarMonth.getMonth();
    document.getElementById('calTitle').textContent = calendarMonth.toLocaleDateString('fr-FR', { month:'long', year:'numeric' });

    const byDate = {};
    expenses.forEach(x => { byDate[x.date] = (byDate[x.date] || 0) + toCurrent(x.amount, x.currency); });
    const monthPrefix = y + '-' + String(m+1).padStart(2,'0');
    const monthValues = Object.entries(byDate).filter(([d]) => d.startsWith(monthPrefix)).map(([,v]) => v);
    const maxVal = Math.max(1, ...monthValues);

    const firstDay = new Date(y, m, 1);
    const startOffset = (firstDay.getDay() + 6) % 7; // Monday=0
    const daysInMonth = new Date(y, m+1, 0).getDate();

    let html = DOW.map(d => `<div class="cal-dow">${d}</div>`).join('');
    for(let i = 0; i < startOffset; i++) html += '<div class="cal-day empty"></div>';

    for(let day = 1; day <= daysInMonth; day++){
      const dateStr = y + '-' + String(m+1).padStart(2,'0') + '-' + String(day).padStart(2,'0');
      const val = byDate[dateStr] || 0;
      const isToday = dateStr === todayISO();
      const isSelected = dateStr === selectedDate;
      let bg = '';
      if(val > 0){
        const intensity = Math.min(1, val / maxVal);
        bg = `style="background:rgba(30,111,92,${0.25 + intensity*0.6}); color:#fff;"`;
      }
      html += `<div class="cal-day ${val>0?'has-spend':''} ${isToday?'today':''} ${isSelected?'selected':''}" ${bg} data-date="${dateStr}">
        <div class="cal-num">${day}</div>
        ${val>0 ? `<div class="cal-amt">${formatConverted(val)}</div>` : ''}
      </div>`;
    }
    document.getElementById('calendarGrid').innerHTML = html;

    document.querySelectorAll('.cal-day[data-date]').forEach(cell => {
      cell.addEventListener('click', function(){
        const d = this.dataset.date;
        selectedDate = (selectedDate === d) ? null : d;
        dateSearch.value = selectedDate || '';
        render();
      });
    });
  }
  document.getElementById('calPrev').addEventListener('click', function(){ calendarMonth.setMonth(calendarMonth.getMonth()-1); renderCalendar(); });
  document.getElementById('calNext').addEventListener('click', function(){ calendarMonth.setMonth(calendarMonth.getMonth()+1); renderCalendar(); });

  // ===================== CATEGORY LEGEND + ICON PICKER =====================
  function renderCatLegend(){
    const CATS = CATEGORIES();
    document.getElementById('catLegend').innerHTML = Object.entries(CATS).map(([cat,info]) =>
      `<span class="cat-chip"><span class="dot" style="background:${info.color}"></span>${info.icon} ${cat}</span>`
    ).join('');
  }

  function renderIconPicker(){
    const CATS = CATEGORIES();
    const box = document.getElementById('iconPickerBox');
    box.innerHTML = Object.entries(CATS).map(([cat, info]) => `
      <div class="icon-picker-row">
        <span class="cat-name">${cat}</span>
        <div class="icon-picker-options">
          ${info.alt.map(ic => `<button type="button" class="icon-opt ${ic===info.icon?'selected':''}" data-cat="${cat}" data-icon="${ic}">${ic}</button>`).join('')}
        </div>
      </div>`).join('');

    box.querySelectorAll('.icon-opt').forEach(btn => {
      btn.addEventListener('click', function(){
        iconOverrides[this.dataset.cat] = this.dataset.icon;
        saveIcons();
        render();
        showToast('🎨 Icône mise à jour', 'success');
      });
    });
  }

  document.getElementById('settingsToggle').addEventListener('click', function(){
    const box = document.getElementById('iconPickerBox');
    const isOpen = box.style.display !== 'none';
    box.style.display = isOpen ? 'none' : 'block';
    this.classList.toggle('open', !isOpen);
  });

  // ===================== EXPORT CSV =====================
  document.getElementById('exportCsvBtn').addEventListener('click', function(){
    if(expenses.length === 0){ showToast('Aucune dépense à exporter', 'warning'); return; }
    let csv = 'Description,Montant,Devise,Catégorie,Date\n';
    expenses.forEach(x => {
      const desc = '"' + x.description.replace(/"/g,'""') + '"';
      csv += `${desc},${x.amount},${x.currency || 'EUR'},${x.category},${x.date}\n`;
    });
    const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url; a.download = 'depenses_' + todayISO() + '.csv';
    a.click();
    URL.revokeObjectURL(url);
    showToast('⬇️ Export CSV téléchargé', 'success');
  });

  // ===================== PDF REPORT (print) =====================
  document.getElementById('pdfReportBtn').addEventListener('click', function(){
    if(expenses.length === 0){ showToast('Aucune dépense à inclure dans le rapport', 'warning'); return; }
    const sorted = expenses.slice().sort((a,b) => new Date(b.date) - new Date(a.date));
    // Chaque ligne affiche le montant dans sa devise d'origine ; le total est converti
    // proprement dans la devise affichée (additionner des montants de devises différentes
    // sans conversion n'a pas de sens).
    const total = sorted.reduce((s,x) => s + toCurrent(x.amount, x.currency), 0);
    const rows = sorted.map(x => `<tr><td>${formatDate(x.date)}</td><td>${escapeHtml(x.description)}</td><td>${x.category}</td><td>${x.currency || 'EUR'}</td><td>${formatMoney(x.amount, x.currency || 'EUR')}</td></tr>`).join('');
    document.getElementById('printReport').innerHTML = `
      <h1>Rapport de dépenses</h1>
      <div class="meta">Généré le ${new Date().toLocaleDateString('fr-FR')} · ${expenses.length} dépense(s)</div>
      <table>
        <thead><tr><th>Date</th><th>Description</th><th>Catégorie</th><th>Devise</th><th>Montant</th></tr></thead>
        <tbody>${rows}</tbody>
      </table>
      <div class="totals"><strong>Total (converti en ${currentCurrency}) : ${formatConverted(total)}</strong></div>`;
    window.print();
  });

  // ===================== MASTER RENDER =====================
  function render(){
    renderCatLegend();
    renderIconPicker();
    renderList();
    renderStats();
    renderCategoryChart();
    renderTrendChart();
    renderCalendar();
  }

  render();
</script>

</body>
</html>