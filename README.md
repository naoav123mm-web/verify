<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<meta name="robots" content="noindex">
<title>التحقق من التقرير الطبي</title>
<link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;600;700;800&display=swap" rel="stylesheet">
<script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-firestore-compat.js"></script>
<style>
  * { box-sizing:border-box; }
  body { font-family:'Cairo',Tahoma,Arial,sans-serif; background:#eef2f5; margin:0; padding:20px; color:#222; }
  .card { max-width:480px; margin:30px auto; background:#fff; border-radius:12px; box-shadow:0 4px 20px rgba(0,0,0,.1); overflow:hidden; }
  .top { padding:26px 20px; text-align:center; color:#fff; }
  .top.ok { background:#0f7b6c; } .top.bad { background:#b5304a; } .top.wait { background:#6b7b87; } .top.warn { background:#d9730d; }
  .top .ic { font-size:48px; line-height:1; }
  .top h1 { margin:8px 0 0; font-size:22px; }
  .body { padding:20px; }
  .row { display:flex; justify-content:space-between; gap:12px; padding:10px 0; border-bottom:1px solid #e3e9ed; font-size:15px; }
  .row:last-child { border-bottom:0; }
  .row span:first-child { color:#667; }
  .row span:last-child { font-weight:700; text-align:left; }
  .note { font-size:12px; color:#778; text-align:center; padding:0 20px 18px; line-height:1.7; }
</style>
</head>
<body>
<div class="card">
  <div class="top wait" id="top"><div class="ic" id="ic">…</div><h1 id="ttl">جارٍ التحقق</h1></div>
  <div class="body" id="body"></div>
  <div class="note" id="note"></div>
</div>
<script>
const FIREBASE_CONFIG = {
  apiKey: "AIzaSyBkXWownNeK0o1OA07pX6oqSkqgSuqJnX4",
  authDomain: "rehab-reports-cd5aa.firebaseapp.com",
  projectId: "rehab-reports-cd5aa"
};
const $ = id => document.getElementById(id);
function show(kind, icon, title, rows, note){
  $("top").className = "top " + kind;
  $("ic").textContent = icon;
  $("ttl").textContent = title;
  const b = $("body");
  b.innerHTML = "";
  (rows || []).forEach(([k, v]) => {
    const r = document.createElement("div"); r.className = "row";
    const a = document.createElement("span"); a.textContent = k;
    const c = document.createElement("span"); c.textContent = v || "—";
    r.appendChild(a); r.appendChild(c); b.appendChild(r);
  });
  $("note").textContent = note || "";
}
(async function(){
  const p = new URLSearchParams(location.search);
  const ref = (p.get("ref") || "").trim(), code = (p.get("code") || "").trim();
  if(!ref || !code){ show("bad", "✖", "رابط غير مكتمل", [], "امسح الباركود من التقرير الأصلي."); return; }
  try{
    firebase.initializeApp(FIREBASE_CONFIG);
    const id = (ref + "_" + code).replace(/[\/\s]/g, "_");
    const snap = await firebase.firestore().collection("reports").doc(id).get();
    if(!snap.exists){
      show("bad", "✖", "لم يُعثر على التقرير", [["رقم التقرير", ref]],
        "لا يوجد سجل مطابق لهذا الرقم والرمز. قد يكون التقرير غير صادر من المنشأة أو تم تعديله بعد الطباعة.");
      return;
    }
    const d = snap.data();
    if(d.code !== code){ show("bad", "✖", "الرمز غير مطابق", [["رقم التقرير", ref]], "بيانات التقرير لا تطابق السجل."); return; }
    if(d.status === "revoked"){
      show("warn", "!", "تقرير ملغى", [["رقم التقرير", d.ref], ["المنشأة", d.facility], ["تاريخ الإصدار", d.date]],
        "تم إلغاء هذا التقرير من قبل المنشأة المصدرة.");
      return;
    }
    show("ok", "✔", "تقرير مسجل لدى المنشأة", [
      ["رقم التقرير", d.ref],
      ["المنشأة", d.facility],
      ["تاريخ الإصدار", d.date],
      ["المريض", d.patient],
      ["رقم الهوية", d.idMasked],
      ["نوع التأهيل", d.type],
      ["الطبيب المعالج", d.doctor],
      ["رقم الترخيص", d.license]
    ], "طابق هذه البيانات مع الورقة التي بين يديك. لا تُعرض التفاصيل الطبية هنا حفاظاً على الخصوصية.");
  }catch(e){
    show("warn", "!", "تعذر الاتصال", [], "حاول مرة أخرى لاحقاً.");
  }
})();
</script>
</body>
</html>
