# Lighting-sim
This is a website lighting 
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Stage Light Lab — Pro Advanced Edition</title>

  <style>
    * { box-sizing: border-box; }
    body {
      margin: 0;
      background: #070b14;
      color: #e2e8f5;
      font-family: system-ui, sans-serif;
    }
    button, select, input { font: inherit; }
    button, select {
      background: #1c2638;
      color: #eef3ff;
      border: 1px solid #30415d;
      border-radius: 6px;
      padding: 7px 10px;
    }
    button { cursor: pointer; }
    button:hover { background: #2a3b57; }
    button.primary { background: #2355cc; }
    input[type="range"] { width: 100%; accent-color: #639aff; }
    input[type="color"] {
      width: 100%; height: 36px; border: 0; padding: 0;
      background: transparent; cursor: pointer;
    }
    input[type="text"], input[type="number"] {
      background: #0d131f; color: white;
      border: 1px solid #30415d; border-radius: 6px;
      padding: 6px 8px; font-size: 13px;
    }
    input[type="number"] { width: 75px; }
    header {
      padding: 10px 16px; border-bottom: 1px solid #1f2d42;
      display: flex; align-items: center;
      justify-content: space-between; gap: 12px;
      background: #0e1522;
    }
    header b { color: #8ab4f8; }
    header small { color: #9aa8bc; }
    .layout {
      display: grid;
      grid-template-columns: 340px minmax(0, 1fr);
      height: calc(100dvh - 49px);
    }
    aside {
      overflow-y: auto; padding: 14px;
      background: #101724; border-right: 1px solid #1f2d42;
    }
    section { margin-bottom: 18px; }
    h2 {
      margin: 0 0 8px; font-size: 14px; color: #9bbdff;
      border-bottom: 1px solid #1f2d42; padding-bottom: 4px;
    }
    .row {
      display: flex; gap: 6px;
      align-items: center; margin: 6px 0;
      flex-wrap: wrap;
    }
    .row > select { flex: 1; min-width: 0; }
    .field { display: block; margin: 8px 0; font-size: 12px; }
    .field span {
      display: flex; justify-content: space-between; gap: 10px;
      color: #b8c5db; margin-bottom: 3px;
    }
    output { color: #7ab3ff; }
    .note {
      font-size: 11px; line-height: 1.5; color: #8898b0;
    }
    #view { position: relative; min-height: 360px; overflow: hidden; }
    #view canvas { display: block; width: 100%; height: 100%; }
    .overlay {
      position: absolute; z-index: 1;
      pointer-events: none; padding: 8px 10px;
      background: #0e1522cc; border: 1px solid #2a3d59;
      border-radius: 6px; font-size: 11px;
    }
    #hint { top: 10px; left: 10px; }
    #status { bottom: 10px; left: 10px; right: 10px; }
    audio { width: 100%; height: 34px; margin-top: 6px; }
    .cue {
      display: flex; gap: 6px; align-items: center;
      padding: 5px 0; border-bottom: 1px solid #1a2638;
      font-size: 11px;
    }
    .cue span { flex: 1; }
    .cue button { padding: 4px 6px; font-size: 11px; }
    #editor[hidden] { display: none; }
    @media (max-width: 760px) {
      .layout { display: flex; flex-direction: column; height: auto; }
      #view { order: -1; height: 50dvh; min-height: 300px; }
      aside { overflow: visible; border-right: 0; }
    }
  </style>

  <script type="importmap">
    {
      "imports": {
        "three": "https://cdn.jsdelivr.net/npm/three@0.170.0/build/three.module.js",
        "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.170.0/examples/jsm/"
      }
    }
  </script>
</head>

<body>
  <header>
    <b>Stage Light Lab — Pro Advanced</b>
    <small>โคมสมจริง • ลิงก์เพลง • Smooth Fade</small>
  </header>

  <div class="layout">
    <aside>
      <section>
        <h2>โปรเจกต์</h2>
        <div class="row">
          <button id="saveProject">บันทึก</button>
          <button id="loadProject">โหลด</button>
          <button id="resetCamera">มุมกล้อง</button>
        </div>
      </section>

      <section>
        <h2>เพิ่มอุปกรณ์ (ไม่จำกัด)</h2>
        <div class="row">
          <select id="fixtureType">
            <option value="beam">Moving Head Beam (คมชัด พุ่งไกล)</option>
            <option value="spot">Moving Head Spot (มี Gobo ลาย)</option>
            <option value="wash">Moving Head Wash (ฟุ้ง นุ่มนวล)</option>
            <option value="profile">Moving Head Profile (มีใบตัด Shutter)</option>
            <option value="par">LED PAR (สาดผนัง/เวที)</option>
            <option value="blinder">Audience Blinder (สาดผู้ชม วอร์มไวท์)</option>
            <option value="strobe">LED Strobe (กระพริบแรงสูง)</option>
          </select>
          <button id="addFixture" class="primary">เพิ่ม</button>
        </div>
        <div class="row">
          <select id="fixtureList"></select>
          <button id="removeFixture">ลบ</button>
        </div>
      </section>

      <section id="editor" hidden>
        <h2>ตั้งค่าโคมที่เลือก</h2>
        <label class="field">
          <span>สีของไฟ (Color)</span>
          <input id="color" type="color">
        </label>
        <div id="sliders"></div>
        <label class="field">
          <input id="enabled" type="checkbox" checked> เปิดโคม (Power/Dimmer ON)
        </label>
      </section>

      <section>
        <h2>บรรยากาศเวที</h2>
        <label class="field">
          <span>หมอกจำลอง (Haze) <output id="hazeValue">35%</output></span>
          <input id="haze" type="range" min="0" max="100" value="35">
        </label>
        <label class="field">
          <input id="sweep" type="checkbox"> ส่าย Moving Head อัตโนมัติ
        </label>
        <label class="field">
          <input id="soundReactive" type="checkbox"> ความสว่างกระพริบตามเสียงเบส
        </label>
      </section>

      <section>
        <h2>แหล่งเพลง (ไฟล์ หรือ ลิงก์)</h2>
        <input id="musicFile" type="file" accept="audio/*">
        <div class="row" style="margin-top:6px;">
          <input id="musicUrlInput" type="text" placeholder="วางลิงก์ไฟล์เสียง (.mp3)" style="flex:1;">
          <button id="loadUrlBtn">โหลดลิงก์</button>
        </div>
        <audio id="audio" controls crossorigin="anonymous"></audio>
        <div id="musicName" class="note">ยังไม่ได้เลือกเพลง</div>
      </section>

      <section>
        <h2>Lighting Cues & Fade</h2>
        <div class="row">
          <span>เวลา(s):</span>
          <input id="cueTime" type="number" min="0" step="0.1" value="0">
          <span>Fade(s):</span>
          <input id="cueFade" type="number" min="0" step="0.1" value="1.5">
        </div>
        <div class="row">
          <button id="useCurrentTime">เวลาปัจจุบัน</button>
          <button id="addCue" class="primary">บันทึกคิว</button>
        </div>
        <div class="row" style="margin-top:8px;">
          <button id="playShow" class="primary">▶ เล่นโชว์อัตโนมัติ</button>
          <button id="stopShow">หยุด</button>
        </div>
        <div id="cueList" style="margin-top:8px;"></div>
      </section>
    </aside>

    <main id="view">
      <div class="overlay" id="hint">
        ลากซ้าย: หมุน • ลากขวา: เลื่อน • สกิล: ซูม
      </div>
      <div class="overlay" id="status">
        กำลังเริ่มระบบ 3D…
      </div>
    </main>
  </div>

  <script type="module">
    import * as THREE from "three";
    import { OrbitControls } from "three/addons/controls/OrbitControls.js";

    const $ = id => document.getElementById(id);
    const status = text => { $("status").textContent = text; };
    const clone = value => JSON.parse(JSON.stringify(value));
    const rad = THREE.MathUtils.degToRad;
    const lerp = THREE.MathUtils.lerp;
    const STORAGE_KEY = "stage-light-pro-v1";

    // รายละเอียดประเภทไฟและค่าควบคุมสมจริง
    const fixtureTypes = {
      beam: { label: "Beam", angle: 4, intensity: 90, range: 40 },
      spot: { label: "Spot", angle: 15, intensity: 80, gobo: true, range: 35 },
      wash: { label: "Wash", angle: 40, intensity: 70, range: 30 },
      profile: { label: "Profile", angle: 18, intensity: 80, shutters: true, range: 35 },
      par: { label: "PAR", angle: 30, intensity: 65, range: 25 },
      blinder: { label: "Blinder", angle: 60, intensity: 100, warm: true, range: 20 },
      strobe: { label: "Strobe", angle: 70, intensity: 100, strobeOnly: true, range: 25 }
    };

    const baseFields = [
      ["x", "ตำแหน่ง X", -12, 12, 0.1, " m"],
      ["y", "ความสูง Y", 1, 12, 0.1, " m"],
      ["z", "ตำแหน่ง Z", -8, 8, 0.1, " m"],
      ["pan", "Pan (มุมแนวนอน)", -180, 180, 1, "°"],
      ["tilt", "Tilt (มุมก้มเงย)", 0, 85, 1, "°"],
      ["zoom", "Zoom (มุมกระจายแสง)", 2, 60, 1, "°"],
      ["intensity", "Dimmer (ความสว่าง)", 0, 100, 1, "%"]
    ];

    function updateSlidersLayout(type) {
      let currentFields = [...baseFields];
      if (type === "profile") {
        currentFields.push(["shutter", "Shutter / ใบตัดแสง", 0, 100, 1, "%"]);
      }
      if (type === "spot") {
        currentFields.push(["gobo", "Gobo (เลือกลาย: 0=เปิด, 1=จุด, 2=วง)", 0, 2, 1, ""]);
      }
      $("sliders").innerHTML = currentFields.map(
        ([key, label, min, max, step, unit]) => `
          <label class="field">
            <span>${label} <output id="${key}Value"></output></span>
            <input id="${key}" type="range" min="${min}" max="${max}" step="${step}">
          </label>
        `
      ).join("");
    }

    const view = $("view");
    const scene = new THREE.Scene();
    scene.background = new THREE.Color("#05080f");
    scene.fog = new THREE.FogExp2("#060a12", 0.016);

    const camera = new THREE.PerspectiveCamera(45, 1, 0.1, 150);
    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 1.7));
    renderer.outputColorSpace = THREE.SRGBColorSpace;
    view.appendChild(renderer.domElement);

    const controls = new OrbitControls(camera, renderer.domElement);
    controls.enableDamping = true;
    controls.maxPolarAngle = Math.PI / 2 - 0.01;
    controls.minDistance = 3;
    controls.maxDistance = 60;

    function resetCamera() {
      camera.position.set(16, 10, 20);
      controls.target.set(0, 2.5, 0);
      controls.update();
    }
    resetCamera();
    $("resetCamera").onclick = resetCamera;

    new ResizeObserver(() => {
      const w = view.clientWidth;
      const h = view.clientHeight;
      if (!w || !h) return;
      camera.aspect = w / h;
      camera.updateProjectionMatrix();
      renderer.setSize(w, h);
    }).observe(view);

    scene.add(new THREE.HemisphereLight(0x8cb0e8, 0x15101f, 0.5));

    // โครงสร้างเวทีหลัก
    function box(w, h, d, color, x, y, z) {
      const mesh = new THREE.Mesh(
        new THREE.BoxGeometry(w, h, d),
        new THREE.MeshStandardMaterial({ color, roughness: 0.8, metalness: 0.2 })
      );
      mesh.position.set(x, y, z);
      scene.add(mesh);
      return mesh;
    }

    box(45, 0.2, 40, 0x0c1017, 0, -0.12, 0);
    box(14, 0.6, 8, 0x28303d, 0, 0.3, 0);
    box(14, 6, 0.18, 0x080a0f, 0, 3.6, -4.2);

    const grid = new THREE.GridHelper(14, 14, 0x3d5478, 0x1a2536);
    grid.position.y = 0.605;
    scene.add(grid);

    for (const x of [-7.5, 7.5]) {
      box(1, 2.8, 1.1, 0x121722, x, 1.4, 2);
    }

    const trussMat = new THREE.MeshStandardMaterial({
      color: 0x7a879e, metalness: 0.85, roughness: 0.3
    });

    function tube(a, b, radius = 0.045) {
      const start = new THREE.Vector3(...a);
      const end = new THREE.Vector3(...b);
      const delta = end.clone().sub(start);
      const mesh = new THREE.Mesh(
        new THREE.CylinderGeometry(radius, radius, delta.length(), 8),
        trussMat
      );
      mesh.position.copy(start).add(end).multiplyScalar(0.5);
      mesh.quaternion.setFromUnitVectors(new THREE.Vector3(0, 1, 0), delta.normalize());
      scene.add(mesh);
    }

    for (const z of [-2.8, 2.8]) {
      for (const x of [-7, 7]) {
        tube([x, 0.6, z], [x, 7, z], 0.09);
      }
      tube([-7, 7, z], [7, 7, z]);
      tube([-7, 7.35, z], [7, 7.35, z]);
    }

    let fixtures = [];
    let selectedId = null;
    let nextId = 1;
    let cues = [];
    let isPlayingShow = false;
    let showTimer = 0;

    const bodyGeometry = new THREE.BoxGeometry(0.42, 0.3, 0.38);
    const headGeometry = new THREE.CylinderGeometry(0.18, 0.22, 0.4, 16);
    const lensGeometry = new THREE.CircleGeometry(0.155, 20);
    const beamGeometry = new THREE.ConeGeometry(1, 1, 32, 1, true);
    beamGeometry.translate(0, -0.5, 0);

    const down = new THREE.Vector3(0, -1, 0);
    const direction = new THREE.Vector3();

    function selected() {
      return fixtures.find(f => f.cfg.id === selectedId);
    }

    function createFixture(config = null, type = "beam") {
      const p = fixtureTypes[type];
      const cfg = config ? clone(config) : {
        id: nextId++,
        type,
        x: ((fixtures.length % 6) - 2.5) * 1.5,
        y: 6.8,
        z: fixtures.length % 2 === 0 ? -2.8 : 2.8,
        pan: 0,
        tilt: 25,
        zoom: p.angle,
        intensity: p.intensity,
        color: type === "blinder" ? "#ffaa55" : "#4488ff",
        enabled: true,
        shutter: 0,
        gobo: 0
      };
      nextId = Math.max(nextId, cfg.id + 1);

      const root = new THREE.Group();
      root.userData.fixtureId = cfg.id;

      const bodyMat = new THREE.MeshStandardMaterial({
        color: 0x1e2430, metalness: 0.6, roughness: 0.4
      });
      const body = new THREE.Mesh(bodyGeometry, bodyMat);
      body.position.y = 0.16;
      root.add(body);

      const pivot = new THREE.Group();
      root.add(pivot);

      const head = new THREE.Mesh(headGeometry, bodyMat);
      head.position.y = -0.18;
      pivot.add(head);

      const lensMat = new THREE.MeshBasicMaterial({ color: cfg.color });
      const lens = new THREE.Mesh(lensGeometry, lensMat);
      lens.rotation.x = Math.PI / 2;
      lens.position.y = -0.385;
      pivot.add(lens);

      const beamMat = new THREE.MeshBasicMaterial({
        color: cfg.color,
        transparent: true,
        opacity: 0.08,
        depthWrite: false,
        side: THREE.DoubleSide,
        blending: THREE.AdditiveBlending
      });
      const beam = new THREE.Mesh(beamGeometry, beamMat);
      beam.position.y = -0.39;
      beam.renderOrder = 2;
      beam.raycast = () => {};
      pivot.add(beam);

      const light = new THREE.SpotLight(cfg.color, 800, p.range, rad(cfg.angle / 2), 0.3, 2);
      light.castShadow = false;
      scene.add(root, light, light.target);

      fixtures.push({ cfg, root, pivot, beam, light, bodyMat, lensMat });
      selectedId = cfg.id;
      refreshFixtureList();
      return cfg;
    }

    function destroyFixture(f) {
      scene.remove(f.root, f.light, f.light.target);
      f.bodyMat.dispose();
      f.lensMat.dispose();
      f.beam.material.dispose();
      f.light.dispose();
    }

    function refreshFixtureList() {
      $("fixtureList").replaceChildren();
      for (const f of fixtures) {
        const option = document.createElement("option");
        option.value = f.cfg.id;
        option.textContent = `${fixtureTypes[f.cfg.type].label} #${f.cfg.id}`;
        $("fixtureList").appendChild(option);
      }
      $("fixtureList").value = selectedId ?? "";
      refreshEditor();
    }

    function refreshEditor() {
      const f = selected();
      $("editor").hidden = !f;
      if (!f) return;
      updateSlidersLayout(f.cfg.type);

      const activeFields = [...baseFields];
      if (f.cfg.type === "profile") activeFields.push(["shutter", "Shutter / ใบตัดแสง", 0, 100, 1, "%"]);
      if (f.cfg.type === "spot") activeFields.push(["gobo", "Gobo", 0, 2, 1, ""]);

      for (const [key] of activeFields) {
        if ($(key)) {
          $(key).value = f.cfg[key];
          $(key + "Value").textContent = f.cfg[key] + (key.match(/pan|tilt|zoom/) ? "°" : key === "intensity" || key === "shutter" ? "%" : key === "gobo" ? "" : " m");
        }
      }
      $("color").value = f.cfg.color;
      $("enabled").checked = f.cfg.enabled;

      // ผูก Event ให้สไลเดอร์ของโคมที่เลือก
      for (const [key] of activeFields) {
        if ($(key)) {
          $(key).oninput = () => {
            const current = selected();
            if (!current) return;
            current.cfg[key] = Number($(key).value);
            $(key + "Value").textContent = current.cfg[key] + (key.match(/pan|tilt|zoom/) ? "°" : key === "intensity" || key === "shutter" ? "%" : key === "gobo" ? "" : " m");
          };
        }
      }
    }

    $("color").oninput = () => { if (selected()) selected().cfg.color = $("color").value; };
    $("enabled").onchange = () => { if (selected()) selected().cfg.enabled = $("enabled").checked; };
    $("fixtureList").onchange = () => {
      selectedId = Number($("fixtureList").value);
      refreshEditor();
    };
    $("addFixture").onclick = () => { createFixture(null, $("fixtureType").value); };
    $("removeFixture").onclick = () => {
      const f = selected();
      if (!f) return;
      destroyFixture(f);
      fixtures = fixtures.filter(item => item !== f);
      selectedId = fixtures[0]?.cfg.id ?? null;
      refreshFixtureList();
    };

    const raycaster = new THREE.Raycaster();
    const pointer = new THREE.Vector2();
    let pointerStart = null;

    renderer.domElement.addEventListener("pointerdown", e => {
      pointerStart = { x: e.clientX, y: e.clientY, button: e.button };
    });

    renderer.domElement.addEventListener("pointerup", e => {
      if (!pointerStart || pointerStart.button !== 0) return;
      if (Math.hypot(e.clientX - pointerStart.x, e.clientY - pointerStart.y) > 5) return;

      const rect = renderer.domElement.getBoundingClientRect();
      pointer.set(
        ((e.clientX - rect.left) / rect.width) * 2 - 1,
        -((e.clientY - rect.top) / rect.height) * 2 + 1
      );
      raycaster.setFromCamera(pointer, camera);
      const hits = raycaster.intersectObjects(fixtures.map(f => f.root), true);
      if (!hits.length) return;

      let object = hits[0].object;
      while (object && object.userData.fixtureId === undefined) {
        object = object.parent;
      }
      if (object) {
        selectedId = object.userData.fixtureId;
        refreshFixtureList();
      }
    });

    function updateHaze() {
      const value = Number($("haze").value);
      $("hazeValue").textContent = value + "%";
      scene.fog.density = 0.005 + value * 0.0003;
    }
    $("haze").oninput = updateHaze;
    updateHaze();

    // ระบบจัดการเสียง (รองรับไฟล์และลิงก์ URL)
    const audio = $("audio");
    let audioContext, analyser, audioSource, frequencyData;
    let bassLevel = 0;

    async function setupAudio() {
      try {
        if (!audioContext) {
          const AudioContextClass = window.AudioContext || window.webkitAudioContext;
          audioContext = new AudioContextClass();
          analyser = audioContext.createAnalyser();
          analyser.fftSize = 1024;
          analyser.smoothingTimeConstant = 0.78;
          audioSource = audioContext.createMediaElementSource(audio);
          audioSource.connect(analyser);
          analyser.connect(audioContext.destination);
          frequencyData = new Uint8Array(analyser.frequencyBinCount);
        }
        if (audioContext.state === "suspended") await audioContext.resume();
      } catch (error) { console.error("Audio context error:", error); }
    }

    $("musicFile").onchange = async event => {
      const file = event.target.files?.[0];
      if (!file) return;
      audio.src = URL.createObjectURL(file);
      audio.load();
      $("musicName").textContent = "ไฟล์: " + file.name;
      await setupAudio();
    };

    $("loadUrlBtn").onclick = async () => {
      const url = $("musicUrlInput").value.trim();
      if (!url) {
        alert("กรุณากรอกลิงก์ไฟล์เสียง (.mp3)");
        return;
      }
      audio.src = url;
      audio.load();
      $("musicName").textContent = "ลิงก์: " + url;
      await setupAudio();
      status("โหลดเพลงจากลิงก์สำเร็จ กด Play ได้เลย");
    };

    audio.addEventListener("play", setupAudio);

    function readBass() {
      let target = 0;
      if (analyser && !audio.paused && !audio.ended) {
        analyser.getByteFrequencyData(frequencyData);
        const hzPerBin = audioContext.sampleRate / analyser.fftSize;
        const first = Math.max(1, Math.floor(40 / hzPerBin));
        const last = Math.min(frequencyData.length - 1, Math.ceil(180 / hzPerBin));
        let total = 0;
        for (let i = first; i <= last; i++) total += frequencyData[i];
        target = total / ((last - first + 1) * 255);
      }
      bassLevel += (target - bassLevel) * 0.18;
    }

    function captureState() {
      return fixtures.map(f => ({
        id: f.cfg.id,
        values: {
          x: f.cfg.x, y: f.cfg.y, z: f.cfg.z,
          pan: f.cfg.pan, tilt: f.cfg.tilt,
          zoom: f.cfg.zoom, intensity: f.cfg.intensity,
          shutter: f.cfg.shutter, gobo: f.cfg.gobo
        },
        color: f.cfg.color,
        enabled: f.cfg.enabled
      }));
    }

    function applyStateInstant(state) {
      state.forEach(saved => {
        const f = fixtures.find(item => item.cfg.id === saved.id);
        if (f) {
          Object.assign(f.cfg, clone(saved.values));
          f.cfg.color = saved.color;
          f.cfg.enabled = saved.enabled;
        }
      });
      refreshEditor();
    }

    function renderCues() {
      const list = $("cueList");
      list.replaceChildren();
      cues.sort((a, b) => a.time - b.time);

      cues.forEach((cue, index) => {
        const row = document.createElement("div");
        row.className = "cue";
        const label = document.createElement("span");
        label.textContent = `คิว ${index + 1} (${cue.time}s, Fade ${cue.fade}s)`;

        const apply = document.createElement("button");
        apply.textContent = "ใช้";
        apply.onclick = () => {
          isPlayingShow = false;
          if (audio.src) audio.pause();
          applyStateInstant(cue.state);
        };

        const remove = document.createElement("button");
        remove.textContent = "ลบ";
        remove.onclick = () => {
          cues.splice(index, 1);
          renderCues();
        };

        row.append(label, apply, remove);
        list.appendChild(row);
      });
    }

    $("useCurrentTime").onclick = () => {
      const t = (audio.src && !audio.paused) ? audio.currentTime : showTimer;
      $("cueTime").value = t.toFixed(1);
    };

    $("addCue").onclick = () => {
      const time = Number($("cueTime").value);
      const fade = Number($("cueFade").value);
      const cue = { time, fade, state: captureState() };
      const existing = cues.findIndex(c => c.time === time);
      if (existing >= 0) cues[existing] = cue;
      else cues.push(cue);
      renderCues();
      status(`บันทึกคิวที่ ${time}s เรียบร้อย`);
    };

    $("playShow").onclick = () => {
      if (cues.length === 0) {
        status("กรุณาบันทึกคิวก่อนเล่นโชว์");
        return;
      }
      isPlayingShow = true;
      showTimer = 0;
      if (audio.src) {
        audio.currentTime = 0;
        audio.play().catch(() => {});
      }
      status("▶ กำลังรันโชว์แสงตามคิว...");
    };

    $("stopShow").onclick = () => {
      isPlayingShow = false;
      if (audio.src) audio.pause();
      status("หยุดโชว์แล้ว");
    };

    $("saveProject").onclick = () => {
      localStorage.setItem(STORAGE_KEY, JSON.stringify({ cues }));
      status("บันทึกข้อมูลคิวลงเบราว์เซอร์แล้ว");
    };

    $("loadProject").onclick = () => {
      const raw = localStorage.getItem(STORAGE_KEY);
      if (!raw) return;
      const project = JSON.parse(raw);
      cues = project.cues;
      renderCues();
      status("โหลดข้อมูลคิวสำเร็จ");
    };

    function getInterpolatedState(currentTime) {
      if (cues.length === 0) return null;
      cues.sort((a, b) => a.time - b.time);
      if (currentTime <= cues[0].time) return cues[0].state;

      for (let i = 0; i < cues.length - 1; i++) {
        const cur = cues[i];
        const nxt = cues[i + 1];
        if (currentTime >= cur.time && currentTime < nxt.time) {
          const startFade = nxt.time - nxt.fade;
          if (currentTime < startFade) return cur.state;
          const progress = THREE.MathUtils.clamp((currentTime - startFade) / (nxt.time - startFade), 0, 1);

          const blended = clone(cur.state);
          blended.forEach(item => {
            const target = nxt.state.find(s => s.id === item.id);
            if (target) {
              item.enabled = progress < 0.5 ? item.enabled : target.enabled;
              const c1 = new THREE.Color(item.color);
              const c2 = new THREE.Color(target.color);
              c1.lerp(c2, progress);
              item.color = "#" + c1.getHexString();

              for (let k in item.values) {
                if (target.values[k] !== undefined) {
                  item.values[k] = lerp(item.values[k], target.values[k], progress);
                }
              }
            }
          });
          return blended;
        }
      }
      return cues[cues.length - 1].state;
    }

    // สร้างไฟเริ่มต้น 4 ประเภทสมจริง
    createFixture(null, "beam");
    createFixture(null, "spot");
    createFixture(null, "wash");
    createFixture(null, "profile");
    refreshEditor();

    const clock = new THREE.Clock();

    function animate() {
      const dt = clock.getDelta();
      readBass();

      if (isPlayingShow) {
        if (audio.src && !audio.paused) {
          showTimer = audio.currentTime;
        } else {
          showTimer += dt;
        }
        const activeState = getInterpolatedState(showTimer);
        if (activeState) {
          activeState.forEach(saved => {
            const f = fixtures.find(item => item.cfg.id === saved.id);
            if (f) {
              f.cfg.x = saved.values.x;
              f.cfg.y = saved.values.y;
              f.cfg.z = saved.values.z;
              f.cfg.pan = saved.values.pan;
              f.cfg.tilt = saved.values.tilt;
              f.cfg.zoom = saved.values.zoom;
              f.cfg.intensity = saved.values.intensity;
              if (saved.values.shutter !== undefined) f.cfg.shutter = saved.values.shutter;
              if (saved.values.gobo !== undefined) f.cfg.gobo = saved.values.gobo;
              f.cfg.color = saved.color;
              f.cfg.enabled = saved.enabled;
            }
          });
        }
      }

      const haze = Number($("haze").value) / 100;
      const reactive = $("soundReactive").checked;
      const motionTime = (audio.src && !audio.paused) ? audio.currentTime : clock.getElapsedTime();

      for (let i = 0; i < fixtures.length; i++) {
        const f = fixtures[i];
        const c = f.cfg;
        const pType = fixtureTypes[c.type];

        let pan = c.pan;
        let tilt = c.tilt;

        if ($("sweep").checked && c.type !== "par" && c.type !== "blinder") {
          pan += Math.sin(motionTime * 0.8 + i * 0.6) * 35;
          tilt += Math.sin(motionTime * 1.0 + i) * 10;
        }
        tilt = THREE.MathUtils.clamp(tilt, 0, 85);

        const p = rad(pan);
        const t = rad(tilt);
        direction.set(
          Math.sin(p) * Math.sin(t),
          -Math.cos(t),
          Math.cos(p) * Math.sin(t)
        ).normalize();

        f.root.position.set(c.x, c.y, c.z);
        f.pivot.quaternion.setFromUnitVectors(down, direction);

        f.light.position.copy(f.root.position).addScaledVector(direction, 0.39);
        f.light.target.position.copy(f.light.position).addScaledVector(direction, 30);

        const bassBoost = reactive ? 0.2 + bassLevel * 1.4 : 1;
        let dimmer = c.enabled ? (c.intensity / 100) * bassBoost : 0;
        
        // จำลองใบตัด Shutter ของ Profile
        if (c.type === "profile" && c.shutter > 0) {
          dimmer *= (1 - c.shutter / 120);
        }

        const halfAngle = rad(c.zoom / 2);
        f.light.color.set(c.color);
        f.light.angle = halfAngle;
        f.light.intensity = dimmer * pType.intensity * 12;

        const distanceToFloor = (f.light.position.y - 0.61) / Math.max(0.08, -direction.y);
        const length = THREE.MathUtils.clamp(distanceToFloor, 0.1, 30);
        const radius = Math.tan(halfAngle) * length;

        f.beam.scale.set(radius, length, radius);
        f.beam.material.color.set(c.color);
        f.beam.material.opacity = Math.min(0.28, haze * dimmer * (c.type === "beam" ? 0.28 : 0.12));
        f.beam.visible = dimmer > 0.001 && haze > 0;

        f.lensMat.color.set(c.color).multiplyScalar(0.1 + Math.min(dimmer, 1) * 2);
        f.bodyMat.emissive.setHex(c.id === selectedId ? 0x1f4e8c : 0x000000);
      }

      controls.update();
    
      renderer.render(scene, camera);
    }

    renderer.setAnimationLoop(animate);
    status("พร้อมใช้งาน — เลือกประเภทโคม วางลิงก์เพลง และกดเล่นโชว์ได้เลย");
  </script>
</body>
</html>
