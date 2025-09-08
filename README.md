# online-server-bd
import React, { useMemo, useState, useEffect, useRef } from "react"; import { motion } from "framer-motion"; import { Search, ExternalLink, Wrench, Link as LinkIcon, Image as ImageIcon, FileText, FileDown, FileUp, Scissors, Settings, ShieldCheck, CheckCircle2, Copy, Trash2, Plus, Sparkles } from "lucide-react"; // shadcn/ui import { Button } from "@/components/ui/button"; import { Input } from "@/components/ui/input"; import { Card, CardContent, CardHeader, CardTitle, CardDescription } from "@/components/ui/card"; import { Badge } from "@/components/ui/badge"; import { Tabs, TabsContent, TabsList, TabsTrigger } from "@/components/ui/tabs"; import { Separator } from "@/components/ui/separator";

// OPTIONAL: pdf-lib for client-side PDF tools // These imports work in the ChatGPT canvas preview and most bundlers (Vite/Next). If your build tool complains, // remove PDF features below or install pdf-lib in your project: npm i pdf-lib. import { PDFDocument } from "pdf-lib";

// --------------------------- // CONFIG: Edit these to customize your shop/site // --------------------------- const BRAND = { name: "Online Service Hub (BD)", tagline: "সব দরকারি লিংক + টুলস এক জায়গায়", owner: "Your Shop Name", accentClass: "bg-indigo-600", accentTextClass: "text-indigo-600", };

const PRICE_LIST: { name: string; price: number; note?: string }[] = [ { name: "Form Fill-up (basic)", price: 30 }, { name: "Photo Resize (Passport/300x300)", price: 30 }, { name: "PDF Merge/Compress", price: 30 }, { name: "Result Check / Print", price: 30 }, { name: "NID/PCC Portal Assistance", price: 50, note: "প্রয়োজনে অতিরিক্ত চার্জ প্রযোজ্য" }, ];

// VERIFIED OFFICIAL LINKS (BD) // You can add more categories/items below const CATEGORIES: { title: string; items: { name: string; url: string; tag?: string }[] }[] = [ { title: "Identity & Immigration", items: [ { name: "NID Services (Election Commission)", url: "https://services.nidw.gov.bd/home", tag: "Official" }, { name: "E‑Passport Portal", url: "https://www.epassport.gov.bd/", tag: "Official" }, { name: "Police Clearance (PCC)", url: "https://pcc.police.gov.bd/", tag: "Official" }, { name: "Birth Registration (BDRIS)", url: "https://bdris.gov.bd/", tag: "Official" }, { name: "Passport (MRP) Status", url: "https://passport.gov.bd/onlinestatus.aspx", tag: "Gov" }, ], }, { title: "Transport & License", items: [ { name: "BRTA Service Portal (BSP)", url: "https://bsp.brta.gov.bd/", tag: "Official" }, ], }, { title: "Education", items: [ { name: "Education Board Results", url: "https://www.educationboardresults.gov.bd/", tag: "Official" }, { name: "eBoardResults (Detailed Marks)", url: "https://www.eboardresults.com/", tag: "Official" }, ], }, { title: "Health", items: [ { name: "Surokkha – Vaccine Certificate", url: "https://surokkha.gov.bd/", tag: "Gov" }, ], }, { title: "Payments", items: [ { name: "EkPay (Govt. Payments)", url: "https://ekpay.gov.bd/", tag: "Gov" }, { name: "bKash", url: "https://www.bkash.com/", tag: "MFS" }, { name: "Nagad", url: "https://nagad.com.bd/", tag: "MFS" }, ], }, ];

// --------------------------- // Helper utilities // --------------------------- const bnDigits = ["০","১","২","৩","৪","৫","৬","৭","৮","৯"]; const enDigits = ["0","1","2","3","4","5","6","7","8","9"];

function toBnNumber(input: string) { return input.replace(/[0-9]/g, d => bnDigits[d as any]); } function toEnNumber(input: string) { return input.replace(/[০-৯]/g, d => enDigits[bnDigits.indexOf(d)]); }

function downloadBlob(blob: Blob, filename: string) { const url = URL.createObjectURL(blob); const a = document.createElement("a"); a.href = url; a.download = filename; a.click(); URL.revokeObjectURL(url); }

// --------------------------- // Small Tools: Image Resize, PDF Merge/Compress, Number Converter // --------------------------- function ImageResizer() { const [file, setFile] = useState<File | null>(null); const [width, setWidth] = useState(300); const [height, setHeight] = useState(300); const [quality, setQuality] = useState(0.9); const [fmt, setFmt] = useState<"image/jpeg" | "image/png">("image/jpeg");

