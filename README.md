# IO
<!--
Plantilla de cálculo - Investigación de Operaciones I
Autor: Generado con ayuda de ChatGPT
Descripción / README (resumido):
- Archivo único HTML que contiene la interfaz (HTML), estilos (CSS) y la lógica (JS)
- Funcionalidades:
  * Ingresar función objetivo (coeficientes de variables x1..xn)
  * Agregar restricciones con signo (<=, >=, =)
  * Seleccionar Maximizar / Minimizar
  * Seleccionar método: Simplex, Dos Fases, Método de la M (Big-M)
  * Ejecutar y ver iteraciones en forma de tablas. Paginación de iteraciones.
  * Mostrar solución óptima al final.
- Instrucciones de uso rápidas:
  1) Especifica el número de variables y presiona 'Configurar variables'.
  2) Ingresa coeficientes de la función objetivo (separados por comas) o usar los inputs.
  3) Añade restricciones con sus coeficientes, sentido y término independiente.
  4) Elige Maximizar/Minimizar y el método.
  5) Presiona 'Resolver'.

Advertencia y alcance:
- Esta plantilla busca cubrir casos típicos de la materia y sirve como soporte de estudio.
- Implementa un solver de tableaux simplex con manejo de slack/surplus y variables artificiales.
- Está pensada para problemas de tamaño pequeño/mediano (pocas variables y restricciones).
- El alumno debe revisar y comprender la transformación a forma estándar; se recomienda validar casos extremos.

Entrega: Copia el contenido a un fichero HTML y súbelo a tu repositorio privado en GitHub.
-->

