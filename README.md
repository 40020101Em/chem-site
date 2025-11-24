<!DOCTYPE html>
<html lang="ar">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>معلومات المركبات الكيميائية</title>
<style>
body{
  font-family: Arial, sans-serif;
  background:#f4f4f9;
  display:flex;
  justify-content:center;
  padding:20px;
}
.container{
  max-width:600px;
  width:100%;
  background:white;
  padding:20px;
  border-radius:10px;
  box-shadow:0 4px 12px rgba(0,0,0,0.1);
}
h1{
  font-size:20px;
  text-align:center;
  margin-bottom:20px;
}
form{
  display:flex;
  gap:10px;
  margin-bottom:20px;
}
input{
  flex:1;
  padding:8px;
  border-radius:5px;
  border:1px solid #ccc;
}
button{
  padding:8px 16px;
  border:none;
  border-radius:5px;
  background:#0b63d6;
  color:white;
  cursor:pointer;
}
.result img{
  max-width:200px;
  display:block;
  margin:10px auto;
}
.result div{
  margin:6px 0;
}
.lang-toggle{
  text-align:center;
  margin-bottom:10px;
}
.lang-toggle button{
  margin:0 5px;
  padding:4px 8px;
  cursor:pointer;
}
</style>
</head>
<body>
<div class="container">
  <h1 id="title">معلومات المركبات الكيميائية | Chemical Compound Info</h1>

  <div class="lang-toggle">
    <button onclick="setLang('ar')">عربي</button>
    <button onclick="setLang('en')">English</button>
  </div>

  <form id="searchForm">
    <input type="text" id="compoundInput" placeholder="اكتب اسم المركب | Enter compound name" required>
    <button type="submit" id="searchBtn">بحث | Search</button>
  </form>

  <div id="result" class="result"></div>
</div>

<script>
let lang = 'ar';

function setLang(l){
  lang = l;
  document.getElementById('title').textContent = (lang==='ar') ? "معلومات المركبات الكيميائية" : "Chemical Compound Info";
  document.getElementById('compoundInput').placeholder = (lang==='ar') ? "اكتب اسم المركب" : "Enter compound name";
  document.getElementById('searchBtn').textContent = (lang==='ar') ? "بحث" : "Search";
}

const form = document.getElementById("searchForm");
const input = document.getElementById("compoundInput");
const resultDiv = document.getElementById("result");

form.addEventListener("submit", async (e)=>{
  e.preventDefault();
  const name = input.value.trim();
  if(!name) return;
  resultDiv.innerHTML = (lang==='ar') ? "جاري التحميل..." : "Loading...";

  try{
    const cidResp = await fetch(`https://pubchem.ncbi.nlm.nih.gov/rest/pug/compound/name/${encodeURIComponent(name)}/cids/JSON`);
    const cidData = await cidResp.json();
    const cid = cidData.IdentifierList?.CID?.[0];
    if(!cid){
      resultDiv.innerHTML = (lang==='ar') ? "المركب غير موجود" : "Compound not found";
      return;
    }

    const propsResp = await fetch(`https://pubchem.ncbi.nlm.nih.gov/rest/pug/compound/cid/${cid}/property/MolecularFormula,MolecularWeight,CanonicalSMILES/JSON`);
    const propsData = await propsResp.json();
    const info = propsData.PropertyTable.Properties[0];
    const imageUrl = `https://pubchem.ncbi.nlm.nih.gov/rest/pug/compound/cid/${cid}/PNG`;

    resultDiv.innerHTML = `
      <img src="${imageUrl}" alt="Structure of ${info.MolecularFormula}">
      <div><strong>${lang==='ar'?'الاسم':'Name'}:</strong> ${name}</div>
      <div><strong>${lang==='ar'?'الصيغة':'Formula'}:</strong> ${info.MolecularFormula}</div>
      <div><strong>${lang==='ar'?'الوزن الجزيئي':'Molecular Weight'}:</strong> ${info.MolecularWeight}</div>
      <div><strong>SMILES:</strong> ${info.CanonicalSMILES}</div>
    `;
  }catch(err){
    resultDiv.innerHTML = (lang==='ar') ? "حدث خطأ" : "Server error";
    console.error(err);
  }
});
</script>
</body>
</html>