const handleResize = async () => { if (!file) return; const img = new Image(); img.src = URL.createObjectURL(file); await new Promise(r => (img.onload = () => r(null))); const canvas = document.createElement("canvas"); canvas.width = width; canvas.height = height; const ctx = canvas.getContext("2d"); if (!ctx) return; ctx.fillStyle = "#ffffff"; // white bg for JPEG ctx.fillRect(0,0,width,height); // cover fit const ratio = Math.max(width / img.width, height / img.height); const w = img.width * ratio; const h = img.height * ratio; const x = (width - w) / 2; const y = (height - h) / 2; ctx.drawImage(img, x, y, w, h); const blob = await new Promise<Blob | null>(res => canvas.toBlob(b => res(b), fmt, quality)); if (blob) downloadBlob(blob, resized_${width}x${height}.${fmt === "image/png" ? "png" : "jpg"}); };

return ( <Card className="w-full shadow-lg"> <CardHeader> <CardTitle className="flex items-center gap-2"><ImageIcon className="h-5 w-5"/>Photo Resize (Passport/300x300)</CardTitle> <CardDescription>Crop-fit to exact size, download as JPEG/PNG</CardDescription> </CardHeader> <CardContent className="space-y-3"> <Input type="file" accept="image/*" onChange={e=>setFile(e.target.files?.[0] ?? null)} /> <div className="grid grid-cols-2 md:grid-cols-4 gap-2"> <Input type="number" value={width} onChange={e=>setWidth(parseInt(e.target.value||"0"))} placeholder="Width" /> <Input type="number" value={height} onChange={e=>setHeight(parseInt(e.target.value||"0"))} placeholder="Height" /> <Input type="number" step="0.05" min="0.1" max="1" value={quality} onChange={e=>setQuality(parseFloat(e.target.value||"0.9"))} placeholder="Quality (0.1-1)" /> <select className="border rounded px-3 py-2" value={fmt} onChange={e=>setFmt(e.target.value as any)}> <option value="image/jpeg">JPEG</option> <option value="image/png">PNG</option> </select> </div> <div className="flex flex-wrap gap-2"> <Button onClick={()=>{setWidth(300); setHeight(300);}} variant="secondary">300×300</Button> <Button onClick={()=>{setWidth(600); setHeight(600);}} variant="secondary">600×600</Button> <Button onClick={handleResize} className={${BRAND.accentClass}}>Download Resized</Button> </div> </CardContent> </Card> ); }

function NumberConverter() { const [text, setText] = useState(""); return ( <Card className="w-full shadow-lg"> <CardHeader> <CardTitle className="flex items-center gap-2"><FileText className="h-5 w-5"/>English↔Bangla Digit Converter</CardTitle> <CardDescription>Convert 0-9 ⇄ ০-৯ instantly</CardDescription> </CardHeader> <CardContent className="space-y-3"> <textarea className="w-full border rounded p-3 min-h-[120px]" placeholder="Type/paste text with numbers…" value={text} onChange={(e)=>setText(e.target.value)} /> <div className="flex flex-wrap gap-2"> <Button onClick={()=>setText(toBnNumber(text))} variant="secondary">To Bangla</Button> <Button onClick={()=>setText(toEnNumber(text))} variant="secondary">To English</Button> <Button onClick={()=>{navigator.clipboard.writeText(text)}} className={${BRAND.accentClass}}>Copy</Button> </div> </CardContent> </Card> ); }

