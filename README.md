# FYNDR
Find nearest to you app
import { useState, useRef, useEffect } from "react";

// ─── Three brand themes ───────────────────────────────────────────────────────
const THEMES = {
  spota: { id:"spota", name:"SPOTA", dot:".", tagline:"WHO'S OPEN RIGHT NOW", bg:"#0a0a0f", surface:"#0d0d18", surface2:"#13131f", border:"#1e1e2e", accent:"#FF6B35", text:"#e0e0e0", sub:"#555", sub2:"#222", font:"'DM Mono','Courier New',monospace", titleFont:"'Syne',sans-serif", dark:true },
  fyndr: { id:"fyndr", name:"FYNDR", dot:".", tagline:"FIND WHAT'S NEAR YOU",  bg:"#f2f2ed", surface:"#ffffff", surface2:"#e8e8e3", border:"#ddddd8", accent:"#0077B6", text:"#111111", sub:"#999", sub2:"#e0e0da", font:"'Inter','Helvetica Neue',sans-serif", titleFont:"'Inter','Helvetica Neue',sans-serif", dark:false },
  khona: { id:"khona", name:"KHONA", dot:"",  tagline:"OPEN · HERE · NOW",     bg:"#0d0800", surface:"#1a1200", surface2:"#221a00", border:"#2e2200", accent:"#F4A261", text:"#f0e0c0", sub:"#5a4530", sub2:"#2a1a00", font:"'Georgia','Times New Roman',serif",  titleFont:"'Georgia','Times New Roman',serif",  dark:true },
};

// ─── Data ─────────────────────────────────────────────────────────────────────
const CATEGORIES = [
  { id:"food",     label:"Food & Meals",      color:"#FF6B35", emoji:"🍱", mobile:false },
  { id:"hair",     label:"Hair & Beauty",      color:"#C77DFF", emoji:"💈", mobile:false },
  { id:"mechanic", label:"Auto & Repairs",     color:"#4CC9F0", emoji:"🔧", mobile:false },
  { id:"goods",    label:"Goods & Supplies",   color:"#F9C74F", emoji:"📦", mobile:false },
  { id:"money",    label:"Financial Services", color:"#43AA8B", emoji:"💸", mobile:false },
  { id:"delivery", label:"Delivery & Courier", color:"#FF3CAC", emoji:"🛵", mobile:true  },
  { id:"rentals",  label:"Rentals",            color:"#A8DADC", emoji:"🏠", mobile:false },
  { id:"forsale",  label:"For Sale",           color:"#E9C46A", emoji:"🛒", mobile:false },
  { id:"other",    label:"Other",              color:"#aaaaaa", emoji:"⚡", mobile:false },
];

const PAYMENT_METHODS = [
  { id:"cash",     label:"Cash",        icon:"💵" },
  { id:"snapscan", label:"SnapScan",    icon:"📷" },
  { id:"eft",      label:"EFT",         icon:"🏦" },
  { id:"capitec",  label:"Capitec Pay", icon:"📲" },
  { id:"ozow",     label:"Ozow",        icon:"⚡" },
];

const MOCK_MERCHANTS = [
  { id:1,  name:"Mama Thandi",      cat:"food",     x:38,y:28, dist:"0.3", status:"open", backIn:null, mobile:false, radius:null, rating:4.8, reviews:112, desc:"Home cooked meals, open till 10pm",            menu:"Umngqusho, pap & stew, vetkoek. Delivery within 1km.", payment:["cash","snapscan"] },
  { id:2,  name:"Sbu Cuts",         cat:"hair",     x:62,y:44, dist:"0.7", status:"soon", backIn:10,   mobile:false, radius:null, rating:4.5, reviews:67,  desc:"Fades & braids, late nights welcome",           menu:"Fades R80, braids R150, locs R120.",                  payment:["cash","eft"] },
  { id:3,  name:"Zithulele Spares", cat:"mechanic", x:25,y:60, dist:"1.1", status:"open", backIn:null, mobile:false, radius:null, rating:4.2, reviews:89,  desc:"Tyres, brakes, no appointment needed",          menu:"Tyre fitting, brake pads, wheel alignment, battery.", payment:["cash","snapscan","eft"] },
  { id:4,  name:"Corner Store Guy", cat:"goods",    x:70,y:22, dist:"0.5", status:"open", backIn:null, mobile:false, radius:null, rating:4.0, reviews:203, desc:"Airtime, data, household basics",               menu:"All networks airtime & data. Basic groceries.",       payment:["cash"] },
  { id:5,  name:"Nkosi Loans",      cat:"money",    x:50,y:68, dist:"1.4", status:"soon", backIn:20,   mobile:false, radius:null, rating:3.8, reviews:31,  desc:"Cash advances, quick & discreet",              menu:"Short term loans. ID & proof of income required.",    payment:["cash","eft"] },
  { id:6,  name:"Thabo Welding",    cat:"other",    x:80,y:55, dist:"1.8", status:"open", backIn:null, mobile:false, radius:null, rating:4.7, reviews:44,  desc:"Gates, burglar bars, same-day",                menu:"Security gates, burglar bars, custom metalwork.",     payment:["cash","eft"] },
  { id:7,  name:"Aunty Nokwanda",   cat:"food",     x:18,y:38, dist:"0.9", status:"open", backIn:null, mobile:false, radius:null, rating:4.9, reviews:178, desc:"Vetkoek & pap, morning & evening",             menu:"Vetkoek R10-R15, pap & wors R35, porridge R20.",      payment:["cash"] },
  { id:8,  name:"Nandi Nails",      cat:"hair",     x:55,y:35, dist:"0.6", status:"soon", backIn:5,    mobile:false, radius:null, rating:4.6, reviews:93,  desc:"Gel nails, acrylics, walk-in or appointment",  menu:"Gel nails R200, acrylics R250, nail art R30+.",       payment:["cash","snapscan"] },
  { id:9,  name:"Sipho Delivery",   cat:"delivery", x:45,y:52, dist:"0.8", status:"open", backIn:null, mobile:true,  radius:5,    rating:4.4, reviews:56,  desc:"Fast scooter, covers 5km radius",              menu:"Documents, parcels, groceries. R15/km or flat rate.", payment:["cash","snapscan","capitec"] },
  { id:10, name:"Room Available",   cat:"rentals",  x:32,y:70, dist:"1.2", status:"open", backIn:null, mobile:false, radius:null, rating:4.1, reviews:12,  desc:"Bachelor flat, R3500/month, available now",    menu:"Furnished, water included. Near taxi rank.",          payment:["cash","eft"] },
  { id:11, name:"Lesedi's Fridge",  cat:"forsale",  x:72,y:65, dist:"1.6", status:"open", backIn:null, mobile:false, radius:null, rating:4.3, reviews:8,   desc:"LG fridge for sale, R1800 negotiable",         menu:"2-door LG. 2 years old, works perfectly.",            payment:["cash"] },
];

