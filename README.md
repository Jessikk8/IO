# IO
<!doctype html>
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
