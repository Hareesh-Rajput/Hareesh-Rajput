import React, { useEffect, useState } from "react";
import { Canvas } from "@react-three/fiber";
import { OrbitControls } from "@react-three/drei";
import { motion } from "framer-motion";
import {
  ResponsiveContainer,
  LineChart,
  Line,
  XAxis,
  YAxis,
  Tooltip,
  CartesianGrid,
} from "recharts";

/*
  Fixed Jarvis-Portfolio-UI-UX React component

  Major fixes in this version:
  - Removed Drei <Stars /> which can fail in some environments
  - Kept <OrbitControls> but wrapped Canvas with a client-only guard
  - Added ErrorBoundary to catch render-time errors inside the 3D scene
  - Used <planeGeometry> and <boxGeometry> (not Buffer variants)
  - SafeImg with fallback to avoid "reading 'source' of undefined" errors
  - Normalized arrays to avoid undefined elements in map()

  Usage notes:
  - This component requires these dependencies: react, react-dom, @react-three/fiber, @react-three/drei, framer-motion, recharts
  - Make sure to run on client (CRA/Vite/Next with CSR for this page). The 3D preview renders only on client.
*/

// ---------- sample data ----------
const linuxLogos = [
  { name: "Ubuntu", src: "https://raw.githubusercontent.com/gliderlabs/docker-alpine/master/logo/alpine.png" },
  { name: "Kali", src: "https://raw.githubusercontent.com/devicons/devicon/master/icons/kali-linux/kali-linux-original.svg" },
  { name: "Arch", src: "https://raw.githubusercontent.com/devicons/devicon/master/icons/archlinux/archlinux-original.svg" },
  { name: "Fedora", src: "https://upload.wikimedia.org/wikipedia/commons/3/3f/Fedora_logo.svg" },
  { name: "Debian", src: "https://www.vectorlogo.zone/logos/debian/debian-icon.svg" },
];

const langLogos = [
  { name: "Python", src: "https://raw.githubusercontent.com/devicons/devicon/master/icons/python/python-original.svg" },
  { name: "C", src: "https://raw.githubusercontent.com/devicons/devicon/master/icons/c/c-original.svg" },
  { name: "JavaScript", src: "https://raw.githubusercontent.com/devicons/devicon/master/icons/javascript/javascript-original.svg" },
  { name: "Java", src: "https://raw.githubusercontent.com/devicons/devicon/master/icons/java/java-original.svg" },
  { name: "Go", src: "https://raw.githubusercontent.com/devicons/devicon/master/icons/go/go-original.svg" },
];

const tools = [
  { name: "Docker", src: "https://raw.githubusercontent.com/devicons/devicon/master/icons/docker/docker-original-wordmark.svg" },
  { name: "Kubernetes", src: "https://www.vectorlogo.zone/logos/kubernetes/kubernetes-icon.svg" },
  { name: "Jenkins", src: "https://www.vectorlogo.zone/logos/jenkins/jenkins-icon.svg" },
  { name: "Terraform", src: "https://www.vectorlogo.zone/logos/hashicorp_terraform/hashicorp_terraform-icon.svg" },
  { name: "Ansible", src: "https://upload.wikimedia.org/wikipedia/commons/9/9f/Ansible_logo.svg" },
  { name: "Prometheus", src: "https://raw.githubusercontent.com/prometheus/prometheus/main/documentation/static/prometheus_logo.png" },
  { name: "Grafana", src: "https://www.vectorlogo.zone/logos/grafana/grafana-icon.svg" },
  { name: "Git", src: "https://www.vectorlogo.zone/logos/git-scm/git-scm-icon.svg" },
];

const performanceData = [
  { month: "Jan", score: 40 },
  { month: "Feb", score: 55 },
  { month: "Mar", score: 62 },
  { month: "Apr", score: 72 },
  { month: "May", score: 78 },
  { month: "Jun", score: 85 },
  { month: "Jul", score: 90 },
  { month: "Aug", score: 95 },
];

