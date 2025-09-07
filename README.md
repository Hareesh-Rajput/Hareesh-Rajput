import React from "react";
import { Canvas } from '@react-three/fiber';
import { OrbitControls, Stars } from '@react-three/drei';
import { motion } from 'framer-motion';
import { ResponsiveContainer, LineChart, Line, XAxis, YAxis, Tooltip, CartesianGrid } from 'recharts';

// NOTE: This is a single-file React component to be used in a Tailwind + Vite/Next app.
// It uses: tailwindcss, framer-motion, @react-three/fiber, @react-three/drei, recharts, lucide-react.

const linuxLogos = [
  { name: 'Ubuntu', src: 'https://raw.githubusercontent.com/gliderlabs/docker-alpine/master/logo/alpine.png' },
  { name: 'Kali', src: 'https://raw.githubusercontent.com/devicons/devicon/master/icons/kali-linux/kali-linux-original.svg' },
  { name: 'Arch', src: 'https://raw.githubusercontent.com/devicons/devicon/master/icons/archlinux/archlinux-original.svg' },
  { name: 'Fedora', src: 'https://upload.wikimedia.org/wikipedia/commons/3/3f/Fedora_logo.svg' },
  { name: 'Debian', src: 'https://www.vectorlogo.zone/logos/debian/debian-icon.svg' },
];

const langLogos = [
  { name: 'Python', src: 'https://raw.githubusercontent.com/devicons/devicon/master/icons/python/python-original.svg' },
  { name: 'C', src: 'https://raw.githubusercontent.com/devicons/devicon/master/icons/c/c-original.svg' },
  { name: 'JavaScript', src: 'https://raw.githubusercontent.com/devicons/devicon/master/icons/javascript/javascript-original.svg' },
  { name: 'Java', src: 'https://raw.githubusercontent.com/devicons/devicon/master/icons/java/java-original.svg' },
  { name: 'Go', src: 'https://raw.githubusercontent.com/devicons/devicon/master/icons/go/go-original.svg' },
];

const tools = [
  { name: 'Docker', src: 'https://raw.githubusercontent.com/devicons/devicon/master/icons/docker/docker-original-wordmark.svg' },
  { name: 'Kubernetes', src: 'https://www.vectorlogo.zone/logos/kubernetes/kubernetes-icon.svg' },
  { name: 'Jenkins', src: 'https://www.vectorlogo.zone/logos/jenkins/jenkins-icon.svg' },
  { name: 'Terraform', src: 'https://www.vectorlogo.zone/logos/hashicorp_terraform/hashicorp_terraform-icon.svg' },
  { name: 'Ansible', src: 'https://upload.wikimedia.org/wikipedia/commons/9/9f/Ansible_logo.svg' },
  { name: 'Prometheus', src: 'https://raw.githubusercontent.com/prometheus/prometheus/main/documentation/static/prometheus_logo.png' },
  { name: 'Grafana', src: 'https://www.vectorlogo.zone/logos/grafana/grafana-icon.svg' },
  { name: 'Git', src: 'https://www.vectorlogo.zone/logos/git-scm/git-scm-icon.svg' },
];

const performanceData = [
  { month: 'Jan', score: 40 },
  { month: 'Feb', score: 55 },
  { month: 'Mar', score: 62 },
  { month: 'Apr', score: 72 },
  { month: 'May', score: 78 },
  { month: 'Jun', score: 85 },
  { month: 'Jul', score: 90 },
  { month: 'Aug', score: 95 },
];

function FloatingBox({ children }) {
  return (
    <motion.div
      initial={{ y: 10, opacity: 0 }}
      animate={{ y: 0, opacity: 1 }}
      transition={{ type: 'spring', stiffness: 60 }}
      className="bg-gradient-to-br from-black/50 to-white/2 border border-white/5 backdrop-blur-md rounded-2xl p-4 shadow-xl"
    >
      {children}
    </motion.div>
  );
}

