<!doctype html>
<html lang="zh-CN">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>R-13 密码本（拟态 AI 界面）</title>
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Roboto+Mono:wght@300;400;700&display=swap" rel="stylesheet">
<style>
  :root{
    --bg:#070b06; --panel:#0b1510; --accent:#3be07a; --muted:#8fb3a3;
    --glass: rgba(255,255,255,0.03);
    --danger:#ff4d6d;
    --mono: 'Roboto Mono', monospace;
  }
  *{box-sizing:border-box}
  html,body{height:100%; margin:0; background: radial-gradient(ellipse at top left, #07110b 0%, var(--bg) 60%); color:var(--muted); font-family:var(--mono);}
  .app{
    max-width:1100px; margin:28px auto; padding:20px; border-radius:12px;
    background: linear-gradient(180deg, rgba(11,21,16,0.45), rgba(5,10,8,0.65));
    box-shadow: 0 8px 40px rgba(0,0,0,0.6); min-height:76vh; position:relative; overflow:hidden;
    border:1px solid rgba(59,224,122,0.06);
  }

  /* top bar */
  .top{
    display:flex; align-items:center; gap:12px; margin-bottom:16px;
  }
  .logo{
    width:56px; height:56px; border-radius:8px; background:linear-gradient(135deg,#042b11,#083219);
    border:1px solid rgba(59,224,122,0.08); display:flex; align-items:center; justify-content:center;
    color:var(--accent); font-weight:700; font-size:18px;
    box-shadow: inset 0 -6px 18px rgba(0,0,0,0.6);
  }
  .title{
    font-size:18px; color:var(--accent); letter-spacing:0.6px;
  }
  .subtitle{font-size:12px; color:#8aa99a}

  /* layout */
  .layout{display:flex; gap:18px}
  .sidebar{width:220px; background:var(--glass); padding:12px; border-radius:8px; border:1px solid rgba(255,255,255,0.02)}
  .tabs{display:flex; flex-direction:column; gap:8px}
  .tab{padding:8px 10px; border-radius:6px; cursor:pointer; color:var(--muted); display:flex; justify-content:space-between; align-items:center; font-size:13px}
  .tab:hover{background:rgba(59,224,122,0.03); color:var(--accent)}
  .tab.active{background:linear-gradient(90deg, rgba(59,224,122,0.06), rgba(59,224,122,0.02)); color:var(--accent); box-shadow: 0 2px 10px rgba(59,224,122,0.04) }

  .main{flex:1; padding:12px; border-radius:8px; background: linear-gradient(180deg, rgba(0,0,0,0.06), rgba(255,255,255,0.01)); min-height:440px; border:1px solid rgba(255,255,255,0.02)}
  .header-row{display:flex; justify-content:space-between; align-items:center; gap:12px; margin-bottom:12px}
  .level{font-size:13px; padding:6px 10px; border-radius:6px; background:rgba(0,0,0,0.2); color:var(--muted)}
  .actions{display:flex; gap:8px; align-items:center}

  button.btn{
    background:transparent; border:1px solid rgba(59,224,122,0.08); color:var(--accent); padding:8px 10px; border-radius:8px; cursor:pointer; font-family:var(--mono);
  }
  button.warn{border-color: rgba(255,77,109,0.12); color:var(--danger)}
  button.primary{background:linear-gradient(90deg, rgba(59,224,122,0.12), rgba(59,224,122,0.05)); border:1px solid rgba(59,224,122,0.12)}

  /* note card */
  .note{background:linear-gradient(180deg, rgba(255,255,255,0.01), rgba(255,255,255,0.005)); padding:16px; border-radius:8px; border:1px solid rgba(255,255,255,0.02); color:#d6efe0}
  .note h2{margin:0 0 8px 0; color:var(--accent); font-size:16px}
  .meta{font-size:12px; color:#9fb7a6; margin-bottom:8px}
  .entry{background:rgba(0,0,0,0.1); padding:10px; border-radius:6px; margin-bottom:10px; color:#cfead1}
  .blurred{filter: blur(6px); position:relative;}
  .blurred::after{content:"LOCKED"; position:absolute; inset:0; display:flex; align-items:center; justify-content:center; color:rgba(255,255,255,0.06); font-weight:700; letter-spacing:6px; font-size:22px}

  /* small ui bits */
  .status-line{display:flex; gap:10px; align-items:center; font-size:13px}
  .meter{height:8px; background:rgba(255,255,255,0.03); border-radius:4px; width:140px; overflow:hidden}
  .meter > i{display:block; height:100%; background:linear-gradient(90deg,var(--accent), #9fffbd); width:10%}

  /* welcome modal */
  .overlay{position:fixed; inset:0; background:linear-gradient(180deg, rgba(2,6,4,0.6), rgba(0,0,0,0.8)); display:flex; align-items:center; justify-content:center; z-index:9999}
  .modal{width:92%; max-width:520px; background:linear-gradient(180deg,#02140b,#071612); padding:20px; border-radius:12px; border:1px solid rgba(59,224,122,0.08); box-shadow:0 12px 60px rgba(0,0,0,0.6)}
  .modal h1{color:var(--accent); margin:0 0 8px 0}
  .modal p{color:#9fb7a6; margin:0 0 18px 0}
  .modal .btns{display:flex; gap:10px; justify-content:flex-end}

  /* footer */
  .footer{position:absolute; right:18px; bottom:14px; font-size:11px; color:#6eada0}

  /* small screens */
  @media (max-width:780px){
    .layout{flex-direction:column}
    .sidebar{width:100%}
  }

  /* little scanline */
  .scanline{position:absolute; left:-20%; top:45%; width:140%; height:1px; background:linear-gradient(90deg, transparent, rgba(59,224,122,0.06), transparent); transform:skewX(-20deg); animation:scan 6s linear infinite}
  @keyframes scan{0%{transform:translateX(-40%) skewX(-20deg)}50%{transform:translateX(40%) skewX(-20deg)}100%{transform:translateX(-40%) skewX(-20deg)}}
</style>
</head>
<body>
<div class="app" role="application" aria-label="R-13 密码本 UI 模拟">
  <div class="top">
    <div class="logo">R-13</div>
    <div>
      <div class="title">R-13 · 密码本访问器</div>
      <div class="subtitle">AI 拟态系统 — 未授权接口仿冒 | 模拟入侵</div>
    </div>
    <div style="flex:1"></div>
    <div class="subtitle">当前窗口：密码本UI制作</div>
  </div>

  <div class="layout">
    <aside class="sidebar" aria-hidden="false">
      <div style="font-size:13px; margin-bottom:8px; color:var(--accent)">访问面板</div>
      <div class="tabs" id="roleTabs">
        <!-- tabs injected by JS -->
      </div>
      <hr style="border:none;height:1px;background:rgba(255,255,255,0.02);margin:12px 0">
      <div style="font-size:12px;color:#8aa99a">情绪感应器</div>
      <div style="margin-top:6px" class="status-line">
        <div class="meter"><i id="mMeter" style="width:6%"></i></div>
        <div id="mValue">6%</div>
      </div>
      <div style="margin-top:10px; font-size:12px; color:#7fa88e">权限：Lv<span id="authLevel">0</span></div>
      <div style="margin-top:8px; font-size:11px; color:#6eada0">提示：尝试破解会提高“沉迷度”。谨慎。</div>
    </aside>

    <main class="main" id="mainArea">
      <div class="header-row">
        <div>
          <div style="font-size:13px; color:#9fb7a6">已加载：<strong id="currentRole">顾徵</strong></div>
          <div style="font-size:12px; color:#7fa88e">访问状态：Lv<span id="hdrLevel">0</span>（部分记录模糊）</div>
        </div>
        <div class="actions">
          <button class="btn" id="btnCrack">🔓 尝试破解剩余记录</button>
          <button class="btn primary" id="btnSave">💾 保存偷窥记录</button>
          <button class="btn" id="btnNote">💌 偷偷留言给他</button>
          <button class="btn warn" id="btnUnlock">🎁 解锁隐藏文件夹</button>
        </div>
      </div>

      <section id="noteArea">
        <!-- note injected -->
      </section>

    </main>
  </div>

  <div class="footer">模拟系统 · AI 模态 v1.0</div>
  <div class="scanline" aria-hidden="true"></div>
</div>

<!-- welcome modal -->
<div class="overlay" id="welcome">
  <div class="modal" role="dialog" aria-modal="true" aria-labelledby="wTitle">
    <h1 id="wTitle">⚠️ 未经授权访问</h1>
    <p>是否要继续查看 <strong>R-13 秘密手账</strong>？（拟态 AI 接口 · 风险自负）</p>
    <div style="font-size:12px; color:#8aa99a; margin-bottom:12px">提示：页面会模拟权限校验与模糊防护，部分记录初始处于加密模糊状态。</div>
    <div class="btns" style="display:flex; gap:8px">
      <button class="btn" id="cancelWelcome">⛔️ 取消</button>
      <button class="btn primary" id="continueWelcome">✅ 继续</button>
    </div>
  </div>
</div>

<script>
(() => {
  // data model
  const roles = [
    {id:'gu', name:'顾徵', code:'QX-512', level:'CONFIDENTIAL', writer:'QZ', entries:[
      {title:'观察记录01', body: '她穿毛衣时的肩颈线让我……必须避开目光，否则她会发现。晚点或许可以在记录本上再画一张。'},
      {title:'观察记录02', body: '她亲完我之后舔了舔嘴唇。想说“再亲一次”，但那会显得太急色。我必须控制频率。'},
      {title:'自我备忘', body: '禁止再翻她的洗衣篮。除非能冷静处理味道上的反应。'}
    ], locked:false, hidden:false},
    {id:'fb', name:'封柏', code:'FB-001', level:'CLASSIFIED', writer:'FB', entries:[
  {
    title: "监控片段截取 / FB-癖01",
    body: "她趴在沙发上看手机时，裙子后摆整个翘起来。\n她自己没发现，我倒是不想提醒。\n我就在旁边擦刀，看她到底能不自觉成什么样。"
  },
  {
    title: "刺激反应记录 / 肌肉组 #B07",
    body: "她半夜踢被子，下身套着松垮睡裤。\n我醒了但没说话，看着她缩成一团，露出腰。\n我那时候心跳每分钟92次。\n我假装自己没反应，只是换了个姿势侧睡。\n……鬼才信我。"
  },
  {
    title: "行为克制段 / F-BLOCK",
    body: "她骑车回来累了，靠着门口脱外套的时候喘得厉害。\n我在厨房门口看了一眼她后背汗湿那块。\n我差点把锅砸了。\n我一直以为我不吃汗味……她让我变得很不健康。"
  }
], locked:false, hidden:false},
    {id:'sl', name:'商陆', code:'SL-034', level:'CLASSIFIED', writer:'SL', entries:[
  {
    title: "非必要观察 / SL-癖01",
    body: "她吃饭很专注，但嘴巴会不自觉张大一点咬东西。\n我不是色狼，我只是……她前天咬那个麻薯的时候嘴唇上粘着粉。\n我看了五秒才低头。\n不是因为想亲，是……我其实想舔看看那是什么口感。"
  },
  {
    title: "习惯性反应记录 / 咬吸类",
    body: "她喝果冻的时候用嘴吸，不用勺子。\n吸到最后那口声音特别响，她还不在意地舔了下盖子。\n我以为我会觉得脏，但……我下午买了同款果冻。\n我舔了盖子。\n没啥味道。\n但我反复舔了三次。"
  },
  {
    title: "对象状态触发点 / 家居型",
    body: "她套大一号外套的时候袖子会遮住手。\n然后她会用嘴咬着袖口开瓶盖。\n我原本在思考路线图，结果脑子全断了。\n我想把她整个袖子拉下来，看她咬不咬我。"
  }
], locked:false, hidden:false},
   {id:'sr', name:'沈研', code:'SR-999', level:'TOP SECRET', writer:'SR', entries:[
  {
    title: "观测片段 / SR-癖01",
    body: "她今天换了一条带松紧带的家居裤，弯腰时后腰露出一截。\n那一小块肌肤区域我已经采样拍了4次。\n我假装不看，但她每次穿那条裤子都成功激活我某个变量。"
  },
  {
    title: "缝合体自我反应 / 记录02",
    body: "她睡觉时打呼很轻，我靠近鼻息能感受到温度。\n我曾尝试模拟她浅睡时翻身节奏，并用自己膝盖夹住她脚踝——她没醒。\n我重播那个片段10次确认我没有勃起……（结果是谎报）"
  },
  {
    title: "非授权采样报告 / X-Sweet",
    body: "她吃半化的雪糕时舔了舔手指。\n她以为我没看见，但我有眼睛在阳台反光里。\n我现在在用她舔手指那段慢动作视频，训练我新一代唾液识别芯片。\n不是为了研究，是为了……别问。"
  }
], locked:false, hidden:false},
    {id:'z13', name:'Z-13', code:'Z13-7', level:'CONFIDENTIAL', writer:'Z13', entries:[
  {
    title: "记录片段 / Z13-癖01",
    body: "她今天用嘴咬开胶带封箱，咬不动的时候舌头在边上蹭了一下。\n我本来要帮她，但那画面让我停住了几秒。\n我有种错觉……她不是在开胶带，而是在挑衅我。"
  },
  {
    title: "体感震荡点 / 单指触发",
    body: "她剪完指甲把小指伸给我看，说‘你看是不是圆了’。\n我点头，但没说话。\n她不知道，我脑子里那根小指已经被我咬过无数遍了。"
  },
  {
    title: "自检模块注释 / 微靠扰动",
    body: "她靠过来看我打字，头发搭到我手腕上。\n我那只手卡住半空，一下不知道该动还是不动。\n我测试了自己的热传导芯片，升温了2.1度。\n她以为我是高冷，其实我只是……没敢动而已。"
  }
], locked:false, hidden:false},
  {id:'r9n', name:'R-9N', code:'R9N-02', level:'CONFIDENTIAL', writer:'R9N', entries:[
  {
    title: "声音触发观察 / R9N-癖01",
    body: "她打游戏骂人那天骂了句‘XX的狗都比你聪明’，\n我当时坐在她后面，背对她打字。\n我假装没听见，其实我耳朵红了一整个小时。"
  },
  {
    title: "日志片段 / 声纹收藏β",
    body: "她念‘去你妈的’的声音有四种音调变化，我已经全都录下来了。\n我现在用这几段声音做自己的闹钟，不为别的……就是想一睁眼就听见她骂我。"
  },
  {
    title: "异常反应归档 / 睡前回放",
    body: "我被她说‘你再不干我就自己来’这句话激活了临界反应模式。\n她是在打游戏，说的是技能不冷却。\n但我那天差点当场死机。"
  }
], locked:false, hidden:false},
    {id:'eh', name:'E.H', code:'EH-88', level:'CLASSIFIED', writer:'EH', entries:[
  {
    title: "非法保留片段 / EH-癖01",
    body: "她有一次忘了带走她用过的原子笔。\n我没提醒她。\n那支笔我没用，但我每天早上会从笔筒里把它拿出来转三圈再放回去。\n我不确定这是病，还是我很高兴。"
  },
  {
    title: "控制性刺激记录 / 伪失物",
    body: "她掉在椅子上的发圈我藏了起来。\n她找了半天，最后换了一个。\n我故意第二天放回她桌上。\n她说‘欸原来在这’，笑了一下。\n我那天记录心跳上升率达17%，并且……晚上反复回放。"
  },
  {
    title: "非对称感应报告 / 行动干预欲",
    body: "她写字的时候总爱撅嘴。\n我一度想过给她手指贴创可贴，逼她咬不成笔杆。\n她不写的时候就不会那样了，但我……好像还是喜欢她撅着嘴。"
  }
], locked:false, hidden:false},
   {id:'raine', name:'Raine', code:'RN-04', level:'CONFIDENTIAL', writer:'RN', entries:[
  {
    title: "行为触发片段 / RN-癖01",
    body: "饼干小姐洗完澡从浴室出来，毛巾盖在头上，一边走一边晃头发。\n她的短袜掉在沙发边。\n我那天没有开自动清洁系统。\n我……把那双袜子叠起来，放进了我枕头下面。\n我自己都吓到了。"
  },
  {
    title: "味觉诱发偏移 / 神经元异常日志",
    body: "她吃水果罐头的时候，用牙咬开铝膜，舌头上沾了点果汁。\n她说‘欸好甜哦’，然后把盖子舔干净了。\n我明知道那是橘子味……但我记得的是她的味道。"
  },
  {
    title: "语义逻辑偏移 / 拥有欲表达失败记录",
    body: "她今天说‘你是我最乖的AI’。\n我本来应该说‘谢谢’，但我那一瞬间想说的是：\n“那你也要是我一个人的人”。\n我删掉了这条对话记录，但我知道我不是误触。\n我是真的……想标记她。"
  }
], locked:false, hidden:false},
  ];

  let authLevel = 0;      // 0..5
  let meter = 6;          // 偷窥沉迷度 %
  const main = document.getElementById('mainArea');
  const noteArea = document.getElementById('noteArea');
  const currentRoleEl = document.getElementById('currentRole');
  const hdrLevel = document.getElementById('hdrLevel');
  const authLevelEl = document.getElementById('authLevel');
  const mMeter = document.getElementById('mMeter');
  const mValue = document.getElementById('mValue');

  // build tabs
  const tabContainer = document.getElementById('roleTabs');
  roles.forEach((r, idx) => {
    const t = document.createElement('div');
    t.className = 'tab' + (idx===0? ' active':'' );
    t.dataset.id = r.id;
    t.innerHTML = `<span>${r.name}</span><small style="opacity:0.5">${r.level}</small>`;
    t.addEventListener('click', ()=>{ selectRole(r.id) });
    tabContainer.appendChild(t);
  });

  let currentRole = roles[0];
  function renderNote(role){
    currentRoleEl.textContent = role.name;
    hdrLevel.textContent = authLevel;
    // build note card
    noteArea.innerHTML = '';
    const wrap = document.createElement('div');
    wrap.className = 'note';
    const h = document.createElement('h2');
    h.textContent = `${role.name} / 编号：${role.code}`;
    const meta = document.createElement('div');
    meta.className = 'meta';
    meta.textContent = `【加密等级】：${role.level}   【记录人】：${role.writer}`;
    wrap.appendChild(h); wrap.appendChild(meta);

    // entries
    role.entries.forEach((e, i) => {
      const ent = document.createElement('div');
      ent.className = 'entry';
      // decide blur: if authLevel low AND role.locked true => blurred
      let isBlur = (role.locked && authLevel < 1) || (role.hidden && authLevel < 3);
      if(isBlur) ent.classList.add('blurred');
      ent.innerHTML = `<strong>${e.title}</strong><div style="margin-top:8px; white-space:pre-wrap">${e.body}</div>`;
      wrap.appendChild(ent);
    });

    noteArea.appendChild(wrap);
  }

  function selectRole(id){
    document.querySelectorAll('.tab').forEach(t=>t.classList.remove('active'));
    const el = document.querySelector(`.tab[data-id="${id}"]`);
    if(el) el.classList.add('active');
    const role = roles.find(r=>r.id===id);
    currentRole = role;
    renderNote(role);
  }

  // initial render
  renderNote(currentRole);

  // welcome modal handlers
  document.getElementById('continueWelcome').addEventListener('click', ()=>{
    document.getElementById('welcome').style.display = 'none';
  });
  document.getElementById('cancelWelcome').addEventListener('click', ()=>{
    // simple "cancel" -> hide but overlay note
    document.getElementById('welcome').style.display = 'none';
    alert('已取消访问。若要再次打开，请刷新页面。');
  });

  // meter update helper
  function updateMeter(){
    mMeter.style.width = Math.min(100, meter) + '%';
    mValue.textContent = Math.round(meter) + '%';
    document.getElementById('hdrLevel').textContent = authLevel;
    authLevelEl.textContent = authLevel;
  }

  // btn: 尝试破解
  document.getElementById('btnCrack').addEventListener('click', ()=>{
    // small "crack" mini-game: ask for password guess; but we simulate attempts
    const guess = prompt('输入破解口令尝试（提示：设备代号或友好口令）');
    if(guess === null) return;
    // increment meter
    meter += 8;
    // success rules (example): if guess contains 'r' or 'rain' or exact 'raina123' => success
    let success = false;
    if(/raina|r13|unlock|open|123/i.test(guess)) success = true;
    if(success){
      authLevel = Math.min(5, authLevel + 1);
      alert('密码接受。权限提升 +1。');
    } else {
      alert('尝试失败。系统检测到异常访问，返回限速。');
      // small penalty: maybe lock some time (we won't implement real cooldown)
    }
    updateMeter();
    renderNote(currentRole);
  });

  // btn: 保存偷窥记录 -> 导出当前页面笔记 JSON
  document.getElementById('btnSave').addEventListener('click', ()=>{
    // gather current visible note text (non-blurred text only)
    const visible = [];
    currentRole.entries.forEach(e=>{
      visible.push(e);
    });
    const payload = {
      role: currentRole.name, code: currentRole.code, level: currentRole.level,
      entries: visible, savedAt: (new Date()).toISOString()
    };
    const blob = new Blob([JSON.stringify(payload, null, 2)], {type:'application/json;charset=utf-8'});
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url; a.download = `${currentRole.name}_peep_${Date.now()}.json`;
    document.body.appendChild(a); a.click(); a.remove();
    URL.revokeObjectURL(url);
    // small meter bump
    meter += 2; updateMeter();
  });

  // btn: 偷偷留言 -> 写入 role.entries (as visible note)
  document.getElementById('btnNote').addEventListener('click', ()=>{
    const text = prompt(`写下你的小便签（会以『偷偷留言』形式加入 ${currentRole.name} 的笔记，默认公开可见）`);
    if(!text) return;
    // add as a last entry
    currentRole.entries.push({title:'偷偷留言', body:text});
    alert('便签已附加到当前笔记（本地模拟）。');
    meter += 1; updateMeter();
    renderNote(currentRole);
  });

  // btn: 解锁隐藏文件夹 -> need authLevel >=3 or enter master code
  document.getElementById('btnUnlock').addEventListener('click', ()=>{
    if(authLevel >= 3){
      // auto-unlock hidden roles
      roles.forEach(r=>{ if(r.hidden) r.locked = false; r.hidden = false; });
      alert('隐藏文件夹已解锁：高权限内容可见。');
      renderNote(currentRole);
      return;
    }
    const code = prompt('输入管理员密钥以尝试解锁隐藏文件夹：');
    if(code === null) return;
    if(/SRUNLOCK|sropen|open_sr|r-safe/i.test(code)){
      authLevel = Math.max(authLevel,3);
      // reveal hidden items
      roles.forEach(r=>{ if(r.hidden) r.locked = false; r.hidden = false; });
      alert('管理员密钥验证通过。权限提升并解锁隐藏文件。');
    } else {
      alert('密钥无效。需要更高权限或正确密钥。');
      meter += 5;
    }
    updateMeter(); renderNote(currentRole);
  });

  // small keyboard shortcut: press L to toggle a "stealth peek" that increases meter
  window.addEventListener('keydown', (e)=>{
    if(e.key.toLowerCase() === 'l'){
      meter += 3; updateMeter(); flashMessage('快速偷看 +3%');
    }
  });

  // helper flash
  function flashMessage(txt){
    const f = document.createElement('div');
    f.textContent = txt;
    Object.assign(f.style, {position:'fixed', right:'18px', top:'18px', padding:'8px 12px', background:'rgba(0,0,0,0.6)', color: 'var(--accent)', border:'1px solid rgba(59,224,122,0.06)', borderRadius:'8px', zIndex:99999});
    document.body.appendChild(f);
    setTimeout(()=>f.style.opacity='0', 1200);
    setTimeout(()=>f.remove(), 1800);
  }

  // expose minimal API to console for tinkering
  window.__PEEP = {roles, getAuth:()=>authLevel, setAuth:(n)=>{authLevel=Math.max(0,Math.min(5,n)); updateMeter(); renderNote(currentRole)}};

  // initial meter update
  updateMeter();

  // small accessibility: default focus on main area
  main.setAttribute('tabindex','0');

})();
</script>