// ---------- helpers ----------
const FALLBACK_SVG = `data:image/svg+xml;utf8,${encodeURIComponent(`
  <svg xmlns='http://www.w3.org/2000/svg' width='120' height='120' viewBox='0 0 120 120'>
    <rect width='100%' height='100%' fill='%230b1220' rx='12' />
    <text x='50%' y='50%' fill='%238a98a8' font-size='12' font-family='Arial' dominant-baseline='middle' text-anchor='middle'>No Image</text>
  </svg>
`)}`;

function SafeImg({ src, alt = "", className = "", title = "" }) {
  const [errored, setErrored] = useState(false);
  const srcToUse = (!src || errored) ? FALLBACK_SVG : src;

  useEffect(() => {
    if (!src) console.warn("[SafeImg] missing src for", alt || title || "unknown");
  }, [src, alt, title]);

  return (
    <img
      src={srcToUse}
      alt={alt}
      title={title || alt}
      className={className}
      onError={() => setErrored(true)}
    />
  );
}

class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }
  static getDerivedStateFromError() {
    return { hasError: true };
  }
  componentDidCatch(error, info) {
    console.error("[ErrorBoundary]", error, info);
  }
  render() {
    if (this.state.hasError) {
      return (
        <div className="w-full h-full flex items-center justify-center bg-black/20 text-xs text-slate-400">
          3D visualization failed to load
        </div>
      );
    }

    return this.props.children;
  }
}

function FloatingBox({ children }) {
  return (
    <motion.div
      initial={{ y: 8, opacity: 0 }}
      animate={{ y: 0, opacity: 1 }}
      transition={{ type: "spring", stiffness: 60 }}
      className="bg-gradient-to-br from-black/50 to-white/5 border border-white/5 backdrop-blur-md rounded-2xl p-4 shadow-xl"
    >
      {children ?? null}
    </motion.div>
  );
}

function ThreeDGraph({ enabled = true }) {
  const [isClient, setIsClient] = useState(false);
  useEffect(() => setIsClient(typeof window !== "undefined"), []);

  if (!enabled) return null;
  if (!isClient) {
    return (
      <div className="w-full h-full flex items-center justify-center bg-black/20 text-xs text-slate-400">
        3D preview (client only)
      </div>
    );
  }

  return (
    <ErrorBoundary>
      <Canvas camera={{ position: [0, 6, 10], fov: 50 }}>
        <ambientLight intensity={0.7} />
        <directionalLight position={[10, 10, 5]} intensity={0.8} />
        <OrbitControls enableZoom={false} />

        {/* Ground */}
        <mesh rotation-x={-Math.PI / 2} position={[0, -0.1, 0]}>
          <planeGeometry args={[40, 40]} />
          <meshStandardMaterial color="#0f172a" metalness={0.2} roughness={0.8} />
        </mesh>

        {/* Bars */}
        {[...Array(8)].map((_, i) => {
          const x = i - 3.5;
          const h = Math.max(0.5, 1 + Math.sin(i * 0.9) * 2.5 + i * 0.6);
          return (
            <mesh key={`bar-${i}`} position={[x * 1.5, h / 2 - 1, 0]}>
              <boxGeometry args={[0.8, h, 0.8]} />
              <meshStandardMaterial color={`hsl(${(i * 30 + 180) % 360},70%,50%)`} />
            </mesh>
          );
        })}
      </Canvas>
    </ErrorBoundary>
  );
}