function PdfTools() { const [files, setFiles] = useState<File[]>([]);

const merge = async () => { if (!files.length) return; const merged = await PDFDocument.create(); for (const f of files) { const buf = await f.arrayBuffer(); const doc = await PDFDocument.load(buf); const pages = await merged.copyPages(doc, doc.getPageIndices()); pages.forEach(p => merged.addPage(p)); } const out = await merged.save(); downloadBlob(new Blob([out], { type: "application/pdf" }), merged_${files.length}_files.pdf); };

const compress = async () => { if (!files.length) return; // Simple re-save which sometimes reduces size; true compression would require image downscaling. const first = await PDFDocument.load(await files[0].arrayBuffer()); const out = await first.save({ useObjectStreams: true }); downloadBlob(new Blob([out], { type: "application/pdf" }), compressed_${files[0].name}); };

return ( <Card className="w-full shadow-lg"> <CardHeader> <CardTitle className="flex items-center gap-2"><FileDown className="h-5 w-5"/>PDF Tools (Merge/Compress)</CardTitle> <CardDescription>Process PDFs fully in your browser – no upload</CardDescription> </CardHeader> <CardContent className="space-y-3"> <Input type="file" multiple accept="application/pdf" onChange={(e)=>setFiles(Array.from(e.target.files ?? []))} /> <div className="text-sm text-muted-foreground">Selected: {files.map(f=>f.name).join(", ") || "No files"}</div> <div className="flex flex-wrap gap-2"> <Button onClick={merge} className={${BRAND.accentClass}}>Merge PDFs</Button> <Button onClick={compress} variant="secondary">Quick Compress</Button> </div> </CardContent> </Card> ); }

// --------------------------- // Main App // --------------------------- export default function OnlineServiceHub() { const [query, setQuery] = useState(""); const [customLinks, setCustomLinks] = useState<{name:string;url:string;tag?:string}[]>(() => { const raw = localStorage.getItem("osh.customLinks"); return raw ? JSON.parse(raw) : []; }); const [newLink, setNewLink] = useState({ name: "", url: "", tag: "" });

useEffect(()=>{ localStorage.setItem("osh.customLinks", JSON.stringify(customLinks)); }, [customLinks]);

const filtered = useMemo(()=>{ const q = query.toLowerCase(); const list = [ ...CATEGORIES.flatMap(c => c.items.map(i => ({...i, _cat: c.title}))), ...customLinks.map(i => ({...i, _cat: "My Shortcuts"})) ]; if (!q) return list; return list.filter(i => ${i.name} ${i.url} ${i.tag ?? ""}.toLowerCase().includes(q)); }, [query, customLinks]);

const addCustom = () => { if (!newLink.name || !newLink.url) return; setCustomLinks(prev => [...prev, { ...newLink }]); setNewLink({ name: "", url: "", tag: "" }); }; const removeCustom = (url: string) => setCustomLinks(prev => prev.filter(l => l.url !== url));

return ( <div className="min-h-screen bg-gradient-to-b from-gray-50 to-white"> <header className="sticky top-0 z-40 backdrop-blur border-b bg-white/70"> <div className="max-w-6xl mx-auto px-4 py-4 flex items-center justify-between gap-3"> <div className="flex items-center gap-3"> <motion.div initial={{ rotate: -10, scale: 0.9 }} animate={{ rotate: 0, scale: 1 }} className={h-10 w-10 rounded-2xl ${BRAND.accentClass} grid place-content-center text-white}> <Wrench className="h-5 w-5"/> </motion.div> <div> <h1 className="text-xl font-semibold">{BRAND.name}</h1> <p className="text-sm text-muted-foreground">{BRAND.tagline}</p> </div> </div>

<div className="flex-1 max-w-xl hidden md:block">
        <div className="relative">
          <Search className="absolute left-3 top-1/2 -translate-y-1/2 h-4 w-4 text-gray-400"/>
          <Input value={query} onChange={(e)=>setQuery(e.target.value)} placeholder="Search NID, Passport, BRTA, Results…" className="pl-9" />
        </div>
      </div>

      <Badge className={`${BRAND.accentClass}`}>{BRAND.owner}</Badge>
    </div>
  </header>

  <main className="max-w-6xl mx-auto px-4 py-6 space-y-8">
    {/* Search (mobile) */}
    <div className="md:hidden">
      <div className="relative">
        <Search className="absolute left-3 top-1/2 -translate-y-1/2 h-4 w-4 text-gray-400"/>
        <Input value={query} onChange={(e)=>setQuery(e.target.value)} placeholder="Search services…" className="pl-9" />
      </div>
    </div>

    {/* Quick verified links */}
    <section className="space-y-4">
      <div className="flex items-center gap-2">
        <h2 className="text-lg font-semibold">Verified Links</h2>
        <ShieldCheck className={`h-5 w-5 ${BRAND.accentTextClass}`} />
      </div>
      <div className="grid md:grid-cols-2 lg:grid-cols-3 gap-4">
        {CATEGORIES.map((cat) => (
          <Card key={cat.title} className="hover:shadow-lg transition-shadow">
            <CardHeader>
              <CardTitle className="text-base flex items-center gap-2"><LinkIcon className="h-4 w-4"/>{cat.title}</CardTitle>
            </CardHeader>
            <CardContent className="space-y-2">
              {cat.items.map((item) => (
                <a key={item.url} href={item.url} target="_blank" rel="noopener noreferrer" className="group flex items-center justify-between rounded-lg border px-3 py-2 hover:bg-gray-50">
                  <div>
                    <div className="font-medium group-hover:underline flex items-center gap-2">{item.name}<ExternalLink className="h-4 w-4 opacity-60"/></div>
                    <div className="text-xs text-muted-foreground break-all">{item.url}</div>
                  </div>
                  {item.tag && <Badge variant="secondary">{item.tag}</Badge>}
                </a>
              ))}
            </CardContent>
          </Card>
        ))}
      </div>
    </section>

    {/* My Shortcuts */}
    <section className="space-y-3">
      <div className="flex items-center justify-between">
        <h2 className="text-lg font-semibold flex items-center gap-2"><Sparkles className="h-5 w-5"/>My Shortcuts</h2>
        <div className="flex gap-2">
          <Input placeholder="Name" value={newLink.name} onChange={(e)=>setNewLink(s=>({...s, name:e.target.value}))} className="w-40"/>
          <Input placeholder="https://…" value={newLink.url} onChange={(e)=>setNewLink(s=>({...s, url:e.target.value}))} className="w-64"/>
          <Input placeholder="Tag (optional)" value={newLink.tag} onChange={(e)=>setNewLink(s=>({...s, tag:e.target.value}))} className="w-32"/>
          <Button onClick={addCustom} className={`${BRAND.accentClass}`}><Plus className="h-4 w-4 mr-1"/>Add</Button>
        </div>
      </div>
      <div className="grid sm:grid-cols-2 lg:grid-cols-3 gap-3">
        {customLinks.length === 0 && <p className="text-sm text-muted-foreground">No shortcuts yet. Add your most-used portals here.</p>}
        {customLinks.map(link => (
          <div key={link.url} className="flex items-center justify-between border rounded-lg px-3 py-2">
            <div>
              <a href={link.url} target="_blank" className="font-medium hover:underline flex items-center gap-2">{link.name}<ExternalLink className="h-4 w-4"/></a>
              <div className="text-xs text-muted-foreground break-all">{link.url}</div>
            </div>
            <div className="flex items-center gap-2">
              {link.tag && <Badge variant="secondary">{link.tag}</Badge>}
              <Button size="icon" variant="ghost" onClick={()=>navigator.clipboard.writeText(link.url)}><Copy className="h-4 w-4"/></Button>
              <Button size="icon" variant="ghost" onClick={()=>removeCustom(link.url)}><Trash2 className="h-4 w-4"/></Button>
            </div>
          </div>
        ))}
      </div>
    </section>

    <Separator/>

    {/* Tools */}
    <section className="space-y-4">
      <h2 className="text-lg font-semibold flex items-center gap-2"><Wrench className="h-5 w-5"/>Quick Tools</h2>
      <div className="grid md:grid-cols-2 gap-4">
        <ImageResizer/>
        <PdfTools/>
        <NumberConverter/>
      </div>
    </section>

    <Separator/>

    {/* Price List */}
    <section className="space-y-3">
      <h2 className="text-lg font-semibold flex items-center gap-2"><FileUp className="h-5 w-5"/>Service Price (Per Task)</h2>
      <div className="grid sm:grid-cols-2 lg:grid-cols-3 gap-3">
        {PRICE_LIST.map((p) => (
          <Card key={p.name} className="hover:shadow-md">
            <CardHeader>
              <CardTitle className="text-base">{p.name}</CardTitle>
              <CardDescription>Start price</CardDescription>
            </CardHeader>
            <CardContent>
              <div className="text-2xl font-bold">৳ {toBnNumber(String(p.price))}</div>
              {p.note && <div className="text-xs mt-1 text-muted-foreground">{p.note}</div>}
            </CardContent>
          </Card>
        ))}
      </div>
      <p className="text-xs text-muted-foreground">নোট: সরকারী ফি/চালান/ব্যাংক ফি এতে অন্তর্ভুক্ত নয়।
      কাস্টমারের ডকুমেন্ট/ডাটা নিরাপত্তা সর্বোচ্চ গুরুত্বে রাখা হবে।</p>
    </section>

    <Separator/>

    {/* Footer */}
    <footer className="py-8 text-center text-sm text-muted-foreground">
      <div className="flex items-center justify-center gap-2 mb-2">
        <CheckCircle2 className={`h-4 w-4 ${BRAND.accentTextClass}`} />
        <span>Trusted by local users • Secure client-side tools • No login required</span>
      </div>
      <div>© {new Date().getFullYear()} {BRAND.owner}. All rights reserved.</div>
    </footer>
  </main>
</div>

); }