function getCat(id) { return CATEGORIES.find(c=>c.id===id); }
function getPayment(id) { return PAYMENT_METHODS.find(p=>p.id===id); }

function playPing() {
  try {
    const ctx = new (window.AudioContext||window.webkitAudioContext)();
    [[1200,600,0],[1500,750,0.28]].forEach(([f1,f2,d])=>{
      const o=ctx.createOscillator(),g=ctx.createGain();
      o.connect(g); g.connect(ctx.destination); o.type="sine";
      o.frequency.setValueAtTime(f1,ctx.currentTime+d);
      o.frequency.exponentialRampToValueAtTime(f2,ctx.currentTime+d+0.2);
      g.gain.setValueAtTime(0.35,ctx.currentTime+d);
      g.gain.exponentialRampToValueAtTime(0.001,ctx.currentTime+d+0.45);
      o.start(ctx.currentTime+d); o.stop(ctx.currentTime+d+0.5);
    });
  } catch(e){}
}

// ─── Shared UI ────────────────────────────────────────────────────────────────
function PulsingDot({color,size=14,status="open"}) {
  const active=status==="open"||status==="soon";
  const dc=status==="soon"?"#F9C74F":active?color:"#444";
  const spd=status==="soon"?"3.4s":"1.8s";
  return (
    <span style={{position:"relative",display:"inline-block",width:size,height:size}}>
      {active&&<span style={{position:"absolute",inset:-6,borderRadius:"50%",background:dc+"44",animation:`ping ${spd} cubic-bezier(0,0,0.2,1) infinite`}}/>}
      <span style={{display:"block",width:size,height:size,borderRadius:"50%",background:dc,boxShadow:active?`0 0 8px ${dc}99`:"none",transition:"all 0.3s"}}/>
    </span>
  );
}

function Stars({rating,reviews,theme}) {
  return (
    <span style={{fontSize:10,color:"#F9C74F"}}>
      {"★".repeat(Math.floor(rating))}{"☆".repeat(5-Math.floor(rating))}
      <span style={{color:theme.sub,marginLeft:3,fontSize:9}}>{rating} ({reviews})</span>
    </span>
  );
}