function ThreeDGraph() {
  // Minimal react-three-fiber scene that gives a 3D grid + floating bars illusion.
  // For a live app you can expand this to animate bars by props or hook into real data.
  return (
    <Canvas camera={{ position: [0, 6, 10], fov: 50 }}>
      <ambientLight intensity={0.7} />
      <directionalLight position={[10, 10, 5]} intensity={0.8} />
      <Stars />
      <OrbitControls enableZoom={false} />

      {/* Ground grid */}
      <mesh rotation-x={-Math.PI / 2} position={[0, -0.1, 0]}>
        <planeBufferGeometry args={[40, 40]} />
        <meshStandardMaterial color="#0f172a" metalness={0.2} roughness={0.8} />
      </mesh>

      {/* sample bars */}
      {[...Array(8)].map((_, i) => {
        const x = i - 3.5;
        const h = 1 + Math.sin(i * 0.9) * 2.5 + i;
        return (
          <mesh key={i} position={[x * 1.5, h / 2 - 1, 0]}>
            <boxBufferGeometry args={[0.8, h, 0.8]} />
            <meshStandardMaterial color={`hsl(${i * 30 + 180},70%,50%)`} />
          </mesh>
        );
      })}
    </Canvas>
  );
}

export default function JarvisPortfolio() {
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
        {/* Left column - profile + icons */}
        <section className="col-span-4 space-y-6">
          <FloatingBox>
            <div className="flex items-center gap-4">
              <img src="https://avatars.githubusercontent.com/u/000?v=4" alt="avatar" className="w-20 h-20 rounded-full border border-white/10 shadow-lg" />
              <div>
                <h2 className="text-xl font-semibold">Aych (Hareesh)</h2>
                <p className="text-sm text-slate-400">CEH · CCNA · Bug Bounty</p>
                <div className="mt-2 text-xs text-slate-300">Location: Hanumangarh · Open to relocation</div>
              </div>
            </div>

            <div className="mt-4 grid grid-cols-4 gap-2">
              {linuxLogos.map(l => (
                <div key={l.name} className="flex flex-col items-center text-xs">
                  <img src={l.src} alt={l.name} className="w-10 h-10 object-contain" />
                  <span className="mt-1 text-slate-300">{l.name}</span>
                </div>
              ))}
            </div>

            <hr className="my-4 border-white/5" />

            <div>
              <h3 className="text-sm font-medium">Top Languages</h3>
              <div className="mt-2 flex gap-3">
                {langLogos.map(l => (
                  <img key={l.name} src={l.src} alt={l.name} title={l.name} className="w-10 h-10 object-contain" />
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
              {tools.map(t => (
                <div key={t.name} className="flex flex-col items-center text-xs">
                  <img src={t.src} alt={t.name} className="w-10 h-10 object-contain" />
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

        {/* Right column - Jarvis Panel + 3D graph + Performance */}
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
                    <ResponsiveContainer width="100%" height="100%">
                      <LineChart data={performanceData} margin={{ top: 10, right: 20, left: -20, bottom: 0 }}>
                        <CartesianGrid strokeDasharray="3 3" stroke="#0b1220" />
                        <XAxis dataKey="month" stroke="#9CA3AF" />
                        <YAxis stroke="#9CA3AF" />
                        <Tooltip />
                        <Line type="monotone" dataKey="score" stroke="#34D399" strokeWidth={3} dot={{ r: 4 }} />
                      </LineChart>
                    </ResponsiveContainer>
                  </div>
                </div>

              </div>

              <div className="w-1/3 flex flex-col gap-4">
                <div className="h-40 rounded-lg overflow-hidden">
                  {/* 3D small preview area */}
                  <ThreeDGraph />
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
                {tools.slice(0, 8).map(t => (
                  <div key={t.name} className="col-span-1 p-3 bg-black/20 rounded-lg flex flex-col items-center">
                    <img src={t.src} alt={t.name} className="w-12 h-12 object-contain" />
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