export default function JarvisPortfolio() {
  const [isClient, setIsClient] = useState(false);
  useEffect(() => setIsClient(typeof window !== "undefined"), []);

  const normalize = (arr = [], prefix = "item") =>
    Array.isArray(arr)
      ? arr.map((it, idx) => ({ name: (it && it.name) || `${prefix}-${idx + 1}`, src: (it && it.src) || null }))
      : [];

  const normalizedLinux = normalize(linuxLogos, "linux");
  const normalizedLangs = normalize(langLogos, "lang");
  const normalizedTools = normalize(tools, "tool");

  useEffect(() => {
    if (!normalizedLinux.length) console.warn("[JarvisPortfolio] linux logos array is empty");
    if (!normalizedLangs.length) console.warn("[JarvisPortfolio] language logos array is empty");
    if (!normalizedTools.length) console.warn("[JarvisPortfolio] tools array is empty");
  }, []);

  return (
    <div className="min-h-screen bg-gradient-to-b from-[#020617] via-[#071133] to-[#000000] text-slate-200 p-6">
      <header className="max-w-7xl mx-auto flex items-center justify-between py-6">
        <div className="flex items-center gap-4">
          <div className="w-12 h-12 rounded-full bg-gradient-to-tr from-cyan-400 to-indigo-600 flex items-center justify-center text-black font-bold shadow-2xl">HK</div>
          <div>
            <h1 className="text-2xl font-semibold tracking-wide">Aych — DevOps & Security</h1>
            <p className="text-sm text-slate-400">Aspiring Penetration Tester · Bug Bounty · Cloud & Automation</p>
          </div>
        </div>

        <nav className="flex items-center gap-4">
          <a className="text-sm px-4 py-2 rounded-md hover:bg-white/5">Projects</a>
          <a className="text-sm px-4 py-2 rounded-md hover:bg-white/5">Tools</a>
          <a className="text-sm px-4 py-2 rounded-md bg-gradient-to-r from-indigo-500 to-cyan-400 text-black font-medium">Contact</a>
        </nav>
      </header>

      <main className="max-w-7xl mx-auto grid grid-cols-12 gap-6">
        {/* Left column */}
        <section className="col-span-4 space-y-6">
          <FloatingBox>
            <div className="flex items-center gap-4">
              <SafeImg src="https://avatars.githubusercontent.com/u/9919?v=4" alt="avatar" className="w-20 h-20 rounded-full border border-white/10 shadow-lg" />
              <div>
                <h2 className="text-xl font-semibold">Aych (Hareesh)</h2>
                <p className="text-sm text-slate-400">CEH · CCNA · Bug Bounty</p>
                <div className="mt-2 text-xs text-slate-300">Location: Hanumangarh · Open to relocation</div>
              </div>
            </div>

            <div className="mt-4 grid grid-cols-4 gap-2">
              {normalizedLinux.map((l, idx) => (
                <div key={`${l.name}-${idx}`} className="flex flex-col items-center text-xs">
                  <SafeImg src={l.src} alt={l.name} className="w-10 h-10 object-contain" />
                  <span className="mt-1 text-slate-300">{l.name}</span>
                </div>
              ))}
            </div>

            <hr className="my-4 border-white/5" />

            <div>
              <h3 className="text-sm font-medium">Top Languages</h3>
              <div className="mt-2 flex gap-3">
                {normalizedLangs.map((l, idx) => (
                  <SafeImg key={`${l.name}-${idx}`} src={l.src} alt={l.name} title={l.name} className="w-10 h-10 object-contain" />
                ))}
              </div>
            </div>

            <div className="mt-4">
              <h3 className="text-sm font-medium">Follow / Contact</h3>
              <div className="mt-2 flex gap-2 text-xs text-slate-300">
                <a className="underline">GitHub</a>
                <a className="underline">LinkedIn</a>
                <a className="underline">Instagram</a>
              </div>
            </div>
          </FloatingBox>

          <FloatingBox>
            <h3 className="font-medium">DevOps Tools</h3>
            <div className="mt-3 grid grid-cols-4 gap-3">
              {normalizedTools.map((t, idx) => (
                <div key={`${t.name}-${idx}`} className="flex flex-col items-center text-xs">
                  <SafeImg src={t.src} alt={t.name} className="w-10 h-10 object-contain" />
                  <span className="mt-1 text-slate-300">{t.name}</span>
                </div>
              ))}
            </div>
          </FloatingBox>

          <FloatingBox>
            <h3 className="font-medium">Quick Stats</h3>
            <div className="mt-2 grid grid-cols-2 gap-2">
              <div className="p-3 rounded-lg bg-white/3">
                <div className="text-2xl font-bold">0+</div>
                <div className="text-xs text-slate-300">Projects</div>
              </div>
              <div className="p-3 rounded-lg bg-white/3">
                <div className="text-2xl font-bold">100+</div>
                <div className="text-xs text-slate-300">Hours Learning</div>
              </div>
            </div>
          </FloatingBox>
        </section>

        {/* Right column */}
        <section className="col-span-8 space-y-6">
          <FloatingBox>
            <div className="flex gap-6">
              <div className="w-2/3">
                <h2 className="text-2xl font-semibold">Jarvis Dashboard</h2>
                <p className="text-sm text-slate-400 mt-1">A futuristic overview of your activity, skills and monthly performance.</p>

                <div className="mt-4 grid grid-cols-2 gap-4">
                  <div className="p-3 rounded-lg bg-white/4">
                    <div className="text-xs text-slate-300">Today's Focus</div>
                    <div className="text-lg font-semibold mt-1">Practice Hacks · CI/CD pipeline</div>
                  </div>
                  <div className="p-3 rounded-lg bg-white/4">
                    <div className="text-xs text-slate-300">Next Goal</div>
                    <div className="text-lg font-semibold mt-1">Kubernetes Certified</div>
                  </div>
                </div>

                <div className="mt-4">
                  <h4 className="text-sm font-medium">Monthly Performance</h4>
                  <div className="w-full h-48 mt-2 bg-transparent rounded-lg p-2">
                    {isClient ? (
                      <ResponsiveContainer width="100%" height="100%">
                        <LineChart data={performanceData} margin={{ top: 10, right: 20, left: -20, bottom: 0 }}>
                          <CartesianGrid strokeDasharray="3 3" stroke="#0b1220" />
                          <XAxis dataKey="month" stroke="#9CA3AF" />
                          <YAxis stroke="#9CA3AF" />
                          <Tooltip />
                          <Line type="monotone" dataKey="score" stroke="#34D399" strokeWidth={3} dot={{ r: 4 }} />
                        </LineChart>
                      </ResponsiveContainer>
                    ) : (
                      <div className="w-full h-full flex items-center justify-center text-xs text-slate-400">Chart preview (client only)</div>
                    )}
                  </div>
                </div>
              </div>

              <div className="w-1/3 flex flex-col gap-4">
                <div className="h-40 rounded-lg overflow-hidden">
                  <ThreeDGraph enabled={isClient} />
                </div>

                <div className="p-3 rounded-lg bg-white/3">
                  <div className="text-xs text-slate-300">Activity</div>
                  <div className="text-lg font-semibold mt-1">Contributions · Bug reports · Labs</div>
                </div>
              </div>
            </div>
          </FloatingBox>

          <FloatingBox>
            <h3 className="font-medium">Detailed Timeline (3D view available)</h3>
            <div className="mt-3 grid grid-cols-3 gap-3">
              <div className="p-3 rounded-lg bg-white/3">
                <div className="text-xs text-slate-300">Apr - Jun</div>
                <div className="font-semibold mt-1">Learned Docker, CI pipelines</div>
              </div>
              <div className="p-3 rounded-lg bg-white/3">
                <div className="text-xs text-slate-300">Jul</div>
                <div className="font-semibold mt-1">Bug bounty project uploaded</div>
              </div>
              <div className="p-3 rounded-lg bg-white/3">
                <div className="text-xs text-slate-300">Aug</div>
                <div className="font-semibold mt-1">Practiced pentesting labs</div>
              </div>
            </div>
          </FloatingBox>

          <FloatingBox>
            <h3 className="font-medium">Visual Skill Map</h3>
            <div className="mt-3 p-2 rounded-lg bg-white/5">
              <div className="grid grid-cols-4 gap-3">
                {normalizedTools.slice(0, 8).map((t, idx) => (
                  <div key={`${t.name}-${idx}`} className="col-span-1 p-3 bg-black/20 rounded-lg flex flex-col items-center">
                    <SafeImg src={t.src} alt={t.name} className="w-12 h-12 object-contain" />
                    <div className="text-xs mt-2">{t.name}</div>
                    <div className="w-full bg-white/5 h-2 rounded-full mt-3">
                      <div className="bg-gradient-to-r from-green-400 to-cyan-400 h-2 rounded-full" style={{ width: `${60 + Math.random() * 40}%` }} />
                    </div>
                  </div>
                ))}
              </div>
            </div>
          </FloatingBox>
        </section>
      </main>

      <footer className="max-w-7xl mx-auto mt-8 text-center text-slate-500 text-sm">
        Built with ❤️ — Dark Jarvis UI · Exportable to GitHub Pages / Vercel
      </footer>
    </div>
  );
}
