<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Al.VIPZEXNET - چت هوش مصنوعی</title>
<style>
:root {
  --bg-dark:#0d1117;
  --text-light:#fff;
  --brand:#58a6ff;
  --success:#238636;
  --danger:#ff5252;
  --card-dark:rgba(33,38,45,0.9);
}

body {
  margin:0;
  font-family:sans-serif;
  background:var(--bg-dark);
  color:var(--text-light);
  display:flex;
  flex-direction:column;
  height:100vh;
}

header {
  background:#161b22;
  padding:10px;
  display:flex;
  align-items:center;
  justify-content:space-between;
  border-bottom:1px solid #30363d;
}
header h1 { margin:0; font-size:18px; color:var(--brand); }
#modelSelect {
  background:var(--bg-dark);
  color:var(--text-light);
  border:1px solid #30363d;
  padding:5px;
  border-radius:5px;
}

#chatWindow {
  flex:1;
  overflow-y:auto;
  padding:20px;
  display:flex;
  flex-direction:column;
  gap:12px;
}
#chatWindow::-webkit-scrollbar { width:6px; }
#chatWindow::-webkit-scrollbar-thumb { background:var(--brand); border-radius:3px; }

.message {
  max-width:70%;
  padding:12px 16px;
  border-radius:12px;
  line-height:1.5;
  white-space:pre-wrap;
  position:relative;
}
.user { background:var(--success); color:#fff; align-self:flex-end; border-bottom-right-radius:0; }
.bot { background:var(--card-dark); color:#fff; align-self:flex-start; border-bottom-left-radius:0; }
.msg-time { font-size:10px; opacity:0.6; text-align:left; margin-top:4px; }
.msg-user { font-weight:bold; font-size:12px; }

#inputArea {
  display:flex;
  padding:10px;
  border-top:1px solid #30363d;
  background:#161b22;
}
#userInput { flex:1; padding:12px; border:none; border-radius:8px; background:var(--bg-dark); color:var(--text-light); font-size:15px; }
#sendBtn, #uploadBtn, #themeBtn { padding:12px 15px; margin-left:5px; background:var(--brand); border:none; border-radius:8px; cursor:pointer; color:#fff; font-size:15px; }
#sendBtn:hover, #uploadBtn:hover, #themeBtn:hover { background:#1f6feb; }

@keyframes fadeIn { from{opacity:0; transform:translateY(5px);} to{opacity:1; transform:translateY(0);} }

#historyMenu {
  display:none;
  position:absolute;
  top:35px;
  right:0;
  background:#161b22;
  border:1px solid #30363d;
  border-radius:6px;
  min-width:220px;
  z-index:100;
  max-height:300px;
  overflow-y:auto;
}
#historyMenu button { width:100%; padding:8px; border:none; background:none; color:#fff; text-align:left; cursor:pointer; }
#historyMenu button:hover { background:#30363d; }
#menuContainer { position:relative; }

#profileModal {
  display:none;
  position:fixed;
  top:50%; left:50%;
  transform:translate(-50%,-50%);
  background:#161b22;
  border:1px solid #30363d;
  border-radius:10px;
  padding:20px;
  z-index:200;
  width:300px;
  color:#fff;
}
#profileModal input { width:100%; margin-bottom:10px; padding:5px; background:#0d1117; color:#fff; border:1px solid #30363d; border-radius:5px; }
#profileModal button { cursor:pointer; border:none; border-radius:5px; padding:5px 10px; }
#saveProfile { background:var(--success); color:#fff; margin-left:5px; }
#closeProfile { background:#30363d; color:#fff; }

.typing { font-style:italic; opacity:0.7; }
.message img { max-width:150px; border-radius:8px; }
</style>
</head>
<body>