<!doctype html>
<html lang="es">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Plantilla de Programación Lineal - Simplex / Dos Fases / M</title>
  <style>
    :root{font-family:Inter,system-ui,Arial;margin:0;color:#111}
    body{margin:0;padding:20px;background:#0f172a;color:#e6eef8}
    .card{background:linear-gradient(180deg,#0b1220 0%,#071226 100%);border-radius:12px;padding:18px;box-shadow:0 6px 20px rgba(2,6,23,.6);max-width:1100px;margin:10px auto}
    h1{margin:0 0 8px;font-size:20px}
    .row{display:flex;gap:12px;flex-wrap:wrap}
    label{font-size:13px;color:#99a6bf}
    input,select,button{padding:8px;border-radius:8px;border:1px solid #1e293b;background:#071028;color:#e6eef8}
    input[type=number]{width:100px}
    textarea{width:100%;min-height:80px;border-radius:8px;padding:8px}
    .controls{display:flex;gap:8px;align-items:center;margin-top:8px}
    .constraints{margin-top:12px}
    .constraint{display:flex;gap:8px;align-items:center;margin-bottom:8px}
    .table-wrap{overflow:auto;margin-top:12px;background:#041025;padding:12px;border-radius:8px}
    table{border-collapse:collapse;width:100%}
    th,td{border:1px solid #10314b;padding:8px;text-align:center;font-family:monospace}
    .btn{cursor:pointer}
    .btn-primary{background:#0ea5a1;border:0;color:#002021}
    .btn-ghost{background:transparent;border:1px solid #123}
    .small{font-size:12px;padding:6px}
    .muted{color:#7b8aa3;font-size:13px}
    footer{margin-top:14px;color:#7b8aa3;font-size:13px}
    .steps-nav{display:flex;gap:6px;align-items:center;margin-top:8px}
  </style>
</head>
<body>
  <div class="card">
    <h1>Plantilla de Programación Lineal — Simplex / Dos Fases / Método de la M</h1>
    <div class="muted">Interfaz para ingresar función objetivo y restricciones. Muestra tableaux iterativos y solución óptima.</div>

    <hr style="border-color:#071a2a;margin:12px 0">

    <div class="row">
      <div>
        <label>Número de variables (n)</label><br>
        <input id="nVars" type="number" min="1" value="2" />
      </div>
      <div>
        <label>Número de restricciones (m)</label><br>
        <input id="nCons" type="number" min="1" value="2" />
      </div>
      <div style="align-self:end">
        <button id="setupBtn" class="btn btn-primary small">Configurar variables</button>
      </div>

      <div style="margin-left:auto;min-width:240px">
        <label>Maximizar / Minimizar</label><br>
        <select id="sense">
          <option value="max">Maximizar</option>
          <option value="min">Minimizar</option>
        </select>
      </div>

      <div style="align-self:end">
        <label>Método</label><br>
        <select id="method">
          <option value="simplex">Simplex</option>
          <option value="two-phase">Dos Fases</option>
          <option value="big-m">Método de la M (Big-M)</option>
        </select>
      </div>

    </div>

    <div id="problemArea" style="margin-top:12px;display:none">
      <h3 style="margin:8px 0">Función objetivo</h3>
      <div id="objInputs" class="row"></div>
      <div style="margin-top:8px">
        <label>Constantes opcionales (si quieres pegar coeficientes separados por comas):</label>
        <input id="objPaste" placeholder="ej: 3,5,0" />
        <button id="pasteObj" class="btn small btn-ghost">Pegar coefs</button>
      </div>

      <h3 style="margin-top:12px">Restricciones</h3>
      <div class="controls">
        <button id="addCons" class="btn small btn-primary">Añadir restricción</button>
        <button id="clearCons" class="btn small btn-ghost">Limpiar</button>
        <div class="muted" style="margin-left:8px">Forma: a1 x1 + a2 x2 + ... (<=, >=, = ) b</div>
      </div>
      <div id="constraints" class="constraints"></div>

      <div style="margin-top:12px">
        <button id="solveBtn" class="btn btn-primary">Resolver</button>
        <button id="exportBtn" class="btn small btn-ghost">Exportar solución (JSON)</button>
      </div>

      <div id="output" style="margin-top:12px"></div>
    </div>

    <footer>Guarda este archivo como <code>Plantilla-Simplex.html</code> y súbelo a tu repositorio privado. Puedes usar la lógica JS como apoyo y documentarla en el README.</footer>
  </div>

  <script>
    // --- Utilidades básicas ---
    const qs = (s)=>document.querySelector(s);
    const qsa = (s)=>document.querySelectorAll(s);

    const nVarsInput = qs('#nVars');
    const nConsInput = qs('#nCons');
    const setupBtn = qs('#setupBtn');
    const problemArea = qs('#problemArea');
    const objInputs = qs('#objInputs');
    const addConsBtn = qs('#addCons');
    const constraintsDiv = qs('#constraints');
    const solveBtn = qs('#solveBtn');
    const pasteObj = qs('#pasteObj');
    const objPaste = qs('#objPaste');
    const clearCons = qs('#clearCons');
    const exportBtn = qs('#exportBtn');

    let state = {n:2,m:2};

    function makeObjInputs(n){
      objInputs.innerHTML = '';
      for(let i=1;i<=n;i++){
        const wrap=document.createElement('div');wrap.style.display='flex';wrap.style.flexDirection='column';
        const label=document.createElement('label');label.textContent='x'+i;
        const inp=document.createElement('input');inp.type='number';inp.step='any';inp.value='0';inp.dataset.idx=i-1;
        wrap.appendChild(label);wrap.appendChild(inp);
        objInputs.appendChild(wrap);
      }
    }

    function addConstraintRow(coefs){
      const row=document.createElement('div');row.className='constraint';
      for(let i=0;i<state.n;i++){
        const inp=document.createElement('input');inp.type='number';inp.step='any';inp.value=(coefs?.[i]??0);inp.style.width='70px';inp.dataset.idx=i;
        row.appendChild(inp);
      }
      const sel=document.createElement('select');['<=','>=','='].forEach(s=>{const o=document.createElement('option');o.value=s;o.textContent=s;sel.appendChild(o)});
      const rhs=document.createElement('input');rhs.type='number';rhs.step='any';rhs.value=(coefs?.[state.n]??0);rhs.style.width='90px';
      const del=document.createElement('button');del.textContent='✖';del.className='btn small btn-ghost';del.onclick=()=>row.remove();
      row.appendChild(sel);row.appendChild(rhs);row.appendChild(del);
      constraintsDiv.appendChild(row);
    }

    setupBtn.addEventListener('click',()=>{
      const n=parseInt(nVarsInput.value)||1;
      const m=parseInt(nConsInput.value)||1;
      state.n=n;state.m=m;problemArea.style.display='block';
      makeObjInputs(n);
      constraintsDiv.innerHTML='';
      for(let i=0;i<m;i++)addConstraintRow();
    });

    addConsBtn.addEventListener('click',()=>addConstraintRow());
    clearCons.addEventListener('click',()=>constraintsDiv.innerHTML='');

    pasteObj.addEventListener('click',()=>{
      const arr=(objPaste.value||'').split(',').map(x=>parseFloat(x.trim())).filter(x=>!isNaN(x));
      for(let i=0;i<state.n;i++){
        const inp=objInputs.querySelector('input[data-idx="'+i+'"]'); if(inp) inp.value=(arr[i]??0);
      }
    });

    exportBtn.addEventListener('click',()=>{
      const prob = readProblem();
      const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(prob,null,2));
      const a = document.createElement('a'); a.setAttribute('href', dataStr); a.setAttribute('download','pl_problem.json'); document.body.appendChild(a); a.click(); a.remove();
    });

    function readProblem(){
      const coeffs = Array.from(objInputs.querySelectorAll('input')).map(i=>parseFloat(i.value)||0);
      const cons = Array.from(constraintsDiv.querySelectorAll('.constraint')).map(row=>{
        const inputs = Array.from(row.querySelectorAll('input[type=number]')).map(i=>parseFloat(i.value)||0);
        const sign = row.querySelector('select').value;
        return {a:inputs.slice(0,state.n), sign:sign, b:inputs[state.n]||0};
      });
      return {n:state.n, m:cons.length, obj:coeffs, sense:qs('#sense').value, constraints:cons, method:qs('#method').value};
    }

    // --- Solver (tableau simplex) ---
    // We implement a general tableau builder that handles <=, >=, = and both Two-Phase and Big-M

    const EPS=1e-9;

    function solveProblem(problem){
      // Build standard form: add slacks/surplus/artificial as needed
      // We'll create arrays describing variables: original x1..xn, slack s1.., surplus, artificials

      const n=problem.n;
      const cons=problem.constraints;

      // Track variable names
      let varNames = [];
      for(let i=1;i<=n;i++)varNames.push('x'+i);

      // Build matrix A and b
      const A = cons.map(c=>c.a.slice());
      const b = cons.map(c=>c.b);
      const signs = cons.map(c=>c.sign);

      // For each constraint, handle based on sign
      const extraCols = [];// for each constraint, record extra column added: {type: 'slack'|'surplus'|'art', name}
      for(let i=0;i<cons.length;i++){
        if(signs[i]=='<='){
          // add slack
          const name='s'+(i+1); varNames.push(name);
          for(let r=0;r<A.length;r++) A[r].push(r==i?1:0);
          extraCols.push({type:'slack',name});
        }else if(signs[i]=='>='){
          // add surplus (-1) and artificial
          const surName='r'+(i+1); varNames.push(surName);
          for(let r=0;r<A.length;r++) A[r].push(r==i?-1:0);
          extraCols.push({type:'surplus',name:surName});
          // add artificial
          const artName='a'+(i+1); varNames.push(artName);
          for(let r=0;r<A.length;r++) A[r].push(r==i?1:0);
          extraCols.push({type:'art',name:artName});
        }else{ // '='
          // add artificial only
          const artName='a'+(i+1); varNames.push(artName);
          for(let r=0;r<A.length;r++) A[r].push(r==i?1:0);
          extraCols.push({type:'art',name:artName});
        }
      }

      // Objective
      let c = Array(varNames.length).fill(0);
      for(let i=0;i<n;i++) c[i] = (problem.obj[i]||0);
      // For minimization, convert to maximization by negating
      let sense = problem.sense; // 'max' or 'min'
      if(sense=='min') c = c.map(x=>-x);

      // method handling
      const method = problem.method; // 'simplex'|'two-phase'|'big-m'

      // Identify basic variables initial: slack and artificials
      let basis = [];
      // build initial tableau rows
      const mcons = A.length;
      // initial basis: for each row, find a column that's identity
      for(let i=0;i<mcons;i++){
        let found=false;
        for(let j=0;j<varNames.length;j++){
          if(Math.abs(A[i][j]-1)<EPS){
            // check other rows 0
            let ok=true; for(let r=0;r<mcons;r++) if(r!=i && Math.abs(A[r][j])>EPS) ok=false;
            if(ok){ basis.push(j); found=true; break; }
          }
        }
        if(!found) basis.push(null);
      }

      // For Two-Phase: Phase1 objective is min sum of artificials -> in tableau as maximize negative sum
      // For Big-M: add penalty -M * artificial into objective (since we maximized, big negative)
      const iterations = [];

      // Helper: build tableau from A,b,c,basis
      function buildTableau(cLocal){
        // tableau with m rows and n+1 columns (last column RHS). row 0 will be objective (Z-row)
        const rows = [];
        // Objective row
        const objRow = cLocal.slice(); objRow.push(0); // add RHS
        rows.push(objRow.map(x=> -x)); // store -c for simplex (we'll treat row0 as -c)
        // constraints
        for(let i=0;i<mcons;i++){
          const row = A[i].slice(); row.push(b[i]); rows.push(row);
        }
        return rows;
      }

      function tableauToString(tab, varNamesLocal){
        // return an object with headers and rows for display
        const headers = varNamesLocal.concat(['RHS']);
        const rows = tab.map(r=>r.map(v=>Number(Math.round(v*10000)/10000)));
        return {headers,rows};
      }

      // We will implement primal simplex iterative routine that supports given basis.
      function runSimplex(cLocal, options={maxIter:200, recordSteps:true}){
        let tab = buildTableau(cLocal);
        let iter=0; const steps=[];
        while(iter<options.maxIter){
          // record
          if(options.recordSteps) steps.push(tableauToString(tab,varNames));
          // check optimality: objective row (row0) coefficients all >=0 -> optimal (since we have -c in row0)
          const objCoeffs = tab[0].slice(0,-1);
          let entering = null; let mostNeg = 0;
          for(let j=0;j<objCoeffs.length;j++){
            if(objCoeffs[j] < -1e-8 && objCoeffs[j] < mostNeg){ mostNeg = objCoeffs[j]; entering = j; }
          }
          if(entering===null) break; // optimal

          // ratio test
          let minRatio = Infinity; let leavingRow = null;
          for(let i=1;i<tab.length;i++){
            const a = tab[i][entering];
            const rhs = tab[i][tab[i].length-1];
            if(a>1e-10){
              const ratio = rhs / a;
              if(ratio < minRatio - 1e-10){ minRatio=ratio; leavingRow = i; }
            }
          }
          if(leavingRow===null) { throw new Error('Solución no acotada (unbounded)'); }

          // pivot at (leavingRow, entering)
          const pivot = tab[leavingRow][entering];
          // normalize leaving row
          for(let j=0;j<tab[leavingRow].length;j++) tab[leavingRow][j] /= pivot;
          // eliminate other rows
          for(let i=0;i<tab.length;i++){
            if(i===leavingRow) continue;
            const factor = tab[i][entering];
            if(Math.abs(factor) < 1e-12) continue;
            for(let j=0;j<tab[i].length;j++) tab[i][j] -= factor * tab[leavingRow][j];
          }

          // update basis index: leavingRow corresponds to basis index leavingRow-1
          basis[leavingRow-1] = entering;
          iter++;
        }
        if(options.recordSteps) steps.push(tableauToString(tab,varNames));
        return {tab,steps,iter};
      }

      // Prepare c for different methods
      // For Big-M: add -M to artificial variable columns in the objective (since we maximize, penalize)
      const M = 1e6;

      // Identify artificial indices
      const artIndices = varNames.map((v,i)=> v.startsWith('a')?i:null).filter(i=>i!==null);

      // If method simplex and there are artificials, we can't start: we should fallback to two-phase or big-m
      let result = {methodUsed:method, iterations:[], optimal:null};

      try{
        if(method==='simplex'){
          if(artIndices.length>0){
            throw new Error('El método Simplex directo requiere solo restricciones <= con variables de holgura. Usa Dos Fases o Big-M para = o >=.');
          }
          // run simplex using c
          const res = runSimplex(c);
          result.iterations=res.steps; result.optimal=extractSolution(res.tab);
        }else if(method==='big-m'){
          // build cBig: subtract M in objective for artificial variable indices
          const cBig = c.slice();
          for(const ai of artIndices) cBig[ai] -= M;
          // For maximization we want to maximize Z = c x - M sum(a)
          const res = runSimplex(cBig);
          result.iterations=res.steps; result.optimal=extractSolution(res.tab);
          // check artificial variables ~ 0
          for(const ai of artIndices){ const val = result.optimal.values[ai]||0; if(Math.abs(val)>1e-4) result.optimal.warning='Variables artificiales no cero -> problema infactible o M insuficiente'; }
        }else if(method==='two-phase'){
          // Phase 1: minimize sum of artificials -> maximize negative sum
          // Build c1
          const c1 = Array(varNames.length).fill(0);
          for(const ai of artIndices) c1[ai] = -1; // because runSimplex uses -c in row0
          // We must set basis to include artificials where present. We'll adjust basis: prefer artificial columns
          for(let i=0;i<basis.length;i++){
            if(basis[i]===null){
              // try find artificial column with 1 at row i and zeros elsewhere
              for(const ai of artIndices){
                let ok=true; for(let r=0;r<A.length;r++) if(r!=i && Math.abs(A[r][ai])>EPS) ok=false;
                if(ok && Math.abs(A[i][ai]-1)<EPS){ basis[i]=ai; break; }
              }
            }
          }
          // run phase1
          const r1 = runSimplex(c1);
          // get tableau after phase1
          const tab1 = r1.tab;
          // check sum of artificials (objective value)
          const zval = -tab1[0][tab1[0].length-1]; // since row0 stores -c
          if(Math.abs(zval) > 1e-6){
            result.iterations = r1.steps; result.optimal={infeasible:true, message:'Problema infactible (fase 1 no pudo llevar artificiales a cero)'};
          }else{
            // remove artificial columns from tableau and varNames, adjust c to original
            // map columns to keep
            const keep = [];
            for(let j=0;j<varNames.length;j++) if(!varNames[j].startsWith('a')) keep.push(j);
            const newVarNames = keep.map(i=>varNames[i]);
            // rebuild A,b from tab1 (rows 1..m)
            const newA = [];
            const newB = [];
            for(let i=1;i<tab1.length;i++){
              const row = keep.map(j=>tab1[i][j]); newA.push(row); newB.push(tab1[i][tab1[i].length-1]);
            }
            // new c
            const newC = keep.map(j=>c[j]||0);
            // overwrite for continuation
            for(let i=0;i<A.length;i++){
              A[i]=newA[i];
              b[i]=newB[i];
            }
            varNames = newVarNames; c = newC;
            // rebuild basis indices: for each row, find basis var index in new varNames
            basis = [];
            for(let i=0;i<A.length;i++){
              let found=null;
              for(let j=0;j<varNames.length;j++) if(Math.abs(A[i][j]-1)<EPS){ let ok=true; for(let r=0;r<A.length;r++) if(r!=i && Math.abs(A[r][j])>EPS) ok=false; if(ok){ found=j; break; } }
              basis.push(found);
            }
            // run phase2
            const r2 = runSimplex(c);
            result.iterations = r1.steps.concat(r2.steps);
            result.optimal = extractSolution(r2.tab);
          }
        }
      }catch(err){ result.error = err.message; }

      function extractSolution(tab){
        // tab has row0 objective, rows 1..m constraints. basis[] tells column indices for each row
        const ncols = tab[0].length-1;
        const sol = Array(varNames.length).fill(0);
        for(let i=1;i<tab.length;i++){
          // find leading 1
          let lead=-1; for(let j=0;j<ncols;j++) if(Math.abs(tab[i][j]-1)<1e-8){ // candidate
            let ok=true; for(let r=1;r<tab.length;r++) if(r!=i && Math.abs(tab[r][j])>1e-8) ok=false;
            if(ok){ lead=j; break; }
          }
          if(lead>=0) sol[lead]=tab[i][tab[i].length-1];
        }
        // objective value
        const z = tab[0][tab[0].length-1] * -1; // due to -c storage
        return {values:sol, objective:z, varNames:varNames};
      }

      return result;
    }

    // --- UI for running and showing iterations ---
    function renderIterationTable(iter){
      const output = qs('#output'); output.innerHTML='';
      if(!iter||!iter.headers) return;
      const wrap = document.createElement('div'); wrap.className='table-wrap';
      const table=document.createElement('table');
      const thead=document.createElement('thead'); const trh=document.createElement('tr');
      iter.headers.forEach(h=>{const th=document.createElement('th'); th.textContent=h; trh.appendChild(th)}); thead.appendChild(trh); table.appendChild(thead);
      const tbody=document.createElement('tbody');
      iter.rows.forEach((r,i)=>{const tr=document.createElement('tr'); r.forEach(c=>{const td=document.createElement('td'); td.textContent=(typeof c==='number'?c.toFixed(6):c); tr.appendChild(td)}); tbody.appendChild(tr)});
      table.appendChild(tbody); wrap.appendChild(table);
      output.appendChild(wrap);
    }

    function showResult(res){
      const out = qs('#output'); out.innerHTML='';
      if(res.error){ const e=document.createElement('div'); e.textContent='Error: '+res.error; e.style.color='salmon'; out.appendChild(e); return; }
      if(res.iterations && res.iterations.length>0){
        // pagination
        let idx=0;
        const nav = document.createElement('div'); nav.className='steps-nav';
        const prev=document.createElement('button'); prev.textContent='◀'; prev.className='btn small';
        const next=document.createElement('button'); next.textContent='▶'; next.className='btn small';
        const info=document.createElement('div'); info.className='muted'; info.style.minWidth='180px';
        prev.onclick=()=>{ if(idx>0) idx--; renderIterationTable(res.iterations[idx]); info.textContent=`Paso ${idx+1} de ${res.iterations.length}`; };
        next.onclick=()=>{ if(idx<res.iterations.length-1) idx++; renderIterationTable(res.iterations[idx]); info.textContent=`Paso ${idx+1} de ${res.iterations.length}`; };
        nav.appendChild(prev); nav.appendChild(next); nav.appendChild(info);
        out.appendChild(nav);
        renderIterationTable(res.iterations[0]); info.textContent=`Paso 1 de ${res.iterations.length}`;
        // final solution box
        const finalBox=document.createElement('div'); finalBox.style.marginTop='12px'; finalBox.style.padding='10px'; finalBox.style.background='#011222'; finalBox.style.borderRadius='8px';
        if(res.optimal){
          if(res.optimal.infeasible){ finalBox.innerHTML=`<b>Resultado:</b> Problema infactible. ${res.optimal.message||''}`; }
          else{
            let html=`<b>Solución óptima (valor Z):</b> ${Number(res.optimal.objective).toFixed(6)}<br><div style='margin-top:6px'><b>Variables:</b><br>`;
            res.optimal.values.forEach((v,i)=>{ html+=`${res.optimal.varNames[i]} = ${Number(v).toFixed(6)}<br>` });
            html+='</div>';
            if(res.optimal.warning) html+='<div style="color:orange;margin-top:8px">Aviso: '+res.optimal.warning+'</div>';
            finalBox.innerHTML=html;
          }
        }
        out.appendChild(finalBox);
      }else{
        out.textContent='No hay iteraciones para mostrar.';
      }
    }

    solveBtn.addEventListener('click',()=>{
      const prob = readProblem();
      qs('#output').innerHTML = '<div class="muted">Resolviendo... (revise mensajes en pantalla)</div>';
      try{
        const res = solveProblem(prob);
        showResult(res);
      }catch(err){ qs('#output').innerHTML = '<div style="color:salmon">Error interno: '+err.message+'</div>'; }
    });

    // Auto-initialize small example
    window.addEventListener('load',()=>{ setupBtn.click();
      // example data
      setTimeout(()=>{
        objPaste.value='3,2'; pasteObj.click();
        // set first constraint 1,2 <= 18 ; second 3,1 <= 42
        const rows = constraintsDiv.querySelectorAll('.constraint');
        if(rows[0]){ rows[0].querySelectorAll('input[type=number]')[0].value=1; rows[0].querySelectorAll('input[type=number]')[1].value=2; rows[0].querySelectorAll('input[type=number]')[2].value=18; rows[0].querySelector('select').value='<='; }
        if(rows[1]){ rows[1].querySelectorAll('input[type=number]')[0].value=3; rows[1].querySelectorAll('input[type=number]')[1].value=1; rows[1].querySelectorAll('input[type=number]')[2].value=42; rows[1].querySelector('select').value='<='; }
      },200);
    });
  </script>
</body>
</html>