// ─── Chat ─────────────────────────────────────────────────────────────────────
function ChatView({merchant,onBack,theme}) {
  const cat=getCat(merchant.cat);
  const [messages,setMessages]=useState([]);
  const [input,setInput]=useState("");
  const [loading,setLoading]=useState(false);
  const [handedOff,setHandedOff]=useState(false);
  const [showPay,setShowPay]=useState(false);
  const [reported,setReported]=useState(false);
  const [showReport,setShowReport]=useState(false);
  const [reportReason,setReportReason]=useState("");
  const bottomRef=useRef(null);
  const acc=theme.accent; const bdr=theme.border; const surf=theme.surface; const surf2=theme.surface2;

  useEffect(()=>{
    setMessages([{role:"assistant",isAi:true,text:`Hey! 👋 I'm the assistant for **${merchant.name}**. I can help with services, pricing and availability. How can I help?`}]);
  },[]);
  useEffect(()=>{bottomRef.current?.scrollIntoView({behavior:"smooth"})},[messages,loading]);

  async function send(override) {
    const text=(override??input).trim(); if(!text||loading)return;
    setInput("");
    const updated=[...messages,{role:"user",text}];
    setMessages(updated); setLoading(true);
    const history=updated.filter((m,i)=>!(m.isAi&&i===0)).map(m=>({role:m.role==="user"?"user":"assistant",content:m.text.replace(/\*\*/g,"")}));
    const payStr=merchant.payment.map(p=>getPayment(p)?.label).join(", ");
    const sys=`You are a friendly AI assistant for "${merchant.name}" on a local merchant discovery app. Category: ${cat.label}. Description: ${merchant.desc}. Services: ${merchant.menu}. Distance: ${merchant.dist}km. Payment: ${payStr}. Rating: ${merchant.rating}/5 (${merchant.reviews} reviews). Rules: 2-3 sentences max. Warm, street-smart. If complex/complaint/owner requested: start with [HANDOFF]. Never invent facts.`;
    try {
      const res=await fetch("https://api.anthropic.com/v1/messages",{method:"POST",headers:{"Content-Type":"application/json"},body:JSON.stringify({model:"claude-sonnet-4-20250514",max_tokens:1000,system:sys,messages:history})});
      const data=await res.json();
      const raw=data.content?.[0]?.text||"Something went wrong.";
      const isHandoff=raw.startsWith("[HANDOFF]");
      if(isHandoff) playPing();
      setMessages(prev=>[...prev,{role:"assistant",isAi:true,text:isHandoff?raw.replace("[HANDOFF]","").trim():raw,isHandoff}]);
    } catch { setMessages(prev=>[...prev,{role:"assistant",isAi:true,text:"Connection issue — try again."}]); }
    finally { setLoading(false); }
  }

  return (
    <div style={{display:"flex",flexDirection:"column",height:"100vh",background:theme.bg,fontFamily:theme.font,maxWidth:420,margin:"0 auto"}}>
      <div style={{padding:"16px 20px",borderBottom:`1px solid ${bdr}`,display:"flex",alignItems:"center",gap:12,flexShrink:0}}>
        <button onClick={onBack} style={{background:"none",border:"none",color:theme.sub,cursor:"pointer",fontSize:20}}>←</button>
        <PulsingDot color={cat.color} size={10} status="open"/>
        <div style={{flex:1}}>
          <div style={{fontFamily:theme.titleFont,fontSize:15,fontWeight:700,color:theme.text}}>{merchant.name}</div>
          <div style={{fontSize:10,color:theme.sub}}>{handedOff?"💬 Owner joined":"🤖 AI assistant"} · {merchant.dist}km</div>
        </div>
        <button onClick={()=>setShowPay(!showPay)} style={{fontSize:10,padding:"3px 10px",borderRadius:10,background:acc+"22",color:acc,border:"none",cursor:"pointer",fontFamily:theme.font}}>💳</button>
        <button onClick={()=>setShowReport(!showReport)} style={{fontSize:10,padding:"3px 10px",borderRadius:10,background:"#ff000015",color:"#ff7777",border:"none",cursor:"pointer",fontFamily:theme.font}}>🚩</button>
      </div>

      {showPay&&<div style={{padding:"10px 16px",borderBottom:`1px solid ${bdr}`,background:surf,display:"flex",gap:8,flexWrap:"wrap"}}>
        {merchant.payment.map(pid=>{const p=getPayment(pid);return p?<span key={pid} style={{padding:"5px 12px",borderRadius:20,background:surf2,border:`1px solid ${bdr}`,fontSize:11,color:theme.sub}}>{p.icon} {p.label}</span>:null;})}
      </div>}

      {showReport&&!reported&&(
        <div style={{padding:"14px 16px",borderBottom:`1px solid #ff000033`,background:"#ff000010"}}>
          <div style={{fontSize:11,color:"#ff9999",marginBottom:8}}>Report {merchant.name} for:</div>
          {["Fake listing","Rude behaviour","Wrong info","Scam"].map(r=>(
            <button key={r} onClick={()=>setReportReason(r)} style={{marginRight:6,marginBottom:6,padding:"5px 12px",borderRadius:20,border:`1px solid ${reportReason===r?"#ff777766":"#ff000033"}`,background:reportReason===r?"#ff000025":"transparent",color:"#ff9999",fontSize:10,cursor:"pointer",fontFamily:theme.font}}>{r}</button>
          ))}
          {reportReason&&<button onClick={()=>{setReported(true);setShowReport(false);}} style={{display:"block",marginTop:6,padding:"8px 20px",borderRadius:20,border:"none",background:"#cc0000",color:"#fff",fontSize:11,cursor:"pointer",fontFamily:theme.font}}>Submit Report</button>}
        </div>
      )}
      {reported&&<div style={{padding:"8px 16px",background:"#ff000012",borderBottom:`1px solid #ff000022`,fontSize:11,color:"#ff9999",textAlign:"center"}}>🚩 Report submitted. Reviewed within 24h. Merchants can be banned for 2 weeks.</div>}

      <div style={{flex:1,overflowY:"auto",padding:"16px 16px 8px"}}>
        {messages.map((msg,i)=>{
          if(msg.isSystem) return <div key={i} style={{textAlign:"center",margin:"10px 0",fontSize:11,color:"#43AA8B",background:"#43AA8B18",borderRadius:8,padding:"8px 14px"}}>{msg.text}</div>;
          const isUser=msg.role==="user";
          return (
            <div key={i} style={{display:"flex",flexDirection:"column",alignItems:isUser?"flex-end":"flex-start",marginBottom:14,animation:"fadeIn 0.2s ease"}}>
              {!isUser&&<div style={{fontSize:9,color:msg.isHandoff?acc:theme.sub,marginBottom:4,letterSpacing:1}}>{msg.isHandoff?"🔔 HANDING OFF":"🤖 AI AGENT"}</div>}
              <div style={{maxWidth:"84%",padding:"10px 14px",borderRadius:14,borderBottomRightRadius:isUser?4:14,borderBottomLeftRadius:isUser?14:4,background:isUser?acc:surf2,border:isUser?"none":`1px solid ${msg.isHandoff?acc+"55":bdr}`,fontSize:13,lineHeight:1.6,color:isUser?"#fff":theme.text}}>
                {msg.text.split(/(\*\*[^*]+\*\*)/).map((p,j)=>p.startsWith("**")?<strong key={j} style={{color:isUser?"#fff":theme.text}}>{p.replace(/\*\*/g,"")}</strong>:p)}
              </div>
              {msg.isHandoff&&!handedOff&&<button onClick={()=>{setHandedOff(true);setMessages(pr=>[...pr,{role:"system",isSystem:true,text:`✅ ${merchant.name} has been notified.`}]);}} style={{marginTop:8,padding:"8px 20px",borderRadius:20,border:`1px solid ${acc}66`,background:acc+"22",color:acc,fontSize:11,cursor:"pointer",fontFamily:theme.font}}>📲 NOTIFY OWNER</button>}
            </div>
          );
        })}
        {loading&&<div style={{display:"flex",marginBottom:14}}><div style={{padding:"12px 16px",borderRadius:14,borderBottomLeftRadius:4,background:surf2,border:`1px solid ${bdr}`,display:"flex",gap:5,alignItems:"center"}}>{[0,1,2].map(i=><span key={i} style={{width:6,height:6,borderRadius:"50%",background:theme.sub,display:"block",animation:`blink 1.2s ${i*0.2}s infinite`}}/>)}</div></div>}
        <div ref={bottomRef}/>
      </div>

      {messages.length===1&&!loading&&(
        <div style={{padding:"4px 16px 8px",display:"flex",gap:8,flexWrap:"wrap"}}>
          {["What's your price?","Available now?","How do I order?","Talk to owner"].map(q=>(
            <button key={q} onClick={()=>send(q)} style={{padding:"6px 14px",borderRadius:20,border:`1px solid ${bdr}`,background:"transparent",color:theme.sub,fontSize:11,cursor:"pointer",fontFamily:theme.font}}>{q}</button>
          ))}
        </div>
      )}

      <div style={{padding:"10px 16px 22px",borderTop:`1px solid ${bdr}`,display:"flex",gap:10,flexShrink:0}}>
        <input value={input} onChange={e=>setInput(e.target.value)} onKeyDown={e=>e.key==="Enter"&&send()} placeholder={handedOff?`Message ${merchant.name.split(" ")[0]}...`:"Ask anything..."} style={{flex:1,background:surf,border:`1px solid ${bdr}`,borderRadius:24,padding:"10px 16px",color:theme.text,fontSize:13,fontFamily:theme.font,outline:"none"}}/>
        <button onClick={()=>send()} disabled={!input.trim()||loading} style={{width:42,height:42,borderRadius:"50%",border:"none",background:input.trim()&&!loading?acc:surf2,color:input.trim()&&!loading?"#fff":theme.sub,cursor:input.trim()&&!loading?"pointer":"default",fontSize:18,display:"flex",alignItems:"center",justifyContent:"center",transition:"all 0.2s",flexShrink:0}}>→</button>
      </div>
    </div>
  );
}

