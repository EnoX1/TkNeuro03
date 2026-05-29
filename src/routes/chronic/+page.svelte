<script>
  import { onMount } from 'svelte';
  import { supabase } from '$lib/supabase';

  // ── State ──────────────────────────────────────────
  let patients = [];
  let currentPt = null;
  let visits = [];
  let searchTerm = '';
  let currentTab = 'summary';
  let dbStatus = 'loading'; // 'ok' | 'err' | 'loading'
  let aiMessages = [];
  let aiInput = '';
  let aiThinking = false;

  // Modal state
  let showModalPt = false;
  let showModalVisit = false;

  // Form: patient
  let ptName = '', ptId = '', ptBirth = '', ptGender = 'M';
  let ckHtn = false, ckDm = false, ckLp = false;
  let ptHistory = '';

  // Form: visit
  let vDate = '', vBW = '', vSBP = '', vDBP = '';
  let vFBG = '', vHbA1c = '', vLDL = '', vEGFR = '';
  let vMeds = '', vNote = '';

  // Toast
  let toastMsg = '';
  let toastVisible = false;

  const today = new Date().toLocaleDateString('zh-TW', {
    year: 'numeric', month: 'long', day: 'numeric', weekday: 'short'
  });

  // ── DB Helpers ─────────────────────────────────────
  async function loadPatients() {
    try {
      const { data, error } = await supabase
        .from('patients').select('*').order('name');
      if (error) throw error;
      patients = data || [];
      dbStatus = 'ok';
      if (patients.length === 0) await seedDemo();
    } catch (e) {
      dbStatus = 'err';
      patients = [];
    }
  }

  async function seedDemo() {
    const demos = [
      { id: 'P001', name: '王大明', birth: '1955-03-15', gender: 'M', diseases: { htn: true, dm: true, lp: false }, history: '無已知藥物過敏' },
      { id: 'P002', name: '李淑芬', birth: '1960-07-22', gender: 'F', diseases: { htn: false, dm: true, lp: true }, history: 'Sulfa 過敏' },
      { id: 'P003', name: '張建國', birth: '1950-11-08', gender: 'M', diseases: { htn: true, dm: false, lp: true }, history: '' },
    ];
    for (const d of demos) {
      await supabase.from('patients').insert(d);
    }
    const { data } = await supabase.from('patients').select('*').order('name');
    patients = data || [];
  }

  async function selectPatient(id) {
    currentPt = patients.find(p => p.id === id);
    currentTab = 'summary';
    aiMessages = [];
    await loadVisits();
  }

  async function loadVisits() {
    if (!currentPt) return;
    const { data } = await supabase
      .from('visits')
      .select('*')
      .eq('patient_id', currentPt.id)
      .order('date', { ascending: false });
    visits = data || [];
  }

  // ── Patient Modal ──────────────────────────────────
  function openAddPt() {
    ptName = ''; ptId = ''; ptBirth = ''; ptGender = 'M';
    ckHtn = false; ckDm = false; ckLp = false; ptHistory = '';
    showModalPt = true;
  }

  function openEditPt() {
    if (!currentPt) return;
    ptName = currentPt.name; ptId = currentPt.id;
    ptBirth = currentPt.birth || ''; ptGender = currentPt.gender || 'M';
    ckHtn = currentPt.diseases?.htn || false;
    ckDm = currentPt.diseases?.dm || false;
    ckLp = currentPt.diseases?.lp || false;
    ptHistory = currentPt.history || '';
    showModalPt = true;
  }

  async function submitPatient() {
    if (!ptName.trim()) { toast('請輸入姓名'); return; }
    const id = ptId.trim() || 'P' + Date.now().toString().slice(-6);
    const pt = {
      id, name: ptName.trim(),
      birth: ptBirth || null,
      gender: ptGender,
      diseases: { htn: ckHtn, dm: ckDm, lp: ckLp },
      history: ptHistory.trim() || null
    };
    const { error } = await supabase.from('patients').upsert(pt);
    if (error) { toast('❌ 儲存失敗：' + error.message); return; }
    toast('✅ 病人已儲存');
    showModalPt = false;
    await loadPatients();
    await selectPatient(id);
  }

  // ── Visit Modal ────────────────────────────────────
  function openAddVisit() {
    vDate = new Date().toISOString().slice(0, 10);
    vBW = ''; vSBP = ''; vDBP = ''; vFBG = '';
    vHbA1c = ''; vLDL = ''; vEGFR = ''; vMeds = ''; vNote = '';
    showModalVisit = true;
  }

  async function submitVisit() {
    if (!currentPt || !vDate) { toast('請填入日期'); return; }
    const visit = {
      patient_id: currentPt.id, date: vDate,
      bw: parseFloat(vBW) || null,
      sbp: parseInt(vSBP) || null, dbp: parseInt(vDBP) || null,
      fbg: parseInt(vFBG) || null, hba1c: parseFloat(vHbA1c) || null,
      ldl: parseInt(vLDL) || null, egfr: parseInt(vEGFR) || null,
      meds: vMeds.trim() || null, note: vNote.trim() || null
    };
    const { error } = await supabase.from('visits').insert(visit);
    if (error) { toast('❌ 儲存失敗：' + error.message); return; }
    toast('✅ 回診記錄已儲存');
    showModalVisit = false;
    await loadVisits();
  }

  async function deleteVisit(id) {
    if (!confirm('確定刪除此回診記錄？')) return;
    await supabase.from('visits').delete().eq('id', id);
    toast('已刪除');
    await loadVisits();
  }

  // ── Computed helpers ───────────────────────────────
  $: filteredPatients = patients.filter(p =>
    p.name.toLowerCase().includes(searchTerm.toLowerCase())
  );

  function latestVal(key) {
    for (const v of visits) if (v[key] != null) return v[key];
    return null;
  }

  function valClass(key, val) {
    if (val == null) return '';
    if (key === 'egfr') return val >= 90 ? 'ok' : val >= 60 ? 'warn' : 'bad';
    const t = {
      sbp: [[130,'warn'],[140,'bad']], dbp: [[85,'warn'],[90,'bad']],
      fbg: [[100,'warn'],[126,'bad']], hba1c: [[7,'warn'],[8,'bad']],
      ldl: [[100,'warn'],[130,'bad']]
    }[key];
    if (!t) return '';
    for (const [th, cls] of t) if (val >= th) return cls;
    return '';
  }

  function calcAge(birth) {
    if (!birth) return '?';
    return Math.floor((Date.now() - new Date(birth)) / 3.156e10);
  }

  // ── AI ─────────────────────────────────────────────
  async function sendAI() {
    const q = aiInput.trim();
    if (!q || !currentPt) return;
    aiInput = '';
    const latest = visits[0] || {};
    const ctx = `病人：${currentPt.name}，${currentPt.gender === 'F' ? '女' : '男'}性，${calcAge(currentPt.birth)}歲。
疾病：${[currentPt.diseases?.htn && '高血壓', currentPt.diseases?.dm && '糖尿病', currentPt.diseases?.lp && '高血脂'].filter(Boolean).join('、') || '無'}。
${currentPt.history ? '過敏/病史：' + currentPt.history + '。' : ''}
最近回診（${latest.date || '無'}）：血壓${latest.sbp && latest.dbp ? latest.sbp + '/' + latest.dbp : '不詳'}，空腹血糖${latest.fbg || '不詳'}，HbA1c ${latest.hba1c || '不詳'}%，LDL ${latest.ldl || '不詳'}，eGFR ${latest.egfr || '不詳'}，用藥：${latest.meds || '不詳'}。`;

    aiMessages = [...aiMessages, { role: 'user', content: q }];
    aiThinking = true;

    try {
      const res = await fetch('https://api.anthropic.com/v1/messages', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          model: 'claude-sonnet-4-20250514',
          max_tokens: 800,
          system: `你是永佳診所的臨床決策輔助系統。以下是當前病人資料：\n\n${ctx}\n\n請用繁體中文，簡潔專業地回答醫師的臨床問題。數值評估要明確指出是否在目標範圍內，並給出具體建議。不要超過300字。`,
          messages: aiMessages
        })
      });
      const data = await res.json();
      const reply = data.content?.[0]?.text || '（無法取得回應）';
      aiMessages = [...aiMessages, { role: 'assistant', content: reply }];
    } catch (e) {
      aiMessages = [...aiMessages, { role: 'assistant', content: '⚠ 連線失敗' }];
    }
    aiThinking = false;
  }

  function askQuick(q) { aiInput = q; sendAI(); }

  // ── Toast ──────────────────────────────────────────
  function toast(msg) {
    toastMsg = msg;
    toastVisible = true;
    setTimeout(() => toastVisible = false, 2500);
  }

  // ── Init ───────────────────────────────────────────
  onMount(loadPatients);