<header>
  <h1>🤖 Al.VIPZEXNET</h1>
  <div style="display:flex; align-items:center; gap:10px;">
    <select id="modelSelect">
      <option value="openai">OpenAI</option>
      <option value="deepseek">DeepSeek</option>
      <option value="mistral">Mistral</option>
      <option value="openai-large">OpenAI Large (GPT-4)</option>
    </select>
    <button id="themeBtn">🌓</button>
    <button id="uploadBtn">📎</button>
    <div id="menuContainer">
      <button id="menuBtn" style="background:none;border:none;color:var(--brand);font-size:20px;cursor:pointer;">⋮</button>
      <div id="historyMenu">
        <button id="newChat">🆕 New Chat</button>
        <button id="profileBtn">👤 پروفایل</button>
        <div id="chatHistoryList"></div>
      </div>
    </div>
  </div>
</header>

<div id="chatWindow"></div>

<div id="inputArea">
  <input type="text" id="userInput" placeholder="پیام خود را بنویسید..." onkeydown="if(event.key==='Enter'){sendMessage()}">
  <button id="sendBtn" onclick="sendMessage()">ارسال</button>
</div>

<div id="profileModal">
  <h3>پروفایل</h3>
  <label>نام:</label><input type="text" id="profileName">
  <label>ایمیل:</label><input type="email" id="profileEmail">
  <div style="text-align:right;">
    <button id="saveProfile">ذخیره</button>
    <button id="closeProfile">بستن</button>
  </div>
</div>

<input type="file" id="fileInput" style="display:none" accept="image/*,.txt">

<script>
let selectedModel="openai";
const chatWindow=document.getElementById("chatWindow");

// تغییر مدل
document.getElementById("modelSelect").addEventListener("change",()=>{selectedModel=document.getElementById("modelSelect").value;});

// تاریخچه
function loadChatHistory(){
  const history=JSON.parse(localStorage.getItem("chatHistory")||"[]");
  history.forEach(msg=>addMessage(msg.text,msg.sender,false,msg.time,msg.name,msg.img));
}

function saveMessage(text,sender,time,name=null,img=null){
  const history=JSON.parse(localStorage.getItem("chatHistory")||"[]");
  history.push({text,sender,time,name,img});
  localStorage.setItem("chatHistory",JSON.stringify(history));
}

function addMessage(text,sender,save=true,time=null,name=null,img=null,isTyping=false){
  const msgDiv=document.createElement("div");
  msgDiv.className="message "+sender;
  if(!time) time=new Date().toLocaleTimeString([], {hour:'2-digit',minute:'2-digit'});
  let userDisplay=name ? `<div class="msg-user">${name}</div>` : "";
  let imgTag=img ? `<img src="${img}">` : "";
  msgDiv.innerHTML=`${userDisplay}<span>${text}</span>${imgTag}<div class="msg-time">${time}</div>`;
  if(isTyping) msgDiv.classList.add("typing");
  chatWindow.appendChild(msgDiv);
  chatWindow.scrollTop=chatWindow.scrollHeight;
  if(save) saveMessage(text,sender,time,name,img);
}

// ارسال پیام متن
async function sendMessage(){
  const input=document.getElementById("userInput");
  const text=input.value.trim();
  if(!text) return;
  const profile=JSON.parse(localStorage.getItem("userProfile")||"{}");
  addMessage(text,"user",true,null,profile.name);
  input.value="";
  const loadingMsg=document.createElement("div");
  loadingMsg.className="message bot typing";
  loadingMsg.textContent="در حال تحلیل Al.VIPZEXNET...";
  chatWindow.appendChild(loadingMsg);
  chatWindow.scrollTop=chatWindow.scrollHeight;

  try{
    const formData=new FormData();
    formData.append("userid","user123");
    formData.append("prompt",text);
    formData.append("model",selectedModel);
    const res=await fetch("https://api2.api-code.ir/gpt-save-v2/",{method:"POST",body:formData});
    const data=await res.text();
    loadingMsg.remove();
    addMessage(data,"bot");
  }catch(err){
    loadingMsg.remove();
    addMessage("❌ خطا در دریافت پاسخ","bot");
    console.error(err);
  }
}