// ─── Inbox ────────────────────────────────────────────────────────────────────
function MerchantInbox({theme}) {
  const CHATS=[
    {id:1,preview:"Is tyre fitting available now?",time:"2m",unread:true,handoff:false},
    {id:2,preview:"Need a custom gate quote",time:"11m",unread:true,handoff:true},
    {id:3,preview:"What braiding styles do you offer?",time:"34m",unread:false,handoff:false},
  ];
  return (
    <div style={{paddingBottom:80}}>
      <div style={{fontSize:10,color:theme.sub,letterSpacing:2,padding:"16px 20px 10px"}}>INCOMING CHATS</div>
      {CHATS.map(c=>(
        <div key={c.id} style={{padding:"14px 20px",borderBottom:`1px solid ${theme.surface2}`,display:"flex",gap:12,alignItems:"center",background:c.handoff?theme.accent+"08":"transparent"}}>
          <div style={{width:38,height:38,borderRadius:"50%",background:theme.surface2,display:"flex",alignItems:"center",justifyContent:"center",fontSize:16,position:"relative",flexShrink:0}}>
            👤{c.unread&&<span style={{position:"absolute",top:1,right:1,width:8,height:8,borderRadius:"50%",background:c.handoff?theme.accent:"#4CC9F0"}}/>}
          </div>
          <div style={{flex:1,minWidth:0}}>
            <div style={{display:"flex",justifyContent:"space-between",marginBottom:3}}>
              <span style={{fontSize:11,color:c.unread?theme.text:theme.sub}}>Customer nearby</span>
              <span style={{fontSize:9,color:theme.sub}}>{c.time} ago</span>
            </div>
            <div style={{fontSize:11,color:theme.sub,whiteSpace:"nowrap",overflow:"hidden",textOverflow:"ellipsis"}}>{c.handoff?"🔴 ":"🤖 "}{c.preview}</div>
          </div>
          {c.handoff&&<span style={{fontSize:9,padding:"4px 9px",borderRadius:8,background:theme.accent+"22",color:theme.accent,flexShrink:0}}>TAKE OVER</span>}
        </div>
      ))}
      <div style={{margin:"16px 20px",padding:14,background:theme.surface,borderRadius:12,border:`1px solid ${theme.border}`,fontSize:11,color:theme.sub,lineHeight:1.8}}>
        🤖 AI handles routine queries automatically.<br/>🔴 Red dot = customer requested you directly.
      </div>
    </div>
  );
}

// ─── Status panel ─────────────────────────────────────────────────────────────
function StatusPanel({myCategory,customCats,theme}) {
  const cat=getCat(myCategory)||customCats.find(c=>c.id===myCategory);
  const [myStatus,setMyStatus]=useState("offline");
  const [backIn,setBackIn]=useState(10);
  const [showTimer,setShowTimer]=useState(false);
  const acc=cat?.color||theme.accent;
  const statuses={offline:{label:"YOU'RE OFFLINE",sub:"Invisible to others"},open:{label:"YOU'RE LIVE",sub:"🤖 AI is handling chats"},soon:{label:"BACK SOON",sub:`Back in ${backIn} min`}};
  const s=statuses[myStatus];
  const borderCol=myStatus==="open"?acc+"66":myStatus==="soon"?"#F9C74F66":theme.border;
  return (
    <div>
      <div style={{background:theme.surface,border:`2px solid ${borderCol}`,borderRadius:20,padding:24,textAlign:"center",transition:"border 0.4s"}}>
        <div style={{marginBottom:20,display:"flex",justifyContent:"center"}}><PulsingDot color={acc} size={28} status={myStatus==="open"?"open":myStatus==="soon"?"soon":"offline"}/></div>
        <div style={{fontFamily:theme.titleFont,fontSize:20,fontWeight:800,color:theme.text,marginBottom:6}}>{s.label}</div>
        <div style={{fontSize:11,color:theme.sub,marginBottom:20}}>{s.sub}</div>
        <div style={{display:"flex",gap:8,justifyContent:"center"}}>
          {[["open","🟢 LIVE"],["soon","🟡 SOON"],["offline","⚫ OFF"]].map(([st,l])=>(
            <button key={st} onClick={()=>{setMyStatus(st);setShowTimer(st==="soon");}} style={{padding:"11px 14px",borderRadius:50,border:st==="offline"&&myStatus==="offline"?"1px solid #ff000044":"none",cursor:"pointer",fontFamily:theme.font,fontSize:11,fontWeight:700,background:myStatus===st?(st==="open"?acc:st==="soon"?"#F9C74F":"#ff000022"):theme.surface2,color:myStatus===st?(st==="soon"?"#111":st==="offline"?"#ff6666":"#fff"):theme.sub,transition:"all 0.2s"}}>{l}</button>
          ))}
        </div>
      </div>
      {showTimer&&(
        <div style={{marginTop:12,background:theme.surface,border:"1px solid #F9C74F44",borderRadius:14,padding:"14px 16px",animation:"fadeIn 0.2s ease"}}>
          <div style={{fontSize:10,color:"#F9C74F",letterSpacing:2,marginBottom:12}}>BACK IN HOW LONG?</div>
          <div style={{display:"flex",gap:8}}>{[5,10,15,20,30].map(t=><button key={t} onClick={()=>setBackIn(t)} style={{flex:1,padding:"10px 4px",borderRadius:10,border:"none",cursor:"pointer",fontFamily:theme.font,fontSize:12,background:backIn===t?"#F9C74F":theme.surface2,color:backIn===t?"#111":theme.sub,fontWeight:backIn===t?700:400}}>{t}m</button>)}</div>
        </div>
      )}
    </div>
  );
}

