<!DOCTYPE html>
<html lang="th">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>หน้าสแกน — เบิก/คืน/ลงทะเบียนใบหน้า (PDA Tracker)</title>
<script src="https://unpkg.com/html5-qrcode@2.3.8/html5-qrcode.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/face-api.js@0.22.2/dist/face-api.min.js"></script>
<style>
  :root{
    --bg:#f4f5f7; --card:#ffffff; --ink:#1f2430; --muted:#6b7280;
    --line:#e5e7eb; --accent:#2f6f4f; --accent-ink:#ffffff;
    --ok:#2f6f4f; --bad:#b3261e; --face:#9d174d;
  }
  *{box-sizing:border-box}
  body{margin:0; background:var(--bg); color:var(--ink); font-family:"Sarabun","Segoe UI",system-ui,-apple-system,sans-serif; font-size:15px;}
  header{background:var(--ink); color:#fff; padding:16px 20px;}
  header h1{font-size:17px; margin:0; font-weight:600;}
  header .sub{font-size:12px; color:#c7ccd6; margin-top:2px;}
  nav{display:flex; gap:6px; padding:10px 16px; background:#fff; border-bottom:1px solid var(--line); overflow-x:auto;}
  nav button{border:none; background:transparent; padding:8px 14px; border-radius:999px; cursor:pointer; font-size:14px; color:var(--muted); white-space:nowrap;}
  nav button.active{background:var(--ink); color:#fff;}
  main{padding:18px; max-width:560px; margin:0 auto;}
  section.panel{background:var(--card); border:1px solid var(--line); border-radius:14px; padding:18px; margin-bottom:16px;}
  section.panel h2{font-size:15px; margin:0 0 4px;}
  .panel-sub{font-size:12.5px; color:var(--muted); margin-bottom:12px;}
  label{display:block; font-size:12.5px; color:var(--muted); margin-bottom:4px; margin-top:10px;}
  input, select, textarea{width:100%; padding:9px 10px; border:1px solid var(--line); border-radius:8px; font-size:14px; font-family:inherit; background:#fbfbfc;}
  input:disabled{color:var(--ink); font-weight:700; background:#eef2f0;}
  textarea{resize:vertical; min-height:44px;}
  .row-actions{margin-top:14px; display:flex; gap:8px; flex-wrap:wrap;}
  button.primary{background:var(--accent); color:var(--accent-ink); border:none; padding:10px 18px; border-radius:8px; font-size:14px; cursor:pointer; font-weight:600;}
  button.secondary{background:#fff; color:var(--ink); border:1px solid var(--line); padding:10px 18px; border-radius:8px; font-size:14px; cursor:pointer;}
  button.scan-btn{background:#eef0ff; color:#3730a3; border:1px solid #d6d9ff; padding:9px 14px; border-radius:8px; font-size:13px; cursor:pointer; font-weight:600;}
  button.face-btn{background:#fdeef4; color:var(--face); border:1px solid #f7cfe0; padding:9px 14px; border-radius:8px; font-size:13px; cursor:pointer; font-weight:600;}
  button:disabled{opacity:.5; cursor:not-allowed;}
  .msg{margin-top:10px; font-size:13px; padding:8px 10px; border-radius:8px; display:none;}
  .msg.ok{display:block; background:#eaf5ee; color:var(--ok);}
  .msg.err{display:block; background:#fdecec; color:var(--bad);}
  .hint{font-size:12px; color:var(--muted); margin-top:6px;}
  .banner{background:#fdecec; color:var(--bad); border:1px solid #f6c8c4; padding:12px 14px; border-radius:10px; font-size:13px; margin-bottom:16px;}
  #qr-reader, #faceVideo{width:100%; border-radius:10px; overflow:hidden;}
  #faceVideo{background:#000; aspect-ratio:4/3;}
  .field-with-btn{display:flex; gap:8px; align-items:stretch;}
  .field-with-btn > div{flex:1;}
</style>
</head>
<body>

<header>
  <h1>📷 หน้าสแกน — PDA Tracker</h1>
  <div class="sub">เบิกออก / ชำรุด(รับคืน) / ลงทะเบียนใบหน้า — ใช้กล้องได้จริงเพราะโฮสต์แยกนอก Apps Script</div>
</header>

<nav id="tabs">
  <button data-tab="checkout" class="active">เบิกออก</button>
  <button data-tab="damaged">ชำรุด (รับคืน)</button>
  <button data-tab="enroll">ลงทะเบียนใบหน้า</button>
</nav>

<main>
  <div id="configBanner" class="banner" style="display:none;">
    ⚠️ ยังไม่ได้ตั้งค่าลิงก์ Apps Script — เปิดไฟล์นี้ด้วยโปรแกรมแก้ไขข้อความ แล้วแก้ค่า
    <code>APPS_SCRIPT_URL</code> ที่ต้นไฟล์ให้เป็นลิงก์เว็บแอปของคุณ (ลงท้ายด้วย <code>/exec</code>)
  </div>

  <!-- เบิกออก -->
  <div class="tab-content" id="tab-checkout">
    <section class="panel">
      <h2>เบิกออกใช้งาน</h2>
      <div class="panel-sub">สแกนใบหน้ายืนยันตัวตนผู้เบิก แล้วสแกน QR เครื่องมือ</div>

      <label>ผู้เบิก</label>
      <div class="field-with-btn">
        <div><input id="coEmployee" placeholder="ยังไม่ยืนยันตัวตน" readonly></div>
        <button type="button" class="face-btn" onclick="openFaceScan('checkout')">🙂 สแกนใบหน้า</button>
      </div>

      <label>หมายเลขเครื่อง (Asset ID)</label>
      <div class="field-with-btn">
        <div><input id="coAssetId" placeholder="เช่น NEC001"></div>
        <button type="button" class="scan-btn" onclick="openQrScan('coAssetId')">📷 สแกน QR</button>
      </div>

      <label>หมายเหตุ (ถ้ามี)</label>
      <textarea id="coNote"></textarea>
      <div class="row-actions">
        <button class="primary" onclick="doCheckout()">บันทึกเบิกออก</button>
      </div>
      <div class="msg" id="coMsg"></div>
    </section>
  </div>

  <!-- ชำรุด (รับคืน) -->
  <div class="tab-content" id="tab-damaged" style="display:none;">
    <section class="panel">
      <h2>ชำรุด — รับคืนเครื่องจากที่เบิกออกไปใช้งาน</h2>
      <div class="panel-sub">ผู้คืนอาจไม่ใช่คนเดียวกับผู้เบิกไปใช้ก็ได้</div>

      <label>ผู้คืนเครื่อง</label>
      <div class="field-with-btn">
        <div><input id="rtReturnedBy" placeholder="ยังไม่ยืนยันตัวตน" readonly></div>
        <button type="button" class="face-btn" onclick="openFaceScan('return')">🙂 สแกนใบหน้า</button>
      </div>

      <label>หมายเลขเครื่อง (Asset ID)</label>
      <div class="field-with-btn">
        <div><input id="rtAssetId" placeholder="เช่น NEC001"></div>
        <button type="button" class="scan-btn" onclick="openQrScan('rtAssetId')">📷 สแกน QR</button>
      </div>

      <label>สภาพเครื่องเมื่อคืน</label>
      <select id="rtCondition">
        <option value="ปกติ">ปกติ — คืนเข้าสต็อกพร้อมใช้งาน</option>
        <option value="ชำรุด">ชำรุด — รอส่งซ่อม</option>
      </select>
      <label>หมายเหตุ (ถ้ามี)</label>
      <textarea id="rtNote"></textarea>
      <div class="row-actions">
        <button class="primary" onclick="doReturn()">บันทึกรับคืน</button>
      </div>
      <div class="msg" id="rtMsg"></div>
    </section>
  </div>

  <!-- ลงทะเบียนใบหน้า -->
  <div class="tab-content" id="tab-enroll" style="display:none;">
    <section class="panel">
      <h2>ลงทะเบียนใบหน้าพนักงาน</h2>
      <div class="panel-sub">ข้อมูลใบหน้าเป็นข้อมูลชีวมิติ (biometric) ตาม PDPA ควรขอความยินยอมจากพนักงานก่อนลงทะเบียน</div>
      <label>ชื่อพนักงาน</label>
      <input id="enrollName" placeholder="ชื่อ-สกุล">
      <div class="row-actions">
        <button class="primary" onclick="openFaceScan('enroll')">🙂 เปิดกล้องลงทะเบียนใบหน้า</button>
      </div>
      <div class="msg" id="enrollMsg"></div>
    </section>
  </div>

  <!-- QR scanner -->
  <section class="panel" id="qrPanel" style="display:none;">
    <h2>สแกน QR Code เครื่องมือ</h2>
    <div id="qr-reader"></div>
    <div class="row-actions"><button class="secondary" onclick="closeQrScan()">ปิด</button></div>
    <div class="hint">เล็งกล้องไปที่ QR Code บนตัวเครื่อง</div>
  </section>

  <!-- Face scanner -->
  <section class="panel" id="facePanel" style="display:none;">
    <h2 id="faceTitle">สแกนใบหน้า</h2>
    <video id="faceVideo" autoplay muted playsinline></video>
    <div class="row-actions">
      <button class="primary" onclick="captureFace()">ถ่ายภาพใบหน้า</button>
      <button class="secondary" onclick="closeFaceScan()">ปิด</button>
    </div>
    <div class="msg" id="faceMsg"></div>
    <div class="hint">จัดใบหน้าให้อยู่กลางกล้อง แสงเพียงพอ แล้วกดถ่ายภาพ</div>
  </section>
</main>

<script>
  /* ======================================================
     ตั้งค่าจุดเชื่อมต่อ: วางลิงก์เว็บแอป Apps Script ของคุณที่นี่
     (Deploy > Manage deployments > คัดลอกลิงก์ที่ลงท้ายด้วย /exec)
     ====================================================== */
  const APPS_SCRIPT_URL = 'https://script.google.com/macros/s/AKfycbwYDWEiHZUT3Rlrev43qU4wYGTAE2-NOocEQcciNXCCkOURL9RuOOR8WTIpRFjpmjhD/exec'; // เช่น https://script.google.com/macros/s/XXXXXXXX/exec

  if(!APPS_SCRIPT_URL || APPS_SCRIPT_URL.indexOf('PASTE_YOUR') === 0){
    document.getElementById('configBanner').style.display = 'block';
  }

  /* ---------- Tabs ---------- */
  document.getElementById('tabs').addEventListener('click', function(e){
    if(e.target.tagName !== 'BUTTON') return;
    document.querySelectorAll('#tabs button').forEach(b=>b.classList.remove('active'));
    e.target.classList.add('active');
    var tab = e.target.getAttribute('data-tab');
    document.querySelectorAll('.tab-content').forEach(el=>el.style.display='none');
    document.getElementById('tab-'+tab).style.display='block';
  });

  function showMsg(id, text, isError){
    var el = document.getElementById(id);
    el.textContent = text;
    el.className = 'msg ' + (isError ? 'err' : 'ok');
  }

  /* ---------- API helpers (เรียก Apps Script เป็น JSON API) ---------- */
  async function apiGet(action, params){
    var url = new URL(APPS_SCRIPT_URL);
    url.searchParams.set('api', '1');
    url.searchParams.set('action', action);
    if(params){ for(var k in params){ url.searchParams.set(k, params[k]); } }
    var res = await fetch(url.toString());
    return res.json();
  }
  async function apiPost(action, payload){
    var body = Object.assign({ action: action }, payload || {});
    var res = await fetch(APPS_SCRIPT_URL, {
      method: 'POST',
      headers: { 'Content-Type': 'text/plain;charset=utf-8' }, // หลีกเลี่ยง CORS preflight
      body: JSON.stringify(body)
    });
    return res.json();
  }

  /* ---------- เบิกออก / รับคืน ---------- */
  async function doCheckout(){
    var id = document.getElementById('coAssetId').value.trim();
    var emp = document.getElementById('coEmployee').value.trim();
    var note = document.getElementById('coNote').value.trim();
    if(!emp){ showMsg('coMsg', 'กรุณาสแกนใบหน้ายืนยันตัวตนผู้เบิกก่อน', true); return; }
    if(!id){ showMsg('coMsg', 'กรุณาระบุหมายเลขเครื่อง', true); return; }
    try{
      var res = await apiPost('checkOut', { assetId:id, employee:emp, note:note });
      if(res.ok){
        showMsg('coMsg', 'บันทึกเบิกออกสำเร็จ: ' + id + ' โดย ' + emp, false);
        document.getElementById('coAssetId').value=''; document.getElementById('coEmployee').value='';
        document.getElementById('coNote').value='';
      } else { showMsg('coMsg', res.error || 'บันทึกไม่สำเร็จ', true); }
    }catch(e){ showMsg('coMsg', 'เชื่อมต่อระบบไม่สำเร็จ: ' + e, true); }
  }

  async function doReturn(){
    var id = document.getElementById('rtAssetId').value.trim();
    var cond = document.getElementById('rtCondition').value;
    var returnedBy = document.getElementById('rtReturnedBy').value.trim();
    var note = document.getElementById('rtNote').value.trim();
    if(!returnedBy){ showMsg('rtMsg', 'กรุณาสแกนใบหน้ายืนยันตัวตนผู้คืนก่อน', true); return; }
    if(!id){ showMsg('rtMsg', 'กรุณาระบุหมายเลขเครื่อง', true); return; }
    try{
      var res = await apiPost('returnEquip', { assetId:id, condition:cond, returnedBy:returnedBy, note:note });
      if(res.ok){
        showMsg('rtMsg', 'บันทึกรับคืนสำเร็จ: ' + id + ' โดย ' + returnedBy, false);
        document.getElementById('rtAssetId').value=''; document.getElementById('rtReturnedBy').value='';
        document.getElementById('rtNote').value='';
      } else { showMsg('rtMsg', res.error || 'บันทึกไม่สำเร็จ', true); }
    }catch(e){ showMsg('rtMsg', 'เชื่อมต่อระบบไม่สำเร็จ: ' + e, true); }
  }

  /* ---------- QR scan ---------- */
  var html5QrCode = null;
  var qrTargetId = null;

  function openQrScan(targetId){
    qrTargetId = targetId;
    document.getElementById('qrPanel').style.display = 'block';
    document.getElementById('qrPanel').scrollIntoView({behavior:'smooth'});
    html5QrCode = new Html5Qrcode('qr-reader');
    Html5Qrcode.getCameras().then(function(cameras){
      if(cameras && cameras.length){
        html5QrCode.start(cameras[0].id, { fps:10, qrbox:220 }, function(decodedText){
          document.getElementById(qrTargetId).value = decodedText.trim();
          closeQrScan();
        }, function(){}).catch(function(err){ alert('ไม่สามารถเปิดกล้องได้: ' + err); closeQrScan(); });
      } else { alert('ไม่พบกล้องในอุปกรณ์นี้'); closeQrScan(); }
    }).catch(function(err){ alert('ไม่สามารถเข้าถึงกล้องได้: ' + err); closeQrScan(); });
  }
  function closeQrScan(){
    document.getElementById('qrPanel').style.display = 'none';
    if(html5QrCode){
      html5QrCode.stop().then(function(){ html5QrCode.clear(); html5QrCode = null; }).catch(function(){ html5QrCode = null; });
    }
  }

  /* ---------- Face scan ---------- */
  const FACE_MODEL_URL = 'https://cdn.jsdelivr.net/gh/justadudewhohacks/face-api.js@master/weights';
  const FACE_MATCH_THRESHOLD = 0.55;
  let modelsLoaded = false;
  let faceStream = null;
  let faceMode = null;

  async function ensureFaceModels(){
    if(modelsLoaded) return;
    await faceapi.nets.tinyFaceDetector.loadFromUri(FACE_MODEL_URL);
    await faceapi.nets.faceLandmark68Net.loadFromUri(FACE_MODEL_URL);
    await faceapi.nets.faceRecognitionNet.loadFromUri(FACE_MODEL_URL);
    modelsLoaded = true;
  }

  async function openFaceScan(mode){
    if(mode === 'enroll' && !document.getElementById('enrollName').value.trim()){
      alert('กรุณากรอกชื่อพนักงานก่อนเปิดกล้อง'); return;
    }
    faceMode = mode;
    document.getElementById('faceTitle').textContent =
      mode==='enroll' ? 'ลงทะเบียนใบหน้าพนักงาน' : (mode==='checkout' ? 'สแกนใบหน้าผู้เบิก' : 'สแกนใบหน้าผู้คืน');
    document.getElementById('facePanel').style.display = 'block';
    document.getElementById('facePanel').scrollIntoView({behavior:'smooth'});
    document.getElementById('faceMsg').className = 'msg';

    try{ await ensureFaceModels(); }
    catch(e){ showMsg('faceMsg', 'โหลดโมเดลจดจำใบหน้าไม่สำเร็จ (ต้องใช้อินเทอร์เน็ต): ' + e, true); return; }

    try{
      faceStream = await navigator.mediaDevices.getUserMedia({ video: true });
      document.getElementById('faceVideo').srcObject = faceStream;
    }catch(e){
      showMsg('faceMsg', 'ไม่สามารถเปิดกล้องได้: ' + e, true);
    }
  }

  function closeFaceScan(){
    document.getElementById('facePanel').style.display = 'none';
    if(faceStream){ faceStream.getTracks().forEach(function(t){ t.stop(); }); faceStream = null; }
  }

  function euclideanDistance(a, b){
    let sum = 0;
    for(let i=0; i<a.length; i++){ sum += Math.pow(a[i]-b[i], 2); }
    return Math.sqrt(sum);
  }

  async function captureFace(){
    var video = document.getElementById('faceVideo');
    showMsg('faceMsg', 'กำลังตรวจจับใบหน้า...', false);

    let detection;
    try{
      detection = await faceapi.detectSingleFace(video, new faceapi.TinyFaceDetectorOptions())
        .withFaceLandmarks().withFaceDescriptor();
    }catch(e){ showMsg('faceMsg', 'เกิดข้อผิดพลาดขณะประมวลผลใบหน้า: ' + e, true); return; }

    if(!detection){
      showMsg('faceMsg', 'ไม่พบใบหน้าในภาพ กรุณาลองใหม่ (แสงเพียงพอ มองกล้องตรง ๆ)', true);
      return;
    }
    var descriptor = Array.from(detection.descriptor);

    if(faceMode === 'enroll'){
      var name = document.getElementById('enrollName').value.trim();
      try{
        var res = await apiPost('addEmployeeFace', { name:name, descriptor:descriptor });
        if(res.ok){
          showMsg('faceMsg', 'ลงทะเบียนใบหน้าของ ' + name + ' สำเร็จ (' + res.data.employeeId + ')', false);
          document.getElementById('enrollName').value = '';
          setTimeout(closeFaceScan, 1200);
        } else { showMsg('faceMsg', res.error || 'ลงทะเบียนไม่สำเร็จ', true); }
      }catch(e){ showMsg('faceMsg', 'เชื่อมต่อระบบไม่สำเร็จ: ' + e, true); }
      return;
    }

    try{
      var empRes = await apiGet('getEmployees');
      if(!empRes.ok || !empRes.data.length){
        showMsg('faceMsg', 'ยังไม่มีข้อมูลใบหน้าพนักงานในระบบ กรุณาลงทะเบียนที่แท็บ "ลงทะเบียนใบหน้า" ก่อน', true);
        return;
      }
      var best = null, bestDist = 999;
      empRes.data.forEach(function(emp){
        try{
          var stored = JSON.parse(emp.FaceDescriptor);
          var dist = euclideanDistance(descriptor, stored);
          if(dist < bestDist){ bestDist = dist; best = emp; }
        }catch(e){}
      });
      if(best && bestDist < FACE_MATCH_THRESHOLD){
        if(faceMode === 'checkout'){ document.getElementById('coEmployee').value = best.Name; }
        else if(faceMode === 'return'){ document.getElementById('rtReturnedBy').value = best.Name; }
        showMsg('faceMsg', 'จดจำใบหน้าสำเร็จ: ' + best.Name, false);
        setTimeout(closeFaceScan, 800);
      } else {
        showMsg('faceMsg', 'ไม่พบใบหน้าที่ตรงกับฐานข้อมูล กรุณาลองใหม่ หรือลงทะเบียนใบหน้าก่อน', true);
      }
    }catch(e){ showMsg('faceMsg', 'เชื่อมต่อระบบไม่สำเร็จ: ' + e, true); }
  }
</script>

</body>
</html>