// آپلود تصویر و فایل
const uploadBtn=document.getElementById("uploadBtn");
const fileInput=document.getElementById("fileInput");
uploadBtn.addEventListener("click",()=>fileInput.click());
fileInput.addEventListener("change",(e)=>{
  const file=e.target.files[0];
  if(!file) return;
  const reader=new FileReader();
  reader.onload=function(){
    const profile=JSON.parse(localStorage.getItem("userProfile")||"{}");
    if(file.type.startsWith("image/")){
      addMessage("", "user", true, null, profile.name, reader.result);
      analyzeImage(reader.result);
    } else if(file.type==="text/plain"){
      addMessage(reader.result, "user");
    }
  };
  reader.readAsDataURL(file);
});

async function analyzeImage(base64Image){
  const loadingMsg=document.createElement("div");
  loadingMsg.className="message bot typing";
  loadingMsg.textContent="در حال تحلیل تصویر توسط Al.VIPZEXNET...";
  chatWindow.appendChild(loadingMsg);
  chatWindow.scrollTop=chatWindow.scrollHeight;

  try{
    const formData=new FormData();
    formData.append("userid","mahdi-vision");
    formData.append("model","mistral");
    formData.append("prompt","این تصویر را توصیف کن: "+base64Image);

    const res=await fetch("https://api2.api-code.ir/gpt-save-v2/",{method:"POST",body:formData});
    const data=await res.text();
    loadingMsg.remove();
    addMessage(data,"bot");
  }catch(err){
    loadingMsg.remove();
    addMessage("❌ خطا در تحلیل تصویر","bot");
    console.error(err);
  }
}

// منوی سه نقطه
const menuBtn=document.getElementById("menuBtn");
const historyMenu=document.getElementById("historyMenu");
const chatHistoryList=document.getElementById("chatHistoryList");
menuBtn.addEventListener("click",()=>{
  historyMenu.style.display=historyMenu.style.display==="block"?"none":"block";
  renderHistoryList();
});
document.getElementById("newChat").addEventListener("click",()=>{
  if(confirm("آیا مطمئن هستید که می‌خواهید چت جدید شروع کنید؟")){
    localStorage.removeItem("chatHistory");
    chatWindow.innerHTML="";
    historyMenu.style.display="none";
  }
});
function renderHistoryList(){
  const history=JSON.parse(localStorage.getItem("chatHistory")||"[]");
  chatHistoryList.innerHTML="";
  history.forEach(msg=>{
    const btn=document.createElement("button");
    btn.textContent=(msg.sender==="user"?"👤 ":"🤖 ")+msg.text.slice(0,30);
    btn.addEventListener("click",()=>alert("پیام کامل:\n"+msg.text));
    chatHistoryList.appendChild(btn);
  });
}
document.addEventListener("click",(e)=>{
  if(!historyMenu.contains(e.target) && e.target!==menuBtn) historyMenu.style.display="none";
});

// پروفایل
const profileBtn=document.getElementById("profileBtn");
const profileModal=document.getElementById("profileModal");
const closeProfile=document.getElementById("closeProfile");
const saveProfile=document.getElementById("saveProfile");
const profileName=document.getElementById("profileName");
const profileEmail=document.getElementById("profileEmail");

profileBtn.addEventListener("click",()=>{
  profileModal.style.display="block";
  historyMenu.style.display="none";
  const profile=JSON.parse(localStorage.getItem("userProfile")||"{}");
  profileName.value=profile.name||"";
  profileEmail.value=profile.email||"";
});
closeProfile.addEventListener("click",()=>{profileModal.style.display="none";});
saveProfile.addEventListener("click",()=>{
  const profile={name:profileName.value.trim(),email:profileEmail.value.trim()};
  localStorage.setItem("userProfile",JSON.stringify(profile));
  alert("پروفایل ذخیره شد ✅");
  profileModal.style.display="none";
});

// حالت شب/روز
const themeBtn=document.getElementById("themeBtn");
let darkMode=true;
themeBtn.addEventListener("click",()=>{
  darkMode=!darkMode;
  document.body.style.background=darkMode?"#0d1117":"#f5f5f5";
  document.body.style.color=darkMode?"#fff":"#000";
});

loadChatHistory();
</script>

</body>
</html>