// ─── Payment panel ────────────────────────────────────────────────────────────
function PaymentPanel({theme}) {
  const [enabled,setEnabled]=useState(["cash"]);
  const [details,setDetails]=useState({});
  function toggle(id){setEnabled(p=>p.includes(id)?p.filter(x=>x!==id):[...p,id]);}
  return (
    <div style={{padding:"20px 20px 80px"}}>
      <div style={{fontSize:10,color:theme.sub,letterSpacing:2,marginBottom:16}}>ACCEPTED PAYMENT METHODS</div>
      {PAYMENT_METHODS.map(p=>{
        const on=enabled.includes(p.id);
        return (
          <div key={p.id} style={{background:theme.surface,border:`1px solid ${on?theme.accent+"55":theme.border}`,borderRadius:14,overflow:"hidden",marginBottom:10,transition:"border 0.2s"}}>
            <div style={{display:"flex",alignItems:"center",gap:14,padding:"14px 16px",cursor:"pointer"}} onClick={()=>toggle(p.id)}>
              <span style={{fontSize:18}}>{p.icon}</span>
              <span style={{fontSize:13,color:on?theme.text:theme.sub,flex:1,fontFamily:theme.font}}>{p.label}</span>
              <span style={{width:22,height:22,borderRadius:6,border:`1px solid ${on?theme.accent:"#444"}`,background:on?theme.accent:"transparent",display:"flex",alignItems:"center",justifyContent:"center",fontSize:11,color:"#fff",transition:"all 0.2s"}}>{on?"✓":""}</span>
            </div>
            {on&&p.id!=="cash"&&(
              <div style={{padding:"0 16px 14px"}}>
                <input value={details[p.id]||""} onChange={e=>setDetails(pr=>({...pr,[p.id]:e.target.value}))} placeholder={p.id==="snapscan"?"SnapScan username":p.id==="eft"?"Account number":"Phone / reference"} style={{width:"100%",background:theme.surface2,border:`1px solid ${theme.border}`,borderRadius:8,padding:"8px 12px",color:theme.text,fontSize:12,fontFamily:theme.font,outline:"none"}}/>
              </div>
            )}
          </div>
        );
      })}
      <div style={{padding:14,background:theme.surface,borderRadius:12,border:`1px solid ${theme.border}`,fontSize:11,color:theme.sub,lineHeight:1.8}}>
        💳 Payment options appear in every customer chat.<br/>💰 Spota never touches your money — all direct.
      </div>
    </div>
  );
}