</script>

<!-- ═══════════════════════════════════════════════════
     LAYOUT
════════════════════════════════════════════════════ -->
<div class="chronic-app">

  <!-- Sidebar -->
  <aside class="chronic-sidebar">
    <div class="sb-header">
      <div class="sb-title">三高管理</div>
      <div class="sb-sub">慢性病追蹤系統</div>
    </div>

    <div class="sb-search">
      <input bind:value={searchTerm} placeholder="搜尋病人..." />
    </div>

    <div class="sb-list">
      {#each filteredPatients as p}
        <button
          class="pt-item {currentPt?.id === p.id ? 'active' : ''}"
          on:click={() => selectPatient(p.id)}
        >
          <span class="pt-dot {p.diseases?.dm ? 'dm' : p.diseases?.htn ? 'htn' : ''}"></span>
          {p.name}
        </button>
      {/each}
      {#if filteredPatients.length === 0}
        <div class="sb-empty">無病人資料</div>
      {/if}
    </div>

    <div class="sb-footer">
      <button class="btn-add-pt" on:click={openAddPt}>＋ 新增病人</button>
    </div>
  </aside>

  <!-- Main -->
  <main class="chronic-main">

    <!-- Topbar -->
    <div class="topbar">
      <div class="topbar-title">📋 永佳診所・三高慢性病管理</div>
      <div class="topbar-date">{today}</div>
      <span class="db-badge {dbStatus}">
        {dbStatus === 'ok' ? '● Supabase 連線' : dbStatus === 'err' ? '⚠ 離線' : '連線中...'}
      </span>
    </div>

    <!-- Content -->
    <div class="content">
      {#if !currentPt}
        <div class="empty-state">
          <div class="empty-icon">🏥</div>
          <div class="empty-title">請從左側選擇病人</div>
          <div class="empty-sub">或點選「＋ 新增病人」建立新病歷</div>
        </div>
      {:else}
        <!-- Patient Header -->
        <div class="pt-header">
          <div class="pt-avatar">{currentPt.name[0]}</div>
          <div class="pt-info">
            <div class="pt-name">{currentPt.name}</div>
            <div class="pt-meta">
              {currentPt.gender === 'F' ? '女' : '男'} · {calcAge(currentPt.birth)}歲 · 病歷號 {currentPt.id}
            </div>
            {#if currentPt.history}
              <div class="pt-allergy">⚠ {currentPt.history}</div>
            {/if}
            <div class="pt-tags">
              {#if currentPt.diseases?.htn}<span class="tag htn">高血壓</span>{/if}
              {#if currentPt.diseases?.dm}<span class="tag dm">糖尿病</span>{/if}
              {#if currentPt.diseases?.lp}<span class="tag lp">高血脂</span>{/if}
            </div>
          </div>
          <div class="pt-actions">
            <button class="btn primary sm" on:click={openAddVisit}>＋ 回診記錄</button>
            <button class="btn outline sm" on:click={openEditPt}>編輯</button>
          </div>
        </div>

        <!-- Tabs -->
        <div class="tabs">
          {#each [['summary','摘要'],['visits','回診記錄'],['ai','AI 問診']] as [t, label]}
            <button class="tab {currentTab === t ? 'active' : ''}" on:click={() => currentTab = t}>
              {label}
            </button>
          {/each}
        </div>

        <!-- Tab: Summary -->
        {#if currentTab === 'summary'}
          {@const sbp = latestVal('sbp')} {@const dbp = latestVal('dbp')}
          {@const fbg = latestVal('fbg')} {@const hba1c = latestVal('hba1c')}
          {@const ldl = latestVal('ldl')} {@const egfr = latestVal('egfr')}
          {@const bw = latestVal('bw')}

          <div class="stats-grid">
            <div class="stat {valClass('sbp', sbp)}">
              <div class="stat-label">血壓</div>
              <div class="stat-val">{sbp && dbp ? `${sbp}/${dbp}` : '—'}</div>
              <div class="stat-unit">mmHg</div>
            </div>
            <div class="stat {valClass('fbg', fbg)}">
              <div class="stat-label">空腹血糖</div>
              <div class="stat-val">{fbg || '—'}</div>
              <div class="stat-unit">mg/dL</div>
            </div>
            <div class="stat {valClass('hba1c', hba1c)}">
              <div class="stat-label">HbA1c</div>
              <div class="stat-val">{hba1c || '—'}</div>
              <div class="stat-unit">%</div>
            </div>
            <div class="stat {valClass('ldl', ldl)}">
              <div class="stat-label">LDL</div>
              <div class="stat-val">{ldl || '—'}</div>
              <div class="stat-unit">mg/dL</div>
            </div>
            <div class="stat {valClass('egfr', egfr)}">
              <div class="stat-label">eGFR</div>
              <div class="stat-val">{egfr || '—'}</div>
              <div class="stat-unit">mL/min</div>
            </div>
            <div class="stat">
              <div class="stat-label">體重</div>
              <div class="stat-val">{bw || '—'}</div>
              <div class="stat-unit">kg</div>
            </div>
          </div>

          <div class="card">
            <div class="card-header">
              <span class="card-title">最近回診</span>
              <span class="card-sub">{visits[0]?.date || '尚無記錄'}</span>
            </div>
            {#if visits.length > 0}
              <table class="vt">
                <thead><tr><th>日期</th><th>血壓</th><th>血糖/HbA1c</th><th>LDL</th><th>eGFR</th><th>備註</th></tr></thead>
                <tbody>
                  {#each visits.slice(0, 5) as v}
                    <tr>
                      <td>{v.date}</td>
                      <td class={valClass('sbp', v.sbp)}>{v.sbp && v.dbp ? `${v.sbp}/${v.dbp}` : '—'}</td>
                      <td>
                        <span class={valClass('fbg', v.fbg)}>{v.fbg || '—'}</span> /
                        <span class={valClass('hba1c', v.hba1c)}>{v.hba1c || '—'}</span>
                      </td>
                      <td class={valClass('ldl', v.ldl)}>{v.ldl || '—'}</td>
                      <td class={valClass('egfr', v.egfr)}>{v.egfr || '—'}</td>
                      <td class="note">{v.note || ''}</td>
                    </tr>
                  {/each}
                </tbody>
              </table>
            {:else}
              <div class="no-data">尚無回診記錄</div>
            {/if}
          </div>

          {#if visits[0]?.meds}
            <div class="card">
              <div class="card-header"><span class="card-title">💊 目前用藥</span></div>
              <div class="card-body">{visits[0].meds}</div>
            </div>
          {/if}

        <!-- Tab: Visits -->
        {:else if currentTab === 'visits'}
          <div class="card">
            <div class="card-header">
              <span class="card-title">所有回診記錄（{visits.length} 筆）</span>
              <button class="btn primary sm" on:click={openAddVisit}>＋ 新增</button>
            </div>
            {#if visits.length > 0}
              <table class="vt">
                <thead><tr><th>日期</th><th>血壓</th><th>血糖</th><th>HbA1c</th><th>LDL</th><th>eGFR</th><th>體重</th><th>用藥</th><th></th></tr></thead>
                <tbody>
                  {#each visits as v}
                    <tr>
                      <td>{v.date}</td>
                      <td class={valClass('sbp', v.sbp)}>{v.sbp && v.dbp ? `${v.sbp}/${v.dbp}` : '—'}</td>
                      <td class={valClass('fbg', v.fbg)}>{v.fbg || '—'}</td>
                      <td class={valClass('hba1c', v.hba1c)}>{v.hba1c || '—'}</td>
                      <td class={valClass('ldl', v.ldl)}>{v.ldl || '—'}</td>
                      <td class={valClass('egfr', v.egfr)}>{v.egfr || '—'}</td>
                      <td>{v.bw || '—'}</td>
                      <td class="note">{v.meds || '—'}</td>
                      <td><button class="btn-del" on:click={() => deleteVisit(v.id)}>🗑</button></td>
                    </tr>
                  {/each}
                </tbody>
              </table>
            {:else}
              <div class="no-data">
                尚無回診記錄
                <button class="btn primary" style="margin-top:12px" on:click={openAddVisit}>新增第一筆</button>
              </div>
            {/if}
          </div>

        <!-- Tab: AI -->
        {:else if currentTab === 'ai'}
          <div class="card ai-panel">
            <div class="ai-title">🤖 AI 臨床問診助手</div>
            <div class="quick-btns">
              {#each ['整體控制評估','用藥調整建議','追蹤項目提醒','腎功能評估','SOAP病歷草稿'] as q}
                <button class="quick-btn" on:click={() => askQuick(q)}>{q}</button>
              {/each}
            </div>
            <div class="ai-messages">
              {#if aiMessages.length === 0}
                <div class="ai-msg assistant">請選擇快速問題或輸入臨床問題，我會根據此病人的最新數值回答。</div>
              {/if}
              {#each aiMessages as m}
                <div class="ai-msg {m.role}">{m.content}</div>
              {/each}
              {#if aiThinking}
                <div class="ai-msg thinking">思考中...</div>
              {/if}
            </div>
            <div class="ai-input-row">
              <input bind:value={aiInput} placeholder="輸入問題..." on:keydown={e => e.key === 'Enter' && sendAI()} />
              <button class="btn primary" on:click={sendAI}>送出</button>
            </div>
          </div>
        {/if}

      {/if}
    </div>
  </main>
</div>

<!-- ═══ Modal: 新增/編輯病人 ═══ -->
{#if showModalPt}
  <div class="modal-overlay" on:click|self={() => showModalPt = false}>
    <div class="modal">
      <h3>🧑‍⚕️ 病人資料</h3>
      <div class="form-row">
        <div class="fg"><label>姓名 *</label><input bind:value={ptName} placeholder="王○○" /></div>
        <div class="fg"><label>病歷號</label><input bind:value={ptId} placeholder="留白自動產生" /></div>
      </div>
      <div class="form-row">
        <div class="fg"><label>出生年月日</label><input type="date" bind:value={ptBirth} /></div>
        <div class="fg"><label>性別</label>
          <select bind:value={ptGender}><option value="M">男</option><option value="F">女</option></select>
        </div>
      </div>
      <div class="fg">
        <label>疾病診斷</label>
        <div class="ck-row">
          <label><input type="checkbox" bind:checked={ckHtn} /> 高血壓</label>
          <label><input type="checkbox" bind:checked={ckDm} /> 糖尿病</label>
          <label><input type="checkbox" bind:checked={ckLp} /> 高血脂</label>
        </div>
      </div>
      <div class="fg"><label>病史備註</label><textarea bind:value={ptHistory} rows="2" placeholder="過敏、重大病史..."></textarea></div>
      <div class="modal-footer">
        <button class="btn outline" on:click={() => showModalPt = false}>取消</button>
        <button class="btn primary" on:click={submitPatient}>儲存</button>
      </div>
    </div>
  </div>
{/if}

<!-- ═══ Modal: 新增回診 ═══ -->
{#if showModalVisit}
  <div class="modal-overlay" on:click|self={() => showModalVisit = false}>
    <div class="modal">
      <h3>📝 新增回診記錄</h3>
      <div class="form-row">
        <div class="fg"><label>看診日期 *</label><input type="date" bind:value={vDate} /></div>
        <div class="fg"><label>體重 (kg)</label><input type="number" bind:value={vBW} step="0.1" placeholder="65.0" /></div>
      </div>
      <div class="form-row">
        <div class="fg"><label>收縮壓</label><input type="number" bind:value={vSBP} placeholder="120" /></div>
        <div class="fg"><label>舒張壓</label><input type="number" bind:value={vDBP} placeholder="80" /></div>
      </div>
      <div class="form-row">
        <div class="fg"><label>空腹血糖</label><input type="number" bind:value={vFBG} placeholder="100" /></div>
        <div class="fg"><label>HbA1c (%)</label><input type="number" bind:value={vHbA1c} step="0.1" placeholder="7.0" /></div>
      </div>
      <div class="form-row">
        <div class="fg"><label>LDL (mg/dL)</label><input type="number" bind:value={vLDL} placeholder="100" /></div>
        <div class="fg"><label>eGFR</label><input type="number" bind:value={vEGFR} placeholder="80" /></div>
      </div>
      <div class="fg"><label>用藥</label><input bind:value={vMeds} placeholder="Amlodipine 5mg..." /></div>
      <div class="fg"><label>醫師備註</label><textarea bind:value={vNote} rows="2" placeholder="今日血壓控制良好..."></textarea></div>
      <div class="modal-footer">
        <button class="btn outline" on:click={() => showModalVisit = false}>取消</button>
        <button class="btn primary" on:click={submitVisit}>儲存</button>
      </div>
    </div>
  </div>
{/if}

<!-- Toast -->
{#if toastVisible}
  <div class="toast">{toastMsg}</div>
{/if}

<!-- ═══════════════════════════════════════════════════
     STYLES
════════════════════════════════════════════════════ -->
<style>
  :global(body) { margin: 0; }

  .chronic-app {
    display: flex; height: calc(100vh - 60px);
    font-family: 'Segoe UI', 'Microsoft JhengHei', sans-serif;
    background: #f8f5ee; color: #1a2a22; overflow: hidden;
  }

  /* ── Sidebar ── */
  .chronic-sidebar {
    width: 220px; min-width: 220px; background: #1e4d3a;
    display: flex; flex-direction: column; flex-shrink: 0;
  }
  .sb-header { padding: 16px 14px 10px; border-bottom: 1px solid rgba(255,255,255,0.1); }
  .sb-title { color: #fff; font-size: 14px; font-weight: 700; letter-spacing: 0.5px; }
  .sb-sub { color: rgba(255,255,255,0.5); font-size: 11px; margin-top: 2px; }
  .sb-search { padding: 8px 10px; }
  .sb-search input {
    width: 100%; background: rgba(255,255,255,0.12); border: 1px solid rgba(255,255,255,0.2);
    border-radius: 6px; color: #fff; padding: 5px 9px; font-size: 12px; outline: none; box-sizing: border-box;
  }
  .sb-search input::placeholder { color: rgba(255,255,255,0.4); }
  .sb-list { flex: 1; overflow-y: auto; padding: 4px 6px; }
  .pt-item {
    display: flex; align-items: center; gap: 7px; width: 100%;
    padding: 8px 9px; border-radius: 6px; border: none; background: transparent;
    color: rgba(255,255,255,0.82); font-size: 13px; cursor: pointer; text-align: left;
    transition: background 0.15s;
  }
  .pt-item:hover { background: rgba(255,255,255,0.1); }
  .pt-item.active { background: rgba(255,255,255,0.2); color: #fff; font-weight: 600; }
  .pt-dot { width: 7px; height: 7px; border-radius: 50%; background: #4a9070; flex-shrink: 0; }
  .pt-dot.dm { background: #e67e22; }
  .pt-dot.htn { background: #e74c3c; }
  .sb-empty { color: rgba(255,255,255,0.35); font-size: 12px; padding: 8px 10px; }
  .sb-footer { padding: 10px; border-top: 1px solid rgba(255,255,255,0.1); }
  .btn-add-pt {
    width: 100%; background: #4a9070; border: none; border-radius: 6px;
    color: #fff; padding: 8px; font-size: 13px; font-weight: 600; cursor: pointer;
  }
  .btn-add-pt:hover { background: #5aaa82; }

  /* ── Main ── */
  .chronic-main { flex: 1; display: flex; flex-direction: column; overflow: hidden; }
  .topbar {
    background: #fff; border-bottom: 1px solid #c8ddd5;
    padding: 10px 18px; display: flex; align-items: center; gap: 10px; flex-shrink: 0;
  }
  .topbar-title { font-size: 14px; font-weight: 700; color: #1e4d3a; flex: 1; }
  .topbar-date { font-size: 11px; color: #5a7a68; }
  .db-badge { font-size: 11px; padding: 2px 8px; border-radius: 20px; }
  .db-badge.ok { background: #d4edda; color: #155724; }
  .db-badge.err { background: #f8d7da; color: #721c24; }
  .db-badge.loading { background: #fff3cd; color: #856404; }

  .content { flex: 1; overflow-y: auto; padding: 16px; }

  /* ── Empty ── */
  .empty-state { display: flex; flex-direction: column; align-items: center; justify-content: center; height: 60%; gap: 10px; color: #5a7a68; }
  .empty-icon { font-size: 44px; opacity: 0.4; }
  .empty-title { font-size: 15px; font-weight: 600; }
  .empty-sub { font-size: 13px; }

  /* ── Patient Header ── */
  .pt-header { background: #fff; border-radius: 10px; padding: 16px 18px; margin-bottom: 14px; box-shadow: 0 2px 10px rgba(30,77,58,0.08); display: flex; align-items: flex-start; gap: 14px; }
  .pt-avatar { width: 48px; height: 48px; border-radius: 50%; background: #2d6a50; display: flex; align-items: center; justify-content: center; color: #fff; font-size: 20px; font-weight: 700; flex-shrink: 0; }
  .pt-info { flex: 1; }
  .pt-name { font-size: 18px; font-weight: 700; color: #1e4d3a; }
  .pt-meta { font-size: 12px; color: #5a7a68; margin-top: 2px; }
  .pt-allergy { font-size: 12px; color: #c0392b; margin-top: 3px; }
  .pt-tags { display: flex; gap: 5px; margin-top: 6px; flex-wrap: wrap; }
  .tag { padding: 2px 9px; border-radius: 20px; font-size: 11px; font-weight: 600; }
  .tag.htn { background: #fde8e8; color: #c0392b; }
  .tag.dm { background: #fef3e2; color: #d68910; }
  .tag.lp { background: #e8f4fd; color: #1a6699; }
  .pt-actions { display: flex; gap: 6px; flex-shrink: 0; }

  /* ── Tabs ── */
  .tabs { display: flex; gap: 3px; margin-bottom: 14px; background: #ede8de; padding: 3px; border-radius: 8px; width: fit-content; }
  .tab { padding: 6px 14px; border-radius: 6px; font-size: 13px; cursor: pointer; border: none; background: transparent; color: #5a7a68; font-weight: 500; transition: all 0.15s; }
  .tab.active { background: #fff; color: #1e4d3a; font-weight: 700; box-shadow: 0 1px 4px rgba(0,0,0,0.08); }

  /* ── Stats ── */
  .stats-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(130px, 1fr)); gap: 10px; margin-bottom: 14px; }
  .stat { background: #fff; border-radius: 10px; padding: 12px; box-shadow: 0 2px 8px rgba(30,77,58,0.08); }
  .stat-label { font-size: 10px; color: #5a7a68; font-weight: 600; text-transform: uppercase; letter-spacing: 0.4px; }
  .stat-val { font-size: 22px; font-weight: 800; color: #1e4d3a; margin: 3px 0 1px; }
  .stat-unit { font-size: 10px; color: #5a7a68; }
  .stat.warn .stat-val { color: #d68910; }
  .stat.bad .stat-val { color: #c0392b; }
  .stat.ok .stat-val { color: #2d6a50; }

  /* ── Card / Table ── */
  .card { background: #fff; border-radius: 10px; box-shadow: 0 2px 8px rgba(30,77,58,0.08); overflow: hidden; margin-bottom: 14px; }
  .card-header { padding: 12px 16px; border-bottom: 1px solid #ede8de; display: flex; align-items: center; justify-content: space-between; }
  .card-title { font-size: 13px; font-weight: 700; color: #1e4d3a; }
  .card-sub { font-size: 11px; color: #5a7a68; }
  .card-body { padding: 12px 16px; font-size: 13px; }
  .no-data { padding: 20px; text-align: center; color: #5a7a68; font-size: 13px; display: flex; flex-direction: column; align-items: center; }
  .vt { width: 100%; border-collapse: collapse; font-size: 12px; }
  .vt th { padding: 8px 10px; text-align: left; background: #f8f5ee; color: #5a7a68; font-weight: 600; font-size: 10px; text-transform: uppercase; }
  .vt td { padding: 8px 10px; border-top: 1px solid #f0ebe0; }
  .vt tr:hover td { background: #fafaf8; }
  .vt .note { color: #5a7a68; max-width: 120px; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
  :global(.ok) { color: #2d6a50 !important; font-weight: 600; }
  :global(.warn) { color: #d68910 !important; font-weight: 600; }
  :global(.bad) { color: #c0392b !important; font-weight: 600; }

  /* ── Buttons ── */
  .btn { padding: 6px 13px; border-radius: 6px; font-size: 13px; font-weight: 600; cursor: pointer; border: none; transition: all 0.15s; }
  .btn.primary { background: #2d6a50; color: #fff; }
  .btn.primary:hover { background: #1e4d3a; }
  .btn.outline { background: transparent; border: 1px solid #c8ddd5; color: #1a2a22; }
  .btn.outline:hover { background: #ede8de; }
  .btn.sm { padding: 4px 10px; font-size: 12px; }
  .btn-del { background: none; border: none; cursor: pointer; color: #5a7a68; padding: 3px 5px; border-radius: 4px; }
  .btn-del:hover { background: #fde8e8; color: #c0392b; }

  /* ── AI Panel ── */
  .ai-panel { padding: 14px 16px; }
  .ai-title { font-size: 13px; font-weight: 700; color: #1e4d3a; margin-bottom: 8px; }
  .quick-btns { display: flex; gap: 5px; flex-wrap: wrap; margin-bottom: 10px; }
  .quick-btn { padding: 3px 10px; border-radius: 20px; border: 1px solid #c8ddd5; background: #f8f5ee; font-size: 12px; cursor: pointer; color: #5a7a68; transition: all 0.1s; }
  .quick-btn:hover { background: #1e4d3a; color: #fff; border-color: #1e4d3a; }
  .ai-messages { min-height: 100px; max-height: 260px; overflow-y: auto; display: flex; flex-direction: column; gap: 7px; margin-bottom: 10px; }
  .ai-msg { padding: 9px 12px; border-radius: 8px; font-size: 13px; line-height: 1.5; white-space: pre-wrap; }
  .ai-msg.assistant { background: #f8f5ee; color: #1a2a22; }
  .ai-msg.user { background: #1e4d3a; color: #fff; align-self: flex-end; max-width: 80%; }
  .ai-msg.thinking { color: #5a7a68; font-style: italic; }
  .ai-input-row { display: flex; gap: 7px; }
  .ai-input-row input { flex: 1; border: 1px solid #c8ddd5; border-radius: 6px; padding: 7px 10px; font-size: 13px; outline: none; background: #f8f5ee; }
  .ai-input-row input:focus { border-color: #4a9070; background: #fff; }

  /* ── Modal ── */
  .modal-overlay { position: fixed; inset: 0; background: rgba(0,0,0,0.4); z-index: 1000; display: flex; align-items: center; justify-content: center; padding: 20px; }
  .modal { background: #fff; border-radius: 12px; padding: 22px; width: 100%; max-width: 480px; max-height: 90vh; overflow-y: auto; box-shadow: 0 20px 60px rgba(0,0,0,0.2); }
  .modal h3 { font-size: 16px; font-weight: 700; color: #1e4d3a; margin-bottom: 16px; }
  .form-row { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-bottom: 10px; }
  .fg { margin-bottom: 10px; }
  .fg label { display: block; font-size: 11px; font-weight: 600; color: #5a7a68; margin-bottom: 4px; text-transform: uppercase; letter-spacing: 0.3px; }
  .fg input, .fg select, .fg textarea { width: 100%; border: 1px solid #c8ddd5; border-radius: 6px; padding: 7px 9px; font-size: 13px; background: #f8f5ee; color: #1a2a22; outline: none; box-sizing: border-box; }
  .fg input:focus, .fg select:focus, .fg textarea:focus { border-color: #4a9070; background: #fff; }
  .ck-row { display: flex; gap: 14px; }
  .ck-row label { display: flex; align-items: center; gap: 5px; font-size: 13px; color: #1a2a22; font-weight: 500; cursor: pointer; text-transform: none; letter-spacing: 0; }
  .modal-footer { display: flex; gap: 7px; justify-content: flex-end; margin-top: 16px; padding-top: 14px; border-top: 1px solid #ede8de; }

  /* ── Toast ── */
  .toast { position: fixed; bottom: 20px; right: 20px; background: #1e4d3a; color: #fff; padding: 9px 16px; border-radius: 8px; font-size: 13px; box-shadow: 0 4px 16px rgba(0,0,0,0.2); z-index: 2000; }

  @media (max-width: 640px) {
    .chronic-sidebar { width: 180px; min-width: 180px; }
    .stats-grid { grid-template-columns: 1fr 1fr; }
    .form-row { grid-template-columns: 1fr; }
  }
</style>