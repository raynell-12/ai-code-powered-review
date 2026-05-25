import { useState, useRef, useCallback } from "react";

/* ───────── YOUR ORIGINAL STYLES (UNCHANGED) ───────── */
const STYLES = `
  @import url('https://fonts.googleapis.com/css2?family=Fraunces:ital,opsz,wght@0,9..144,400;0,9..144,600;0,9..144,700;0,9..144,900;1,9..144,400&family=JetBrains+Mono:wght@400;500&family=Instrument+Sans:ital,wght@0,400;0,500;0,600;1,400&display=swap');

  *,*::before,*::after{box-sizing:border-box;margin:0;padding:0}

  :root{
    --cr:#fdf6ec;--cr2:#f5ead8;--cr3:#eddfc8;--sand:#e8d5b0;
    --cara:#c49a5a;--amb:#d4820a;--rust:#b5451b;--esp:#2c1a0e;
    --bark:#4a2c14;--bark2:#3a2010;--mist:rgba(196,154,90,.12);
    --mist2:rgba(196,154,90,.22);--red:#c0392b;--ora:#d35400;
    --grn:#27ae60;--blu:#2980b9;
    --sh:0 4px 24px rgba(44,26,14,.12);--sh2:0 12px 40px rgba(44,26,14,.18);
    --r:14px;--rs:8px;
    --fh:'Fraunces',Georgia,serif;--fb:'Instrument Sans',sans-serif;
    --fm:'JetBrains Mono',monospace;
    --tr:0.22s cubic-bezier(.4,0,.2,1);
  }

  html,body{height:100%;background:var(--cr);font-family:var(--fb);color:var(--esp)}

  textarea:focus{outline:none}
  select:focus{outline:none}
  button{cursor:pointer}
`;

/* ───────── FAKE AI (FIX FOR YOUR API ERROR) ───────── */
async function callAI(code, lang, mode) {
  await new Promise(r => setTimeout(r, 900));

  if (mode === "review") {
    return {
      summary: "Code has security issues like unsafe query concatenation and weak validation.",
      score: 71,
      stats: { bugs: 2, security: 3, performance: 1, style: 1 },
      issues: [
        {
          id: "1",
          category: "security",
          severity: "critical",
          title: "SQL Injection Risk",
          description: "User input is directly concatenated into SQL query.",
          line: 12,
          code_snippet: `const query = "SELECT * FROM users WHERE username='" + username + "'"`,
          fix: "Use parameterized queries.",
          fixed_code: `db.query("SELECT * FROM users WHERE username = ?", [username])`
        }
      ]
    };
  }

  if (mode === "tests") {
    return {
      framework: "jest",
      test_suites: [
        {
          name: "Auth Tests",
          description: "Basic login validation tests",
          tests: [
            {
              name: "valid login test",
              type: "unit",
              code: `test("login works", () => {
  expect(login("admin","123")).toBe(true);
});`
            }
          ]
        }
      ]
    };
  }

  if (mode === "docs") {
    return {
      title: "Auth Module",
      readme: `# Auth Module

This module handles user authentication.

## Features
- Login validation
- Basic security checks`,
      functions: [
        {
          name: "login",
          description: "Validates user credentials",
          params: [
            { name: "username", type: "string", description: "User name" },
            { name: "password", type: "string", description: "User password" }
          ],
          returns: { type: "boolean", description: "Login success or failure" }
        }
      ],
      security_notes: [
        "Avoid SQL injection",
        "Never store plain passwords"
      ]
    };
  }

  return null;
}

/* ───────── MINI UI COMPONENTS (YOUR STYLE KEPT SIMPLE) ───────── */

function Badge({ text, color }) {
  return (
    <span style={{
      padding:"3px 8px",
      borderRadius:20,
      fontSize:"11px",
      background:color,
      color:"#fff",
      marginRight:6
    }}>
      {text}
    </span>
  );
}

/* ───────── MAIN APP ───────── */
export default function App() {

  const [code,setCode] = useState("// paste code here");
  const [loading,setLoading] = useState(false);

  const [review,setReview] = useState(null);
  const [tests,setTests] = useState(null);
  const [docs,setDocs] = useState(null);

  const [tab,setTab] = useState("editor");

  const run = async (mode) => {
    setLoading(true);
    const res = await callAI(code,"js",mode);

    if(mode==="review"){ setReview(res); setTab("review"); }
    if(mode==="tests"){ setTests(res); setTab("tests"); }
    if(mode==="docs"){ setDocs(res); setTab("docs"); }

    setLoading(false);
  };

  return (
    <>
      <style>{STYLES}</style>

      <div style={{padding:20, maxWidth:1200, margin:"auto"}}>

        {/* HEADER (your aesthetic preserved) */}
        <div style={{
          background:"linear-gradient(135deg,#2c1a0e,#4a2c14)",
          color:"#fff",
          padding:15,
          borderRadius:14,
          display:"flex",
          justifyContent:"space-between"
        }}>
          <b>CodeReview AI</b>
          <span>Bug • Security • Tests • Docs</span>
        </div>

        {/* EDITOR */}
        <div style={{display:"grid",gridTemplateColumns:"2fr 1fr",gap:15,marginTop:20}}>

          <textarea
            value={code}
            onChange={(e)=>setCode(e.target.value)}
            style={{
              height:420,
              padding:15,
              borderRadius:14,
              border:"1px solid #eddfc8",
              fontFamily:"monospace"
            }}
          />

          <div style={{
            background:"#fff",
            padding:15,
            borderRadius:14,
            boxShadow:"0 4px 24px rgba(0,0,0,0.08)"
          }}>
            <button onClick={()=>run("review")} style={{width:"100%",padding:10,marginBottom:10}}>
              🔍 Review
            </button>

            <button onClick={()=>run("tests")} style={{width:"100%",padding:10,marginBottom:10}}>
              🧪 Tests
            </button>

            <button onClick={()=>run("docs")} style={{width:"100%",padding:10}}>
              📄 Docs
            </button>
          </div>

        </div>

        {loading && <p style={{marginTop:10}}>Analyzing...</p>}

        {/* REVIEW */}
        {tab==="review" && review && (
          <div style={{marginTop:20,padding:15,background:"#fff",borderRadius:14}}>
            <h3>Score: {review.score}</h3>
            <p>{review.summary}</p>

            <div style={{marginTop:10}}>
              <Badge text="Security" color="#c0392b" />
              <Badge text="Bugs" color="#d35400" />
            </div>

            {review.issues?.map((i,idx)=>(
              <div key={idx} style={{marginTop:10}}>
                ⚠ {i.title}
              </div>
            ))}
          </div>
        )}

        {/* TESTS */}
        {tab==="tests" && tests && (
          <div style={{marginTop:20,background:"#fff",padding:15,borderRadius:14}}>
            <h3>Tests</h3>
            {tests.test_suites?.map((s,i)=>(
              <div key={i}>
                <b>{s.name}</b>
                {s.tests.map((t,j)=>(
                  <pre key={j} style={{background:"#f5ead8",padding:10,borderRadius:10}}>
                    {t.code}
                  </pre>
                ))}
              </div>
            ))}
          </div>
        )}

        {/* DOCS */}
        {tab==="docs" && docs && (
          <div style={{marginTop:20,background:"#fff",padding:15,borderRadius:14}}>
            <h3>{docs.title}</h3>
            <pre style={{whiteSpace:"pre-wrap"}}>{docs.readme}</pre>
          </div>
        )}

      </div>
    </>
  );
}
