const state = { activities: JSON.parse(localStorage.getItem('natproActivities') || '[]') };

const navButtons = document.querySelectorAll('[data-view]');
navButtons.forEach(btn => btn.addEventListener('click', () => showView(btn.dataset.view)));
document.getElementById('menuToggle').addEventListener('click',()=>document.querySelector('.sidebar').classList.toggle('open'));

function showView(id){
  document.querySelectorAll('.view').forEach(v=>v.classList.remove('active-view'));
  document.getElementById(id).classList.add('active-view');
  document.querySelectorAll('.nav-item').forEach(b=>b.classList.toggle('active', b.dataset.view===id));
  document.querySelector('.sidebar').classList.remove('open');
}
function saveActivity(type, site, details){
  const entry = { type, site, details, time: new Date().toLocaleString(), id: `NAT-${Date.now().toString().slice(-6)}` };
  state.activities.unshift(entry);
  localStorage.setItem('natproActivities', JSON.stringify(state.activities.slice(0,25)));
  renderActivities();
  postToDiscord(entry);
  alert(`${type} submitted successfully. Report ID: ${entry.id}`);
}
function renderActivities(){
  const list = document.getElementById('activityList');
  const rows = state.activities.length ? state.activities : [
    {type:'Incident Report',site:'Jumbo Foods',details:'Trespass - Subject refused to leave',time:'Demo',id:'IR# 2026-001'},
    {type:'Daily Activity Report',site:'Comfort Inn',details:'DAR Completed',time:'Demo',id:'DAR# 2026-045'},
    {type:'Clock In',site:'Unit 03',details:'Officer Smith clocked in',time:'Demo',id:'CLK# 2026-010'}
  ];
  list.innerHTML = rows.slice(0,8).map(r=>`<li><strong>${r.type}</strong><br>${r.details}<br><small>${r.site} • ${r.time} • ${r.id}</small></li>`).join('');
}
async function postToDiscord(entry){
  if(!DISCORD_WEBHOOK_URL || DISCORD_WEBHOOK_URL.includes('PASTE_')) return;
  try{
    await fetch(DISCORD_WEBHOOK_URL,{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify({content:`**${entry.type}**\nSite: ${entry.site}\nDetails: ${entry.details}\nTime: ${entry.time}\nReport ID: ${entry.id}`})});
  }catch(err){ console.warn('Discord post failed', err); }
}
function setStatus(status){
  document.getElementById('currentStatus').textContent = status;
  saveActivity('Status Update','Officer Portal',`Status changed to ${status}`);
}
function formData(form){ return Object.fromEntries(new FormData(form).entries()); }

document.getElementById('clockForm').addEventListener('submit', e=>{e.preventDefault(); const d=formData(e.target); saveActivity(d.action,d.site,`${d.officer} submitted ${d.action}`); e.target.reset();});
document.getElementById('darForm').addEventListener('submit', e=>{e.preventDefault(); const d=formData(e.target); saveActivity('Daily Activity Report',d.site,`${d.officer}: ${d.summary}`); e.target.reset();});
document.getElementById('incidentForm').addEventListener('submit', e=>{e.preventDefault(); const d=formData(e.target); saveActivity('Incident Report',d.site,`${d.type}: ${d.details}`); e.target.reset();});
document.getElementById('trespassForm').addEventListener('submit', e=>{e.preventDefault(); const d=formData(e.target); saveActivity('Trespass Report',d.site,`${d.subject}: ${d.reason}`); e.target.reset();});
document.getElementById('emergencyBtn').addEventListener('click',()=>saveActivity('EMERGENCY ALERT','Current Assignment','Officer requested immediate supervisor/dispatch assistance.'));
renderActivities();