// ─── Main App ─────────────────────────────────────────────────────────────────
function SpotaApp({theme}) {
  const [view,setView]=useState("map");
  const [selected,setSelected]=useState(null);
  const [chatWith,setChatWith]=useState(null);
  const [myCategory,setMyCategory]=useState("mechanic");
  const [customCats,setCustomCats]=useState([]);
  const [customInput,setCustomInput]=useState("");
  const [filterCat,setFilterCat]=useState(null);
  const [merchantTab,setMerchantTab]=useState("status");
  const [searchQ,setSearchQ]=useState("");
  const [radiusKm,setRadiusKm]=useState(2);
  const [fullscreen,setFullscreen]=useState(false);
  const [suggestionDismissed,setSuggestionDismissed]=useState(false);
  const [suggestionsOff,setSuggestionsOff]=useState(false);
  const [showSuggestion,setShowSuggestion]=useState(false);

  useEffect(()=>{
    if(suggestionsOff||suggestionDismissed)return;
    const t=setTimeout(()=>setShowSuggestion(true),3500);
    return()=>clearTimeout(t);
  },[suggestionsOff,suggestionDismissed]);

  const allCats=[...CATEGORIES,...customCats];
  const acc=theme.accent; const bdr=theme.border; const surf=theme.surface; const surf2=theme.surface2;

  const filtered=MOCK_MERCHANTS.filter(m=>{
    if(parseFloat(m.dist)>radiusKm)return false;
    if(filterCat&&m.cat!==filterCat)return false;
    if(searchQ){const q=searchQ.toLowerCase(); const c=getCat(m.cat); return m.name.toLowerCase().includes(q)||m.desc.toLowerCase().includes(q)||(c?.label.toLowerCase().includes(q));}
    return true;
  });

  function addCustomCat(){
    const name=customInput.trim(); if(!name)return;
    const id="custom_"+name.toLowerCase().replace(/\s+/g,"_");
    if(!customCats.find(c=>c.id===id)) setCustomCats(p=>[...p,{id,label:name,color:"#FF6B35",emoji:"✨",mobile:false,custom:true}]);
    setMyCategory(id); setCustomInput("");
  }

  if(chatWith) return <ChatView merchant={chatWith} onBack={()=>setChatWith(null)} theme={theme}/>;

  return (
    <div style={{fontFamily:theme.font,background:theme.bg,minHeight:"100vh",color:theme.text,maxWidth:420,margin:"0 auto"}}>
      {/* Proximity toast */}
      {showSuggestion&&!suggestionDismissed&&!suggestionsOff&&!fullscreen&&view==="map"&&(
        <div style={{position:"fixed",top:80,left:"50%",transform:"translateX(-50%)",zIndex:200,background:surf,border:`1px solid ${acc}55`,borderRadius:14,padding:"10px 16px",display:"flex",alignItems:"center",gap:10,boxShadow:`0 4px 24px ${acc}22`,maxWidth:360,width:"calc(100% - 32px)",animation:"fadeIn 0.3s ease"}}>
          <span style={{fontSize:20}}>🔧</span>
          <div style={{flex:1}}>
            <div style={{fontSize:12,color:theme.text,fontWeight:600}}>Auto repair nearby</div>
            <div style={{fontSize:10,color:theme.sub}}>Zithulele Spares is 1.1km away</div>
          </div>
          <button onClick={()=>setSuggestionDismissed(true)} style={{background:"none",border:"none",color:theme.sub,cursor:"pointer",fontSize:18,lineHeight:1}}>×</button>
          <button onClick={()=>{setSuggestionsOff(true);setSuggestionDismissed(true);}} style={{background:"none",border:"none",color:theme.sub,cursor:"pointer",fontSize:9,textDecoration:"underline",fontFamily:theme.font,whiteSpace:"nowrap"}}>turn off</button>
        </div>
      )}

      {/* Header */}
      <div style={{padding:"18px 20px 12px",borderBottom:`1px solid ${bdr}`,display:"flex",alignItems:"center",justifyContent:"space-between"}}>
        <div>
          <div style={{fontFamily:theme.titleFont,fontSize:24,fontWeight:800,color:theme.text,letterSpacing:theme.id==="spota"?"-0.5px":"0"}}>
            {theme.name}<span style={{color:acc}}>{theme.dot}</span>
          </div>
          <div style={{fontSize:9,color:theme.sub,letterSpacing:2,marginTop:1}}>{theme.tagline}</div>
        </div>
        <div style={{display:"flex",gap:8}}>
          {[["map","MAP"],["merchant","MY DOT"]].map(([v,l])=>(
            <button key={v} onClick={()=>setView(v)} style={{padding:"6px 14px",borderRadius:20,border:"none",cursor:"pointer",fontSize:11,letterSpacing:1,fontFamily:theme.font,background:view===v?acc:surf2,color:view===v?"#fff":theme.sub,transition:"all 0.2s"}}>{l}</button>
          ))}
        </div>
      </div>

      {/* MAP VIEW */}
      {view==="map"&&(
        <div>
          {/* Search */}
          <div style={{padding:"12px 16px 0"}}>
            <div style={{position:"relative"}}>
              <span style={{position:"absolute",left:12,top:"50%",transform:"translateY(-50%)",fontSize:13,color:theme.sub}}>🔍</span>
              <input value={searchQ} onChange={e=>setSearchQ(e.target.value)} placeholder="Search services, names, keywords..." style={{width:"100%",background:surf,border:`1px solid ${bdr}`,borderRadius:24,padding:"9px 36px",color:theme.text,fontSize:12,fontFamily:theme.font,outline:"none",boxSizing:"border-box"}}/>
              {searchQ&&<button onClick={()=>setSearchQ("")} style={{position:"absolute",right:12,top:"50%",transform:"translateY(-50%)",background:"none",border:"none",color:theme.sub,cursor:"pointer",fontSize:18}}>×</button>}
            </div>
          </div>

          {/* Filter bar */}
          <div style={{padding:"10px 16px",display:"flex",gap:8,overflowX:"auto",scrollbarWidth:"none"}}>
            <button onClick={()=>setFilterCat(null)} style={{flexShrink:0,padding:"5px 14px",borderRadius:20,border:`1px solid ${!filterCat?acc:"#33333388"}`,background:!filterCat?acc+"20":"transparent",color:!filterCat?acc:theme.sub,fontSize:10,cursor:"pointer",fontFamily:theme.font}}>ALL</button>
            {CATEGORIES.map(cat=>(
              <button key={cat.id} onClick={()=>setFilterCat(filterCat===cat.id?null:cat.id)} style={{flexShrink:0,padding:"5px 12px",borderRadius:20,border:`1px solid ${filterCat===cat.id?cat.color:"#33333388"}`,background:filterCat===cat.id?cat.color+"20":"transparent",color:filterCat===cat.id?cat.color:theme.sub,fontSize:10,cursor:"pointer",fontFamily:theme.font}}>{cat.emoji}</button>
            ))}
          </div>

          {/* Radar */}
          {!fullscreen&&(
            <div style={{margin:"0 16px",borderRadius:16,background:theme.dark?"#0d0d18":surf,border:`1px solid ${bdr}`,position:"relative",height:280,overflow:"hidden"}}>
              {[1,2,3].map(i=><div key={i} style={{position:"absolute",inset:0,borderRadius:"50%",border:`1px solid ${theme.dark?"#ffffff08":"#00000008"}`,margin:`${i*15}%`}}/>)}
              {/* Dynamic radius ring */}
              <div style={{position:"absolute",left:"50%",top:"50%",width:`${Math.min(radiusKm/5*95,92)}%`,paddingBottom:`${Math.min(radiusKm/5*95,92)}%`,transform:"translate(-50%,-50%)",borderRadius:"50%",border:`1px dashed ${acc}55`,pointerEvents:"none",transition:"all 0.3s"}}/>
              <div style={{position:"absolute",left:"50%",top:0,bottom:0,width:1,background:theme.dark?"#ffffff06":"#00000006"}}/>
              <div style={{position:"absolute",top:"50%",left:0,right:0,height:1,background:theme.dark?"#ffffff06":"#00000006"}}/>
              <div style={{position:"absolute",left:"50%",top:"50%",width:"50%",height:1,background:`linear-gradient(to right,transparent,${acc}55)`,transformOrigin:"left center",animation:"radarSweep 4s linear infinite"}}/>
              <div style={{position:"absolute",left:"50%",top:"50%",transform:"translate(-50%,-50%)",zIndex:10}}>
                <span style={{display:"block",width:10,height:10,borderRadius:"50%",background:acc,boxShadow:`0 0 0 3px ${acc}44`}}/>
              </div>
              <div style={{position:"absolute",left:"50%",top:"calc(50% + 14px)",transform:"translateX(-50%)",fontSize:7,color:theme.dark?"#ffffff44":theme.sub,letterSpacing:1}}>YOU</div>
              {filtered.map(m=>{const cat=getCat(m.cat);return(
                <button key={m.id} onClick={()=>setSelected(selected?.id===m.id?null:m)} style={{position:"absolute",left:`${m.x}%`,top:`${m.y}%`,transform:"translate(-50%,-50%)",background:"none",border:"none",cursor:"pointer",zIndex:5,padding:8}}>
                  <PulsingDot color={cat.color} size={12} status={m.status}/>
                </button>
              );})}
              <button onClick={()=>setFullscreen(true)} style={{position:"absolute",top:8,right:8,background:theme.dark?"#ffffff18":"#00000018",border:"none",borderRadius:8,padding:"5px 8px",cursor:"pointer",color:theme.dark?"#fff":theme.text,fontSize:12}}>⛶</button>
              {selected&&(()=>{const cat=getCat(selected.cat); return(
                <div style={{position:"absolute",bottom:12,left:12,right:12,background:theme.dark?"#13131f":surf,border:`1px solid ${cat.color}44`,borderRadius:12,padding:"12px 14px",animation:"fadeIn 0.2s ease",zIndex:20}}>
                  <div style={{display:"flex",alignItems:"center",gap:8,marginBottom:4}}>
                    <PulsingDot color={cat.color} size={10} status={selected.status}/>
                    <span style={{fontFamily:theme.titleFont,fontSize:14,fontWeight:700,color:theme.text}}>{selected.name}</span>
                    {selected.status==="soon"?<span style={{marginLeft:"auto",fontSize:9,color:"#F9C74F",background:"#F9C74F22",padding:"2px 8px",borderRadius:10}}>⏱ {selected.backIn}m</span>:<span style={{marginLeft:"auto",fontSize:9,color:cat.color,background:cat.color+"22",padding:"2px 8px",borderRadius:10}}>{cat.emoji}</span>}
                  </div>
                  <div style={{marginBottom:5}}><Stars rating={selected.rating} reviews={selected.reviews} theme={theme}/></div>
                  <div style={{fontSize:11,color:theme.sub,lineHeight:1.5,marginBottom:8}}>{selected.desc}</div>
                  <div style={{display:"flex",justifyContent:"space-between",alignItems:"center"}}>
                    <div style={{fontSize:10,color:theme.sub}}>{selected.dist}km away</div>
                    <button onClick={()=>setChatWith(selected)} style={{padding:"7px 18px",borderRadius:20,border:"none",background:cat.color,color:"#fff",fontSize:11,cursor:"pointer",fontFamily:theme.font}}>💬 CHAT</button>
                  </div>
                </div>
              );})()} 
            </div>
          )}

          {/* Fullscreen radar */}
          {fullscreen&&(
            <div style={{position:"fixed",inset:0,zIndex:100,background:theme.dark?"#0a0a0f":surf,display:"flex",flexDirection:"column"}}>
              <div style={{flex:1,position:"relative"}}>
                {[1,2,3,4,5].map(i=><div key={i} style={{position:"absolute",inset:0,borderRadius:"50%",border:`1px solid ${theme.dark?"#ffffff06":"#00000006"}`,margin:`${i*8}%`}}/>)}
                <div style={{position:"absolute",left:"50%",top:"50%",width:`${Math.min(radiusKm/5*90,88)}%`,paddingBottom:`${Math.min(radiusKm/5*90,88)}%`,transform:"translate(-50%,-50%)",borderRadius:"50%",border:`2px dashed ${acc}66`,pointerEvents:"none"}}/>
                <div style={{position:"absolute",left:"50%",top:"50%",width:"50%",height:1,background:`linear-gradient(to right,transparent,${acc}77)`,transformOrigin:"left center",animation:"radarSweep 3.5s linear infinite"}}/>
                <div style={{position:"absolute",left:"50%",top:"50%",transform:"translate(-50%,-50%)",zIndex:10}}>
                  <span style={{display:"block",width:14,height:14,borderRadius:"50%",background:acc,boxShadow:`0 0 0 5px ${acc}33`}}/>
                </div>
                {filtered.map(m=>{const cat=getCat(m.cat); return(
                  <button key={m.id} onClick={()=>{setSelected(m);setFullscreen(false);}} style={{position:"absolute",left:`${m.x}%`,top:`${m.y}%`,transform:"translate(-50%,-50%)",background:"none",border:"none",cursor:"pointer",zIndex:5,padding:10}}>
                    <PulsingDot color={cat.color} size={14} status={m.status}/>
                    <div style={{position:"absolute",top:"100%",left:"50%",transform:"translateX(-50%)",fontSize:8,color:cat.color,whiteSpace:"nowrap",marginTop:4,textShadow:"0 1px 4px #00000088"}}>{m.name.split(" ")[0]}</div>
                  </button>
                );})}
              </div>
              <div style={{padding:"12px 20px 24px",borderTop:`1px solid ${bdr}`,display:"flex",alignItems:"center",gap:12,background:theme.bg}}>
                <div style={{flex:1}}>
                  <div style={{fontSize:9,color:theme.sub,marginBottom:6}}>RADIUS: {radiusKm}km</div>
                  <input type="range" min="0.5" max="5" step="0.5" value={radiusKm} onChange={e=>setRadiusKm(parseFloat(e.target.value))} style={{width:"100%",accentColor:acc,cursor:"pointer"}}/>
                </div>
                <button onClick={()=>setFullscreen(false)} style={{padding:"10px 18px",borderRadius:20,border:"none",background:acc,color:"#fff",cursor:"pointer",fontFamily:theme.font,fontSize:12,flexShrink:0}}>✕ EXIT</button>
              </div>
            </div>
          )}

          {/* Radius slider (normal mode) */}
          {!fullscreen&&(
            <div style={{padding:"12px 20px 6px"}}>
              <div style={{display:"flex",justifyContent:"space-between",marginBottom:5}}>
                <span style={{fontSize:10,color:theme.sub,letterSpacing:1}}>SEARCH RADIUS</span>
                <span style={{fontSize:10,color:acc,fontWeight:600}}>{radiusKm}km · {filtered.length} visible</span>
              </div>
              <input type="range" min="0.5" max="5" step="0.5" value={radiusKm} onChange={e=>setRadiusKm(parseFloat(e.target.value))} style={{width:"100%",accentColor:acc,cursor:"pointer"}}/>
            </div>
          )}

          {/* Merchant list */}
          {!fullscreen&&(
            <div style={{padding:"8px 16px 100px"}}>
              <div style={{fontSize:10,color:theme.sub,letterSpacing:2,marginBottom:12}}>{filtered.length} OPEN NEAR YOU{searchQ?` · "${searchQ}"`:""}</div>
              {filtered.length===0&&<div style={{textAlign:"center",padding:"40px 20px",color:theme.sub,fontSize:12}}>Nothing found in this area. Try expanding radius.</div>}
              {filtered.map(m=>{const cat=getCat(m.cat); return(
                <div key={m.id} style={{display:"flex",alignItems:"center",gap:14,padding:"12px 0",borderBottom:`1px solid ${surf2}`}}>
                  <div onClick={()=>setSelected(selected?.id===m.id?null:m)} style={{display:"flex",alignItems:"center",gap:14,flex:1,cursor:"pointer"}}>
                    <PulsingDot color={cat.color} size={10} status={m.status}/>
                    <div style={{flex:1}}>
                      <div style={{fontSize:13,color:theme.text,fontWeight:500}}>{m.name}</div>
                      <div style={{display:"flex",alignItems:"center",gap:8,marginTop:2,flexWrap:"wrap"}}>
                        <span style={{fontSize:9,color:m.status==="soon"?"#F9C74F":theme.sub}}>{m.status==="soon"?`⏱ ${m.backIn}m`:`${cat.emoji} ${cat.label}`}</span>
                        <Stars rating={m.rating} reviews={m.reviews} theme={theme}/>
                      </div>
                    </div>
                    <div style={{fontSize:10,color:theme.sub}}>{m.dist}km</div>
                  </div>
                  <button onClick={()=>setChatWith(m)} style={{padding:"5px 12px",borderRadius:16,border:`1px solid ${cat.color}44`,background:"transparent",color:cat.color,fontSize:11,cursor:"pointer",fontFamily:theme.font,flexShrink:0}}>💬</button>
                </div>
              );})}
            </div>
          )}
        </div>
      )}

      {/* MERCHANT VIEW */}
      {view==="merchant"&&(
        <div>
          <div style={{display:"flex",borderBottom:`1px solid ${bdr}`}}>
            {[["status","STATUS"],["payment","💳 PAY"],["inbox","INBOX ●"]].map(([t,l])=>(
              <button key={t} onClick={()=>setMerchantTab(t)} style={{flex:1,padding:"11px 4px",border:"none",background:"none",fontFamily:theme.font,fontSize:10,letterSpacing:0.5,cursor:"pointer",color:merchantTab===t?acc:theme.sub,borderBottom:merchantTab===t?`2px solid ${acc}`:"2px solid transparent"}}>{l}</button>
            ))}
          </div>

          {merchantTab==="status"&&(
            <div style={{padding:"24px 20px"}}>
              <StatusPanel myCategory={myCategory} customCats={customCats} theme={theme}/>
              <div style={{height:24}}/>
              <div style={{fontSize:10,color:theme.sub,letterSpacing:2,marginBottom:12}}>WHAT ARE YOU OFFERING?</div>
              <div style={{display:"flex",flexDirection:"column",gap:8,paddingBottom:80}}>
                {allCats.map(cat=>(
                  <button key={cat.id} onClick={()=>setMyCategory(cat.id)} style={{display:"flex",alignItems:"center",gap:14,padding:"13px 16px",borderRadius:12,border:`1px solid ${myCategory===cat.id?cat.color+"88":bdr}`,background:myCategory===cat.id?cat.color+"15":surf,cursor:"pointer",textAlign:"left",transition:"all 0.2s"}}>
                    <span style={{width:10,height:10,borderRadius:"50%",background:cat.color,flexShrink:0,boxShadow:myCategory===cat.id?`0 0 8px ${cat.color}`:"none"}}/>
                    <span style={{fontSize:13,fontFamily:theme.font,color:myCategory===cat.id?theme.text:theme.sub,flex:1}}>
                      {cat.emoji} {cat.label}
                      {cat.mobile&&<span style={{fontSize:9,color:cat.color,marginLeft:8}}>MOBILE</span>}
                      {cat.custom&&<span style={{fontSize:9,color:theme.sub,marginLeft:8}}>MY CATEGORY</span>}
                    </span>
                    {myCategory===cat.id&&<span style={{fontSize:10,color:cat.color}}>✓</span>}
                  </button>
                ))}
                {myCategory==="other"&&(
                  <div style={{padding:14,background:surf,border:`1px solid ${bdr}`,borderRadius:12,marginTop:4,animation:"fadeIn 0.2s ease"}}>
                    <div style={{fontSize:10,color:theme.sub,letterSpacing:1,marginBottom:8}}>NOT IN THE LIST? CREATE YOUR OWN</div>
                    <div style={{display:"flex",gap:8}}>
                      <input value={customInput} onChange={e=>setCustomInput(e.target.value)} onKeyDown={e=>e.key==="Enter"&&addCustomCat()} placeholder="e.g. Photography, Tutoring..." style={{flex:1,background:surf2,border:`1px solid ${bdr}`,borderRadius:8,padding:"8px 12px",color:theme.text,fontSize:12,fontFamily:theme.font,outline:"none"}}/>
                      <button onClick={addCustomCat} style={{padding:"8px 14px",borderRadius:8,border:"none",background:acc,color:"#fff",cursor:"pointer",fontFamily:theme.font,fontSize:12}}>ADD</button>
                    </div>
                    <div style={{fontSize:10,color:theme.sub,marginTop:8}}>If others offer the same service, you'll be grouped together automatically.</div>
                  </div>
                )}
              </div>
            </div>
          )}
          {merchantTab==="payment"&&<PaymentPanel theme={theme}/>}
          {merchantTab==="inbox"&&<MerchantInbox theme={theme}/>}
        </div>
      )}
    </div>
  );
}

