import React, { useState, useMemo, useEffect } from "react";

// Simple, single-file React app with Tailwind classes
// Pages: Home, Generate, Pricing, Account
// Pricing: $2.99/mo, $24.9/yr; 1-day free trial (UI + mock state)
// Notes for integration are in comments inside the component.

export default function App() {
  const [route, setRoute] = useState("home");
  const [user, setUser] = useState(null); // { email, plan: 'free'|'trial'|'monthly'|'yearly', trialEndsAt }
  const [theme, setTheme] = useState("light");
  const [isGenerating, setIsGenerating] = useState(false);
  const [genImage, setGenImage] = useState(null);

  useEffect(() => {
    // Prefer system theme on first load (cosmetic only)
    const prefersDark = window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches;
    setTheme(prefersDark ? 'dark' : 'light');
  }, []);

  useEffect(() => {
    document.documentElement.classList.toggle('dark', theme === 'dark');
  }, [theme]);

  const navItem = (key, label) => (
    <button
      key={key}
      onClick={() => setRoute(key)}
      className={`px-4 py-2 rounded-xl text-sm font-medium transition-all ${
        route === key
          ? 'bg-black text-white dark:bg-white dark:text-black shadow'
          : 'hover:bg-black/10 dark:hover:bg-white/10'
      }`}
    >
      {label}
    </button>
  );

  const Header = () => (
    <header className="sticky top-0 z-30 backdrop-blur bg-white/70 dark:bg-black/40 border-b border-black/5 dark:border-white/10">
      <div className="max-w-6xl mx-auto px-4 py-3 flex items-center justify-between">
        <div className="flex items-center gap-2">
          <div className="w-8 h-8 rounded-2xl bg-gradient-to-br from-indigo-500 via-cyan-500 to-emerald-400 shadow"></div>
          <span className="font-semibold tracking-tight text-lg">Aira – AI Character Generator</span>
        </div>
        <nav className="flex items-center gap-2">
          {navItem('home','Home')}
          {navItem('generate','Generate')}
          {navItem('pricing','Pricing')}
          {navItem('account','Account')}
          <button
            onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}
            className="ml-2 px-3 py-2 rounded-xl text-sm border border-black/10 dark:border-white/10 hover:bg-black/5 dark:hover:bg-white/5"
            aria-label="Toggle theme"
          >{theme === 'light' ? '🌙' : '☀️'}</button>
        </nav>
      </div>
    </header>
  );

  const GradientBg = ({ children }) => (
    <div className="relative">
      <div className="absolute inset-0 -z-10 bg-[radial-gradient(circle_at_20%_10%,rgba(99,102,241,0.15),transparent_40%),radial-gradient(circle_at_80%_20%,rgba(34,211,238,0.18),transparent_35%),radial-gradient(circle_at_50%_90%,rgba(16,185,129,0.18),transparent_40%)]" />
      {children}
    </div>
  );

  const Card = ({ children, className="" }) => (
    <div className={`rounded-2xl border border-black/10 dark:border-white/10 bg-white/70 dark:bg-white/5 backdrop-blur p-5 shadow-sm ${className}`}>{children}</div>
  );

  function Hero() {
    return (
      <section className="max-w-6xl mx-auto px-4 py-14 grid md:grid-cols-2 gap-8 items-center">
        <div>
          <h1 className="text-4xl md:text-5xl font-extrabold leading-tight tracking-tight">
            Create stunning characters
            <span className="block text-transparent bg-clip-text bg-gradient-to-r from-indigo-500 via-cyan-500 to-emerald-500">with one click.</span>
          </h1>
          <p className="mt-4 text-lg text-black/70 dark:text-white/70">
            Aira turns simple prompts into beautiful, unique characters for games, stories, and social media. Try it free for 1 day.
          </p>
          <div className="mt-6 flex gap-3">
            <button
              onClick={() => setRoute('generate')}
              className="px-5 py-3 rounded-2xl bg-black text-white dark:bg-white dark:text-black font-medium shadow hover:scale-[1.01] active:scale-100 transition"
            >Try Free</button>
            <button
              onClick={() => setRoute('pricing')}
              className="px-5 py-3 rounded-2xl border border-black/10 dark:border-white/10 hover:bg-black/5 dark:hover:bg-white/5"
            >See Pricing</button>
          </div>
          <ul className="mt-6 text-sm text-black/60 dark:text-white/60 space-y-1">
            <li>• 1-day free trial · no commitment</li>
            <li>• $2.99/month or $24.9/year after trial</li>
            <li>• Full-resolution downloads</li>
          </ul>
        </div>
        <Card className="md:ml-auto">
          <div className="aspect-[4/3] w-full rounded-xl bg-gradient-to-br from-indigo-200 via-cyan-200 to-emerald-200 dark:from-indigo-900/30 dark:via-cyan-900/30 dark:to-emerald-900/30 grid place-items-center text-center">
            <div>
              <div className="text-sm uppercase tracking-widest opacity-70">Live Preview</div>
              <div className="text-2xl font-bold">Your character appears here</div>
              <div className="mt-2 text-sm opacity-70">Tap Generate to see a sample</div>
            </div>
          </div>
        </Card>
      </section>
    );
  }

  function Generate() {
    const [prompt, setPrompt] = useState("");

    const canGenerate = useMemo(() => {
      if (!user) return true; // allow preview before signup
      if (user.plan === 'monthly' || user.plan === 'yearly') return true;
      if (user.plan === 'trial') {
        // check if trial still active
        return user.trialEndsAt && new Date(user.trialEndsAt) > new Date();
      }
      return false;
    }, [user]);

    const handleGenerate = async () => {
      setIsGenerating(true);
      setGenImage(null);
      // DEMO: simulate generation with placeholder image.
      // TODO (Backend): Call your API endpoint (Render/FastAPI) which proxies to Stable Diffusion or another model.
      await new Promise(r => setTimeout(r, 1200));
      const seed = Math.floor(Math.random()*1000);
      const url = `https://picsum.photos/seed/aira-${seed}/1024/768`;
      setGenImage(url);
      setIsGenerating(false);
    };

    return (
      <section className="max-w-6xl mx-auto px-4 py-10 grid lg:grid-cols-2 gap-8">
        <Card>
          <h2 className="text-xl font-semibold">Describe your character</h2>
          <p className="text-sm opacity-70 mt-1">Example: "elven ranger, emerald cloak, silver hair, cinematic lighting"</p>
          <div className="mt-4 space-y-3">
            <textarea
              value={prompt}
              onChange={(e) => setPrompt(e.target.value)}
              rows={4}
              placeholder="Type your prompt..."
              className="w-full rounded-xl border border-black/10 dark:border-white/10 bg-white/70 dark:bg-white/5 backdrop-blur px-3 py-2 outline-none focus:ring-2 focus:ring-indigo-400"
            />
            <div className="flex flex-wrap gap-2 text-xs">
              {['anime hero','medieval knight','space explorer','cyberpunk hacker','fairy archer'].map(p => (
                <button key={p} onClick={() => setPrompt(p)} className="px-3 py-1.5 rounded-xl border border-black/10 dark:border-white/10 hover:bg-black/5 dark:hover:bg-white/5">{p}</button>
              ))}
            </div>
            <button
              onClick={handleGenerate}
              disabled={!canGenerate || isGenerating}
              className={`w-full px-5 py-3 rounded-2xl font-medium transition ${
                !canGenerate ? 'bg-black/10 dark:bg-white/10 text-black/40 dark:text-white/40 cursor-not-allowed' :
                'bg-black text-white dark:bg-white dark:text-black hover:scale-[1.01] active:scale-100'
              }`}
            >{isGenerating ? 'Generating…' : !canGenerate ? 'Upgrade or Start Trial to Generate' : 'Generate'}</button>
            {!user && (
              <div className="text-xs opacity-70 mt-2">Tip: You haven’t signed in yet. You can preview generation, but create an account to save results.</div>
            )}
          </div>
        </Card>
        <Card>
          <h2 className="text-xl font-semibold">Result</h2>
          <div className="mt-4 aspect-[4/3] w-full rounded-xl bg-black/5 dark:bg-white/5 grid place-items-center overflow-hidden">
            {genImage ? (
              <img src={genImage} alt="Generated character" className="w-full h-full object-cover" />
            ) : (
              <div className="text-sm opacity-70">Your image will appear here.</div>
            )}
          </div>
          <div className="mt-4 flex gap-2">
            <button className="px-4 py-2 rounded-xl border border-black/10 dark:border-white/10 hover:bg-black/5 dark:hover:bg-white/5">Download</button>
            <button className="px-4 py-2 rounded-xl border border-black/10 dark:border-white/10 hover:bg-black/5 dark:hover:bg-white/5">Save to Account</button>
          </div>
        </Card>
      </section>
    );
  }

  function Pricing() {
    return (
      <section className="max-w-6xl mx-auto px-4 py-12">
        <div className="text-center max-w-3xl mx-auto">
          <h2 className="text-3xl md:text-4xl font-extrabold tracking-tight">Simple pricing, global payments</h2>
          <p className="mt-3 text-black/70 dark:text-white/70">Start with a free 1-day trial. Cancel anytime. Payments via Paddle (worldwide) and optional crypto.</p>
        </div>
        <div className="mt-8 grid md:grid-cols-2 gap-6">
          <Card className="relative">
            <div className="absolute right-5 -top-3 text-xs px-2 py-1 rounded-full bg-emerald-500/10 text-emerald-600 dark:text-emerald-400 border border-emerald-500/20">Most Popular</div>
            <h3 className="text-xl font-semibold">Monthly</h3>
            <div className="mt-2 text-4xl font-extrabold">$2.99<span className="text-base font-medium">/mo</span></div>
            <ul className="mt-4 text-sm space-y-2 opacity-80">
              <li>• Unlimited generations</li>
              <li>• Full-resolution downloads</li>
              <li>• Priority queue</li>
            </ul>
            <button
              onClick={() => alert('TODO: Open Paddle checkout (Monthly)')}
              className="mt-5 w-full px-5 py-3 rounded-2xl bg-black text-white dark:bg-white dark:text-black font-medium"
            >Continue</button>
          </Card>
          <Card>
            <h3 className="text-xl font-semibold">Yearly</h3>
            <div className="mt-2 text-4xl font-extrabold">$24.9<span className="text-base font-medium">/yr</span></div>
            <ul className="mt-4 text-sm space-y-2 opacity-80">
              <li>• Save 30% vs monthly</li>
              <li>• Unlimited generations</li>
              <li>• VIP support</li>
            </ul>
            <button
              onClick={() => alert('TODO: Open Paddle checkout (Yearly)')}
              className="mt-5 w-full px-5 py-3 rounded-2xl bg-black text-white dark:bg-white dark:text-black font-medium"
            >Continue</button>
          </Card>
        </div>
        <Card className="mt-6">
          <h4 className="text-lg font-semibold">Free trial</h4>
          <p className="text-sm opacity-80 mt-1">Try everything for 1 day. We’ll remind you before it ends. No commitment.</p>
          <div className="mt-3 flex gap-2">
            <button
              onClick={() => {
                const now = new Date();
                const ends = new Date(now.getTime() + 24*60*60*1000);
                setUser({ email: 'demo@trial', plan: 'trial', trialEndsAt: ends.toISOString() });
                setRoute('generate');
              }}
              className="px-4 py-2 rounded-xl border border-black/10 dark:border-white/10 hover:bg-black/5 dark:hover:bg-white/5"
            >Start 1‑day trial</button>
            <button onClick={() => setRoute('account')} className="px-4 py-2 rounded-xl border border-black/10 dark:border-white/10 hover:bg-black/5 dark:hover:bg-white/5">Sign in</button>
          </div>
        </Card>
      </section>
    );
  }

  function Account() {
    const [email, setEmail] = useState("");
    const trialLeft = useMemo(() => {
      if (!user || user.plan !== 'trial' || !user.trialEndsAt) return null;
      const ms = new Date(user.trialEndsAt) - new Date();
      if (ms <= 0) return 'Expired';
      const hours = Math.floor(ms/1000/60/60);
      const mins = Math.floor((ms/1000/60)%60);
      return `${hours}h ${mins}m left`;
    }, [user]);

    return (
      <section className="max-w-4xl mx-auto px-4 py-10 grid md:grid-cols-2 gap-6">
        <Card>
          <h3 className="text-xl font-semibold">Sign in / Create account</h3>
          <p className="text-sm opacity-70 mt-1">We’ll keep it simple. Just your email for now.</p>
          <div className="mt-4 space-y-3">
            <input
              type="email"
              value={email}
              onChange={e=>setEmail(e.target.value)}
              placeholder="you@example.com"
              className="w-full rounded-xl border border-black/10 dark:border-white/10 bg-white/70 dark:bg-white/5 backdrop-blur px-3 py-2 outline-none focus:ring-2 focus:ring-indigo-400"
            />
            <button
              onClick={() => setUser({ email, plan: 'free' })}
              className="w-full px-5 py-3 rounded-2xl bg-black text-white dark:bg-white dark:text-black font-medium"
            >Continue</button>
            <div className="text-xs opacity-70">Next step (backend): connect Supabase auth to handle real accounts.</div>
          </div>
        </Card>
        <Card>
          <h3 className="text-xl font-semibold">Your plan</h3>
          {!user ? (
            <p className="opacity-70 mt-2">You’re not signed in yet.</p>
          ) : (
            <div className="mt-2 space-y-2 text-sm">
              <div><span className="opacity-60">Email:</span> <span className="font-medium">{user.email}</span></div>
              <div><span className="opacity-60">Status:</span> <span className="font-medium capitalize">{user.plan}</span> {trialLeft && <span className="ml-2 text-xs opacity-70">{trialLeft}</span>}</div>
              <div className="pt-2">
                <button onClick={() => alert('TODO: Open Paddle customer portal')} className="px-3 py-2 rounded-xl border border-black/10 dark:border-white/10 hover:bg-black/5 dark:hover:bg-white/5">Manage billing</button>
              </div>
            </div>
          )}
          <div className="text-xs opacity-70 mt-3">Billing: Paddle (global cards, Apple Pay, Google Pay, PayPal). Optional: add crypto gateway.</div>
        </Card>
      </section>
    );
  }

  return (
    <GradientBg>
      <Header />
      {route === 'home' && <Hero />}
      {route === 'generate' && <Generate />}
      {route === 'pricing' && <Pricing />}
      {route === 'account' && <Account />}
      <footer className="mt-10 border-t border-black/10 dark:border-white/10">
        <div className="max-w-6xl mx-auto px-4 py-8 text-sm flex flex-col md:flex-row gap-4 md:items-center md:justify-between">
          <div className="opacity-70">© {new Date().getFullYear()} Aira. All rights reserved.</div>
          <div className="flex gap-4">
            <a className="opacity-70 hover:opacity-100" href="#" onClick={(e)=>{e.preventDefault(); alert('TODO: Terms of Service page');}}>Terms</a>
            <a className="opacity-70 hover:opacity-100" href="#" onClick={(e)=>{e.preventDefault(); alert('TODO: Privacy Policy page');}}>Privacy</a>
            <a className="opacity-70 hover:opacity-100" href="#" onClick={(e)=>{e.preventDefault(); setRoute('pricing');}}>Pricing</a>
          </div>
        </div>
      </footer>
    </GradientBg>
  );
}