// ─── Root ─────────────────────────────────────────────────────────────────────
export default function App() {
  const [themeName,setThemeName]=useState("spota");
  const theme=THEMES[themeName];

  return (
    <div style={{background:theme.bg,minHeight:"100vh"}}>
      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=DM+Mono:wght@300;400;500&family=Syne:wght@700;800&family=Inter:wght@400;500;600;700;800&display=swap');
        @keyframes ping{75%,100%{transform:scale(2.2);opacity:0}}
        @keyframes fadeIn{from{opacity:0;transform:translateY(8px)}to{opacity:1;transform:translateY(0)}}
        @keyframes radarSweep{from{transform:rotate(0deg)}to{transform:rotate(360deg)}}
        @keyframes blink{0%,100%{opacity:0.2}50%{opacity:1}}
        *{box-sizing:border-box;margin:0;padding:0}
        ::-webkit-scrollbar{width:4px}::-webkit-scrollbar-track{background:transparent}::-webkit-scrollbar-thumb{background:#33333366;border-radius:2px}
        input[type=range]{-webkit-appearance:none;height:4px;border-radius:2px;outline:none;background:#33333355;}
        input[type=range]::-webkit-slider-thumb{-webkit-appearance:none;width:18px;height:18px;border-radius:50%;cursor:pointer;}
        input::placeholder{opacity:0.35}
      `}</style>

      {/* Brand switcher */}
      <div style={{display:"flex",justifyContent:"center",gap:6,padding:"10px 16px",background:theme.dark?"rgba(0,0,0,0.5)":"rgba(255,255,255,0.8)",backdropFilter:"blur(12px)",position:"sticky",top:0,zIndex:50,borderBottom:`1px solid ${theme.border}`}}>
        {Object.values(THEMES).map(t=>(
          <button key={t.id} onClick={()=>setThemeName(t.id)} style={{padding:"6px 20px",borderRadius:20,border:"none",cursor:"pointer",fontFamily:t.titleFont,fontSize:11,fontWeight:800,letterSpacing:t.id==="spota"?1:0,background:themeName===t.id?t.accent:"transparent",color:themeName===t.id?"#fff":theme.sub,transition:"all 0.2s"}}>
            {t.name}
          </button>
        ))}
      </div>

      <SpotaApp theme={theme}/>
    </div>
  );
}
